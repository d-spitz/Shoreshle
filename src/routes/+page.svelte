<script lang="ts">
	import { roots } from '$lib/roots';
	import type { Root } from '$lib/models/root';
	import Keyboard from '$lib/Keyboard.svelte';
	import InputBlocks from '$lib/InputBlocks.svelte';
	import Alert from '$lib/Alert.svelte';

	const hebrewKeyboardRows: string[][] = [
		Array.from('קראטופ'),
		Array.from('שדגכעיחל'),
		Array.from('זסבהנמצת')
	];

	const finalToRegular: Record<string, string> = {
		ך: 'כ',
		ם: 'מ',
		ן: 'נ',
		ף: 'פ',
		ץ: 'צ'
	};

	const maxGuesses = 6;
	let currentRoot: Root = getDailyRoot();
	let guesses: string[] = [];
	let rootLength = currentRoot.root.length;
	let currentGuess = '';
	let gameState: 'playing' | 'won' | 'lost' = 'playing';
	let dialogMessage = '';
	let alertMessage = '';
	let showAlert = false;
	let normalizedRoots = roots.map((r) => normalizeWord(r.root));

	let letterStatuses: Record<string, 'correct' | 'present' | 'absent' | undefined> = {};

	function getDailyRoot(): Root {
		const today = new Date().toISOString().slice(0, 10);
		// FNV-1a hash
		let hash = 2166136261;
		for (let i = 0; i < today.length; i++) {
			hash ^= today.charCodeAt(i);
			hash += (hash << 1) + (hash << 4) + (hash << 7) + (hash << 8) + (hash << 24);
		}
		const idx = Math.abs(hash) % roots.length;
		return roots[idx];
	}

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
			if (
				status === 'correct' ||
				(status === 'present' && letterStatuses[letter] !== 'correct') ||
				(status === 'absent' && !letterStatuses[letter])
			) {
				letterStatuses[letter] = status;
			}
		}
	}

	function checkGuess() {
		const input = currentGuess;
		const normInput = normalizeWord(input);
		const normAnswer = normalizeWord(currentRoot.root);
		if (normInput.length < 3 || normInput.length > 4) {
			alertMessage = 'שורש חייב להיות 3 או 4 אותיות';
			showAlert = true;
			currentGuess = '';
			return;
		}
		if (!normalizedRoots.includes(normInput)) {
			alertMessage = 'שורש לא קיים ברשימה';
			showAlert = true;
			currentGuess = '';
			return;
		}
		if (guesses.includes(input)) {
			alertMessage = 'כבר ניסית את השורש הזה';
			showAlert = true;
			currentGuess = '';
			return;
		}
		guesses = [...guesses, input];
		updateLetterStatuses(input);
		if (normInput === normAnswer) {
			gameState = 'won';
			dialogMessage = 'ניחשת נכון!';
		} else if (guesses.length >= maxGuesses) {
			gameState = 'lost';
			dialogMessage = `הפסדת! השורש היה: ${currentRoot.root}`;
		} else {
			dialogMessage = '';
		}
		currentGuess = '';
	}

	function getGuessStatuses(guess: string): Array<'correct' | 'present' | 'absent'> {
		const normGuess = normalizeWord(guess);
		const normAnswer = normalizeWord(currentRoot.root);

		// First pass: mark correct letters and count remaining letters in answer
		const statuses = Array(normGuess.length).fill('absent');
		const remainingLetters = [...normAnswer];

		// First mark all correct letters
		for (let i = 0; i < normGuess.length; i++) {
			if (normGuess[i] === remainingLetters[i]) {
				statuses[i] = 'correct';
				remainingLetters[i] = '#'; // Mark as used
			}
		}

		// Then handle present letters, only if they remain in answer
		for (let i = 0; i < normGuess.length; i++) {
			if (statuses[i] !== 'correct') {
				const remainingIdx = remainingLetters.indexOf(normGuess[i]);
				if (remainingIdx !== -1) {
					statuses[i] = 'present';
					remainingLetters[remainingIdx] = '#'; // Mark as used
				}
			}
		}

		return statuses;
	}

	function getLetterStatus(guess: string, index: number, answerOverride?: string) {
		return getGuessStatuses(guess)[index];
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
		dialogMessage = '';
	}

	let messageDialog: HTMLDialogElement;
	$: if (dialogMessage && messageDialog && !messageDialog.open) {
		messageDialog.showModal();
	} // TODO make this svelte 5
