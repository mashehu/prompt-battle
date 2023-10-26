<script lang="ts">
	import { enhance, type SubmitFunction } from '$app/forms';
	import { slide, fly } from 'svelte/transition';
	import { quintOut } from 'svelte/easing';

	import { user, colorizedBackground, theme } from '$lib/store';

	import ActionBar from '$lib/layout/ActionBar.svelte';
	import Button from '$lib/components/Button.svelte';
	import Loading from '$lib/components/Loading.svelte';
	import Result from '$lib/components/Result.svelte';

	import { onMount } from 'svelte';

	$colorizedBackground = true;

	let loading = false;
	let prompt: string;
	let urls: string[];

	const handleSubmit: SubmitFunction = () => {
		$colorizedBackground = false;
		loading = true;

		return async ({ action, result }) => {
			let response = JSON.parse(JSON.stringify(result));

			if (response.status == 200) {
				urls = response.data.urls;

				// Makes sure that the image is loaded properly
				setTimeout(() => {
					loading = false;
				}, 1000);
			} else {
				loading = false;
				alert('An error occurred, please try again.');
			}
		};
	};

	let duration = 30; // in seconds
	const circumference = 2 * Math.PI * 30; // 30 is the radius of the circle
	let elapsed = 0;
	let timeLeft = duration;
	let circleColor: string;

	onMount(() => {
		circleColor = getComputedStyle(
			document.querySelector('main')
		).backgroundColor;
		const interval = setInterval(() => {
			if (elapsed < duration) {
				elapsed += 1;
				timeLeft = duration - elapsed;
			} else {
				// submit
				document.getElementById('promptForm').requestSubmit();
				clearInterval(interval);
			}
		}, 1000);
		return () => clearInterval(interval); // Cleanup
	});

	$: offset = circumference * (1 - elapsed / duration);
</script>

<div class="flex min-h-screen flex-col">
	<div class="flex min-h-[90vh] items-center justify-center">
		<!-- Prompting -->
		{#if !loading && urls == undefined}
			<form
				id="promptForm"
				method="POST"
				use:enhance={handleSubmit}
				class="h-full w-full px-6 pb-6 pt-14"
			>
				<label for="prompt">
					<textarea
						name="prompt"
						class="prompt-input"
						bind:value={prompt}
						placeholder="Enter prompt"
					/>
				</label>
			</form>
		{/if}

		<!-- Loading state / Sending the request to the api -->
		{#if loading}
			<div
				in:fly={{
					delay: 300,
					duration: 1000,
					y: 25,
					opacity: 0,
					easing: quintOut
				}}
			>
				<Loading />
			</div>
		{/if}

		<!-- Display the result -->
		{#if !loading && urls != undefined}
			<div
				in:fly={{
					delay: 300,
					duration: 1000,
					y: 25,
					opacity: 0,
					easing: quintOut
				}}
			>
				<Result image={urls} {prompt} user={$user} />
			</div>
		{/if}
	</div>

	<!-- Prompting -->
	{#if !loading && urls == undefined}
		<div out:slide={{ duration: 1000, easing: quintOut }}>
			<ActionBar>
				<div class="my-4 ml-4">
					<svg
						width="70"
						height="70"
						viewbox="0 0 70 70"
						xmlns="http://www.w3.org/2000/svg"
					>
						<!-- Background circle -->
						<circle
							cx="35"
							cy="35"
							r="30"
							fill="none"
							stroke="lightgrey"
							stroke-width="2"
						/>

						<!-- Countdown circle -->
						<circle
							cx="35"
							cy="35"
							r="30"
							fill="none"
							stroke={circleColor}
							stroke-width="2"
							stroke-dasharray={circumference}
							stroke-dashoffset={offset}
							transform="rotate(-90 35 35)"
						/>

						<!-- Time text in the center -->
						<text
							x="35"
							y="45"
							font-family="Arial"
							font-size="30"
							fill="black"
							text-anchor="middle">{timeLeft}s</text
						>
					</svg>

					
				</div>
				<div class="my-4 ml-4">
				<p class="text-lg text-slate-900">{$user}</p>
				</div>
				<div class="ml-auto">
					<Button label="Generate the image" type="submit" form="promptForm" />
				</div>
			</ActionBar>
		</div>
	{/if}

	<!-- Start a new game -->
	{#if !loading && urls != undefined}
		<div in:slide={{ duration: 1000, easing: quintOut }}>
			<ActionBar>
				<Button
					label="Start a new game"
					type="button"
					href="/"
					class="w-full"
				/>
			</ActionBar>
		</div>
	{/if}
</div>

<style lang="postcss">
	.prompt-input {
		@apply h-full w-full bg-transparent font-redaction text-8xl text-slate-100 focus:outline-none focus:ring-0;
	}
	.prompt-input::placeholder {
		@apply text-slate-100;
	}
</style>
