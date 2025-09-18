<template>
    <div id="app">
        <Header @add-note="addNote" @update-search="searchQuery = $event"></Header>
        <div class="main-layout">
         
            <div class="editor">
                <NoteCard
                v-if="activeNote"
                :note="activeNote"
                @save-note="save-note"/>
                <p v-else>Select or add a new note.</p>

            </div>

        </div>
    </div>
</template>

<script setup>
import {ref,computed} from 'vue'
import Header from './components/Header.vue';
import NoteCard from './components/NoteCard.vue';



const notes = ref([])
const activeNoteId = ref(null)

function uid() { 
    return 'n_' + Math.random().toString(36).substr(2, 9)
     }
     function addNote(){
        const newNote = { id: uid(), title: '', content: '' }
       notes.value.push(newNote)
       activeNoteId.value = newNote.id
}
const activeNote = computed({
  get() {
    return notes.value.find(n => n.id === activeNoteId.value)
  },
  set(updatedNote) {
    const index = notes.value.findIndex(n => n.id === updatedNote.id)
    if (index !== -1) notes.value[index] = updatedNote
  }
})
     
</script>