<script lang="ts">
export let inputBoxes: string[];
export let onInput: (e: Event, idx: number) => void;
export let onKeydown: (e: KeyboardEvent, idx: number) => void;
export let disabled: boolean;

function setCursorToStart(e: Event) {
    const input = e.target as HTMLInputElement;
    // If input is empty, place cursor at start; if filled, place at end
    if (!input.value) {
        input.setSelectionRange(0, 0);
    } else {
        input.setSelectionRange(1, 1);
    }
}

function handleKeydown(e: KeyboardEvent, idx: number) {
    if (e.key === 'Backspace') {
        handleBackspace();
    } else if (e.key === 'ArrowLeft' && idx > 0) {
        // RTL: Move to previous (lower index, visually right)
        const prev = document.getElementById(`box-${idx - 1}`) as HTMLInputElement;
        prev?.focus();
        e.preventDefault();
    } else if (e.key === 'ArrowRight' && idx < inputBoxes.length - 1) {
        // RTL: Move to next (higher index, visually left)
        const next = document.getElementById(`box-${idx + 1}`) as HTMLInputElement;
        next?.focus();
        e.preventDefault();
    } else {
        onKeydown?.(e, idx);
    }
}

export function handleBackspace() {
    // Remove from leftmost (highest index) filled box and focus the previous box (or first if none)
    for (let i = 0; i < inputBoxes.length; i++) {
        if (inputBoxes[i]) {
            inputBoxes[i] = '';
            // Focus the previous box (i-1), or box-0 if i==0
            const focusIdx = i < inputBoxes.length - 1 ?  i + 1 : inputBoxes.length;
            setTimeout(() => {
                const input = document.getElementById(`box-${focusIdx}`) as HTMLInputElement;
                input?.focus();
                input?.setSelectionRange(1, 1);
            }, 0);
            break;
        }
    }
}
</script>

<div class="input-boxes">
    {#each inputBoxes as val, i (i)}
        <input
            id={`box-${i}`}
            class="guess-char-box"
            type="text"
            bind:value={inputBoxes[i]}
            maxlength="1"
            pattern="[א-ת]"
            inputmode="text"
            autocomplete="off"
            dir="rtl"
            required
            {disabled}
            on:input={(e) => onInput(e, i)}
            on:keydown={(e) => handleKeydown(e, i)}
            on:focus={(e) => setCursorToStart(e)}
        />
    {/each}
</div>

<style>
.input-boxes {
    display: flex;
    gap: 0.5rem;
    justify-content: center;
    margin-bottom: 1rem;
}
.guess-char-box {
    width: 2.2rem;
    height: 2.2rem;
    font-size: 1.5rem;
    text-align: center;
    border-radius: 0.4rem;
    border: 2px solid var(--input-border);
    background: var(--input-bg);
    direction: rtl;
    font-family: inherit;
    transition: border 0.2s;
    color: var(--foreground);
}
.guess-char-box:focus {
    border-color: var(--btn-bg);
    outline: none;
}
</style>
