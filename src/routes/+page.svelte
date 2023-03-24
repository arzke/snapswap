<script lang="ts">
	let link: string | undefined;
	let status: 'idle' | 'loading' | 'loaded' = 'idle';

	const currentTab = { active: true, currentWindow: true };

	async function getUrl() {
		try {
			status = 'loading';

			const [tab] = await chrome.tabs.query(currentTab);
			const url = tab.url;

			const response = await fetch(`https://archive.is/${url}`);
			const data = await response.text();

			const container = document.createElement('div');
			container.innerHTML = data;

			link =
				(
					container.querySelector('#row0 > div.TEXT-BLOCK > a:nth-child(1)') as
						| HTMLAnchorElement
						| undefined
				)?.href ?? '';

			link &&
				(await chrome.tabs.update(tab.id as number, {
					url: link
				}));
		} finally {
			status = 'loaded';
		}
	}
</script>

{#if status === 'idle'}
	<button on:click={getUrl}>Go to archived version</button>
{:else if status === 'loading'}
	<p>Looking for an archived version of this page.</p>
{:else if status === 'loaded' && !link}
	<p>No archive found</p>
{:else if status === 'loaded'}
	<p>
		Archive found. <br />
		Redirecting to page...
	</p>
{/if}

<style>
	button {
		color: #222;
		display: inline-block;
		width: 100%;
		height: 100%;
		border: none;
		transition: all 0.15s ease-in;
	}

	button:hover {
		background: #ccc;
	}
</style>
