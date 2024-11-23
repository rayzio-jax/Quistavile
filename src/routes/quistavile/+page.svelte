<script>
	// components
	import TypingText from '$lib/components/TypingText.svelte';
	import GameButton from '$lib/components/GameButton.svelte';

	// import song from "$lib/audio/glimpse-loop.mp3";
	import { beforeNavigate } from '$app/navigation';
	import { browser } from '$app/environment';
	import { onMount } from 'svelte';

	// Text variables
	const starterText = 'WHO ARE YOU?';
	let paragraph;
	let isParagraphStart = false;
	let isTypingFinished = false;

	// Music variables
	let isMusicPlaying = false;
	let songPlayer;
	let buttonAudio;

	// Rotation animation variables
	let angle = 0;
	let rotating = false;
	let animationFrame;

	// Splitted char config
	let chars;
	let words;
	let charWrappers;
	let viewportWidth;
	let size = 20; // Wrapper size

	// Set paragraph with inputted text
	const startText = () => {
		paragraph = `I'M ∎∎∎∎∎ ,{300}<br>
		YOUR ∎∎∎∎∎ !{300}<br>
		YOU REALLY DON'T REMEMBER?{1000}
		YOU'VE BEEN DAYDREAMING FOR YEARS.{500}{reset}
		I'VE BEEN WAITING FOR YOU TO BACK...{500}<br>
		BUT NOW{500} YOU'VE FORGOTTEN ME{300}.{300}.{300}.`;
		isParagraphStart = true;

		startMusic(); // Play the song
		buttonSfxPlay();
	};

	// MUSIC PLAYER
	if (browser) {
		// Song player
		const songMp3 =
			'https://res.cloudinary.com/dseyaxooa/video/upload/v1732238913/ibxds0nxoy5ittmsfopd.mp3';
		songPlayer = new Audio(songMp3);
		songPlayer.volume = 0.5;
		songPlayer.loop = true;

		// Button audio
		const btnSFX =
			'https://res.cloudinary.com/dseyaxooa/video/upload/v1732249085/fg8pgikhzne5intfgzdy.wav';
		buttonAudio = new Audio(btnSFX);
		buttonAudio.volume = 0.3;

		// Pause the song when the page is unloaded
		beforeNavigate(() => {
			if (isMusicPlaying) {
				songPlayer.pause();
				isMusicPlaying = false;
			}
			paragraph = '';
		});
	}

	// Get state when typing finished
	function handleTyping(e) {
		isTypingFinished = e.detail;
	}

	// Init button sound
	const buttonSfxPlay = () => {
		if (buttonAudio.paused || !isParagraphStart) {
			buttonAudio.play();
		}
	};

	// Init start music
	const startMusic = () => {
		if (songPlayer.paused || !isParagraphStart) {
			startRotateDisc();
			songPlayer.play();
			isMusicPlaying = true;
		}
	};

	// Toggle music play/pause
	const toggleMusic = () => {
		if (songPlayer.paused) {
			startRotateDisc();
			songPlayer.play();
			isMusicPlaying = true;
		} else {
			stopRotateDisc();
			songPlayer.pause();
			isMusicPlaying = false;
		}
	};

	// Update rotation state
	const rotateDisc = () => {
		if (rotating) {
			angle += 2;
			if (angle >= 360) {
				angle = angle % 360;
			}
			animationFrame = requestAnimationFrame(rotateDisc);
		}
	};

	const startRotateDisc = () => {
		if (!rotating) {
			rotating = true;
			rotateDisc();
		}
	};

	const stopRotateDisc = () => {
		rotating = false;
		cancelAnimationFrame(animationFrame);
	};

	// Handle `Enter` key
	const handleKeyDown = (e) => {
		if (e.key === 'Enter' || e.key === ' ') {
			toggleMusic();
		}
	};

	const animateChars = () => {
		words.forEach((word) => {
			word.classList.add('flex', 'justify-center', 'items-center', 'gap-4', 'md:gap-6', 'xl:gap-8');
		});

		if (!chars.length) {
			console.error('No characters found for animation');
			return;
		}

		chars.forEach((char, index) => {
			// Calculate delay based on character index
			const total = 0.825; // Total time for the effect
			const delay = (index / chars.length) * total; // Staggered delay

			setTimeout(() => {
				let opacity = 0; // Initial opacity

				// Gradually increase opacity to create a fade-in effect
				const increaseOpacity = setInterval(() => {
					opacity += 0.1; // Increment opacity (adjust step as needed)
					char.style.opacity = opacity.toFixed(1); // Apply to character style

					// Stop the interval when opacity reaches 1
					if (opacity >= 1) {
						clearInterval(increaseOpacity);
					}
				}, 50); // Interval time (adjust for smoother/faster fade-in)
			}, delay); // Convert delay to milliseconds
		});

		setInterval(() => {
			chars.forEach((char) => {
				const randomX = Math.floor(Math.random() * 61 - 10) - 30 * 0.2; // Random X between -20 and 20
				const randomY = Math.floor(Math.random() * 61 - 10) - 30 * 0.2; // Random Y between -20 and 20

				char.style.transform = `translate(${randomX}px, ${randomY}px)`;
			});
		}, 500); // Adjust interval as needed
	};

	let movementText = '';
	onMount(() => {
		movementText = 'Y O U';
	});

	onMount(() => {
		// Update viewportWidth on mount and when the window resizes
		const updateViewportWidth = () => {
			viewportWidth = window.innerWidth;
			size = (viewportWidth / 40) * 2; // Adjust calculation as needed
		};

		const changeWrapperSize = () => {
			updateViewportWidth();
			charWrappers = document.querySelectorAll('p#split .word .wrapper');
			if (charWrappers) {
				charWrappers.forEach((wrapper) => {
					wrapper.style.width = `${size}px`;
					wrapper.style.height = `${size}px`;
				});
			}
		};

		updateViewportWidth(); // Set the initial value
		window.addEventListener('resize', changeWrapperSize);

		return () => {
			// Clean up event listener on component destruction
			window.removeEventListener('resize', changeWrapperSize);
		};
	});

	// Splitted text random movement
	onMount(async () => {
		const Splitting = (await import('splitting')).default;
		Splitting({ target: '#split' });

		chars = document.querySelectorAll('p#split .word .char');
		words = document.querySelectorAll('p#split .word');

		chars.forEach((char) => {
			char.style.opacity = 0;
		});

		chars.forEach((char) => {
			const wrapper = document.createElement('div');
			wrapper.style.position = 'relative';
			wrapper.style.width = `${size}px`;
			wrapper.style.height = `${size}px`;
			wrapper.classList.add('wrapper', 'flex', 'justify-center', 'items-center');

			char.parentNode.replaceChild(wrapper, char);
			wrapper.appendChild(char);

			char.style.position = 'absolute'; // Ensure absolute positioning
			char.classList.add(
				'drop-shadow-[0px_0px_12px_rgba(255,255,255,1)]',
				'transition-opacity',
				'duration-500'
			);
		});
	});

	$: if (isTypingFinished) animateChars();

	// Music player animation SlideIn
	let musicPlayerSlideIn;
	$: musicPlayerSlideIn = isParagraphStart ? '' : 'translate-x-[220px]';

	let btnDisappear;
	$: if (isParagraphStart) {
		btnDisappear = 'opacity-0 transition-opacity duration-1000';
		setTimeout(() => {
			btnDisappear = 'opacity-0 transition-opacity duration-1000 hidden';
		}, 1000);
	}
