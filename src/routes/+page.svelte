<script lang="ts">
	// ── Audio ──────────────────────────────────────────────────────────────────
	let audioCtx: AudioContext | null = $state(null);

	function getAudioCtx() {
		if (!audioCtx) audioCtx = new window.AudioContext();
		return audioCtx;
	}

	/**
	 * Pluck a string at the given frequency.
	 * Uses a short sine burst with an exponential decay.
	 */
	function pluck(freq: number) {
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
		[0, 1].map(() => BASS_FREQS.map(() => ({ phase: 0, active: false, frame: Number.NaN })))
	);

	function triggerString(bassIdx: number, strIdx: number) {
		pluck(BASS_FREQS[strIdx]);
		const s = strings[bassIdx][strIdx];
		s.active = true;
		s.phase = 0;

		if (s.frame) cancelAnimationFrame(s.frame);

		const startTime = performance.now();
		const duration = 1400; // ms — matches audio decay

		function tick(now: number) {
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
	 */
	function wavePath(x1: number, x2: number, y: number, phase: number, active: boolean) {
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
	function hitString(svgEl: SVGSVGElement, e: MouseEvent, stringYs: number[], rotation: number) {
		const pt = svgEl.createSVGPoint();
		pt.x = e.clientX;
		pt.y = e.clientY;

		const ctm = svgEl.getScreenCTM();
		if (!ctm) return -1;

		const svgPt = pt.matrixTransform(ctm.inverse());

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

	function onMouseMove(bassIdx: number, e: MouseEvent) {
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

<!-- ── Page ───────────────────────────────────────────────────────────────── -->
<main
	class="relative flex min-h-screen w-full flex-col items-center justify-center overflow-hidden bg-[#0e0e0e]"
>
	<!-- Bass guitar stage -->
	<div class="relative flex w-full max-w-5xl flex-col items-center" style="height: 680px;">
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
	</div>

	<!-- Hint label -->
	<p
		class="absolute bottom-6 left-1/2 -translate-x-1/2 font-mono text-[10px] tracking-[0.25em] text-white/20 uppercase"
	>
		work in progress...
	</p>
</main>

<style>
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
