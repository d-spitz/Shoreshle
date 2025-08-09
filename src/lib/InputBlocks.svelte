<script lang="ts">
	let {
		guess = '',
		rootLength = 3,
		filled = false, // for current guess, highlight filled letters
		disabled = false, // for past guesses, show disabled look
		statuses = undefined
	}: {
		guess?: string;
		rootLength?: number;
		filled?: boolean;
		disabled?: boolean;
		statuses?: Array<'correct' | 'present' | 'absent'> | undefined;
	} = $props();
</script>

{#each Array(rootLength) as _, i}
	{#if statuses}
		<span class="letter {statuses[i]}">
			{guess[i] || ''}
		</span>
	{:else}
		<span class="letter {disabled ? 'disabled-row' : ''} {filled && guess[i] ? 'filled' : ''}">
			{guess[i] || ''}
		</span>
	{/if}
{/each}

<style>
	.letter {
		block-size: 4rem;
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
