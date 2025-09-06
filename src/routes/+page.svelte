<script>
	import { onMount } from 'svelte';

	let monologue = $state('');
	let currentMode = $state('input'); // 'input' | 'practice'
	let lines = $state(/** @type {string[]} */ ([]));
	let currentLineIndex = $state(0);
	let scrambledWords = $state(/** @type {string[]} */ ([]));
	let availableWords = $state(/** @type {string[]} */ ([]));
	let allMonologueWords = $state(/** @type {string[]} */ ([]));
	let userInput = $state('');
	let selectedWords = $state(/** @type {string[]} */ ([]));
	let showFeedback = $state(false);
	let isCorrect = $state(false);

	// Load saved monologue from localStorage
	onMount(() => {
		const saved = localStorage.getItem('promptly-monologue');
		if (saved) {
			monologue = saved;
		}
	});

	function saveMonologue() {
		localStorage.setItem('promptly-monologue', monologue);
	}

	function startPractice() {
		if (!monologue.trim()) return;

		lines = monologue.split('\n').filter((line) => line.trim());

		// Extract all unique words from the monologue for decoys
		allMonologueWords = extractAllWords(monologue);

		currentLineIndex = 0;
		currentMode = 'practice';
		generateScrambledWords();
	}

	function extractAllWords(/** @type {string} */ text) {
		// Split text into words and clean them
		const words = text
			.toLowerCase()
			.split(/\s+/)
			.map((/** @type {string} */ word) => word.replace(/[^\w]/g, '')) // Remove punctuation
			.filter((/** @type {string} */ word) => word.length > 0);

		// Return unique words only
		return [...new Set(words)];
	}

	function cleanWord(/** @type {string} */ word) {
		return word.toLowerCase().replace(/[^\w]/g, '');
	}

	function generateScrambledWords() {
		const currentLine = lines[currentLineIndex];
		const words = currentLine.split(' ').filter((/** @type {string} */ word) => word.trim());

		// Get cleaned versions of current line words for comparison
		const currentLineCleanWords = words.map(cleanWord);

		// Find decoy words from the monologue that aren't in the current line
		const availableDecoys = allMonologueWords.filter(
			(/** @type {string} */ word) => !currentLineCleanWords.includes(word)
		);

		// Add 3-7 random decoy words from the monologue
		const numDecoys = Math.min(7, Math.max(3, Math.floor(words.length * 0.6)));
		const selectedDecoys = availableDecoys.sort(() => Math.random() - 0.5).slice(0, numDecoys);

		// Combine original words with selected decoys
		// Use a Set to ensure no duplicates (comparing cleaned versions)
		const allWords = [...words];
		const usedCleanWords = new Set(currentLineCleanWords);

		// Add decoys, but only if their cleaned version isn't already used
		selectedDecoys.forEach((/** @type {string} */ decoyWord) => {
			if (!usedCleanWords.has(decoyWord)) {
				allWords.push(decoyWord);
				usedCleanWords.add(decoyWord);
			}
		});

		// Use Fisher-Yates shuffle algorithm for better randomization
		for (let i = allWords.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[allWords[i], allWords[j]] = [allWords[j], allWords[i]];
		}

		scrambledWords = allWords;
		availableWords = [...allWords];

		selectedWords = [];
		userInput = '';
		showFeedback = false;
	}

	function submitLine() {
		const currentLine = lines[currentLineIndex];
		const normalizedUser = userInput.trim().toLowerCase();
		const normalizedCorrect = currentLine.trim().toLowerCase();

		isCorrect = normalizedUser === normalizedCorrect;
		showFeedback = true;

		if (isCorrect) {
			setTimeout(() => {
				if (currentLineIndex < lines.length - 1) {
					currentLineIndex++;
					generateScrambledWords();
				} else {
					// Completed all lines
					currentMode = 'completed';
				}
			}, 1500);
		}
	}

	function resetPractice() {
		currentMode = 'input';
		currentLineIndex = 0;
		selectedWords = [];
		userInput = '';
		showFeedback = false;
	}

	function nextLine() {
		if (currentLineIndex < lines.length - 1) {
			currentLineIndex++;
			generateScrambledWords();
		} else {
			// If it's the last line, complete the practice
			currentMode = 'completed';
		}
	}

	function retryLine() {
		generateScrambledWords();
	}

	function addWordToSubmission(/** @type {string} */ word) {
		selectedWords = [...selectedWords, word];
		availableWords = availableWords.filter(w => w !== word);
		userInput = selectedWords.join(' ');
	}

	function removeWordFromSubmission(/** @type {number} */ index) {
		const removedWord = selectedWords[index];
		selectedWords = selectedWords.filter((_, i) => i !== index);
		availableWords = [...availableWords, removedWord];
		userInput = selectedWords.join(' ');
	}

	function clearSubmission() {
		availableWords = [...scrambledWords];
		selectedWords = [];
		userInput = '';
	}
