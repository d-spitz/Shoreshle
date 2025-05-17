<script lang="ts">
export let rows: string[][];
export let letterStatuses: Record<string, 'correct' | 'present' | 'absent' | undefined> = {};
export let onKey: (letter: string) => void;
export let onBackspace: () => void;
export let onEnter: () => void;
export let disabled: boolean = false;
export let currentInputLength: number = 0;
export let requiredLength: number = 0;
</script>

<div class="alephbet-keyboard">
    {#each rows as row, rowIdx}
        <div class="alephbet-row">
            {#if rowIdx === 2}
                <button type="button" class="alephbet-key special-key" on:click={onBackspace} disabled={disabled}>⌫</button>
            {/if}
            {#each row as letter}
                <button
                    type="button"
                    class="alephbet-key {letterStatuses[letter]}"
                    on:click={() => onKey(letter)}
                    disabled={disabled}
                >{letter}</button>
            {/each}
            {#if rowIdx === 2}
                <button
                    type="button"
                    class="alephbet-key enter-key"
                    style="width:calc(2.2rem * 1.5);height:2.2rem;"
                    on:click={onEnter}
                    disabled={disabled || currentInputLength !== requiredLength}
                >⏎</button>
            {/if}
        </div>
    {/each}
</div>

<style>
.alephbet-keyboard {
    margin: 1.2rem 0 0.5rem 0;
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
    gap: 0.2rem;
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
    transition: background 0.2s, border 0.2s;
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
    width: calc(2.2rem * 1.5);
}

@media (max-width: 600px) {
    .alephbet-keyboard {
        padding: 0 0.1rem;
    }
    .alephbet-row {
        gap: 0.05rem;
    }
    .alephbet-key {
        width: 1.5rem;
        height: 1.5rem;
        font-size: 0.9rem;
        margin: 0 0.01rem;
    }
    .alephbet-key.enter-key {
        width: calc(1.5rem * 1.5);
        font-size: 0.85rem;
    }
    .alephbet-key.special-key {
        width: 1.5rem;
        font-size: 0.85rem;
    }
}
</style>