</script>

<main class="container">
	<Alert message={alertMessage} show={showAlert} onClose={() => (showAlert = false)} />
	<div class="hint-details">
		{currentRoot.meaning}
	</div>
	<div class="guesses">
		{#each Array(maxGuesses) as _, rowIdx}
			{#if rowIdx < guesses.length}
				<!-- Filled guess row -->
				<InputBlocks
					guess={guesses[rowIdx]}
					{rootLength}
					disabled={true}
					statuses={getGuessStatuses(guesses[rowIdx])}
				/>
			{:else if rowIdx === guesses.length && gameState === 'playing'}
				<!-- Current guess row -->
				<InputBlocks guess={currentGuess} {rootLength} filled={true} />
			{:else}
				<!-- Blank row -->
				<InputBlocks guess={''} {rootLength} />
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
	<dialog bind:this={messageDialog} class="message-dialog" on:close={() => (dialogMessage = '')}>
		<div class="message-content">{dialogMessage}</div>
		{#if gameState !== 'playing'}
			<button
				class="restart-btn"
				on:click={() => {
					restart();
					messageDialog.close();
				}}>שחק שוב</button
			>
		{/if}
		<button class="close-btn" on:click={() => messageDialog.close()}>✕</button>
	</dialog>
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
		}
	}

	.container {
		max-width: 400px;
		margin: 2rem auto;
		padding: 1rem;
		background: var(--container-bg);
		border-radius: 1rem;
		box-shadow: 0 2px 16px rgba(0, 0, 0, 0.08);
		text-align: center;
		font-family: 'Assistant', Arial, sans-serif;
		color: var(--foreground);
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		gap: 1rem;
		align-items: center;
	}

	@media (max-width: 600px) {
		.container {
			padding: 1rem 0.2rem;
			margin: 0;
			box-sizing: border-box;
		}
	}
	.guesses {
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
		flex: 1 1 auto;
		justify-content: center;
	}
	.restart-btn {
		font-size: 1.1rem;
		border-radius: 0.4rem;
		border: none;
		background: var(--btn-bg);
		color: #fff;
		cursor: pointer;
		transition: background 0.2s;
		display: inline-block;
	}
	.restart-btn {
		background: var(--btn-bg-restart);
		margin-top: 1rem;
		padding: 0.5rem 1.2rem;
	}
	.hint-details {
		background: var(--box-bg);
		border-radius: 0.5rem;
		box-shadow: 0 1px 4px rgba(0, 0, 0, 0.04);
		padding: 0.5rem;
		color: var(--hint);
		border: 1px solid var(--box-border);
		transition: background 0.2s;
		width: 9rem;
		text-align: center;
	}
	.message-dialog {
		min-width: 220px;
		max-width: 90vw;
		border: none;
		border-radius: 1rem;
		background: var(--container-bg);
		color: var(--foreground);
		box-shadow: 0 4px 32px rgba(0, 0, 0, 0.18);
		padding: 1.2rem 1.2rem 1.2rem 1.2rem;
		text-align: center;
		position: relative;
		z-index: 1000;
	}
	.message-content {
		font-size: 1.2rem;
		margin-bottom: 1.2rem;
		width: 70vw;
		max-width: 300px;
	}
	.close-btn {
		position: absolute;
		top: 0.7rem;
		left: 0.7rem;
		background: none;
		border: none;
		color: var(--hint);
		font-size: 1.3rem;
		cursor: pointer;
		padding: 0.2rem 0.5rem;
		border-radius: 0.3rem;
		transition: background 0.15s;
	}
	.close-btn:hover {
		background: var(--box-bg);
	}
	@media (max-width: 600px) {
		.message-dialog {
			min-width: 0;
			padding: 0.7rem 0.2rem 1.2rem 0.2rem;
		}
	}
</style>
