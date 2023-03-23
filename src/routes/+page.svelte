<script lang="ts">
	let link: string | undefined;
	let loaded = false;

	const currentTab = { active: true, currentWindow: true };

	async function getUrl() {
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

		link && await chrome.tabs.update(tab.id as number, {
			url: link
		});
		loaded = true;
	}
</script>

<button on:click={getUrl}>Go to archived version</button>

{#if loaded && !link}
	<div>No archive found</div>
{/if}
