<script lang="ts">
	export let guess: string = '';
	export let rootLength: number = 3;
	export let filled: boolean = false; // for current guess, highlight filled letters
	export let disabled: boolean = false; // for past guesses, show disabled look
	export let statuses: Array<'correct' | 'present' | 'absent'> | undefined = undefined;
</script>

<div class="input-blocks">
	{#each Array(rootLength) as _, i}
		{#if statuses}
			<span class="letter {statuses[rootLength - 1 - i]}">
				{guess[rootLength - 1 - i] || ''}
			</span>
		{:else}
			<span
				class="letter {disabled ? 'disabled-row' : ''} {filled && guess[rootLength - 1 - i]
					? 'filled'
					: ''}"
			>
				{guess[rootLength - 1 - i] || ''}
			</span>
		{/if}
	{/each}
</div>

<style>
	:root {
		--disabled-bg: #e9e9e9;
		--disabled-color: #b0b0b0;
		--disabled-border: #d0d0d0;
	}
	@media (prefers-color-scheme: dark) {
		:root {
			--disabled-bg: #23272a;
			--disabled-color: #888;
			--disabled-border: #444;
		}
	}
	.input-blocks {
		display: flex;
		gap: 0.25rem;
		justify-content: center;
	}
	.letter {
		display: inline-block;
		width: 4rem;
		height: 4rem;
		line-height: 4rem;
		font-size: 2.6rem;
		border-radius: 0.4rem;
		background: var(--box-bg);
		color: var(--foreground);
		border: 2px solid var(--box-border);
		font-weight: bold;
		text-align: center;
		transition:
			background 0.2s,
			border 0.2s,
			color 0.2s;
	}
	.letter.filled {
		border-color: var(--btn-bg);
	}
	.letter.disabled-row {
		background: var(--disabled-bg);
		color: var(--disabled-color);
		border-color: var(--disabled-border);
	}
	.letter.correct {
		background: var(--box-correct);
		color: #fff;
		border-color: var(--box-correct-border);
	}
	.letter.present {
		background: var(--box-present);
		color: #222;
		border-color: var(--box-present-border);
	}
	.letter.absent {
		background: var(--box-absent);
		color: #aaa;
		border-color: var(--box-absent-border);
	}
</style>