</script>

<div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4">
	<div class="mx-auto max-w-2xl">
		<header class="mb-8 text-center">
			<h1 class="mb-2 text-3xl font-bold text-indigo-800">Promptly</h1>
			<p class="text-indigo-600">Practice memorizing your monologues</p>
		</header>

		{#if currentMode === 'input'}
			<div class="mb-4 rounded-lg bg-white p-6 shadow-lg">
				<h2 class="mb-4 text-xl font-semibold text-gray-800">Enter Your Monologue</h2>
				<textarea
					bind:value={monologue}
					oninput={saveMonologue}
					placeholder="Paste your monologue here... Each line will be practiced separately."
					class="h-64 w-full resize-none rounded-md border border-gray-300 p-3 focus:border-transparent focus:ring-2 focus:ring-indigo-500"
				></textarea>

				{#if monologue.trim()}
					<div class="mt-4 flex flex-col gap-3 sm:flex-row">
						<button
							onclick={startPractice}
							class="flex-1 rounded-md bg-indigo-600 px-6 py-3 font-medium text-white transition-colors hover:bg-indigo-700"
						>
							Start Practice
						</button>
						<div class="flex items-center text-sm text-gray-600">
							{monologue.split('\n').filter((line) => line.trim()).length} lines to practice
						</div>
					</div>
				{/if}
			</div>
		{/if}

		{#if currentMode === 'practice'}
			<div class="rounded-lg bg-white p-6 shadow-lg">
				<div class="mb-6">
					<div class="mb-2 flex items-center justify-between">
						<h2 class="text-xl font-semibold text-gray-800">
							Line {currentLineIndex + 1} of {lines.length}
						</h2>
						<button
							onclick={resetPractice}
							class="text-sm font-medium text-indigo-600 hover:text-indigo-700"
						>
							← Back to Input
						</button>
					</div>
					<div class="h-2 w-full rounded-full bg-gray-200">
						<div
							class="h-2 rounded-full bg-indigo-600 transition-all duration-300"
							style="width: {(currentLineIndex / lines.length) * 100}%"
						></div>
					</div>
				</div>

				<div class="mb-4 rounded-md bg-gray-50 p-3 border border-gray-200">
					<h3 class="mb-2 text-sm font-medium text-gray-700">Previous Line:</h3>
					<p class="text-gray-600 italic">
						{currentLineIndex === 0 ? 'First Line' : lines[currentLineIndex - 1]}
					</p>
				</div>

				<div class="mb-6">
					<h3 class="mb-3 text-lg font-medium text-gray-700">Available Words:</h3>
					<div class="flex flex-wrap gap-2">
						{#each availableWords as word, wordIndex}
							<button
								onclick={() => addWordToSubmission(word)}
								class="word-card rounded-md border border-gray-300 bg-gray-100 px-3 py-2 text-sm transition-colors hover:bg-indigo-100"
								data-word-index={wordIndex}
							>
								{word}
							</button>
						{/each}
					</div>
				</div>

				<div class="mb-4">
					<h3 class="mb-2 text-sm font-medium text-gray-700">
						Submission Line:
					</h3>
					<div class="submission-line min-h-[5rem] w-full rounded-md border border-gray-300 p-3 bg-gray-50 flex flex-wrap items-start gap-2">
						{#if selectedWords.length === 0}
							<span class="text-gray-500 italic">Click words above to build your line...</span>
						{:else}
							{#each selectedWords as word, index}
								<button
									onclick={() => removeWordFromSubmission(index)}
									class="submission-word inline-block rounded-md bg-indigo-100 border border-indigo-200 px-2 py-1 text-sm font-medium text-indigo-800 hover:bg-indigo-200 transition-colors"
								>
									{word}
								</button>
							{/each}
						{/if}
					</div>
				</div>

				{#if showFeedback}
					<div
						class="mb-4 rounded-md p-4 {isCorrect
							? 'border border-green-200 bg-green-50'
							: 'border border-red-200 bg-red-50'}"
					>
						{#if isCorrect}
							<p class="font-medium text-green-800">✓ Correct! Moving to next line...</p>
						{:else}
							<p class="mb-2 font-medium text-red-800">✗ Not quite right. The correct line is:</p>
							<p class="mb-3 text-gray-700 italic">"{lines[currentLineIndex]}"</p>
							<div class="flex flex-col gap-2 sm:flex-row">
								<button
									onclick={retryLine}
									class="rounded-md bg-orange-600 px-4 py-2 text-sm font-medium text-white hover:bg-orange-700"
								>
									Try Again
								</button>
								<button
									onclick={nextLine}
									class="rounded-md bg-indigo-600 px-4 py-2 text-sm font-medium text-white hover:bg-indigo-700"
								>
									{currentLineIndex < lines.length - 1 ? 'Skip to Next Line' : 'Finish Practice'}
								</button>
							</div>
						{/if}
					</div>
				{:else}
					<div class="flex flex-col gap-3 sm:flex-row">
						<button
							onclick={submitLine}
							disabled={!userInput.trim()}
							class="flex-1 rounded-md bg-indigo-600 px-6 py-3 font-medium text-white transition-colors hover:bg-indigo-700 disabled:cursor-not-allowed disabled:bg-gray-300"
						>
							Submit Line
						</button>
						<button
							onclick={clearSubmission}
							class="rounded-md bg-gray-500 px-6 py-3 font-medium text-white transition-colors hover:bg-gray-600"
						>
							Clear
						</button>
					</div>
				{/if}
			</div>
		{/if}

		{#if currentMode === 'completed'}
			<div class="rounded-lg bg-white p-6 text-center shadow-lg">
				<div class="mb-4 text-6xl">🎉</div>
				<h2 class="mb-4 text-2xl font-bold text-gray-800">Congratulations!</h2>
				<p class="mb-6 text-gray-600">
					You've completed all {lines.length} lines of your monologue.
				</p>
				<div class="flex flex-col justify-center gap-3 sm:flex-row">
					<button
						onclick={() => {
							currentLineIndex = 0;
							currentMode = 'practice';
							generateScrambledWords();
						}}
						class="rounded-md bg-indigo-600 px-6 py-3 font-medium text-white transition-colors hover:bg-indigo-700"
					>
						Practice Again
					</button>
					<button
						onclick={resetPractice}
						class="rounded-md bg-gray-500 px-6 py-3 font-medium text-white transition-colors hover:bg-gray-600"
					>
						New Monologue
					</button>
				</div>
			</div>
		{/if}
	</div>
</div>

<style>
	.submission-word {
		animation: wordAppear 0.3s ease-out;
	}

	@keyframes wordAppear {
		0% {
			transform: scale(0.5) translateY(-10px);
			opacity: 0;
		}
		100% {
			transform: scale(1) translateY(0);
			opacity: 1;
		}
	}
</style>
