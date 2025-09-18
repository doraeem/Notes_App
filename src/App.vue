<template>
  <div id="app">
    <Header @add-note="addNote" @update-search="searchQuery = $event" />

    <div class="main-layout">
      <Sidebar
        :notes="filteredNotes"
        :activeNoteId="activeNoteId"
        @select-note="selectNote"
        @reorder-notes="reorderNotes"
        @delete-note="deleteNote"
      />

      <div class="editor">
        
        <NoteCard
           v-if="activeNote"
           :note="activeNote"
           @save-note="saveNote"
        />
         <p v-else>Select a note or add a new one.</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import Header from './components/Header.vue'
import Sidebar from './components/Sidebar.vue'
import NoteCard from "./components/NoteCard.vue";


const notes = ref([])
const activeNoteId = ref(null)
const searchQuery = ref('')

function uid() {
  return 'n_' + Math.random().toString(36).substr(2, 9)
}


function addNote() {
  const newNote = { id: uid(), title: '', content: '' }
  notes.value.push(newNote)
  activeNoteId.value = newNote.id
}

function selectNote(id) {
  activeNoteId.value = id
}
function saveNote(updatedNote) {
  const index = notes.value.findIndex(n => n.id === updatedNote.id)
  if (index !== -1){
     notes.value[index] = updatedNote
  }
 
  localStorage.setItem('notes', JSON.stringify(notes.value))

  activeNoteId.value = null
}

function reorderNotes(newOrder) {
  notes.value = newOrder
}

function deleteNote(id) {
  notes.value = notes.value.filter(n => n.id !== id)
  if (activeNoteId.value === id) activeNoteId.value = null
}


const filteredNotes = computed(() => {
  if (!searchQuery.value) return notes.value
  return notes.value.filter(n =>
    (n.title || '').toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

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