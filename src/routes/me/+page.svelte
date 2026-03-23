<script lang="ts">
	// Replace this import with your actual photo when ready
	import photo from '$lib/assets/me_00.jpeg';

	// Staggered entrance — track whether page has mounted
	import { onMount } from 'svelte';
	let mounted = $state(false);
	onMount(() => {
		// Small delay so CSS transition fires after first paint
		requestAnimationFrame(() => {
			mounted = true;
		});
	});

	function getAge() {
		let birthDate = '2002-03-31'; // Input date
		const birthday = new Date(birthDate);
		const today = new Date();

		// Basic year subtraction
		let age = today.getFullYear() - birthday.getFullYear();

		// Check if the birthday has happened this year
		const monthDiff = today.getMonth() - birthday.getMonth();
		const dayDiff = today.getDate() - birthday.getDate();

		if (monthDiff < 0 || (monthDiff === 0 && dayDiff < 0)) {
			age--;
		}

		return age;
	}
</script>

<main class="flex min-h-screen flex-col bg-[#0e0e0e]">
	<!-- Grain overlay (matches main page) -->
	<div
		class="pointer-events-none fixed inset-0 z-50 opacity-[0.04]"
		style="background-image: url('data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 width=%22200%22 height=%22200%22><filter id=%22n%22><feTurbulence type=%22fractalNoise%22 baseFrequency=%220.9%22 numOctaves=%224%22/></filter><rect width=%22200%22 height=%22200%22 filter=%22url(%23n)%22 opacity=%221%22/></svg>')"
	></div>

	<!-- Nav -->
	<nav class="flex w-full items-center justify-between px-12 pt-8">
		<a
			href="/"
			class="font-mono text-[11px] tracking-[0.25em] text-white/30 uppercase transition-colors duration-300 hover:text-white/70"
		>
			← back
		</a>
		<div class="flex gap-10">
			{#each ['Me', 'Projects', 'Contact'] as label}
				<a
					href={`/${label.toLowerCase()}`}
					class={`font-mono text-[11px] tracking-[0.2em] uppercase transition-colors duration-300 ${label === 'Me' ? 'text-white/80' : 'text-white/30 hover:text-white/70'}`}
				>
					{label}
				</a>
			{/each}
		</div>
	</nav>

	<!-- Main content: photo + text side by side -->
	<div class="flex flex-1 items-center justify-center px-12 py-16">
		<div class="mx-auto flex max-w-5xl items-center gap-16 lg:gap-24">
			<!-- ── Left: Photo ── -->
			<div
				class="relative shrink-0 transition-all duration-700 ease-out"
				class:opacity-0={!mounted}
				class:translate-y-4={!mounted}
				class:opacity-100={mounted}
				class:translate-y-0={mounted}
				style="transition-delay: 0.05s;"
			>
				<!-- Decorative offset border -->
				<div class="absolute -top-3 -left-3 h-full w-full border border-white/10"></div>

				<div class="relative h-96 w-72 overflow-hidden bg-white/5 lg:h-105 lg:w-80">
					{#if photo}
						<img
							src={photo}
							alt="Portrait"
							class="h-full w-full object-cover object-center transition-all duration-700 hover:grayscale-0"
						/>
					{:else}
						<!-- Placeholder: textured rectangle -->
						<div
							class="flex h-full w-full flex-col items-center justify-center gap-3"
							style="background: linear-gradient(135deg, #1a1a1a 0%, #111 50%, #161616 100%);"
						>
							<div
								class="flex h-16 w-16 items-center justify-center rounded-full border border-white/10"
							>
								<svg
									width="28"
									height="28"
									viewBox="0 0 24 24"
									fill="none"
									stroke="rgba(255,255,255,0.2)"
									stroke-width="1"
								>
									<circle cx="12" cy="8" r="4" />
									<path d="M4 20c0-4 3.6-7 8-7s8 3 8 7" />
								</svg>
							</div>
							<span class="font-mono text-[9px] tracking-[0.3em] text-white/20 uppercase"
								>photo here</span
							>
						</div>
					{/if}
				</div>

				<!-- Small accent line below photo -->
				<div class="mt-4 h-px w-12 bg-white/20"></div>
			</div>

			<!-- ── Right: Text ── -->
			<div class="flex max-w-lg flex-col gap-8">
				<!-- Title block -->
				<div
					class="transition-all duration-700 ease-out"
					class:opacity-0={!mounted}
					class:translate-y-4={!mounted}
					class:opacity-100={mounted}
					class:translate-y-0={mounted}
					style="transition-delay: 0.15s;"
				>
					<!-- Eyebrow label -->
					<p class="mb-4 font-mono text-[10px] tracking-[0.35em] text-white/30 uppercase">
						about me
					</p>
					<!-- Big display name -->
					<h1
						class="text-6xl leading-none font-light tracking-tight text-white/90 lg:text-7xl"
						style="font-family: 'Cormorant Garamond', serif;"
					>
						Hi!<br />
						<em class="text-white/50 italic">I'm Èric Ryhr.</em>
					</h1>
				</div>

				<!-- Divider -->
				<div
					class="flex items-center gap-4 transition-all duration-700 ease-out"
					class:opacity-0={!mounted}
					class:translate-y-4={!mounted}
					class:opacity-100={mounted}
					class:translate-y-0={mounted}
					style="transition-delay: 0.25s;"
				>
					<div class="h-px flex-1 bg-white/10"></div>
					<span class="font-mono text-[9px] tracking-[0.3em] text-white/20">///</span>
				</div>

				<!-- Paragraph 1 -->
				<p
					class="text-base leading-relaxed font-light text-white/60 lg:text-lg"
					style="font-family: 'Cormorant Garamond', serif; transition-delay: 0.35s;"
					class:opacity-0={!mounted}
					class:translate-y-4={!mounted}
					class:opacity-100={mounted}
					class:translate-y-0={mounted}
				>
					I'm a {getAge()} year old AI engineer based in Barcelona with a side passion for playing music.
					I've been coding since I was 13, and professionally for the past 3 years. I specialize in building
					applications that solve real-world problems, and I'm always eager to learn new technologies
					and take on exciting projects.
				</p>

				<!-- Paragraph 2 -->
				<p
					class="text-base leading-relaxed font-light text-white/40 lg:text-lg"
					style="font-family: 'Cormorant Garamond', serif; transition-delay: 0.45s;"
					class:opacity-0={!mounted}
					class:translate-y-4={!mounted}
					class:opacity-100={mounted}
					class:translate-y-0={mounted}
				>
					When I'm not coding, I'm usually playing bass with my band <a
						href="https://www.instagram.com/strayfrogs.band/"
						target="_blank"
						class="text-white/60 underline hover:text-white/90">Stray Frogs</a
					>. I see music and coding as similar crafts. Both require careful composition and
					attention to detail to create something meaningful.
				</p>

				<!-- Footer detail -->
				<div
					class="pt-4 transition-all duration-700 ease-out"
					class:opacity-0={!mounted}
					class:translate-y-4={!mounted}
					class:opacity-100={mounted}
					class:translate-y-0={mounted}
					style="transition-delay: 0.55s;"
				>
					<span class="font-mono text-[10px] tracking-[0.25em] text-white/20 uppercase">
						Barcelona, Cat
					</span>
				</div>
			</div>
		</div>
	</div>
</main>

<style>
	:global(html, body) {
		margin: 0;
		padding: 0;
		background: #0e0e0e;
	}
</style>
