<script lang="ts">
	import type { NoteDocType } from '../../schema';
	import { onMount } from 'svelte';
	import { db } from '../../store';

	let db$;
	let notesList: NoteDocType[] = [];
	let name: string, body: string;

	const resetForm = () => {
		name = '';
		body = '';
	};

	onMount(() => {
		const getNotesList = async () => {
			db$ = await db();
			db$.notes
				.find()
				.sort({ updatedAt: 'desc' })
				.$.subscribe((notes) => (notesList = notes));
		};
		getNotesList();
	});

	const saveNote = async () => {
		const db$ = await db();

		await db$.notes
			.insert({
				name,
				body,
				createdAt: new Date().getTime(),
        		updatedAt: new Date().getTime(),
                })
			.then(resetForm);
	};

	function onSubmit() {
        saveNote();
    }
</script>

<h1>Notes</h1>

<main>
	<form on:submit|preventDefault={onSubmit}>
		<div>
			<label for="itemId">Name</label>
			<input type="text" id="name" name="name" bind:value={name} />
		</div>
		<div>
			<label for="name">Body</label>
			<input type="text" id="body" name="body" bind:value={body} />
		</div>
		<button type="submit">Submit</button>
	</form>

	<div>
		<h2>Notes</h2>
		<ul id="note-list" class="nostyle">
			{#await notesList}
				Loading Notes...
			{:then results}
				{#each results as note}
					<li>{note.name} - {note.body} - {new Date(note.createdAt).toLocaleDateString('en-US')} </li>
				{/each}
			{/await}
		</ul>
	</div>
</main>

<style>
	* {
		box-sizing: border-box;
	}
	form {
		display: flex;
		flex-direction: column;
		width: 300px;
	}

	form > div {
		display: flex;
		justify-content: space-between;
	}

	form > div + * {
		margin-top: 10px;
	}
</style>