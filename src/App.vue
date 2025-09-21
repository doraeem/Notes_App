<template>
  <div id="app">
    <Header @update-search="searchQuery = $event" />

    <!-- Mobile sidebar toggle button -->
    <button class="menu-toggle" @click="toggleSidebar">☰</button>

    <!-- Overlay for mobile sidebar -->
    <div class="sidebar-overlay" :class="{ active: showSidebar }" @click="toggleSidebar"></div>

    <div class="main-layout">
      <Sidebar
        :notes="filteredNotes"
        :activeNoteId="activeNoteId"
        @add-note="addNote"
        @select-note="selectNote"
        @reorder-notes="reorderNotes"
        @delete-note="deleteNote"
        @pin-note="togglePinNote"
        @rename-note="renameSidebarTitle"
        :class="{ show: showSidebar }"
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
import { ref, computed, watch } from 'vue'
import Header from './components/Header.vue'
import Sidebar from './components/Sidebar.vue'
import NoteCard from './components/NoteCard.vue'

const notes = ref(JSON.parse(localStorage.getItem('notes')) || [])
const activeNoteId = ref(null)
const searchQuery = ref('')
const showSidebar = ref(false)

function uid() {
  return 'n_' + Math.random().toString(36).substr(2, 9)
}

// Persist notes
watch(notes, (newVal) => {
  localStorage.setItem('notes', JSON.stringify(newVal))
}, { deep: true })

function addNote() {
  const newNote = {
    id: uid(),
    title: '',
    sidebarTitle: '', 
    content: '',
    pinned: false,
    favorite: false,
    stickies: [],
  }
  notes.value.push(newNote)
  activeNoteId.value = newNote.id
  showSidebar.value = false
}

function selectNote(id) {
  activeNoteId.value = id
  showSidebar.value = false
}

function saveNote(updatedNote) {
  const index = notes.value.findIndex(n => n.id === updatedNote.id)
  if (index !== -1) 
    notes.value[index] = { ...updatedNote }
  
  activeNoteId.value = null
}

function deleteNote(id) {
  if (!confirm("Are you sure you want to delete this note?")) return
  notes.value = notes.value.filter(n => n.id !== id)
  if (activeNoteId.value === id) 
    activeNoteId.value = null
}

function reorderNotes(newOrder) {
  notes.value = [...newOrder]
}

function togglePinNote(noteId) {
  const note = notes.value.find(n => n.id === noteId)
  if (!note) return

  if (!note.pinned) note.originalIndex = notes.value.indexOf(note)
  note.pinned = !note.pinned

  if (!note.pinned && note.originalIndex !== undefined) {
    const currentIndex = notes.value.indexOf(note)
    notes.value.splice(currentIndex, 1)
    notes.value.splice(note.originalIndex, 0, note)
    delete note.originalIndex
  } else reorderPinnedNotes()
}

function reorderPinnedNotes() {
  notes.value.sort((a, b) => {
    if (a.pinned && !b.pinned) return -1
    if (!a.pinned && b.pinned) return 1
    return 0
  })
}

const filteredNotes = computed(() => {
  if (!searchQuery.value) return notes.value
  return notes.value.filter(n =>
    (n.title || '').toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

const activeNote = computed(() => notes.value.find(n => n.id === activeNoteId.value))

function goBack() {
  activeNoteId.value = null
}

function renameSidebarTitle({ id, title }) {
  const note = notes.value.find(n => n.id === id)
  if (note) note.sidebarTitle = title
}

// Sidebar toggle
function toggleSidebar() {
  showSidebar.value = !showSidebar.value
}
</script>
