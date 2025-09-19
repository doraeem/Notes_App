<template>
    <div class="note-card">

        <div class="note-card-actions">
             <button class="action-btn" @click="goBack" title="Go Back">
               <img src="/icons/back.png" alt="back" class="icon" /> 
             </button>
             <button class="action-btn" @click="addImage" title="Add Image">
                 <img src="/icons/image.png" alt="add" class="icon">
             </button>
             <button class="action-btn" @click="makeSticky" title="Sticky Note">
               <img :class="{ active: note.sticky }" src="/icons/sticky.png" alt="Sticky" class="icon" />
             </button>
            <button class="action-btn" @click="saveAsFile" title="Save as File">
              <img src="/icons/saveAs.png" alt="Save" class="icon" /> 
            </button>
           <button class="action-btn" @click="saveNote" title="Save">
             <img src="/icons/save.png" alt="save-note" class="icon">
           </button>
           <button class="action-btn" @click="refreshNote" title="Refresh">
             <img src="/icons/refresh.png" alt="refresh" class="icon">
           </button>
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
           class="note-content"
          :class="{ sticky: note.sticky }">
           </textarea>

        </div>
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
  emit('toggle-sticky', props.note.id)  
}

function refreshNote(){
    if(!props.note) return
    props.note.title = ''
    props.note.content = ''
}
function goBack(){
    emit('go-back')
}
function saveNote(){
    emit('save-note',props.note)
}
</script>