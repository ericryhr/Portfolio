<script lang="ts">
	import { page } from '$app/state';

	const nav = ['Me', 'Projects', 'Contact'];
</script>

<!-- Nav -->
<nav class="flex w-full items-center justify-between bg-[#0e0e0e] px-12 pt-8">
	<a
		href="/"
		class="font-mono text-[11px] tracking-[0.25em] text-white/30 uppercase transition-colors duration-300 hover:text-white/70"
	>
		← back
	</a>
	<div class="flex gap-10">
		{#each nav as label}
			<a
				href={`/${label.toLowerCase()}`}
				class={`font-mono text-[11px] tracking-[0.2em] uppercase transition-colors duration-300 ${page.url.pathname.includes(label.toLowerCase()) ? 'text-white/80' : 'text-white/30 hover:text-white/70'}`}
			>
				{label}
			</a>
		{/each}
	</div>
</nav>

<slot />
