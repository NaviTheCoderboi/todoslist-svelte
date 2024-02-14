<script lang="ts">
	import Bin from './(components)/bin.svelte';
	import Todos from './(components)/todos.svelte';

	let currentTab = $state<'notes' | 'bin'>('notes');
</script>

<main>
	<div class="main">
		<h1>Todos List</h1>
		<a href="https://svelte.dev/" target="_blank" class="link-to-svelte"
			>made using svelte 5(runes mode)</a
		>
		<div class="switcher">
			<button
				onclick={() => (currentTab = 'notes')}
				class="switch"
				style={currentTab == 'notes'
					? 'background-color: rgb(203 213 225) !important; color: black !important;'
					: ''}>Todos</button
			>
			<button
				onclick={() => (currentTab = 'bin')}
				class="switch"
				style={currentTab == 'bin'
					? 'background-color: rgb(203 213 225) !important; color: black !important;'
					: ''}>Bin</button
			>
		</div>
		{#if currentTab === 'notes'}
			<Todos />
		{:else}
			<Bin />
		{/if}
	</div>
</main>

<style>
	main {
		min-height: 100vh;
		width: 100%;
		background-color: rgb(0, 0, 0);
		color: rgb(255, 255, 255);
	}

	main .main {
		margin: 0 auto;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		width: 55%;
		min-height: 100vh;
	}

	.main h1 {
		font-size: 3.5rem;
	}

	.main .link-to-svelte {
		font-size: 1.5rem;
		color: #f96743;
		font-weight: 100;
		text-decoration: none;
	}

	.main .link-to-svelte:hover {
		text-decoration: underline;
	}

	.switcher {
		display: flex;
		justify-content: center;
		align-items: center;
		gap: 40px;
		padding-top: 20px;
		padding-bottom: 20px;
		width: 100%;
		background-color: rgb(30 41 59);
		margin-top: 30px;
	}

	.switcher .switch {
		padding: 10px 20px;
		background-color: rgb(51 65 85);
		text-align: center;
		border-radius: 1rem;
		font-size: 1.5rem;
		font-weight: 800;
		outline: none;
		border: none;
		color: white;
	}

	.switcher .switch:hover {
		filter: brightness(1.2);
	}
</style>
