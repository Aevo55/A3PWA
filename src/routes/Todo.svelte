<script>
import {onMount} from 'svelte'

let emptyList = []

let newItem = $state("")

let listList = $state([])

let currList = $state(null)

onMount(() => {
     load()
})

function load(){
     let storedList = localStorage.getItem('storedList')
     if (storedList) {
          listList = (JSON.parse(storedList))
          for(let i = 0; i < listList.length; i++){
               listList[i].edit = false
               for(let j = 0; j < listList[i].items.length; j++){
                    listList[i].items[j].edit = false
               }
          }
     }else{
          listList = emptyList
     }
}

function goBack(){
     currList = null
}

function saveAll() {
     return localStorage.setItem('storedList', JSON.stringify(listList))
}

function nuke(){
     localStorage.setItem('storedList', JSON.stringify(emptyList))
     load();
}

function trashList(){
     let currListIndex = listList.indexOf(currList)
     listList.splice(currListIndex, 1)
     if(listList.length == 0){
     }
     currList = null
}

function newList(){
     listList.push({
          text: "New List " + (listList.length + 1),
          items: [],
          edit: false,
          color: "#FFECA1"
     })
}

function editListName(index){
     let oldEdit = listList[index].edit
     for(let i = 0; i < listList.length; i++){
          listList[i].edit = false;
     }
     listList[index].edit = !oldEdit;
}



function openList(index){
     currList = listList[index]
}

function createItem(){
     currList.items.push({
          text: "New Item " + (currList.items.length + 1),
          done: false,
          edit: false
     })
}

function textBoxEnterCheck(event){
     if(event.key === 'Enter'){
          clearEdits()
     }
}

function clearEdits(){
     for(let i = 0; i < listList.length; i++){
          listList[i].edit = false;
          for(let j = 0; j < listList[i].items.length; j++){
               listList[i].items[j].edit = false;
          }
     }
}

function allDone(index){
     if(listList[index].items.length == 0){
          return false;
     }
     for(let i = 0; i < listList[index].items.length; i++){
          if(!listList[index].items[i].done){
               return false;
          }
     }
     return true;
}

function removeItem(index){
     currList.items = currList.items.toSpliced(index, 1)
}

function toggleDone(index){
     currList.items[index].done = !currList.items[index].done
}

function toggleEdit(index){
     let oldEdit = currList.items[index].edit
     for(let i = 0; i < currList.items.length; i++){
          currList.items[i].edit = false;
     }
     currList.items[index].edit = !oldEdit;
}

$inspect(listList)


</script>

<div class="_col">
     <div class="_row buttonBar">
          <button class="headerButton" type="button" title="Save" onclick={() => saveAll()}>&#128190;</button>
          <button class="headerButton" type="button" title="Undo" onclick={() => load()}>↻</button>
          <button class="headerButton" type="button" title="Kaboom" onclick={() => nuke()}>💣</button>
     </div>
     {#if currList == null}
     <div class="listHolder">
          {#each listList as item, index}
          <div class="listItem _row _jbetween ">
               {#if item.edit}
               <input class="rowTextInput" type="text" autofocus bind:value={item.text} onkeydown={(event) => textBoxEnterCheck(event)}>
               {:else}
               <div class={allDone(index) ? "rowText rowTextDone" : "rowText"}>{item.text}</div>
               {/if}
               <div class="_row _jend rowButtons">
                    {#if !item.edit}
                    <button class="rowButton" type="button" title="Rename" onclick={() => editListName(index)}>✎</button>
                    {:else}
                    <button class="rowButton" type="button" title="Rename" onclick={() => editListName(index)}>&#10003;</button>
                    {/if}
                    <button class="rowButton" type="button" title="Open" onclick={() => openList(index)}>&#128193;</button>
                    <button class="rowButton" type="button" title="Remove" onclick={() => trashList(index)}>🗑</button>
               </div>
          </div>
          {/each}
          <div class="_row listItem _jend _100">
               <button class="addButton" type="button" title="New List" onclick={() => newList()}>+</button>
          </div>
     </div> 
     {:else}

     <div class="listHolder">
          <div class="_row _acenter _jbetween listTitle"> 
               <h2>{currList.text}</h2>
               <button class="backButton" type="button" title="Open" onclick={() => goBack()}>⮐</button>
          </div>
         
          {#each currList.items as item, index}
          <div class="listItem _row _jbetween">
               {#if item.edit}
               <input class="rowTextInput" type="text" autofocus bind:value={item.text} onkeydown={(event) => textBoxEnterCheck(event)}>
               {:else}
               <div class={item.done ? "rowText rowTextDone" : "rowText"}>{item.text}</div>
               {/if}
               <div class="_row _jend rowButtons">
                    {#if !item.edit}
                    <button class="rowButton" type="button" title="Edit" onclick={() => toggleEdit(index)}>✎</button>
                    {:else}
                    <button class="rowButton" type="button" title="Edit" onclick={() => toggleEdit(index)}>&#10003;</button>
                    {/if}
                    <button class="rowButton" type="button" title="Done" onclick={() => toggleDone(index)}>&#9745;</button>
                    <button class="rowButton" type="button" title="Remove" onclick={() => removeItem(index)}>&#128465;</button>
               </div>
          </div>
          {/each}
          <div class="_row listItem _jend _100">
               <button class="addButton" type="button" title="New Item" onclick={() => createItem()}>+</button>
          </div>
     </div>
     {/if}
</div>



<style>
     .listHolder{
          margin: 20px;
          margin-top: 15px;
          padding: 10px 30px 0px 30px;
          width: 80vw;
          background-color: #f9ff3e;
          display: flex;
          flex-direction: column;
          align-items: flex-start;
          box-shadow: 5px 5px;
          border-color: black;
          border-radius: 1px;
          border-width: 1px;
          border-style: solid;
          position: relative;
     }

     .listItem{
          position: relative;
          height: 55px;
          align-items: center;
          width: 100%;
     }

     .listItem:not(:first-child)::before {
          position: absolute;
          display: block;
          content: "";
          width: calc(80vw + 10px);
          top: 0px;
          left: -5px;
          height: 1.5px;
          background-color: grey; 
     }

     .rowText{
          margin-left: 0px;
          font-size: 1.4em;
     }

     .rowTextDone{
          text-decoration: line-through;
     }

     .rowTextInput{
          margin-left: 0px;
          font-size: 1.4em;
          font-family: "Mulish", sans-serif;
          font-optical-sizing: auto;
          font-weight: 5vw;
          font-style: normal;
     }

     button{
          margin: 0px 0.3vw;
     }

     .headerButton{
          width: 60px;
          height: 40px;
          font-size: 1.5em;
          margin-left: 0px;
          margin-right: 15px;
          padding: 0px;
     }

     .rowButton{
          width: 40px;
          font-weight: 800;
          font-size: 1em;
     }

     .addButton{
          width: 130px;
          height: 30px;
          font-weight: 800;
          font-size: 1em;
     }

     .rowButtons{
          height: 30px;
     }

     .backButton{
          height: 80%;
          width: 130px;
          font-size: large;
     }

     .buttonBar{
          margin-left: 20px;
          padding: 0px;
     }

     .listTitle{
          width: 100%;
          height: 50px;
          margin: 5px 0px;
          padding: 0px;
     }

     .listTitle::before{
          content:"";
          width: 130px;
          display: block;
     }

     h2{
          margin: 0px;
          padding: 0px;
          font-size: 1.5em;
          text-wrap: nowrap;
     }
</style>