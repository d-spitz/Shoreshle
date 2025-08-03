<script lang="ts">
	let {
		rows,
		letterStatuses = {},
		onKey,
		onBackspace,
		onEnter,
		disabled = false,
		currentInputLength = 0,
		requiredLength = 0
	}: {
		rows: string[][];
		letterStatuses?: Record<string, 'correct' | 'present' | 'absent' | undefined>;
		onKey: (letter: string) => void;
		onBackspace: () => void;
		onEnter: () => void;
		disabled?: boolean;
		currentInputLength?: number;
		requiredLength?: number;
	} = $props();
</script>

<div class="alephbet-keyboard">
	{#each rows as row, rowIdx}
		<div class="alephbet-row" dir="rtl">
			{#if rowIdx === 0}
				<button type="button" class="alephbet-key special-key" onclick={onBackspace} {disabled}
					>⌫</button
				>
			{/if}
			{#if rowIdx === 2}
				<button
					type="button"
					class="alephbet-key enter-key"
					onclick={onEnter}
					disabled={disabled || currentInputLength !== requiredLength}>⏎</button
				>
			{/if}
			{#each row as letter}
				<button
					type="button"
					class="alephbet-key {letterStatuses[letter]}"
					onclick={() => onKey(letter)}
					{disabled}>{letter}</button
				>
			{/each}
		</div>
	{/each}
</div>

<style>
	.alephbet-keyboard {
		user-select: none;
		max-width: 100vw;
		width: 100%;
		box-sizing: border-box;
		overflow-x: auto;
	}
	.alephbet-row {
		display: flex;
		flex-wrap: wrap;
		justify-content: center;
		gap: 1px;
		margin-bottom: 0.2rem;
		width: 100%;
		box-sizing: border-box;
	}
	.alephbet-key {
		width: 2.2rem;
		height: 2.2rem;
		font-size: 1.3rem;
		border-radius: 0.4rem;
		background: var(--box-bg);
		color: var(--foreground);
		border: 2px solid var(--box-border);
		text-align: center;
		font-weight: bold;
		transition:
			background 0.2s,
			border 0.2s;
		margin: 0 0.05rem;
		cursor: pointer;
		outline: none;
		padding: 0;
		display: inline-flex;
		align-items: center;
		justify-content: center;
	}
	.alephbet-key:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}
	.alephbet-key.correct {
		background: var(--box-correct);
		color: #fff;
		border-color: var(--box-correct-border);
	}
	.alephbet-key.present {
		background: var(--box-present);
		color: #222;
		border-color: var(--box-present-border);
	}
	.alephbet-key.absent {
		background: var(--box-absent);
		color: #aaa;
		border-color: var(--box-absent-border);
	}
	.alephbet-key.special-key {
		width: 2.2rem;
		font-size: 1.2rem;
		background: var(--btn-bg-disabled);
		color: var(--foreground);
	}
	.alephbet-key.enter-key {
		background: var(--btn-bg);
		color: #fff;
		border-color: var(--btn-bg);
		font-size: 1.2rem;
		width: calc(2.2rem * 1.2);
	}
	.alephbet-row.enter-row {
		justify-content: center;
		margin-top: 0.4rem;
		margin-bottom: 0;
		gap: 0;
	}

	@media (max-width: 600px) {
		.alephbet-keyboard {
			overflow: visible;
		}
		.alephbet-key {
			width: clamp(1.5rem, 8vw, 2rem);
			height: 2.5rem;
			font-size: 1.4rem;
		}
		.alephbet-row.enter-row {
			margin-top: 0.3rem;
		}
	}
</style>
