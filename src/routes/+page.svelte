<script lang="ts">
	import bassV5 from '$lib/assets/Bass-V5.svg';

	let imgElement: HTMLImageElement | null = null;
	let imgWidth = 0;
	let imgHeight = 0;

	// Svelte reactive block to track image dimensions when it loads/resizes
	$: if (imgElement) {
		imgWidth = imgElement.clientWidth;
		imgHeight = imgElement.clientHeight;
	}

	const STRING_X1 = 43;
	const STRING_X2S = [938, 984, 1027, 1065];
	const STRING_Y1S = [598, 619, 639, 657];
	const STRING_Y2S = [619, 626, 634, 640];
</script>

<!-- ── Page ───────────────────────────────────────────────────────────────── -->
<main
	class="relative flex min-h-screen w-full flex-col items-center justify-center overflow-hidden bg-[#0e0e0e]"
>
	<!-- Bass guitar stage -->
	<div class="relative flex w-full max-w-5xl flex-col items-center" style="height: 680px;">
		<div class="relative flex w-full flex-col items-center justify-center">
			<div class="absolute" style="top: -80px; scale: 3.5; transform-origin: center;">
				<img bind:this={imgElement} src={bassV5} alt="Bass V5" />
			</div>
			<svg class="absolute" width={imgWidth * 3.5} height={imgHeight * 3.5}>
				<line
					x1={STRING_X1}
					y1={STRING_Y1S[0]}
					x2={STRING_X2S[0]}
					y2={STRING_Y2S[0]}
					stroke="rgb(109, 93, 93)"
					stroke-width="3.0"
				/>
				<line
					x1={STRING_X1}
					y1={STRING_Y1S[1]}
					x2={STRING_X2S[1]}
					y2={STRING_Y2S[1]}
					stroke="rgb(109, 93, 93)"
					stroke-width="2.5"
				/>
				<line
					x1={STRING_X1}
					y1={STRING_Y1S[2]}
					x2={STRING_X2S[2]}
					y2={STRING_Y2S[2]}
					stroke="rgb(109, 93, 93)"
					stroke-width="2.0"
				/>
				<line
					x1={STRING_X1}
					y1={STRING_Y1S[3]}
					x2={STRING_X2S[3]}
					y2={STRING_Y2S[3]}
					stroke="rgb(109, 93, 93)"
					stroke-width="1.5"
				/>
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
