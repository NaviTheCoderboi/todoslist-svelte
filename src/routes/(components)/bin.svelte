<script lang="ts">
	import { quintIn } from 'svelte/easing';
	import { fade, fly } from 'svelte/transition';

	interface TodoType {
		id: number;
		text: string;
		completed: boolean;
	}

	let bin = $state<TodoType[]>([]);

	$effect(() => {
		const localBin = localStorage.getItem('bin');
		if (localBin) {
			bin = JSON.parse(localBin);
		}
	});

	const permanentlyDeleteTodo = (id: number) => {
		const deletingTodo = bin.find((todo) => todo.id === id);
		if (deletingTodo) {
			bin = bin.filter((todo) => todo.id !== id);
			localStorage.setItem('bin', JSON.stringify(bin));
		} else {
			alert('Todo not found!');
		}
	};

	const restoreTodo = (id: number) => {
		const restoringTodo = bin.find((todo) => todo.id === id);
		if (restoringTodo) {
			bin = bin.filter((todo) => todo.id !== id);
			localStorage.setItem('bin', JSON.stringify(bin));
			const todos = JSON.parse(localStorage.getItem('todos') || '[]');
			localStorage.setItem('todos', JSON.stringify([...todos, restoringTodo]));
		} else {
			alert('Todo not found!');
		}
	};

	const restoreAll = () => {
		const todos = JSON.parse(localStorage.getItem('todos') || '[]');
		localStorage.setItem('todos', JSON.stringify([...todos, ...bin]));
		bin = [];
		localStorage.setItem('bin', JSON.stringify(bin));
	};

	const permanentlyDeleteAll = () => {
		bin = [];
		localStorage.setItem('bin', JSON.stringify(bin));
	};
</script>

{#snippet todo(td)}
	<li
		in:fly={{
			y: -20,
			duration: 300,
			easing: quintIn
		}}
		out:fade={{
			duration: 500,
			easing: quintIn
		}}
		class="todo"
	>
		<span>{td.text}</span>
		<div>
			<button onclick={() => restoreTodo(td.id)}>
				<svg
					stroke="#000"
					fill="#000"
					stroke-width="0"
					viewBox="0 0 24 24"
					height="1.5rem"
					width="1.5rem"
					xmlns="http://www.w3.org/2000/svg"
					><path fill="none" d="M0 0h24v24H0V0z"></path><path
						d="M14 12c0-1.1-.9-2-2-2s-2 .9-2 2 .9 2 2 2 2-.9 2-2zm-2-9a9 9 0 0 0-9 9H0l4 4 4-4H5c0-3.87 3.13-7 7-7s7 3.13 7 7a6.995 6.995 0 0 1-11.06 5.7l-1.42 1.44A9 9 0 1 0 12 3z"
					></path></svg
				>
			</button>
			<button onclick={() => permanentlyDeleteTodo(td.id)}>
				<svg
					stroke="#000"
					fill="#000"
					stroke-width="0"
					viewBox="0 0 24 24"
					height="1.5rem"
					width="1.5rem"
					xmlns="http://www.w3.org/2000/svg"
					><path
						d="M7 4V2H17V4H22V6H20V21C20 21.5523 19.5523 22 19 22H5C4.44772 22 4 21.5523 4 21V6H2V4H7ZM6 6V20H18V6H6ZM9 9H11V17H9V9ZM13 9H15V17H13V9Z"
					></path></svg
				>
			</button>
		</div>
	</li>
{/snippet}

<section>
	<div class="utils-bar-wrapper">
		<div class="utils-bar">
			<button class="util-bar-item" onclick={restoreAll}>
				<svg
					stroke="#fff"
					fill="#fff"
					stroke-width="0"
					viewBox="0 0 24 24"
					height="1.5rem"
					width="1.5rem"
					xmlns="http://www.w3.org/2000/svg"
					><path fill="none" d="M0 0h24v24H0V0z"></path><path
						d="M14 12c0-1.1-.9-2-2-2s-2 .9-2 2 .9 2 2 2 2-.9 2-2zm-2-9a9 9 0 0 0-9 9H0l4 4 4-4H5c0-3.87 3.13-7 7-7s7 3.13 7 7a6.995 6.995 0 0 1-11.06 5.7l-1.42 1.44A9 9 0 1 0 12 3z"
					></path></svg
				>
				Restore All
			</button>
			<button class="util-bar-item" onclick={permanentlyDeleteAll}>
				<svg
					stroke="#fff"
					fill="#fff"
					stroke-width="0"
					viewBox="0 0 24 24"
					height="1.5rem"
					width="1.5rem"
					xmlns="http://www.w3.org/2000/svg"
					><path
						d="M7 4V2H17V4H22V6H20V21C20 21.5523 19.5523 22 19 22H5C4.44772 22 4 21.5523 4 21V6H2V4H7ZM6 6V20H18V6H6ZM9 9H11V17H9V9ZM13 9H15V17H13V9Z"
					></path></svg
				>
				Permanently Delete All
			</button>
		</div>
	</div>
	<ul>
		{#each bin as td}
			{@render todo(td)}
		{/each}
	</ul>
</section>

<style>
	.utils-bar {
		display: flex;
		justify-content: space-between;
		align-items: center;
		width: 60%;
		padding: 0 20px;
	}

	.util-bar-item {
		background-color: transparent;
		border: none;
		color: #fff;
		display: flex;
		align-items: center;
		gap: 10px;
		cursor: pointer;
		font-size: 1.4rem;
		filter: brightness(0.8);
	}

	.util-bar-item:hover {
		filter: brightness(1);
	}

	section {
		width: 100%;
		height: 100%;
		padding: 40px;
		background-color: rgb(30 41 59);
	}

	section .utils-bar-wrapper {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		gap: 20px;
		padding: 20px 0;
	}

	section .add-btn {
		background-color: #f96743;
		color: #fff;
		border: none;
		border-radius: 9999px;
		padding: 10px 20px;
		font-size: 1.5rem;
		cursor: pointer;
	}

	section .add-btn:hover {
		filter: brightness(1.2);
	}

	ul {
		height: 100%;
		width: 100%;
		display: grid;
		grid-auto-flow: row;
		grid-template-columns: repeat(2, 1fr);
		gap: 20px;
		list-style: none;
	}

	.todo {
		display: flex;
		align-items: center;
		gap: 20px;
		padding: 10px 20px;
		background-color: #fff;
		border-radius: 0.2rem;
		color: black;
		font-size: 1.5rem;
		font-weight: 700;
		filter: brightness(0.9);
		cursor: pointer;
		transition: all 0.3s;
	}

	.todo:hover {
		filter: brightness(1);
	}

	.todo .todo-check {
		width: 1.5rem;
		height: 1.5rem;
	}

	.todo span {
		flex: 1;
	}

	.todo div {
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 10px;
	}

	.todo div button {
		background-color: transparent;
		border: none;
		cursor: pointer;
	}
</style>
