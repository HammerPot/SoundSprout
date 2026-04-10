<script lang="ts">
	import { onDestroy, onMount } from 'svelte';
	import * as Tone from 'tone';
	import type { RecursivePartial } from 'tone/build/esm/core/util/Interface.js';

	type OscShape = 'sine' | 'triangle' | 'square' | 'sawtooth';

	let octave = $state(3);
	const keys = $derived([
		{ note: `C${octave}`, key: 'a' },
		{ note: `D${octave}`, key: 's' },
		{ note: `E${octave}`, key: 'd' },
		{ note: `F${octave}`, key: 'f' },
		{ note: `G${octave}`, key: 'g' },
		{ note: `A${octave}`, key: 'h' },
		{ note: `B${octave}`, key: 'j' },
		{ note: `C${octave + 1}`, key: 'k' },
		{ note: `D${octave + 1}`, key: 'l' },

	]);

	const envelope = { attack: 0.02, decay: 0.1, sustain: 0.35, release: 0.75 };

	let waveform = $state<OscShape>('sawtooth');
	let volumeDb = $state(-6);

	let poly = $state<Tone.PolySynth<Tone.Synth> | null>(null);

	function voiceOpts(): RecursivePartial<Tone.SynthOptions> {
		return { oscillator: { type: waveform }, envelope };
	}

	function getPoly(): Tone.PolySynth<Tone.Synth> {
		if (!poly) {
			poly = new Tone.PolySynth(Tone.Synth, {
				...voiceOpts(),
				volume: volumeDb
			}).toDestination();
		}
		return poly;
	}

	$effect(() => {
		const p = poly;
		if (!p) return;
		p.set(voiceOpts());
		p.volume.value = volumeDb;
	});

	async function startNote(note: string) {
		await Tone.start();
		getPoly().triggerAttack(note);
	}

	function stopNote(note: string) {
		poly?.triggerRelease(note);
	}

	onDestroy(() => {
		poly?.dispose();
		poly = null;
	});

	/** Physical key → pitch that was actually triggered (so octave changes don’t break keyup). */
	const computerHeld: Record<string, string> = {};

	onMount(() => {
		const down = (e: KeyboardEvent) => {
			if (e.repeat) return;
			const k = e.key.toLowerCase();
			const row = keys.find((r) => r.key === k);
			if (!row) return;
			e.preventDefault();
			if (computerHeld[k]) return;
			computerHeld[k] = row.note;
			void startNote(row.note);
		};
		const up = (e: KeyboardEvent) => {
			const k = e.key.toLowerCase();
			const attacked = computerHeld[k];
			if (!attacked) return;
			delete computerHeld[k];
			stopNote(attacked);
		};
		const octaveUp = (e: KeyboardEvent) => {
			if (e.key !== 'x') return;
			e.preventDefault();
			octave = Math.min(6, octave + 1);
		};
		const octaveDown = (e: KeyboardEvent) => {
			if (e.key !== 'z') return;
			e.preventDefault();
			octave = Math.max(1, octave - 1);
		};

		window.addEventListener('keydown', down);
		window.addEventListener('keyup', up);
		window.addEventListener('keydown', octaveUp);
		window.addEventListener('keydown', octaveDown);
		return () => {
			window.removeEventListener('keydown', down);
			window.removeEventListener('keyup', up);
			window.removeEventListener('keydown', octaveUp);
			window.removeEventListener('keydown', octaveDown);
		};
	});

	function pointerDown(note: string, el: HTMLElement, e: PointerEvent) {
		el.setPointerCapture(e.pointerId);
		void startNote(note);
	}

	function pointerUp(note: string) {
		stopNote(note);
	}
</script>

