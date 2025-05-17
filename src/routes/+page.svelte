<script lang="ts">
import { roots } from '$lib/roots';
import type { Root } from '$lib/models/root';
import { onMount } from 'svelte';
import InputBlocks from '$lib/InputBlocks.svelte';
import Keyboard from '$lib/Keyboard.svelte';

function getDailyRoot(): Root {
    // Use today's date in ISO format as a seed
    const today = new Date().toISOString().slice(0, 10);
    // Simple hash function for string to int
    let hash = 0;
    for (let i = 0; i < today.length; i++) {
        hash = today.charCodeAt(i) + ((hash << 5) - hash);
    }
    const idx = Math.abs(hash) % roots.length;
    return roots[idx];
}

const maxGuesses = 6;
let currentRoot: Root = getDailyRoot();
let guesses: string[] = [];
let rootLength = currentRoot.root.length;
let inputBoxes: string[] = Array(rootLength).fill('');
let gameState: 'playing' | 'won' | 'lost' = 'playing';
let message = '';

let letterStatuses: Record<string, 'correct' | 'present' | 'absent' | undefined> = {};

// Hebrew keyboard rows for on-screen keyboard
const hebrewKeyboardRows: string[][] = [
    Array.from('קראטוןםפ'),
    Array.from('שדגכעיחלךף'),
    Array.from('זסבהנמצתץ')
];

// Map final forms to regular forms
const finalToRegular: Record<string, string> = {
    ך: 'כ', ם: 'מ', ן: 'נ', ף: 'פ', ץ: 'צ'
};
function normalizeLetter(l: string) {
    return finalToRegular[l] || l;
}
function normalizeWord(w: string) {
    return w.split('').map(normalizeLetter).join('');
}

function updateLetterStatuses(guess: string) {
    const normGuess = normalizeWord(guess);
    const normAnswer = normalizeWord(currentRoot.root);
    for (let i = 0; i < normGuess.length; i++) {
        const letter = normGuess[i];
        const status = getLetterStatus(normGuess, i, normAnswer);
        // Only upgrade status (absent < present < correct)
        if (status === 'correct' ||
            (status === 'present' && letterStatuses[letter] !== 'correct') ||
            (status === 'absent' && !letterStatuses[letter])) {
            letterStatuses[letter] = status;
        }
    }
}

function checkGuess() {
    // Join inputBoxes right-to-left for Hebrew (rightmost is first letter)
    const input = inputBoxes.slice().reverse().join('');
    const normInput = normalizeWord(input);
    const normAnswer = normalizeWord(currentRoot.root);
    if (normInput.length < 3 || normInput.length > 4) {
        message = 'שורש חייב להיות 3 או 4 אותיות';
        return;
    }
    if (guesses.includes(input)) {
        message = 'כבר ניסית את השורש הזה';
        return;
    }
    guesses = [...guesses, input];
    updateLetterStatuses(input);
    if (normInput === normAnswer) {
        gameState = 'won';
        message = 'ניחשת נכון!';
    } else if (guesses.length >= maxGuesses) {
        gameState = 'lost';
        message = `הפסדת! השורש היה: ${currentRoot.root}`;
    } else {
        message = '';
    }
    inputBoxes = Array(rootLength).fill('');
    setTimeout(() => {
        const first = document.getElementById(`box-${rootLength - 1}`) as HTMLInputElement;
        first?.focus();
    }, 0);
}

function getLetterStatus(guess: string, index: number, answerOverride?: string) {
    // guess and answer are both normalized and left-to-right
    const answer = answerOverride || normalizeWord(currentRoot.root);
    // For RTL, guess is reversed, so index 0 is rightmost
    if (guess[index] === answer[index]) return 'correct';
    if (answer.includes(guess[index])) return 'present';
    return 'absent';
}

function handleBoxInput(e: Event, idx: number) {
    const target = e.target as HTMLInputElement;
    let val = target.value.replace(/[^א-ת]/g, '');
    if (val.length > 1) val = val.slice(-1); // Only last char
    inputBoxes[idx] = val;
    // Move to next box (lower index, to the left visually) if filled, since Hebrew is RTL
    if (val && idx > 0) {
        const next = document.getElementById(`box-${idx - 1}`) as HTMLInputElement;
        next?.focus();
    }
}

function handleBoxKeydown(e: KeyboardEvent, idx: number) {
    // For RTL: backspace moves right (higher index)
    if (e.key === 'Backspace' && !inputBoxes[idx] && idx < inputBoxes.length - 1) {
        const prev = document.getElementById(`box-${idx + 1}`) as HTMLInputElement;
        prev?.focus();
    }
}

function handleKeyboardClick(letter: string) {
    // Fill from right to left (highest empty index)
    const idx = inputBoxes.lastIndexOf('');
    if (idx !== -1) {
        inputBoxes[idx] = letter;
        // Focus next box (lower index, to the left visually)
        setTimeout(() => {
            if (idx > 0) {
                const next = document.getElementById(`box-${idx - 1}`) as HTMLInputElement;
                next?.focus();
            }
        }, 0);
    }
}

function handleKeyboardBackspace() {
    // Simulate a Backspace keydown on the currently focused input
    const active = document.activeElement as HTMLInputElement;
    if (active && active.classList.contains('guess-char-box')) {
        const event = new KeyboardEvent('keydown', { key: 'Backspace', bubbles: true });
        active.dispatchEvent(event);
    }
}

