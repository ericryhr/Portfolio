<script>
	// ── Audio ──────────────────────────────────────────────────────────────────
	let audioCtx = $state(null);

	function getAudioCtx() {
		if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
		return audioCtx;
	}

	/**
	 * Pluck a string at the given frequency.
	 * Uses a short sine burst with an exponential decay.
	 */
	function pluck(freq) {
		const ctx = getAudioCtx();
		const osc = ctx.createOscillator();
		const gain = ctx.createGain();
		osc.connect(gain);
		gain.connect(ctx.destination);

		osc.type = 'sine';
		osc.frequency.setValueAtTime(freq, ctx.currentTime);
		gain.gain.setValueAtTime(0.35, ctx.currentTime);
		gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 1.4);

		osc.start(ctx.currentTime);
		osc.stop(ctx.currentTime + 1.4);
	}

	// ── Bass definitions ───────────────────────────────────────────────────────
	// Each bass has: id, x/y position, rotation (deg), color scheme, strings[]
	// String frequencies (bass, low→high): E1 410Hz, A1 550Hz, D2 730Hz, G2 980Hz
	const BASS_FREQS = [410.2, 550.0, 730.4, 980.0];

	// We track per-string animation state: { wave: number (0–1 phase), active: bool }
	let strings = $state(
		[0, 1].map(() => BASS_FREQS.map(() => ({ phase: 0, active: false, frame: null })))
	);

	function triggerString(bassIdx, strIdx) {
		pluck(BASS_FREQS[strIdx]);
		const s = strings[bassIdx][strIdx];
		s.active = true;
		s.phase = 0;

		if (s.frame) cancelAnimationFrame(s.frame);

		const startTime = performance.now();
		const duration = 1400; // ms — matches audio decay

		function tick(now) {
			const t = (now - startTime) / duration;
			if (t >= 1) {
				s.active = false;
				s.phase = 0;
				return;
			}
			s.phase = t;
			s.frame = requestAnimationFrame(tick);
		}

		s.frame = requestAnimationFrame(tick);
	}

	// ── SVG string wave path helper ────────────────────────────────────────────
	/**
	 * Build a wavy SVG path for a string.
	 * @param {number} x1   start x
	 * @param {number} x2   end x
	 * @param {number} y    baseline y
	 * @param {number} phase  animation phase 0–1
	 * @param {boolean} active
	 */
	function wavePath(x1, x2, y, phase, active) {
		if (!active) return `M ${x1} ${y} L ${x2} ${y}`;

		const len = x2 - x1;
		const amp = 5 * Math.sin(phase * Math.PI); // envelope: 0 → peak → 0
		const freq = 4; // number of full waves across the string
		const steps = 80;
		let d = `M ${x1} ${y}`;

		for (let i = 1; i <= steps; i++) {
			const frac = i / steps;
			const px = x1 + frac * len;
			const py = y + amp * Math.sin(2 * Math.PI * freq * frac - phase * Math.PI * 8);
			d += ` L ${px} ${py}`;
		}
		return d;
	}

	// ── Mouse hit detection ────────────────────────────────────────────────────
	/**
	 * Given an SVG element and a pointer event, return the string index hit (-1 if none).
	 * stringYs: array of y coordinates (in SVG space) for each string.
	 */
	function hitString(svgEl, e, stringYs, rotation) {
		const pt = svgEl.createSVGPoint();
		pt.x = e.clientX;
		pt.y = e.clientY;
		const svgPt = pt.matrixTransform(svgEl.getScreenCTM().inverse());

		// The SVG is rotated as a whole group; we need to un-rotate around center
		const cx = 300,
			cy = 120; // SVG viewBox center
		const rad = (rotation * Math.PI) / 180;
		const dx = svgPt.x - cx;
		const dy = svgPt.y - cy;
		const lx = dx * Math.cos(-rad) - dy * Math.sin(-rad) + cx;
		const ly = dx * Math.sin(-rad) + dy * Math.cos(-rad) + cy;

		for (let i = 0; i < stringYs.length; i++) {
			if (Math.abs(ly - stringYs[i]) < 9) return i;
		}
		return -1;
	}

	// We store SVG element refs
	let svgRefs = $state([null, null]);

	// ── Bass SVG configs ───────────────────────────────────────────────────────
	// Both basses share a 600×240 viewBox
	// Strings run along the neck from about x=80 to x=520, evenly spaced in y
	const STRING_X1 = 75;
	const STRING_X2 = 525;
	// 4 strings centered in neck area, y = 85 to 115 (neck center ~100)
	const STRING_YS = [88, 96, 104, 112];

	const basses = [
		{
			id: 'jazz',
			label: 'Sire Marcus Miller V7',
			rotation: -8,
			bodyColor: '#7B3A1A',
			bodyColor2: '#3B1A0A',
			pickguardColor: '#111',
			neckColor: '#C4922A',
			headstockColor: '#6B300F'
		},
		{
			id: 'mustang',
			label: 'Squier Mustang Bass',
			rotation: 12,
			bodyColor: '#E8E0CC',
			bodyColor2: '#C8C0B0',
			pickguardColor: '#8B1A1A',
			neckColor: '#B8832A',
			headstockColor: '#C8C0B0'
		}
	];

	// ── Cursor tracking per SVG ────────────────────────────────────────────────
	let lastHit = [-1, -1]; // last string index hit per bass, to avoid re-triggering

	function onMouseMove(bassIdx, e) {
		const svgEl = svgRefs[bassIdx];
		if (!svgEl) return;
		const hit = hitString(svgEl, e, STRING_YS, 0); // rotation handled in hit fn, pass 0 here since we rotate the group
		if (hit !== -1 && hit !== lastHit[bassIdx]) {
			lastHit[bassIdx] = hit;
			triggerString(bassIdx, hit);
		}
		if (hit === -1) lastHit[bassIdx] = -1;
	}
