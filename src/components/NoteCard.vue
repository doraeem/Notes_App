<template>
    <div class="note-card">

        <div class="note-card-header">
            <span class="menu-dots" @click="menuOpen = !menuOpen">⋮</span>
            <div v-if="menuOpen" class="menu-options">
               <button @click="addImage">Add Image</button>
               <button @click="makeSticky">Sticky Note</button>
               <button @click="saveAsFile">Save as File</button>
            </div>
        </div>
        <div class="note-card-body">
            <input type="text"
            v-model="note.title"
            placeholder="Note Title"
            class="note-title"
            />
            <textarea 
            v-model="note.content"
            placeholder="Write your note here.."
            class="note-content">
            </textarea>
        </div>
        <button class="save-btn" @click="saveNote">Save</button>
        <button class="refresh-btn" @click="refreshNote">Refresh</button>

    </div>
    </template>

<script setup>
import {ref} from 'vue'

const props= defineProps({
    note:Object
})
const emit = defineEmits(['save-note'])

const menuOpen = ref(false)

function addImage() {
  alert('Add Image feature placeholder')
}

function makeSticky() {
  alert('Sticky Note feature placeholder')
}
function refreshNote(){
    if(!props.note) return
    props.note.title = ''
    props.note.content = ''
}

function saveNote(){
    emit('save-note',props.note)
}
</script>