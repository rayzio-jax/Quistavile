<script>
	import { createEventDispatcher } from 'svelte';

	export let text; // The text to type
	export let counter = null; // Counter to check current text in array
	export let compClass = '';
	export let typingSFX = '';

	const dispatch = createEventDispatcher();

	let isBtnDisabled = false;
	let isTypingFinish = false;

	let typedChar = ''; // Text that has been typed so far
	let index = 0; // Current character position in text
	let typewriter; // Interval for typing animation
	let keySFX; // Audio player

	const typingSpeed = 100; // Typing speed in ms

	// AUDIO SFX
	const playSFX = () => {
		keySFX = new Audio(typingSFX);
		keySFX.play();
		keySFX.volume = 0.5;
	};

	const startTyping = () => {
		isTypingFinish = false;
		// Start typing the current text prop
		if (text && text.length > 0) {
			typewriter = setInterval(() => {
				// Default typewritter effect
				if (index < text.length) {
					let matchNewLine;
					let matchCommand;

					matchCommand = text.slice(index).match(/^\{([^}]+)\}/);
					if (matchCommand) {
						let command = matchCommand[1];
						if (/^reset$/i.test(command)) {
							clearInterval(typewriter);
							typedChar = '';
							index += matchCommand[0].length;
							startTyping();
							return;
						} else {
							let delay = parseInt(command); // Parse command to integer

							// Add typing delay if command is match
							if (!isNaN(delay)) {
								clearInterval(typewriter);
								setTimeout(() => {
									index += matchCommand[0].length;
									startTyping();
								}, delay);
								return;
							}
						}
					}

					matchNewLine = text.slice(index).match(/^<br\s*\/?>/i);
					if (matchNewLine) {
						clearInterval(typewriter);
						index += matchNewLine[0].length;
						typedChar += '<br>'; // Skip <br> tag
						startTyping();
						return;
					}

					typedChar += text[index];
					if (playSFX) playSFX(); // Ensure playSFX is defined before calling it
					index++;

					isBtnDisabled = true; // Disable button when typing is done
				} else {
					// If index is out of bounds, stop typing
					isTypingFinish = true;
					clearInterval(typewriter); // Stop typing when done
					isBtnDisabled = false;
				}

				dispatch('toggleDisabled', isBtnDisabled); // Dispatch event that button on parent have to disabled
				dispatch('typingFinished', isTypingFinish); // Dispatch event that typing is finished
			}, typingSpeed);
		}
	};

	let prevCounter; // Monitoring current text index in array

	$: if (text) {
		// Reset typed char if current index of text is changing
		if (prevCounter !== counter) {
			typedChar = ''; // Reset typed text
			index = 0; // Reset index
		}
		clearInterval(typewriter); // Stop any ongoing typing
		startTyping(); // Restart typing whenever text changes
		prevCounter = counter;
	}
</script>

<!-- Display the typed characters -->
<p
	class="{compClass} select-none text-white"
	class:hidden={counter === 0}
	class:block={!counter === 0}
>
	{@html typedChar}{#if text}
		<span class="blink-cursor">_</span>
	{/if}
</p>

<style>
	.blink-cursor {
		animation: blink 800ms step-end infinite;
		animation-timing-function: cubic-bezier(0.175, 0.885, 0.32, 1.275);
	}

	@keyframes blink {
		from,
		to {
			opacity: 1;
		}
		50% {
			opacity: 0;
		}
	}
</style>
