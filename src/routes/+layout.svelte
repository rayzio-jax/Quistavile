<script>
	import { page } from "$app/stores";
	import { onDestroy, onMount } from "svelte";
	import "../app.css";

	let gradientPercentages = [0, 0, 0];
	let delay = 0; // 0.2 seconds

	function animateMask() {
		function updateMask() {
			for (let i = 0; i < gradientPercentages.length; i++) {
				if (gradientPercentages[i] > 0) {
					gradientPercentages[i] -= 1;
					break;
				} else if (i === gradientPercentages.length - 1) {
					gradientPercentages = [100, 100, 100];
				}
			}

			for (let i = 0; i < gradientPercentages.length; i++) {
				document.body.style.setProperty(`--percent${i + 1}`, `${gradientPercentages[i]}%`);
			}
		}

		setInterval(updateMask, delay);
	}

	// Start animate mask if mounted
	onMount(() => {
		document.body.classList.add("mounted");
		animateMask();
	});
</script>

<div class="body min-h-screen">
	{#if $page.url.pathname.startsWith("/quistavile")}
		<main class="h-screen p-5">
			<slot />
		</main>

		<style>
			div.body {
				cursor: url("/cursor/green-pixelate.png"), auto;
			}
		</style>
	{:else}
		<main class="flex items-center justify-center h-screen">
			<slot />
		</main>
	{/if}
</div>

<style>
	:global(html, body) {
		margin: 0;
		padding: 0;
		width: 100%;
		height: 100%;
		overflow-x: hidden; /* Prevent horizontal scrolling */
	}

	:global(body.mounted::before) {
		--size: 45px;
		--line: color-mix(in lch, rgba(255, 255, 255, 0.5), transparent 50%);
		content: "";
		height: 100vh;
		width: 100vw;
		position: fixed;
		background:
			linear-gradient(90deg, var(--line) 1px, transparent 1px var(--size)) 50% 50% / var(--size) var(--size),
			linear-gradient(var(--line) 1px, transparent 1px var(--size)) 50% 50% / var(--size) var(--size);
		mask: linear-gradient(150deg, transparent var(--percent1), white var(--percent2), transparent var(--percent3));
		top: 0;
		transform-style: flat;
		pointer-events: none;
		z-index: -999999;
	}
</style>
