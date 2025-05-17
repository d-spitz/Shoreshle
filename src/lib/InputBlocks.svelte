<script lang="ts">
export let guess: string = '';
export let rootLength: number = 3;
</script>

<div class="input-blocks">
    {#each Array(rootLength) as _, i}
        <span class="letter {guess.length > rootLength - 1 - i ? 'filled' : ''}">
            {guess[rootLength - 1 - i] || ''}
        </span>
    {/each}
</div>

<style>
.input-blocks {
    display: flex;
    gap: 2px;
    justify-content: center;
}
.letter {
    display: inline-block;
    width: 4rem;
    height: 4rem;
    line-height: 4rem;
    font-size: 2.5rem;
    border-radius: 0.4rem;
    background: var(--box-bg);
    color: var(--foreground);
    border: 2px solid var(--box-border);
    font-weight: bold;
    text-align: center;
    transition: background 0.2s, border 0.2s;
}
.letter.filled {
    border-color: var(--btn-bg);
}
</style>
