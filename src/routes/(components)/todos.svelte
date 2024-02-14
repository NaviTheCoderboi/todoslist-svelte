<script lang="ts">
	import { quintIn } from 'svelte/easing';
	import { fade, fly } from 'svelte/transition';

	interface TodoType {
		id: number;
		text: string;
		completed: boolean;
	}

	let todos = $state<TodoType[]>([]);
	const filteredTodos = $derived(() => {
		switch (todoFilter) {
			case 'all':
				return todos;
			case 'completed':
				return todos.filter((todo) => todo.completed);
			case 'active':
				return todos.filter((todo) => !todo.completed);
		}
	});
	let todoInput = $state<string>('');
	let todoInputMode = $state<['add', any] | ['edit', any]>(['add', null]);
	let todoFilter = $state<'all' | 'completed' | 'active'>('all');

	$effect(() => {
		const localTodos = localStorage.getItem('todos');
		if (localTodos) {
			todos = JSON.parse(localTodos);
		}
	});

	const addTodo = () => {
		if (todoInput.trim() === '') {
			alert('Please enter a todo first!');
			return;
		}
		const newTodo = {
			id: Math.floor(Math.random() * 100000),
			text: todoInput,
			completed: false
		};
		todos = [newTodo, ...todos];
		localStorage.setItem('todos', JSON.stringify(todos));
		todoInput = '';
	};

	const deleteTodo = (id: number) => {
		const deletingTodo = todos.find((todo) => todo.id === id);
		if (deletingTodo) {
			const bin = localStorage.getItem('bin');
			if (bin) {
				localStorage.setItem('bin', JSON.stringify([...JSON.parse(bin), deletingTodo]));
			} else {
				localStorage.setItem('bin', JSON.stringify([deletingTodo]));
			}
			todos = todos.filter((todo) => todo.id !== id);
			localStorage.setItem('todos', JSON.stringify(todos));
		} else {
			alert('Todo not found!');
		}
	};

	const _editTodo = (id: number) => {
		const todo = todos.find((todo) => todo.id === id);
		if (todo) {
			todoInput = todo.text;
			todoInputMode = ['edit', todo];
		} else {
			alert('Todo not found!');
		}
	};

	const editTodo = () => {
		todos = todos.map((t) => {
			if (t.id === todoInputMode[1].id) {
				t.text = todoInput;
			}
			return t;
		});
		localStorage.setItem('todos', JSON.stringify(todos));
		todoInput = '';
		todoInputMode = ['add', null];
	};

	const editCompleted = (id: number) => {
		todos = todos.map((t) => {
			if (t.id === id) {
				t.completed = !t.completed;
			}
			return t;
		});
		localStorage.setItem('todos', JSON.stringify(todos));
	};

	const completeAll = () => {
		todos = todos.map((t) => {
			t.completed = true;
			return t;
		});
		localStorage.setItem('todos', JSON.stringify(todos));
	};

	const deleteCompleted = () => {
		todos.map((todo) => {
			if (todo.completed) {
				deleteTodo(todo.id);
			}
		});
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
		style={td.completed ? 'background-color: rgb(16 185 129); color: #fff;' : ''}
	>
		<input
			type="checkbox"
			checked={td.completed}
			class="todo-check"
			on:change={() => editCompleted(td.id)}
		/>
		<span>{td.text}</span>
		<div>
			<button onclick={() => _editTodo(td.id)}>
				<svg
					stroke="#000"
					fill="#000"
					stroke-width="0"
					viewBox="0 0 576 512"
					height="1.5rem"
					width="1.5rem"
					xmlns="http://www.w3.org/2000/svg"
					><path
						d="M402.6 83.2l90.2 90.2c3.8 3.8 3.8 10 0 13.8L274.4 405.6l-92.8 10.3c-12.4 1.4-22.9-9.1-21.5-21.5l10.3-92.8L388.8 83.2c3.8-3.8 10-3.8 13.8 0zm162-22.9l-48.8-48.8c-15.2-15.2-39.9-15.2-55.2 0l-35.4 35.4c-3.8 3.8-3.8 10 0 13.8l90.2 90.2c3.8 3.8 10 3.8 13.8 0l35.4-35.4c15.2-15.3 15.2-40 0-55.2zM384 346.2V448H64V128h229.8c3.2 0 6.2-1.3 8.5-3.5l40-40c7.6-7.6 2.2-20.5-8.5-20.5H48C21.5 64 0 85.5 0 112v352c0 26.5 21.5 48 48 48h352c26.5 0 48-21.5 48-48V306.2c0-10.7-12.9-16-20.5-8.5l-40 40c-2.2 2.3-3.5 5.3-3.5 8.5z"
					></path></svg
				>
			</button>
			<button onclick={() => deleteTodo(td.id)}>
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
	<div class="input-wrapper">
		<div class="todo-input">
			<svg
				stroke="#000"
				fill="#000"
				stroke-width="0"
				viewBox="0 0 512 512"
				height="1.8rem"
				width="1.8rem"
				xmlns="http://www.w3.org/2000/svg"
				><path
					d="M131.3 20.35c-14.6.1-28.1 10-31.93 24.82-2.33 9.13-.55 18.4 4.13 25.84-7.67 4.26-13.69 11.53-16.03 20.66-2.32 9.13-.56 18.33 4.1 25.83a32.687 32.687 0 0 0-15.96 20.6c-2.34 9.1-.54 18.4 4.18 25.8-7.72 4.3-13.75 11.5-16.09 20.7-2.33 9.1-.54 18.4 4.19 25.8-7.72 4.3-13.75 11.5-16.09 20.7-2.34 9.1-.54 18.4 4.18 25.8-7.72 4.3-13.75 11.5-16.08 20.7-2.34 9.1-.54 18.4 4.18 25.8-7.72 4.3-13.75 11.5-16.09 20.7-2.35 9.2-.51 18.5 4.3 26a32.915 32.915 0 0 0-16.28 20.8c-4.48 17.5 6.25 35.6 23.79 40.1l.1-.2 31.71 8.2-1.47 5.7 261.56 67L374 326.5l-22.4 21.2-87.8 26.5 15.5-42.5-151.7-38.8 4.4-17.4 153.5 39.3 9.7-26.7 15.3-14.4-167-42.8 4.4-17.4 178 45.6 39.6-37.4-206.1-52.8 4.4-17.4L380.7 207l-.1.4 31.5-29.8 18.3-71.4-261.6-67.04-4.8 18.66c2.2-16.32-8.1-32.27-24.5-36.44-2.7-.7-5.5-1.04-8.2-1.03zm.3 17.99c1.2 0 2.4.19 3.5.48 8.1 2.09 12.9 10.13 10.8 18.27l17.2 4.4-11 42.81c2.2-16.35-8.2-32.26-24.5-36.43l-.6-.15c-7.8-2.34-12.2-10.15-10.2-18.07 1.7-6.61 7.3-11 13.7-11.3h1.1zm-11.9 46.51c.9 0 1.9.14 2.9.36l.6.15c8.1 2.08 12.9 10.12 10.8 18.24l17.2 4.4-11 43c2.4-16.4-8-32.6-24.4-36.7-.7-.2-1.3-.4-1.9-.5-7-2.7-10.9-10.1-9-17.62 1.7-6.97 7.9-11.45 14.8-11.29zm59.9 4.59 217 55.66-4.4 17.4-217-55.6zm-72.9 41.86h1.3c.5 0 .9 0 1.4.1.6.2 1.2.3 1.8.5l.1-.2c8.1 2.1 12.9 10.1 10.8 18.3l17.2 4.4-11 43c2.3-16.3-8.1-32.4-24.4-36.6-8.18-2.1-12.94-10.1-10.85-18.3 1.69-6.6 7.25-10.9 13.65-11.2zM465.4 152l-10.2 9.6 31.6 33.5 10.2-9.6zm-23.3 22L315.7 293.5l31.5 33.5 126.5-119.5zm-347.23 3.7c1.48 0 3 .1 4.53.5 8.1 2.1 12.9 10.1 10.8 18.3l17.2 4.4-11 43c2.3-16.4-8.1-32.4-24.44-36.6-8.14-2.1-12.9-10.1-10.82-18.3 1.7-6.6 7.32-11 13.73-11.3zm-11.91 46.5c1.48 0 3 .1 4.53.5 8.14 2.1 12.91 10.1 10.81 18.3l17.2 4.4-11 42.9c2.3-16.3-8.1-32.3-24.45-36.5-8.14-2.1-12.89-10.1-10.81-18.3 1.69-6.6 7.31-11 13.72-11.3zm-11.9 46.5c1.48 0 3 .1 4.53.5 8.13 2.1 12.89 10.1 10.81 18.3l17.2 4.3-10.94 42.8c2.16-16.3-8.25-32.1-24.51-36.3-8.14-2.1-12.9-10.1-10.82-18.3 1.7-6.6 7.32-11 13.73-11.3zm235.34 39.2L293 346.6l37.4-11.3zm-247.25 7.3c1.48 0 3 .1 4.53.5 8.14 2.1 12.9 10.1 10.81 18.3l17.21 4.3-11 43c2.1-16.2-8.3-32-24.53-36.2l.1-.3c-8.16-2.1-12.92-10.1-10.84-18.3 1.69-6.6 7.31-11 13.72-11.3zm56.95 20.3L333.2 393l-4.4 17.4-217.1-55.5zM47.18 364c1.48 0 3 .1 4.52.5 8.14 2.1 12.9 10.1 10.82 18.3l17.2 4.3-3.69 14.4-31.92-8.2v.2c-8.01-2.2-12.67-10.1-10.61-18.2 1.7-6.6 7.32-11 13.73-11.3z"
				></path></svg
			>
			<input
				type="text"
				placeholder="Add your todo"
				class="todo-input"
				bind:value={todoInput}
				on:keydown={(e) => {
					if (e.key === 'Enter') {
						if (todoInputMode[0] === 'edit') {
							editTodo();
						} else {
							addTodo();
						}
					}
				}}
			/>
			{#if todoInputMode[0] === 'edit'}
				<button class="add-btn" onclick={editTodo}>Edit</button>
			{:else}
				<button class="add-btn" onclick={addTodo}>Add</button>
			{/if}
		</div>
		<div class="utils-bar">
			<button class="util-bar-item" onclick={completeAll}>
				<svg
					stroke="#fff"
					fill="#fff"
					stroke-width="0"
					viewBox="0 0 448 512"
					height="1.5rem"
					width="1.5rem"
					xmlns="http://www.w3.org/2000/svg"
					><path
						d="M342.6 86.6c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L160 178.7l-57.4-57.4c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3l80 80c12.5 12.5 32.8 12.5 45.3 0l160-160zm96 128c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L160 402.7 54.6 297.4c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3l128 128c12.5 12.5 32.8 12.5 45.3 0l256-256z"
					></path></svg
				>
				Complete All
			</button>
			<button class="util-bar-item" onclick={deleteCompleted}>
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
				Delete completed
			</button>
		</div>
	</div>
	<ul>
		{#each filteredTodos() as td}
			{@render todo(td)}
		{/each}
	</ul>
	<div class="input-wrapper">
		<div class="utils-bar-2">
			<select class="filter" bind:value={todoFilter}>
				<option value="all">All</option>
				<option value="completed">Completed</option>
				<option value="active">Active</option>
			</select>
			<div class="stats">
				Completed: {todos.filter((todo) => todo.completed).length}
			</div>
			<div class="stats">
				Totol: {todos.length}
			</div>
		</div>
	</div>
</section>

<style>
	.utils-bar-2 {
		display: flex;
		justify-content: space-between;
		align-items: center;
		width: 60%;
		margin-top: 20px;
		font-size: 1.3rem;
	}

	.utils-bar-2 .filter {
		background-color: rgb(203 213 225);
		border-radius: 4px;
		outline: none;
		border: none;
		font-size: 1.3rem;
	}

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

	section .input-wrapper {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		gap: 20px;
		padding: 20px 0;
	}

	section .input-wrapper .todo-input {
		width: 60%;
		background-color: rgb(203 213 225);
		border-radius: 9999px;
		display: flex;
		align-items: center;
		gap: 20px;
		padding: 10px 20px;
	}

	section .todo-input {
		font-size: 1.5rem;
		border: none;
		outline: none;
		background-color: transparent;
		flex: 1;
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

	section .todo-input::placeholder {
		color: black;
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