</script>

<div class="h-full w-full overflow-hidden px-[10vw] pt-[10vh]">
	<div
		id="window"
		class="pointer-events-none absolute left-0 top-0 flex h-full w-full items-center justify-center"
	>
		<p
			id="split"
			data-splitting
			class="flex select-none gap-12 text-6xl font-bold text-slate-300 transition-opacity duration-500 md:text-8xl xl:text-9xl"
			class:opacity-0={!movementText}
			class:opacity-100={movementText}
		>
			{movementText}
		</p>
	</div>

	<TypingText
		on:typingFinished={handleTyping}
		typingSFX="https://res.cloudinary.com/dseyaxooa/video/upload/v1732239065/wgqhp9pnw0nenrfqiwj1.mp3"
		compClass="text-xl md:text-3xl"
		text={paragraph}
	/>

	<GameButton
		compClass="hover:animate-none absolute right-1/2 translate-x-1/2 top-1/2 -translate-y-1/2 {btnDisappear} {!isParagraphStart
			? 'animate-pulse'
			: ''}"
		text={starterText}
		onClick={startText}
	/>

	<div
		id="music-player"
		class="fixed bottom-5 right-0 flex w-[220px] items-center gap-4 overflow-hidden rounded-l-xl bg-gray-300/50 pl-3 transition-transform duration-300 {musicPlayerSlideIn}"
	>
		<div class="relative">
			<div
				style:transform={isParagraphStart && isMusicPlaying
					? `rotate(${angle}deg)`
					: `rotate(${angle}deg)`}
				class="h-[100px] w-[100px] overflow-hidden"
			>
				<img
					class="h-full w-full object-contain"
					src="https://i.ibb.co.com/PzsmR8M/vinyl.png"
					alt=""
				/>
			</div>
			<div
				class="left-0-0 absolute top-0 -z-10 h-[100px] w-[100px] scale-[0.3] overflow-hidden [clip-path:ellipse(50%_50%_at_50%_50%)]"
			>
				<img
					style:transform={isParagraphStart && isMusicPlaying
						? `rotate(${angle}deg)`
						: `rotate(${angle}deg)`}
					class="h-full w-full object-cover"
					src="https://i.ibb.co.com/nDMYxcc/album-image.jpg"
					alt=""
				/>
			</div>
		</div>
		<div class="my-4 flex flex-col items-center justify-center gap-2">
			<div class="flex flex-col">
				<h1 class="w-full text-right text-xl leading-none">glimpse</h1>
				<p class="text-right leading-none">blush</p>
			</div>
			{#if isMusicPlaying}
				<div
					role="button"
					tabindex="0"
					on:click={toggleMusic}
					on:keydown={handleKeyDown}
					class="flex h-6 w-6 shrink-0 overflow-hidden"
				>
					<svg
						class="h-full w-full object-contain"
						xmlns="http://www.w3.org/2000/svg"
						width="32"
						height="32"
						fill="#000000"
						viewBox="0 0 256 256"
					>
						<path
							d="M216,48V208a16,16,0,0,1-16,16H160a16,16,0,0,1-16-16V48a16,16,0,0,1,16-16h40A16,16,0,0,1,216,48ZM96,32H56A16,16,0,0,0,40,48V208a16,16,0,0,0,16,16H96a16,16,0,0,0,16-16V48A16,16,0,0,0,96,32Z"
						></path>
					</svg>
				</div>
			{:else}
				<div
					role="button"
					tabindex="0"
					on:click={toggleMusic}
					on:keydown={handleKeyDown}
					class="flex h-6 w-6 shrink-0 overflow-hidden"
				>
					<svg
						class="h-full w-full object-contain"
						xmlns="http://www.w3.org/2000/svg"
						width="32"
						height="32"
						fill="#000000"
						viewBox="0 0 256 256"
					>
						<path
							d="M240,128a15.74,15.74,0,0,1-7.6,13.51L88.32,229.65a16,16,0,0,1-16.2.3A15.86,15.86,0,0,1,64,216.13V39.87a15.86,15.86,0,0,1,8.12-13.82,16,16,0,0,1,16.2.3L232.4,114.49A15.74,15.74,0,0,1,240,128Z"
						></path>
					</svg>
				</div>
			{/if}
		</div>
	</div>
</div>

<style>
</style>
