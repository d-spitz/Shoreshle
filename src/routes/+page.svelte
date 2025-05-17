<script lang="ts">
import { roots } from '$lib/roots';
import type { Root } from '$lib/models/root';
import Keyboard from '$lib/Keyboard.svelte';
import InputBlocks from '$lib/InputBlocks.svelte';

function getDailyRoot(): Root {
    const today = new Date().toISOString().slice(0, 10);
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
let currentGuess = '';
let gameState: 'playing' | 'won' | 'lost' = 'playing';
let message = '';

let letterStatuses: Record<string, 'correct' | 'present' | 'absent' | undefined> = {};

const hebrewKeyboardRows: string[][] = [
    Array.from('קראטוןםפ'),
    Array.from('שדגכעיחלךף'),
    Array.from('זסבהנמצתץ')
];

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
        if (status === 'correct' ||
            (status === 'present' && letterStatuses[letter] !== 'correct') ||
            (status === 'absent' && !letterStatuses[letter])) {
            letterStatuses[letter] = status;
        }
    }
}

function checkGuess() {
    const input = currentGuess;
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
    currentGuess = '';
}

function getLetterStatus(guess: string, index: number, answerOverride?: string) {
    const answer = answerOverride || normalizeWord(currentRoot.root);
    if (guess[index] === answer[index]) return 'correct';
    if (answer.includes(guess[index])) return 'present';
    return 'absent';
}

function handleKeyboardClick(letter: string) {
    if (gameState !== 'playing') return;
    if (currentGuess.length < rootLength) {
        currentGuess += letter;
    }
}

function handleKeyboardBackspace() {
    if (gameState !== 'playing') return;
    if (currentGuess.length > 0) {
        currentGuess = currentGuess.slice(0, -1);
    }
}

function restart() {
    currentRoot = getDailyRoot();
    guesses = [];
    currentGuess = '';
    letterStatuses = {};
    gameState = 'playing';
    message = '';
}
</script>

<main class="container">
    <div class="header">
        <h1>שורשל</h1>
        <div class="hint">רמז: {currentRoot.meaning}</div>
    </div>
    <div class="guesses">
        {#each Array(maxGuesses) as _, rowIdx}
            {#if rowIdx < guesses.length}
                <!-- Filled guess row -->
                <div class="guess-row">
                    {#each Array(rootLength) as _, i}
                        <span class="letter {getLetterStatus(normalizeWord(guesses[rowIdx]), rootLength - 1 - i)}">
                            {guesses[rowIdx][rootLength - 1 - i] || ''}
                        </span>
                    {/each}
                </div>
            {:else if rowIdx === guesses.length && gameState === 'playing'}
                <!-- Current guess row -->
                <InputBlocks guess={currentGuess} rootLength={rootLength} />
            {:else}
                <!-- Blank row -->
                <div class="guess-row">
                    {#each Array(rootLength) as _, i}
                        <span class="letter"></span>
                    {/each}
                </div>
            {/if}
        {/each}
    </div>
    <Keyboard
        rows={hebrewKeyboardRows}
        {letterStatuses}
        onKey={handleKeyboardClick}
        onBackspace={handleKeyboardBackspace}
        onEnter={checkGuess}
        disabled={gameState !== 'playing'}
        currentInputLength={currentGuess.length}
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
    display: flex;
    flex-direction: column;
    justify-content: space-between;
}

@media (max-width: 600px) {
    .container {
        padding: 0.2rem;
        margin: 0;
        width: 100vw;
        min-height: 100vh;
        box-sizing: border-box;
    }
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
    display: flex;
    flex-direction: column;
    gap: 2px;
    flex: 1 1 auto;
    justify-content: center;
}
.guess-row {
    display: flex;
    justify-content: center;
    gap: 2px;
}
.letter {
    display: inline-block;
    width: 3.5rem;
    height: 3.5rem;
    line-height: 3.5rem;
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
.header {
    flex: 0 0 auto;
    width: 100%;
}
</style>
