<script lang="ts">
	import { onMount } from 'svelte';

	let mounted = $state(false);

	// ── Obfuscated email ───────────────────────────────────────────────────────
	// Replace each part with your actual address.
	const emailParts = ['ryhrmateu', '@', 'gmail', '.', 'com'];
	let emailHref = $state('');

	onMount(() => {
		requestAnimationFrame(() => {
			mounted = true;
		});
		emailHref = 'mailto:' + emailParts.join('');
	});

	// ── Social links ───────────────────────────────────────────────────────────
	const links = [
		{
			label: 'LinkedIn',
			sub: 'linkedin.com/in/èric-ryhr-mateu-a9aa80250',
			href: 'https://linkedin.com/in/èric-ryhr-mateu-a9aa80250',
			external: true
		},
		{
			label: 'Email',
			sub: emailParts.join(''),
			get href() {
				return emailHref;
			},
			external: false
		},
		{
			label: 'GitHub',
			sub: 'github.com/ericryhr',
			href: 'https://github.com/ericryhr',
			external: true
		}
	];
</script>

<main class="flex min-h-screen flex-col bg-[#0e0e0e]">
	<!-- Content -->
	<div class="flex flex-1 items-center justify-center px-12 py-16">
		<div class="flex w-full max-w-lg flex-col gap-3">
			<!-- Header -->
			<div
				class="transition-all duration-700 ease-out"
				class:opacity-0={!mounted}
				class:translate-y-4={!mounted}
				class:opacity-100={mounted}
				class:translate-y-0={mounted}
				style="transition-delay: 0.05s;"
			>
				<p class="mb-4 font-mono text-[10px] tracking-[0.35em] text-white/30 uppercase">
					get in touch
				</p>
				<h1
					class="text-6xl leading-none font-light tracking-tight text-white/90 lg:text-7xl"
					style="font-family: 'Cormorant Garamond', serif;"
				>
					Let's<br />
					<em class="text-white/50 italic">talk.</em>
				</h1>
			</div>

			<!-- Divider -->
			<div
				class="flex items-center gap-4 transition-all duration-700 ease-out"
				class:opacity-0={!mounted}
				class:translate-y-4={!mounted}
				class:opacity-100={mounted}
				class:translate-y-0={mounted}
				style="transition-delay: 0.15s;"
			>
				<div class="h-px flex-1 bg-white/10"></div>
				<span class="font-mono text-[9px] tracking-[0.3em] text-white/20">///</span>
			</div>

			<!-- Link list -->
			<ul class="flex flex-col">
				{#each links as link, i}
					<li
						class="transition-all duration-700 ease-out"
						class:opacity-0={!mounted}
						class:translate-y-4={!mounted}
						class:opacity-100={mounted}
						class:translate-y-0={mounted}
						style="transition-delay: {0.25 + i * 0.1}s;"
					>
						<a
							href={link.href}
							target={link.external ? '_blank' : undefined}
							rel={link.external ? 'noopener noreferrer' : undefined}
							class="link-row group flex items-center justify-between border-b border-white/10 py-5 transition-all duration-300 hover:border-white/25"
						>
							<!-- Left: icon + label -->
							<div class="flex items-center gap-3">
								{#if link.label === 'GitHub'}
									<svg
										width="18"
										height="18"
										viewBox="0 0 24 24"
										fill="none"
										class="link-icon"
										stroke-width="1.5"
										stroke-linecap="round"
										stroke-linejoin="round"
									>
										<path
											d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"
										/>
									</svg>
								{:else if link.label === 'LinkedIn'}
									<svg
										width="18"
										height="18"
										viewBox="0 0 24 24"
										fill="none"
										class="link-icon"
										stroke-width="1.5"
										stroke-linecap="round"
										stroke-linejoin="round"
									>
										<path
											d="M16 8a6 6 0 0 1 6 6v7h-4v-7a2 2 0 0 0-2-2 2 2 0 0 0-2 2v7h-4v-7a6 6 0 0 1 6-6z"
										/>
										<rect x="2" y="9" width="4" height="12" />
										<circle cx="4" cy="4" r="2" />
									</svg>
								{:else}
									<svg
										width="18"
										height="18"
										viewBox="0 0 24 24"
										fill="none"
										class="link-icon"
										stroke-width="1.5"
										stroke-linecap="round"
										stroke-linejoin="round"
									>
										<path
											d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"
										/>
										<polyline points="22,6 12,13 2,6" />
									</svg>
								{/if}

								<span
									class="link-label text-2xl font-light transition-colors duration-300"
									style="font-family: 'Cormorant Garamond', serif;"
								>
									{link.label}
								</span>
							</div>

							<!-- Right: handle + arrow -->
							<div class="flex items-center gap-4">
								<span
									class="link-arrow inline-block transition-all duration-300 group-hover:translate-x-1"
								>
									→
								</span>
							</div>
						</a>
					</li>
				{/each}
			</ul>
		</div>
	</div>
</main>

<style>
	/* Base state — all dimmed white */
	.link-icon {
		stroke: rgba(255, 255, 255, 0.3);
		transition: stroke 0.3s;
	}
	.link-label {
		color: rgba(255, 255, 255, 0.65);
		transition: color 0.3s;
	}
	.link-arrow {
		color: rgba(255, 255, 255, 0.3);
		transition: color 0.3s;
	}

	/* Hover state — everything brightens */
	.link-row:hover .link-icon {
		stroke: rgba(255, 255, 255, 0.75);
	}
	.link-row:hover .link-label {
		color: rgba(255, 255, 255, 0.95);
	}
	.link-row:hover .link-arrow {
		color: rgba(255, 255, 255, 0.75);
	}
</style>