</script>

<!-- ── Fonts ──────────────────────────────────────────────────────────────── -->
<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="" />
	<link
		href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=DM+Mono:wght@300;400&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<!-- ── Page ───────────────────────────────────────────────────────────────── -->
<main
	class="relative flex min-h-screen w-full flex-col items-center justify-center overflow-hidden bg-[#0e0e0e]"
>
	<!-- Subtle grain overlay -->
	<div
		class="pointer-events-none fixed inset-0 z-50 opacity-[0.04]"
		style="background-image: url('data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 width=%22200%22 height=%22200%22><filter id=%22n%22><feTurbulence type=%22fractalNoise%22 baseFrequency=%220.9%22 numOctaves=%224%22/></filter><rect width=%22200%22 height=%22200%22 filter=%22url(%23n)%22 opacity=%221%22/></svg>')"
	></div>

	<!-- Bass guitar stage -->
	<div class="relative flex w-full max-w-5xl flex-col items-center" style="height: 680px;">
		<!-- ── Bass 1 (top-left, tilted left) ── -->
		<div
			class="absolute"
			style="top: 20px; left: -60px; transform: rotate(-8deg); transform-origin: center;"
		>
			<!-- svelte-ignore a11y_no_static_element_interactions -->
			<svg
				bind:this={svgRefs[0]}
				viewBox="0 0 600 240"
				width="640"
				height="256"
				class="cursor-crosshair drop-shadow-[0_8px_40px_rgba(180,80,20,0.25)]"
				onmousemove={(e) => onMouseMove(0, e)}
				onmouseleave={() => {
					lastHit[0] = -1;
				}}
			>
				<!-- Jazz Bass body (offset cutaway shape) -->
				<g>
					<!-- Lower bout -->
					<ellipse cx="118" cy="130" rx="105" ry="88" fill="url(#grad0)" />
					<!-- Upper bout (smaller) -->
					<ellipse cx="100" cy="62" rx="75" ry="58" fill="url(#grad0)" />
					<!-- Join them -->
					<rect x="15" y="60" width="210" height="100" fill="url(#grad0)" />
					<!-- Waist cutout -->
					<ellipse cx="220" cy="112" rx="30" ry="22" fill="#0e0e0e" />

					<!-- Pickguard -->
					<path
						d="M 70 90 L 200 78 L 215 108 L 195 145 L 60 155 Z"
						fill={basses[0].pickguardColor}
						opacity="0.92"
					/>

					<!-- Pickups -->
					<rect
						x="88"
						y="108"
						width="56"
						height="14"
						rx="3"
						fill="#1a1a1a"
						stroke="#333"
						stroke-width="1"
					/>
					<rect
						x="152"
						y="100"
						width="52"
						height="13"
						rx="3"
						fill="#1a1a1a"
						stroke="#333"
						stroke-width="1"
					/>

					<!-- Control knobs -->
					<circle cx="72" cy="148" r="7" fill="#222" stroke="#444" stroke-width="1.5" />
					<circle cx="91" cy="155" r="7" fill="#222" stroke="#444" stroke-width="1.5" />
					<circle cx="110" cy="159" r="7" fill="#222" stroke="#444" stroke-width="1.5" />
				</g>

				<!-- Neck -->
				<rect x="78" y="90" width="448" height="30" rx="4" fill={basses[0].neckColor} />
				<!-- Fret lines -->
				{#each Array(19) as _, i}
					<line
						x1={160 + i * 19}
						y1="90"
						x2={160 + i * 19}
						y2="120"
						stroke="#5a3a10"
						stroke-width="1.2"
					/>
				{/each}
				<!-- Fret markers -->
				{#each [3, 5, 7, 9, 12, 15, 17] as pos}
					<circle cx={160 + (pos - 0.5) * 19} cy="105" r="2.5" fill="#d4b06060" />
				{/each}

				<!-- Headstock -->
				<path
					d="M 526 84 L 600 78 L 602 88 L 604 118 L 602 128 L 526 120 Z"
					fill={basses[0].headstockColor}
				/>
				<!-- Tuning pegs -->
				{#each [0, 1, 2, 3] as i}
					<circle
						cx={535 + i * 17}
						cy={i % 2 === 0 ? 88 : 116}
						r="6"
						fill="#aaa"
						stroke="#888"
						stroke-width="1"
					/>
					<rect
						x={533 + i * 17}
						y={i % 2 === 0 ? 78 : 118}
						width="4"
						height="10"
						rx="1"
						fill="#999"
					/>
				{/each}

				<!-- Strings -->
				{#each STRING_YS as sy, si}
					<path
						d={wavePath(STRING_X1, STRING_X2, sy, strings[0][si].phase, strings[0][si].active)}
						stroke={`hsl(${40 + si * 10}, 70%, ${55 + si * 5}%)`}
						stroke-width={2.2 - si * 0.3}
						fill="none"
						stroke-linecap="round"
					/>
				{/each}

				<!-- Bridge -->
				<rect
					x="75"
					y="88"
					width="18"
					height="34"
					rx="2"
					fill="#555"
					stroke="#666"
					stroke-width="1"
				/>

				<!-- Gradient defs -->
				<defs>
					<radialGradient id="grad0" cx="40%" cy="40%">
						<stop offset="0%" stop-color={basses[0].bodyColor} />
						<stop offset="100%" stop-color={basses[0].bodyColor2} />
					</radialGradient>
				</defs>
			</svg>
		</div>

		<!-- ── Nav (centered, floating) ── -->
		<nav
			class="absolute z-10 flex items-center gap-10"
			style="top: 50%; left: 50%; transform: translate(-50%, -50%);"
		>
			{#each ['Me', 'Projects', 'Contact'] as label}
				<a
					href={`/${label.toLowerCase()}`}
					class="nav-link font-mono text-sm tracking-[0.2em] text-white/60 uppercase transition-all duration-300 hover:tracking-[0.3em] hover:text-white"
				>
					{label}
				</a>
			{/each}
		</nav>

		<!-- ── Bass 2 (bottom-right, tilted right) ── -->
		<div
			class="absolute"
			style="bottom: 20px; right: -60px; transform: rotate(12deg); transform-origin: center;"
		>
			<!-- svelte-ignore a11y_no_static_element_interactions -->
			<svg
				bind:this={svgRefs[1]}
				viewBox="0 0 600 240"
				width="600"
				height="240"
				class="cursor-crosshair drop-shadow-[0_8px_40px_rgba(220,210,180,0.15)]"
				onmousemove={(e) => onMouseMove(1, e)}
				onmouseleave={() => {
					lastHit[1] = -1;
				}}
			>
				<!-- Mustang Bass body (offset, shorter, rounder) -->
				<g>
					<!-- Main body blob -->
					<ellipse cx="120" cy="115" rx="108" ry="92" fill="url(#grad1)" />
					<ellipse cx="100" cy="68" rx="70" ry="55" fill="url(#grad1)" />
					<rect x="14" y="66" width="215" height="98" fill="url(#grad1)" />

					<!-- Upper horn cutaway -->
					<ellipse cx="225" cy="85" rx="28" ry="22" fill="#0e0e0e" />
					<!-- Lower cutaway (Mustang has one) -->
					<ellipse cx="225" cy="145" rx="22" ry="18" fill="#0e0e0e" />

					<!-- Pickguard (Mustang style, tortoiseshell) -->
					<path
						d="M 85 72 L 210 68 L 220 90 L 218 145 L 210 158 L 75 158 L 68 130 Z"
						fill={basses[1].pickguardColor}
						opacity="0.88"
					/>

					<!-- Single split-coil pickup -->
					<rect
						x="130"
						y="108"
						width="52"
						height="12"
						rx="2"
						fill="#111"
						stroke="#444"
						stroke-width="1"
					/>
					<rect
						x="130"
						y="108"
						width="25"
						height="12"
						rx="2"
						fill="#181818"
						stroke="#333"
						stroke-width="0.5"
					/>

					<!-- Controls -->
					<circle cx="92" cy="152" r="8" fill="#111" stroke="#555" stroke-width="1.5" />
					<circle cx="115" cy="158" r="8" fill="#111" stroke="#555" stroke-width="1.5" />
				</g>

				<!-- Neck -->
				<rect x="78" y="92" width="430" height="26" rx="4" fill={basses[1].neckColor} />
				<!-- Fret lines -->
				{#each Array(17) as _, i}
					<line
						x1={165 + i * 20}
						y1="92"
						x2={165 + i * 20}
						y2="118"
						stroke="#7a5520"
						stroke-width="1.1"
					/>
				{/each}
				<!-- Dot markers -->
				{#each [3, 5, 7, 9, 12] as pos}
					<circle cx={165 + (pos - 0.5) * 20} cy="105" r="2.5" fill="#e0c08060" />
				{/each}

				<!-- Headstock (Mustang style — wider, rounder) -->
				<path
					d="M 508 88 L 580 82 L 590 95 L 590 115 L 580 128 L 508 122 Z"
					fill={basses[1].headstockColor}
					stroke="#aaa8"
					stroke-width="0.5"
				/>
				<!-- Tuning pegs (all on one side like a Mustang) -->
				{#each [0, 1, 2, 3] as i}
					<circle
						cx={515 + i * 17}
						cy={i % 2 === 0 ? 90 : 118}
						r="6"
						fill="#bbb"
						stroke="#999"
						stroke-width="1"
					/>
					<rect
						x={513 + i * 17}
						y={i % 2 === 0 ? 80 : 120}
						width="4"
						height="10"
						rx="1"
						fill="#aaa"
					/>
				{/each}

				<!-- Strings -->
				{#each STRING_YS as sy, si}
					<path
						d={wavePath(STRING_X1, STRING_X2 - 16, sy, strings[1][si].phase, strings[1][si].active)}
						stroke={`hsl(${40 + si * 10}, 60%, ${60 + si * 4}%)`}
						stroke-width={2.2 - si * 0.3}
						fill="none"
						stroke-linecap="round"
					/>
				{/each}

				<!-- Bridge -->
				<rect
					x="75"
					y="90"
					width="20"
					height="30"
					rx="2"
					fill="#666"
					stroke="#777"
					stroke-width="1"
				/>

				<defs>
					<radialGradient id="grad1" cx="45%" cy="40%">
						<stop offset="0%" stop-color={basses[1].bodyColor} />
						<stop offset="100%" stop-color={basses[1].bodyColor2} />
					</radialGradient>
				</defs>
			</svg>
		</div>
	</div>

	<!-- Hint label -->
	<p
		class="absolute bottom-6 left-1/2 -translate-x-1/2 font-mono text-[10px] tracking-[0.25em] text-white/20 uppercase"
	>
		move your cursor across the strings
	</p>
</main>

<style>
	:global(html, body) {
		margin: 0;
		padding: 0;
		background: #0e0e0e;
		font-family: 'DM Mono', monospace;
	}

	.nav-link {
		position: relative;
	}

	.nav-link::after {
		content: '';
		position: absolute;
		bottom: -3px;
		left: 0;
		width: 0;
		height: 1px;
		background: white;
		transition: width 0.3s ease;
	}

	.nav-link:hover::after {
		width: 100%;
	}
</style>