function restart() {
    currentRoot = getDailyRoot();
    guesses = [];
    inputBoxes = Array(rootLength).fill('');
    letterStatuses = {};
    gameState = 'playing';
    message = '';
    setTimeout(() => {
        const first = document.getElementById(`box-${rootLength - 1}`) as HTMLInputElement;
        first?.focus();
    }, 0);
}

let inputBlocksRef: any;

onMount(() => {
    setTimeout(() => {
        const first = document.getElementById(`box-${rootLength - 1}`) as HTMLInputElement;
        first?.focus();
    }, 0);
});
</script>

<main class="container">
    <h1>שורשל</h1>
    <div class="hint">רמז: {currentRoot.meaning}</div>
    <div class="guesses">
        {#each guesses as guess}
            <div class="guess-row">
                {#each Array(guess.length) as _, i}
                    <span class="letter {getLetterStatus(normalizeWord(guess), guess.length - 1 - i)}">{guess[guess.length - 1 - i]}</span>
                {/each}
            </div>
        {/each}
        {#if gameState === 'playing' && guesses.length < maxGuesses}
            <form on:submit|preventDefault={checkGuess} class="guess-form">
                <InputBlocks
                    bind:this={inputBlocksRef}
                    {inputBoxes}
                    onInput={handleBoxInput}
                    onKeydown={handleBoxKeydown}
                    disabled={gameState !== 'playing'}
                />
            </form>
        {/if}
    </div>
    <Keyboard
        rows={hebrewKeyboardRows}
        {letterStatuses}
        onKey={handleKeyboardClick}
        onBackspace={() => inputBlocksRef?.handleBackspace()}
        onEnter={checkGuess}
        disabled={gameState !== 'playing'}
        currentInputLength={inputBoxes.join('').length}
        requiredLength={rootLength}
    />
    <div class="message">{message}</div>
    {#if gameState !== 'playing'}
        <button class="restart-btn" on:click={restart}>שחק שוב</button>
    {/if}
</main>

<style>
:root {
    --background: #f5f6fa;
    --container-bg: #f0f1f6;
    --foreground: #2d2d2d;
    --hint: #666;
    --box-bg: #f0f0f0;
    --box-border: #e0e0e0;
    --box-correct: #4caf50;
    --box-correct-border: #388e3c;
    --box-present: #ffeb3b;
    --box-present-border: #fbc02d;
    --box-absent: #e0e0e0;
    --box-absent-border: #bdbdbd;
    --input-bg: #f9f9f9;
    --input-border: #bbb;
    --btn-bg: #1976d2;
    --btn-bg-disabled: #90caf9;
    --btn-bg-restart: #43a047;
    --message: #d32f2f;
}
@media (prefers-color-scheme: dark) {
    :root {
        --background: #181a1b;
        --container-bg: #23242a;
        --foreground: #f5f5f5;
        --hint: #bdbdbd;
        --box-bg: #23272a;
        --box-border: #444;
        --box-correct: #388e3c;
        --box-correct-border: #4caf50;
        --box-present: #bfae2c;
        --box-present-border: #ffeb3b;
        --box-absent: #333;
        --box-absent-border: #666;
        --input-bg: #23272a;
        --input-border: #666;
        --btn-bg: #1565c0;
        --btn-bg-disabled: #37474f;
        --btn-bg-restart: #388e3c;
        --message: #ef5350;
    }
}
.container {
    max-width: 400px;
    margin: 2rem auto;
    padding: 1rem;
    background: var(--container-bg);
    border-radius: 1rem;
    box-shadow: 0 2px 16px rgba(0,0,0,0.08);
    text-align: center;
    font-family: 'Assistant', Arial, sans-serif;
    color: var(--foreground);
}
h1 {
    font-size: 2.2rem;
    margin-bottom: 0.5rem;
    color: var(--foreground);
    letter-spacing: 0.1em;
}
.hint {
    font-size: 1.1rem;
    color: var(--hint);
    margin-bottom: 1.5rem;
}
.guesses {
    margin-bottom: 1.5rem;
}
.guess-row {
    display: flex;
    justify-content: center;
    margin-bottom: 0.5rem;
}
.letter {
    display: inline-block;
    width: 2.2rem;
    height: 2.2rem;
    line-height: 2.2rem;
    margin: 0 0.15rem;
    font-size: 1.5rem;
    border-radius: 0.4rem;
    background: var(--box-bg);
    color: var(--foreground);
    border: 2px solid var(--box-border);
    font-weight: bold;
    text-align: center;
    transition: background 0.2s, border 0.2s;
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
.guess-form {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.5rem;
}
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
.submit-btn, .restart-btn {
    font-size: 1.1rem;
    border-radius: 0.4rem;
    border: none;
    background: var(--btn-bg);
    color: #fff;
    cursor: pointer;
    transition: background 0.2s;
    display: inline-block;
}
.submit-btn {
    padding: 0;
    width: auto;
    min-width: 0;
    height: 2.2rem;
    font-size: 1.2rem;
}
.submit-btn:disabled {
    background: var(--btn-bg-disabled);
    cursor: not-allowed;
}
.restart-btn {
    background: var(--btn-bg-restart);
    margin-top: 1rem;
    padding: 0.5rem 1.2rem;
}
.alephbet-keyboard {
    margin: 1.2rem 0 0.5rem 0;
    user-select: none;
}
.alephbet-row {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 0.2rem;
    margin-bottom: 0.2rem;
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
</style>
