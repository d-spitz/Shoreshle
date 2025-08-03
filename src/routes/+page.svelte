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
	const shareUrl = 'https://d-spitz.github.io/Shoreshle/';
	let currentRoot = $state(getDailyRoot());
	let guesses = $state<string[]>([]);
	let rootLength = $derived(currentRoot.root.length);
	let currentGuess = $state('');
	let gameState = $state<'playing' | 'won' | 'lost'>('playing');
	let showDialog = $state(false);
	let alertMessage = $state('');
	let showAlert = $state(false);
	let alertType = $state<'success' | 'warning'>('warning');
	let normalizedRoots = $derived(roots.map((r) => normalizeWord(r.root)));

	let letterStatuses = $state<Record<string, 'correct' | 'present' | 'absent' | undefined>>({});

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
			alertMessage = 'השורש חייב להיות 3 או 4 אותיות';
			showAlert = true;
			currentGuess = '';
			return;
		}
		if (!normalizedRoots.includes(normInput)) {
			alertMessage = 'השורש לא קיים ברשימה';
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
			showDialog = true;
			if (typeof document !== 'undefined') {
				setCookie(getTodayKey(), 'won');
				setGuessesCookie();
			}
		} else if (guesses.length >= maxGuesses) {
			gameState = 'lost';
			showDialog = true;
			if (typeof document !== 'undefined') {
				setCookie(getTodayKey(), 'lost');
				setGuessesCookie();
			}
		} else {
			showDialog = false;
			if (typeof document !== 'undefined') {
				setGuessesCookie();
			}
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
		showDialog = false;
	}

	let messageDialog = $state<HTMLDialogElement>();

	$effect(() => {
		if (showDialog && messageDialog && !messageDialog.open) {
			messageDialog.showModal();
		}
	});

	// Emoji grid for sharing
	function getEmojiGrid() {
		// Use colored squares for each guess
		const colorMap = {
			correct: '🟩',
			present: '🟨',
			absent: '⬜'
		};
		return guesses
			.map((guess) =>
				getGuessStatuses(guess)
					.toReversed()
					.map((status) => colorMap[status])
					.join('')
			)
			.join('\n');
	}

	function getShareText() {
		const dateStr = new Date().toLocaleDateString('he-IL', {
			day: '2-digit',
			month: '2-digit',
			year: '2-digit'
		});
		return `שורשל\n${dateStr} | ${guesses.length}/${maxGuesses}\n${getEmojiGrid()}`;
	}

	async function handleShare() {
		const text = getShareText();
		let shared = false;
		try {
			if (navigator.share) {
				await navigator.share({ text, url: shareUrl });
				shared = true;
			}
		} catch {}
		if (!shared) {
			alert('לא ניתן לשתף בדפדפן זה');
		}
	}

	async function handleCopy() {
		const text = getShareText();
		if (navigator.clipboard) {
			await navigator.clipboard.writeText(text);
			alertMessage = 'ההישג הועתק!';
			alertType = 'success';
			showAlert = true;
		} else {
			alertMessage = 'לא ניתן להעתיק בדפדפן זה';
			alertType = 'warning';
			showAlert = true;
		}
	}

	// Cookie helpers
	function setCookie(name: string, value: string, days = 365) {
		const expires = new Date(Date.now() + days * 864e5).toUTCString();
		document.cookie = name + '=' + encodeURIComponent(value) + '; expires=' + expires + '; path=/';
	}
	function getCookie(name: string): string | undefined {
		return document.cookie
			.split('; ')
			.find((row) => row.startsWith(name + '='))
			?.split('=')[1];
	}
	function getTodayKey() {
		return 'shoreshle_' + new Date().toISOString().slice(0, 10);
	}
	function checkCookieGameState() {
		const key = getTodayKey();
		const val = getCookie(key);
		if (val === 'won') {
			gameState = 'won';
			showDialog = true;
			return true;
		} else if (val === 'lost') {
			gameState = 'lost';
			showDialog = true;
			return true;
		}
		return false;
	}

	// Save gameState in guesses cookie for emoji grid
	function setGuessesCookie() {
		const key = getTodayKey() + '_guesses';
		setCookie(key, JSON.stringify(guesses), 365);
	}
	function getGuessesFromCookie(): string[] {
		const key = getTodayKey() + '_guesses';
		const val = getCookie(key);
		if (val) {
			try {
				return JSON.parse(decodeURIComponent(val));
			} catch {
				return [];
			}
		}
		return [];
	}

	$effect(() => {
		if (typeof document !== 'undefined') {
			checkCookieGameState();
			const savedGuesses = getGuessesFromCookie();
			if (savedGuesses.length > 0 && guesses.length === 0) {
				guesses = savedGuesses;
			}
		}
	});
</script>

<main class="container">
	<Alert
		message={alertMessage}
		show={showAlert}
		type={alertType}
		onClose={() => (showAlert = false)}
	/>
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
	<dialog bind:this={messageDialog} class="message-dialog" onclose={() => (showDialog = false)}>
		{#if gameState !== 'playing'}
			<div class="dialog-header">
				<span class="game-title">שורשל</span>
				<span class="game-date"
					>{new Date().toLocaleDateString('he-IL', {
						day: '2-digit',
						month: '2-digit',
						year: '2-digit'
					})}</span
				>
				<span class="game-guesses">{guesses.length}/{maxGuesses}</span>
			</div>
			<div class="message-content">
				{#if gameState === 'won'}
					ניחשת נכון!
				{:else}
					הפסדת! השורש היה: <strong>{currentRoot.root.split('').join('.')}</strong>
				{/if}
			</div>
			<div class="emoji-grid">
				{#each guesses as guess}
					<div class="emoji-row" dir="rtl">
						{#each getGuessStatuses(guess) as status}
							<span class="emoji-cell {status}"
								>{status === 'correct' ? '🟩' : status === 'present' ? '🟨' : '⬜'}</span
							>
						{/each}
					</div>
				{/each}
			</div>
			<div class="dialog-actions">
				<button class="share-btn" onclick={handleShare} aria-label="שתף">
					<svg
						xmlns="http://www.w3.org/2000/svg"
						width="24"
						height="24"
						style="vertical-align: middle;"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="2"
						stroke-linecap="round"
						stroke-linejoin="round"
						><path d="M12 2v13" /><path d="m16 6-4-4-4 4" /><path
							d="M4 12v8a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-8"
						/></svg
					>
				</button>
				<button class="copy-btn" onclick={handleCopy} aria-label="העתק">
					<svg
						xmlns="http://www.w3.org/2000/svg"
						width="24"
						height="24"
						style="vertical-align: middle;"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="2"
						stroke-linecap="round"
						stroke-linejoin="round"
						class="lucide lucide-copy-icon lucide-copy"
						><rect width="14" height="14" x="8" y="8" rx="2" ry="2" /><path
							d="M4 16c-1.1 0-2-.9-2-2V4c0-1.1.9-2 2-2h10c1.1 0 2 .9 2 2"
						/></svg
					>
				</button>
			</div>
		{/if}
		<button
			class="close-btn"
			onclick={() => {
				if (messageDialog) messageDialog.close();
			}}>✕</button
		>
	</dialog>
</main>

<style>
	@import './page.css';
</style>
