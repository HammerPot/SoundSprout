<script lang="ts">
	import * as Tone from 'tone';
	import { onDestroy, tick } from 'svelte';
	import PolySynthKeys from '$lib/PolySynthKeys.svelte';

	const titleText = 'Sound Sprout';
	const notes = ['C4', 'D4', 'E4', 'F4', 'G4', '', 'A4', 'B4', 'C5', 'D5', 'E5', 'F5'];

	const ABOUT_PHOTOS = [
		{ src: '/imgs/groundcarrots.jpg', alt: 'Carrots growing in a raised bed' },
		{ src: '/imgs/tablecarrots.jpg', alt: 'Table with three carrots' }
	] as const;

	const GARDENING_KIT_PHOTO = {
		src: '/imgs/gardeningkit.png',
		alt: 'Example gardening kit'
	} as const;

	let lightbox = $state<{ src: string; alt: string } | null>(null);
	let lightboxDialog = $state<HTMLDialogElement | null>(null);

	async function openLightbox(entry: { src: string; alt: string }) {
		lightbox = entry;
		await tick();
		lightboxDialog?.showModal();
	}

	function onLightboxClose() {
		lightbox = null;
	}

	let synth: Tone.Synth | null = null;


	function getSynth() {
		if (!synth) {
			synth = new Tone.Synth().toDestination();
		}
		return synth;
	}

	async function playNote(note: string) {
		await Tone.start();
		getSynth().triggerAttackRelease(note, '8n');
	}

	onDestroy(() => {
		synth?.dispose();
		synth = null;
	});
</script>

<div class=" w-full max-w-32 border-0">
	<a href="https://hackclub.com/">
		<img  src="https://assets.hackclub.com/flag-orpheus-left.svg" alt="Hack Club" style=" padding-top: var(--spacing-2);"/>
	</a>
</div>

