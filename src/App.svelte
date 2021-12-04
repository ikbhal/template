<script>
import {onMount} from 'svelte';
//https://svelte.dev/tutorial/writable-stores
import {writable} from 'svelte/store';
//../../template/src/firebasevTest from './EnvTest.svelte';

import {db} from './firebase';
let todayDateString = new Date().toISOString().slice(0,10);
export let date= todayDateString;
let template = '';
let replaceNote = '';
let instance = '';

export let instanceList=writable([]);

// https://vim.fandom.com/wiki/Cut/copy_and_paste_using_visual_selection -> V visual select line, d delete, y copy, p paste
async function loadinstanceList(){
	let colRef = db.collection('template');
  	let allDocs= await colRef.get();
	let docListCopy = [];
	for(const doc of allDocs.docs){
		let docData = doc.data();
		let docCopy = {
			id:doc.id, 
			template: doc.template,
			instance: doc.instance,
			replaceNote: doc.replaceNote
		};
		docListCopy.push(docCopy);
   		console.log(doc.id, '=>', doc.data());
  	}
	instanceList.set(docListCopy);
}

onMount(async () => {
	loadInstanceList();
});

function handleOnSubmit() {template,
		replaceNote,
		instance
	console.log("form submitted");
	db.collection("template").add({
	    date,	
		template,
		replaceNote,
		instance
	})
	.then(() => {
		console.log("Document successfully written!");
		loadInstanceList();
	})
	.catch((error) => {
		console.error("Error writing document: ", error);
	});
}

function deleteInstance(id){
	console.log("remove instance id:", id);
	db.collection("template").doc(id).delete().then(() => {
		console.log("Document successfully deleted!");
		loadBathList();
	}).catch((error) => {
		console.error("Error removing document: ", error);
	});
}
</script>

<main>

<h1>Template</h1>
<p>Replace word1 with wrord in template create instance, use template template1</p>
<p>Operation: create instance from template template1 with text, given replace words </p>
<form on:submit|preventDefault={handleOnSubmit}>
	
	<h3>Create Template Instance</h3>
	<label>Date:<input type="date" bind:value={date}/></label>
	<label>Template</label><br/>
	<textarea
	 class="template" bind:value={template}></textarea>

	
	<label>Replace Note</label><br/>
	<textarea  class="replaceNote" bind:value={replaceNote}></textarea>

	<label>Final Text/instance</label>
	<textarea class="instance" bind:value={instance}></textarea>
	<button type="submit">Save</button>
</form>

<hr/>
<h3>List Template Instance</h3>
<div class="instanceList">
	{#each $instanceList as tinstance}
		<div>
			{tinstance.date} <br/>
			Template<br/>
			{tinstance.template}
			<br/>
			Replace Notes<br>
			{tinstance.replaceNotes}
			<br/>
			instance <br/>
			{tinstance.instance}
			<br/>
			<button on:click={deleteInstance(tinstance.id)}>Delete</button>
		</div>
	{/each}
</div>
</main>


<main>

	<h3>Create Template Instance</h3>
	<form >

</form>
</main>

<style>
	textarea { width: 100%; height: 200px; }
	.replaceNote {height:100px;}
</style>