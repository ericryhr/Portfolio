<script lang="ts">
	import { onMount } from 'svelte';
	import smarttracking_00 from '$lib/assets/smarttracking_00.png';

	// ── Projects data ──────────────────────────────────────────────────────────
	// Replace with your actual projects. `image` can be an import path or null for placeholder.
	const projects = [
		{
			title: 'Hawk-Eye Clone',
			blurb:
				'Recreation of the Hawk-Eye visualization system used in tennis broadcasts, but using AI-upscaled real video for the ball impact points instead of CGI.',
			stack: ['Unity', 'C#', 'C++', 'Blender'],
			video: null,
			image: null,
			accent: '#1a3a2a', // placeholder gradient accent colour — swap per project
			accent2: '#0a1a14'
		},
		{
			title: 'Padel Smart Tracker',
			blurb:
				'Triangulation of ball and player positions in real-time using six cameras around a padel court and estimation of trajectories for prediction of future positions, for use in match analysis.',
			stack: ['Unreal Engine', 'C++', 'Blender'],
			video: null,
			image: smarttracking_00,
			accent: '#1a1a3a',
			accent2: '#0a0a1a'
		},
		{
			title: 'Training Object Detection Models on Synthetic Images',
			blurb:
				"Master's thesis on the use of synthetic data generated from 3D models and a game engine for training object detection models, as an alternative to manual annotation of real images.",
			stack: ['YOLO', 'Unity', 'C#', 'Blender', 'Python'],
			video: null,
			image: null,
			accent: '#1a3a1a',
			accent2: '#0a1a0a'
		},
		{
			title: 'Inertial Movement Detector Deployment',
			blurb:
				"Bachelor's thesis on the mobile deployment of an inertial movement detection model for tracking human movement using smartphone sensors, without the use of GPS.",
			stack: ['Android', 'iOS', 'Unity', 'C#', 'ONNX', 'PyTorch'],
			video: '/videos/demo_escales_opt.mp4',
			image: null,
			accent: '#3a1a1a',
			accent2: '#1a0a0a'
		},
		{
			title: 'Climbing Safety System',
			blurb:
				'Design and deployment of a computer vision system that detects if a climber is well tied in before they start climbing, to prevent accidents.',
			stack: ['Raspberry Pi', 'Python'],
			video: null,
			image: null,
			accent: '#3a3a1a',
			accent2: '#1a1a0a'
		}
	];

	// ── Scroll reveal ──────────────────────────────────────────────────────────
	let visible = $state(projects.map(() => false));

	onMount(() => {
		const observers: IntersectionObserver[] = [];

		projects.forEach((_, i) => {
			const el = document.getElementById(`project-${i}`);
			if (!el) return;

			const obs = new IntersectionObserver(
				([entry]) => {
					if (entry.isIntersecting) {
						visible[i] = true;
						obs.disconnect();
					}
				},
				{ threshold: 0.15 }
			);
			obs.observe(el);
			observers.push(obs);
		});

		return () => observers.forEach((o) => o.disconnect());
	});
</script>

<main class="flex min-h-screen flex-col bg-[#0e0e0e]">
	<!-- Page header -->
	<header class="mx-auto w-full max-w-5xl px-12 pt-20 pb-16">
		<p class="mb-4 font-mono text-[10px] tracking-[0.35em] text-white/30 uppercase">
			selected work
		</p>
		<h1
			class="text-6xl leading-none font-light tracking-tight text-white/90 lg:text-7xl"
			style="font-family: 'Cormorant Garamond', serif;"
		>
			Projects<br />
			<em class="text-white/40 italic">&amp; things I've worked on.</em>
		</h1>
	</header>

	<!-- Project list -->
	<section class="mx-auto flex w-full max-w-5xl flex-col gap-32 px-12 pb-32">
		{#each projects as project, i}
			{@const flipped = i % 2 !== 0}

			<article
				id="project-{i}"
				class="flex items-center gap-12 transition-all duration-700 ease-out lg:gap-20"
				class:opacity-0={!visible[i]}
				class:translate-y-8={!visible[i]}
				class:opacity-100={visible[i]}
				class:translate-y-0={visible[i]}
				class:flex-row-reverse={flipped}
			>
				<!-- ── Graphic / image ── -->
				<div class="relative h-56 w-72 shrink-0 overflow-hidden lg:h-64 lg:w-96">
					{#if project.image}
						<img src={project.image} alt={project.title} class="h-full w-full object-cover" />
					{:else if project.video}
						<video
							src={project.video}
							autoplay
							muted
							loop
							playsinline
							preload="none"
							class="h-full w-full object-cover"
						>
						</video>
					{:else}
						<!-- Animated gradient placeholder -->
						<div
							class="animated-gradient h-full w-full"
							style="--c1: {project.accent}; --c2: {project.accent2};"
						></div>
						<!-- Decorative offset border -->
						<div
							class="pointer-events-none absolute border border-white/10"
							class:-top-2={!flipped}
							class:-right-2={!flipped}
							class:-bottom-2={flipped}
							class:-left-2={flipped}
							style="inset: {flipped ? '8px -8px -8px 8px' : '-8px 8px 8px -8px'};"
						></div>
					{/if}
				</div>

				<!-- ── Description ── -->
				<div class="flex flex-1 flex-col gap-6">
					<!-- Index + title -->
					<div>
						<span class="font-mono text-[10px] tracking-[0.3em] text-white/20 uppercase">
							{String(i + 1).padStart(2, '0')}
						</span>
						<h2
							class="mt-1 text-4xl leading-tight font-light text-white/85 lg:text-5xl"
							style="font-family: 'Cormorant Garamond', serif;"
						>
							{project.title}
						</h2>
					</div>

					<!-- Thin rule -->
					<div class="h-px w-12 bg-white/15"></div>

					<!-- Blurb -->
					<p
						class="text-base leading-relaxed font-light text-white/50"
						style="font-family: 'Cormorant Garamond', serif;"
					>
						{project.blurb}
					</p>

					<!-- Stack tags -->
					<div class="flex flex-wrap gap-2">
						{#each project.stack as tag}
							<span
								class="border border-white/15 px-3 py-1 font-mono text-[9px] tracking-[0.2em] text-white/40 uppercase"
							>
								{tag}
							</span>
						{/each}
					</div>
				</div>
			</article>
		{/each}
	</section>
</main>

<style>
	/* Animated gradient placeholder */
	@keyframes gradient-shift {
		0% {
			background-position: 0% 50%;
		}
		50% {
			background-position: 100% 50%;
		}
		100% {
			background-position: 0% 50%;
		}
	}

	.animated-gradient {
		background: linear-gradient(
			135deg,
			var(--c2) 0%,
			var(--c1) 40%,
			#0e0e0e 60%,
			var(--c1) 80%,
			var(--c2) 100%
		);
		background-size: 300% 300%;
		animation: gradient-shift 6s ease infinite;
	}
</style>