<div class="flex flex-col items-center " style="padding-top: var(--spacing-2); padding-bottom: var(--spacing-4);">
	<!-- <h1 class="text-center ultratitle" style="color: var(--color-primary);"> Sound Sprout </h1> -->
	<h1 class="text-center ultratitle" style="color: var(--color-primary); font-family: 'HojasDePlata'; font-weight: 400;">
		{#each titleText as char, i}
			{#if char === ' '}
				<span class="">{char}</span>
			{:else}
				<span class="hover:underline" role="button" tabindex="-1" onclick={() => playNote(notes[i])} style="cursor: pointer;">{char}</span>
			{/if}
		{/each}
	</h1>
	<p class="text-center headline" style="color: var(--color-text);"> Make a Website that Generates Music, Get Seeds to Grow a Garden! </p>
</div>

<div class="info-card-grid grid grid-cols-1 gap-4 md:grid-cols-2">
	<div class="info-card-grid__cell" style="padding-top: var(--spacing-2); padding-bottom: var(--spacing-2);">
		<div class="container">
			<div class="card" style="background-color: var(--color-background-3);">
				<h2 class="eyebrow"> What is Sound Sprout </h2>
				<p style="color: var(--color-text);"> This YSWS requires you to create a website that generates and plays music in some way. </p>
			</div>
		</div>
	</div>

	<div class="info-card-grid__cell" style="padding-top: var(--spacing-2); padding-bottom: var(--spacing-2);">
		<div class="container">
			<div class="card" style="background-color: var(--color-background-3);">
				<h2 class="eyebrow"> What do I get for submitting </h2>
				<p style="color: var(--color-text);">Ship a project with at least 6 hours of work, and you will receive a gardening kit to start growing your own garden!</p>
			</div>
		</div>
	</div>

	<div class="info-card-grid__cell" style="padding-top: var(--spacing-2); padding-bottom: var(--spacing-2);">
		<div class="container">
			<div class="card" style="background-color: var(--color-background-3);">
				<h2 class="eyebrow"> Wait, how </h2>
				<p style="color: var(--color-text);"> The web browser is a powerful tool, and there are many resources available for beginners to learn how to start creating!</p>
				<p style="color: var(--color-text);">A great resource you may find useful are the <a href="https://developer.mozilla.org/en-US/docs/Web/HTML" target="_blank">MDN Web Docs</a>, where you can find in-depth explanations of the APIs and useful libraries that can be used to achieve your goal.</p>
			</div>
		</div>
	</div>

	<div class="info-card-grid__cell" style="padding-top: var(--spacing-2); padding-bottom: var(--spacing-2);">
		<div class="container">
			<div class="card" style="background-color: var(--color-background-3);">
				<h2 class="eyebrow"> Did you say gardening kit </h2>
				<p style="color: var(--color-text);">That's right! Complete this YSWS and you will receive one gardening kit in the mail to grow into your very own garden!</p>
			</div>
		</div>
	</div>
</div>

<div style="padding-top: var(--spacing-2); padding-bottom: var(--spacing-2);">
	<div class="container ">
		<div class="card" style="background-color: var(--color-background-3);">
			<h2 class="eyebrow text-center"> Why are you running this </h2>
			<p style="color: var(--color-text); text-align: center;"> I have a huge passion for both music and technology. I also love gardening and think it's a great way to connect with nature and appreciate the world around us.</p>
			<p style="color: var(--color-text); text-align: center;"> Below are some photos of some carrots I have grown in the past! </p>
			<div class="about-photo-grid">
				{#each ABOUT_PHOTOS as photo}
					<button
						type="button"
						class="lightbox-thumb about-photo-grid__thumb"
						aria-label={`View larger: ${photo.alt}`}
						onclick={() => openLightbox(photo)}
					>
						<img src={photo.src} alt="" loading="lazy" decoding="async" />
					</button>
				{/each}
			</div>
		</div>
	</div>
</div>



<div style="padding-top: var(--spacing-2); padding-bottom: var(--spacing-2);">
	<PolySynthKeys />
</div>

<!-- <div style="padding-top: var(--spacing-2); padding-bottom: var(--spacing-2);">
	<div class="container">
		<div class="card" style="background-color: var(--color-background-3);">
			<h2 class="text-center eyebrow"> What resources? </h2>
			<p class="text-center" style="color: var(--color-text);"> Whether you are a beginner or an expert, the <a href="https://developer.mozilla.org/en-US/docs/Web/HTML" target="_blank">MDN Web Docs</a> may provide a good launchpad to learn about relevant APIs. </p>
			</div>
			</div>
			</div> -->
			
<div style="padding-top: var(--spacing-4); padding-bottom: var(--spacing-4);">
	<div class="container">
		<div class="card" style="background-color: var(--color-background-3);">
			<h2 class="text-center eyebrow"> Get Started </h2>
			<div class="grid min-w-0 grid-cols-1 gap-4 md:grid-cols-3" style="padding-bottom: var(--spacing-1);">
				<div class="card interactive" style="" >
					<h2 class="text-center heading" style="color: var(--color-accent-text)"> Step 1: </h2>
					<p class="text-center" style="color: var(--color-accent-text)"> Create a new website that generates music! </p>
				</div>
				<div class="card interactive" style="">
					<h2 class="text-center heading" style="color: var(--color-accent-text)"> Step 2: </h2>
					<p class="text-center" style="color: var(--color-accent-text)"> Submit your website to the Sound Sprout YSWS! </p>
				</div>
				<div class="card interactive" style="">
					<h2 class="text-center heading" style="color: var(--color-accent-text)"> Step 3: </h2>
					<p class="text-center" style="color: var(--color-accent-text)"> Get mailed seeds to grow your own muse! </p>
				</div>
			</div>
			<div class="flex justify-center">
				<button class="btn lg" style=" width: 95%; margin-top: var(--spacing-3);" onclick={() => window.location.href = 'https://slack.hackclub.com'}> RSVP! </button>
			</div>
		</div>
	</div>
</div>

<div style="padding-top: var(--spacing-4); padding-bottom: var(--spacing-4);">
	<div class="container">
		<div class="card" style="background-color: var(--color-background-3);">
			<h2 class="text-center eyebrow" style="">FAQ</h2>
			<!-- <div style="padding-top: var(--spacing-1); padding-bottom: var(--spacing-2);">
				<details class="card" style="background-color: var(--color-accent);">
					<summary class="heading" style="color: var(--color-accent-text)"> What is Hack Club? </summary>
					<p class="" style="color: var(--color-accent-text)"> Hack Club is a 501(c)(3) nonprofit network of high school hackers. </p>
					</details>
					</div> -->
				<div style="padding-top: var(--spacing-1); padding-bottom: var(--spacing-2);">
					<details class="card" style="background-color: var();">
						<summary class="heading" style="color: var(); cursor: pointer;"> What is a YSWS? </summary>
						<p class="" style="color: var()"> YSWS (You Ship We Ship) are programs where teens can build cool projects and get cool things in return. </p>
					</details>
				</div>
				<div style="padding-top: var(--spacing-1); padding-bottom: var(--spacing-2);">
					<details class="card" style="background-color: var();">
						<summary class="heading" style="color: var(); cursor: pointer;"> Can I participate? </summary>
						<p class="" style="color: var()"> As long as you are 18 years old or younger, you can participate! </p>
					</details>
				</div>
				<div style="padding-top: var(--spacing-1); padding-bottom: var(--spacing-2);">
					<details class="card" style="">
						<summary class="heading" style="color: var(); cursor: pointer;"> What is required for this YSWS? </summary>
						<p class="" style="color: var()"> You need to spend at least 6 hours creating a website that can either play back audio generated by the user on your website, or play back a sequence in real time that you prepared. The goal is to create something that is fun and engaging while learning something new! </p>
						<p class="" style="color: var()"> To prove you have spent at least 6 hours working on your website, you must use <a href="https://hackatime.hackclub.com/" style="color: var(--color-neutral)">Hackatime</a> while working on your website. </p>
					</details>
				</div>
				<div style="padding-top: var(--spacing-1); padding-bottom: var(--spacing-2);">
					<details class="card" style="background-color: var();">
						<summary class="heading" style="color: var(); cursor: pointer;"> What is a Gardening Kit? </summary>
						<div class="gardening-kit-faq-row">
							<p class="" style="color: var()">
								A gardening kit is a kit that contains everything you need to start growing your own garden in your very own home! This includes trays, soil pucks, and of course seeds. The photo to the right <a type="button" class="" aria-label={`View larger: ${GARDENING_KIT_PHOTO.alt}`} onclick={() => openLightbox(GARDENING_KIT_PHOTO)} style="cursor: pointer; color: var(--color-neutral)">(click to enlarge)</a> shows one of the gardening kits that you could receive.</p>
							<button
								type="button"
								class="lightbox-thumb content-thumb-wrap gardening-kit-faq-row__thumb"
								aria-label={`View larger: ${GARDENING_KIT_PHOTO.alt}`}
								onclick={() => openLightbox(GARDENING_KIT_PHOTO)}
							>
								<img
									class="content-thumb"
									src={GARDENING_KIT_PHOTO.src}
									alt=""
									loading="lazy"
									decoding="async"
								/>
							</button>
						</div>
					</details>
				</div>
			</div>
			
		</div>
	</div>

<dialog
	bind:this={lightboxDialog}
	class="image-lightbox"
	aria-label="Enlarged photo"
	onclose={onLightboxClose}
>
	{#if lightbox}
		<p class="sr-only">Enlarged image: {lightbox.alt}</p>
		<img src={lightbox.src} alt={lightbox.alt} class="image-lightbox__img" />
		<form method="dialog" class="image-lightbox__actions">
			<button type="submit" class="btn">Close</button>
		</form>
	{/if}
</dialog>