<div class="container">
	<div class="card" style="background-color: var(--color-background-3);">
		<h2 class="text-center eyebrow">Try a <a href="https://en.wikipedia.org/wiki/Polyphony">polyphonic</a> synth</h2>

		<div class="poly-controls">
			<label class="caption poly-field-hint poly-wf-label" for="poly-synth-waveform">Waveform</label>
			<label class="caption poly-field-hint poly-vol-label" for="poly-synth-volume"
				>Volume {volumeDb.toFixed(0)} dB</label>
			<div class="poly-control-slot poly-wf-ctl">
				<select id="poly-synth-waveform" class="poly-select" bind:value={waveform}>
					<option value="sine">Sine</option>
					<option value="triangle">Triangle</option>
					<option value="square">Square</option>
					<option value="sawtooth">Sawtooth</option>
				</select>
			</div>
			<div class="poly-control-slot poly-vol-ctl">
				<input
					id="poly-synth-volume"
					type="range"
					min="-36"
					max="6"
					step="1"
					bind:value={volumeDb}
					class="poly-volume-range"
				/>
			</div>
		</div>

		<div class="poly-keys" role="group">
			{#each keys as row (row.note)}
				<div
					class="card interactive poly-key text-center justify-center"
					style="cursor: pointer; background-color: var(--color-background-3); border: 1px solid var(--color-neutral); color: var(--color-text); display: block;"
					role="button"
					tabindex="-1"
					onpointerdown={(e) => pointerDown(row.note, e.currentTarget, e)}
					onpointerup={() => pointerUp(row.note)}
					onpointercancel={() => pointerUp(row.note)}
					onlostpointercapture={() => pointerUp(row.note)}
					onkeydown={(e) => {
						if (e.key !== 'Enter' && e.key !== ' ') return;
						e.preventDefault();
						if (e.repeat) return;
						void startNote(row.note);
					}}
					onkeyup={(e) => {
						if (e.key !== 'Enter' && e.key !== ' ') return;
						pointerUp(row.note);
					}}
				>
					<span class="heading text-center">{row.note}</span>
					<br>
					<span class="caption" style="color: var(--color-neutral-text);">{row.key}</span>
				</div>
			{/each}
		</div>
	</div>
</div>

<style>
	.poly-controls {
		display: grid;
		gap: var(--spacing-1);
		margin-top: var(--spacing-3);
		margin-bottom: var(--spacing-3);
		color: var(--color-text);
		text-align: center;
		grid-template-columns: 1fr;
		grid-template-areas:
			'wf-label'
			'wf-ctl'
			'vol-label'
			'vol-ctl';
	}

	.poly-wf-label {
		grid-area: wf-label;
	}
	.poly-vol-label {
		grid-area: vol-label;
	}
	.poly-wf-ctl {
		grid-area: wf-ctl;
	}
	.poly-vol-ctl {
		grid-area: vol-ctl;
	}

	@media screen and (min-width: 48em) {
		.poly-controls {
			grid-template-columns: 1fr 2fr;
			grid-template-rows: auto auto;
			column-gap: var(--spacing-3);
			row-gap: var(--spacing-1);
			align-items: start;
			grid-template-areas:
				'wf-label vol-label'
				'wf-ctl vol-ctl';
		}
	}

	.poly-field-hint {
		margin: 0;
		color: var(--color-neutral-text);
		line-height: var(--line-height-caption, 1.375);
		text-align: center;
	}

	/* Shared row height on desktop: center short range input with taller select. */
	.poly-control-slot {
		display: flex;
		align-items: center;
		min-width: 0;
		min-height: 100%;
	}

	@media screen and (min-width: 48em) {
		.poly-control-slot {
			align-self: stretch;
		}
	}

	.poly-select {
		width: 100%;
		padding: var(--spacing-2);
		border-radius: var(--radii-small);
		background-color: var(--color-background-3);
		border: 1px solid var(--color-neutral);
		color: var(--color-text);
		text-align: center;
		text-align-last: center;
	}

	.poly-keys {
		display: flex;
		flex-wrap: wrap;
		justify-content: center;
		gap: var(--spacing-2);
	}

	/* Extra behavior only: theme `.card` supplies look. */
	.poly-key {
		display: flex;
		flex-direction: column;
		align-items: center;
		min-width: 2.75rem;
		touch-action: none;
		cursor: pointer;
		color: var(--color-text);
	}

	.poly-volume-range {
		width: 100%;
		display: block;
		height: 1.5rem;
		-webkit-appearance: none;
		appearance: none;
		background: transparent;
	}

	.poly-volume-range:focus-visible {
		outline: 2px solid var(--color-primary);
		outline-offset: 2px;
	}

	.poly-volume-range::-webkit-slider-runnable-track {
		height: 6px;
		border-radius: var(--radii-circle);
		background: var(--color-background-3);
		border: 1px solid var(--color-neutral);
	}

	.poly-volume-range::-webkit-slider-thumb {
		-webkit-appearance: none;
		appearance: none;
		width: 14px;
		height: 14px;
		margin-top: -5px;
		border-radius: 50%;
		background: var(--color-primary);
		border: none;
		cursor: pointer;
	}

	.poly-volume-range::-moz-range-track {
		height: 6px;
		border-radius: var(--radii-circle);
		background: var(--color-background-3);
		border: 1px solid var(--color-neutral);
	}

	.poly-volume-range::-moz-range-thumb {
		width: 14px;
		height: 14px;
		border-radius: 50%;
		background: var(--color-primary);
		border: none;
		cursor: pointer;
	}
</style>
