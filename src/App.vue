<template>
  <div id="app">
    <Header @update-search="searchQuery = $event" />

    <div class="main-layout">
      <Sidebar
        :notes="filteredNotes"
        :activeNoteId="activeNoteId"
        @add-note="addNote" 
        @select-note="selectNote"
        @reorder-notes="reorderNotes"
        @delete-note="deleteNote"
        @pin-note="togglePinNote"
      />

      <div class="editor">
        <NoteCard
           v-if="activeNote"
           :note="activeNote"
           @save-note="saveNote" 
           @go-back="goBack"
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

const notes = ref(JSON.parse(localStorage.getItem('notes') || '[]'))
const activeNoteId = ref(null)
const searchQuery = ref('')

function uid() {
  return 'n_' + Math.random().toString(36).substr(2, 9)
}

function addNote() {
  const newNote = { 
    id: uid(), 
    title: '', 
    content: '', 
    pinned: false, 
    favorite: false, 
    stickies: [] 
  }
  notes.value.push(newNote)
  activeNoteId.value = newNote.id
  persistNotes()
}

function selectNote(id) {
  activeNoteId.value = id
}

function saveNote(updatedNote) {
  const index = notes.value.findIndex(n => n.id === updatedNote.id)
  if (index !== -1) 
    notes.value[index] = { ...updatedNote }
  persistNotes()
  activeNoteId.value = null
}

function deleteNote(id) {
  const confirmed = confirm("Are you sure you want to delete this note?");
  if (!confirmed) return
  notes.value = notes.value.filter(n => n.id !== id)
  if (activeNoteId.value === id) 
    activeNoteId.value = null
  persistNotes()
}

function reorderNotes(newOrder) {
  notes.value = [...newOrder]
  persistNotes()
}

function togglePinNote(noteId) {
  const note = notes.value.find(n => n.id === noteId)
  if (!note) return
  note.pinned = !note.pinned
  reorderPinnedNotes()
  persistNotes()
}

function reorderPinnedNotes() {
  notes.value.sort((a, b) => {
    if (a.pinned && !b.pinned) return -1
    if (!a.pinned && b.pinned) return 1
    return 0
  })
  notes.value = [...notes.value] 
}

const filteredNotes = computed(() => {
  let list = notes.value
  if (searchQuery.value) {
    list = list.filter(n => (n.title || '').toLowerCase().includes(searchQuery.value.toLowerCase()))
  }
  return list
})

const activeNote = computed({
  get() { 
    return notes.value.find(n => n.id === activeNoteId.value) 
  },
  set(updatedNote) {
    const idx = notes.value.findIndex(n => n.id === updatedNote.id)
    if (idx !== -1) 
      notes.value[idx] = { ...updatedNote }
    persistNotes()
  }
})

function goBack() { 
  activeNoteId.value = null 
}

function persistNotes() {
  localStorage.setItem('notes', JSON.stringify(notes.value))
}
</script>
