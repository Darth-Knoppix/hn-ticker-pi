<script lang="ts">
	import QR from 'svelte-qr'
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';

	let showQR = false
	let page = 1;
	let storyIndex = 0;
	let stories = []
	let interval

	async function getNextPage() {
		const res = await fetch(`https://api.hackerwebapp.com/news?page=${page}`)
		const data = await res.json()

		if (res.ok) {
			return data
		}
	}

	async function getNextStory(index) {
		if (stories.length === 0 || index === stories.length) {
			page += 1
			const nextPage = await getNextPage()
			stories = stories.concat(nextPage)
		}

		return stories[index]
	}

	function toggleView() {
		if (showQR) {
			interval = startTicker()
		} else {
			clearInterval(interval)
		}
		showQR = !showQR
	}

	function startTicker() {
		return setInterval(() => {
			storyIndex += 1
			currentStory = getNextStory(storyIndex)
		}, 6000);
	}

	let currentStory = getNextStory(storyIndex)

	onMount(() => {
		interval = startTicker()
	});
</script>

<main on:click={toggleView}>
	{#await currentStory}
		<p>loading</p>
	{:then story} 
		<h1 class={showQR ? 'normal' : 'large'}>{story.title}</h1>
		<div class="flex">
			<h2>{story.domain || '...'}</h2>
			<time>{story.time_ago}</time>
		</div>
		{#if showQR}
			<div class="qr-url" transition:fade>
				<QR text={story.url} />
			</div>
		{/if}
	{/await}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		margin: 0 auto;
	}

	h1 {
		color: #490089;
		font-size: 10vh;
		font-weight: 600;
		margin-bottom: 0;
		transition: 0.5s linear;
	}

	.normal {
		font-size: 10vh;
	}

	.large {
		font-size: 14vh;
	}

	h2 {
		font-weight: 300;
		
	}

	time {
		color: #5e5e5e;
		margin-left: 1rem;
	}

	.qr-url {
		width: 25vw;
	}

	.flex {
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: baseline;
		font-size: 1em;
	}
</style>