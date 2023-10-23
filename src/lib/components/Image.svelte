<script lang="ts">
	export let src: string;
	export let alt: string;

	export let clientHeight;

	let isFullscreen = false;
</script>

<div class="group relative h-max" bind:clientHeight>
	<!-- Download Button -->
	<a
		href={src}
		target="_blank"
		rel="noopener noreferrer"
		download
		class="absolute right-4 top-4 rounded-lg bg-white px-3 py-2 opacity-0 transition duration-200 hover:bg-slate-200 group-hover:opacity-100"

		
	>
		↓
	</a>

	<!-- Image -->
	<img on:click={()=>{isFullscreen = !isFullscreen}} {src} {alt} class="hover:border-8 border-slate-100 {isFullscreen ? 'fixed z-50 top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 h-[90vh]': ''}" />
	<div class="h-screen w-screen bg-white/30  backdrop-blur  fixed top-0 left-0 z-40 " class:hidden={!isFullscreen} on:click={()=>{isFullscreen = !isFullscreen}}></div>
</div>
