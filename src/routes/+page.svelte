<script>
	import { writable } from "svelte/store";
	import { goto } from "$app/navigation"; // Import goto for navigation
	import { beforeNavigate } from "$app/navigation";
	import { browser } from "$app/environment";
	import { onMount } from "svelte";

	// Import components
	import TypingText from "$lib/components/TypingText.svelte";
	import GameButton from "../lib/components/GameButton.svelte";

	// Button disable state
	let buttonDisable = false;

	// Title page
	// Add `{delay}` within the string to add text delay
	const titleText = writable(["", "HELLO THERE!", "YOU DON'T REMEMBER ME?", "IT'S ME!{300}{reset} YOUR ∎∎∎∎∎", "ME!{300} YOUR{300} ∎{300}∎{300}∎{300}∎{300}∎"]);

	// Button text
	const btnText = writable(["WHAT IS THIS?", "WHO ARE YOU?", "NO.", "WHO?", "I DON'T KNOW"]);

	// Click counter
	let counter = writable(0);

	// Change text index on click
	function changeTextClick() {
		// Play sound on click
		const audio = new Audio("https://res.cloudinary.com/dseyaxooa/video/upload/v1732249278/ozkyvxtiagxe4moirnva.wav");
		audio.volume = 0.3;
		if (audio.paused) audio.play();

		// Update index
		counter.update((n) => {
			if (n >= $btnText.length - 1) {
				goto("/quistavile");
				return n; // Do not increment further
			}
			return n + 1; // Increment the counter
		});
	}

	// Get disable state from child comp
	function handleDisable(e) {
		buttonDisable = e.detail;
	}

	let btnState;
	$: btnState = counter && buttonDisable ? "opacity-0 hidden" : "opacity-100 block";

	let textState;
	$: if (buttonDisable) {
		textState = ".";
	} else {
		setTimeout(() => {
			textState = $btnText[$counter];
		}, 100);
	}

	// Set text to empty before navigating
	if (browser) {
		beforeNavigate(() => {
			titleText.set([]);
			btnText.set([]);
		});
	}

	onMount(() => {
		document.documentElement.scrollTop = 0; // Reset scroll position
		document.body.style.overflow = "hidden"; // Ensure no overflow
	});
</script>

<div class="flex flex-col justify-center items-center">
	<TypingText typingSFX="https://res.cloudinary.com/dseyaxooa/video/upload/v1732239065/wgqhp9pnw0nenrfqiwj1.mp3" on:toggleDisabled={handleDisable} compClass="text-[40px] text-pretty mx-12 md:text-[46px] xl:text-[64px] leading-tight xl:h-full min-h-[72px] drop-shadow-lg select-none text-center" text={$titleText[$counter]} counter={$counter} />

	<GameButton compClass="mt-8 xl:mt-12 transition-transform duration-300 {btnState}" disableMode={buttonDisable} text={textState} onClick={changeTextClick} />
</div>
