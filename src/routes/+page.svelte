<script lang="ts">
	import { onMount } from 'svelte';

	let sentence: string = '';
	let typedSentence: string = '';
	let untypedSentence: string = '';

	let words: Array<string> = [];
	let wordsPerMinute: number = 0;

    let timer: Function | null = null // startTimer();

	let correctText: boolean = true;

	// This is a function that returns a random number between 1 and 100
	function getRandomNumber(): number {
		return Math.floor(Math.random() * 100) + 1;
	}

	async function fetchSentence() {
		const data = await fetch(`https://dummyjson.com/quotes/${getRandomNumber()}`);
		const jsonData = await data.json();
		sentence = jsonData.quote;
		sentence = sentence.replace(/’/g, "'");
        sentence = sentence.replace(/‘/g, "'");
		sentence = sentence.replace('—', '-');
		words = sentence.split(' ');
	}

	function resetStates() {
		sentence = '';
		typedSentence = '';
		untypedSentence = '';
		correctText = true;
		words = [];
        timer = null;
	}

	function calculateWordsPerMinute(timer: Function) {
        const minutesElapsed = timer();
		const numberOfWords = words.length;
		wordsPerMinute = Math.round(numberOfWords / minutesElapsed);
	}

	function startTimer() {
		// start time in milliseconds
		const startTime = new Date().getTime();
		// return a function that calculates the elapsed time
		return function () {
			const currentTime = new Date().getTime();
			// return the elapsed time in minutes
			return (currentTime - startTime) / 1000 / 60;
		};
	}

	// generates a sentence on first load
	onMount(async () => {
		await fetchSentence();
	});

	// generates a new sentence when the button is clicked
	async function generateNewSentence() {
		resetStates();
		await fetchSentence();
	}

	function handleInput(e: Event) {
		if (!e.target) return;

		// typed sentence is the sentence that the user has typed
		typedSentence = (e.target as HTMLInputElement).value;

		if (typedSentence.length === 0) correctText = true;

         // start the timer if the user has typed the first character
        if (typedSentence.length === 1) {
            timer = startTimer();
        }

		correctText = typedSentence === sentence.slice(0, typedSentence.length);

		// untyped sentence is essentially the sentence without the typed sentence
		if (correctText) {
			untypedSentence = sentence.slice(typedSentence.length);
		} else {
            // if the user has typed incorrectly, the untyped sentence is the last character of the sentence
			untypedSentence = sentence.slice(typedSentence.length - 1);
		}

		// if the typed sentence is the same as the sentence, generate a new sentence and calculate the words per minute
		if (typedSentence === sentence && timer) {
            calculateWordsPerMinute(timer);
            generateNewSentence();
        }
	}
</script>

<div class="container">
	<h1>Welcome to Sentence Writer</h1>
	<h2>Test your typing skills by writing the sentence on the screen</h2>

	{#if sentence}
		{#if typedSentence.length === 0}
			<p class="untyped-text">{sentence}</p>
		{:else}
			<div>
				<span class={correctText ? 'typed-text-correct' : 'typed-text-incorrect'}>
					{typedSentence}
				</span>
				<span class="untyped-text"> {untypedSentence}</span>
			</div>
		{/if}
	{:else}
		<p>Generating new sentence...</p>
	{/if}

	<input
		type="text"
		on:input={handleInput}
		value={typedSentence}
		maxLength={correctText ? sentence.length : typedSentence.length}
	/>

	<button on:click={generateNewSentence}> Generate new sentence </button>

	<p>WPM: {wordsPerMinute}</p>
</div>

<style>
	h1 {
		text-align: center;
	}

	h2 {
		text-align: center;
	}

	p {
		text-align: center;
		font-size: 2rem;
		width: fit-content;
		margin: 0;
	}

	span {
		text-align: center;
		font-size: 2rem;
		width: fit-content;
	}

	button {
		display: block;
		margin: 0 auto;
		margin-top: 2rem;
		padding: 1rem;
		font-size: 1.5rem;
		border: none;
		border-radius: 5px;
		background-color: #f1f1f1;
		cursor: pointer;
	}
	button:hover {
		background-color: #e1e1e1;
	}

	.untyped-text {
		color: gray;
	}

	.typed-text-correct {
		color: #99cc00;
		background-color: #e8edea;
		font-weight: bold;
	}

	.typed-text-incorrect {
		color: red;
		background-color: #e8edea;
		font-weight: bold;
	}

	input {
		display: block;
		margin: 0 auto;
		margin-top: 2rem;
		padding: 1rem;
		font-size: 1.5rem;
		border: none;
		border-radius: 5px;
		background-color: #f1f1f1;
		width: 50%;
	}

	.container {
		margin-top: 2rem;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
</style>
