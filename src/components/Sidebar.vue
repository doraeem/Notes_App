<template>
  <aside class="sidebar">
    <div class="sidebar-header">
      <h3>Notes</h3>
     <div class="header-actions">
        <button class="drag-btn"  @click="toggleDragMode"> 
          <img src="/icons/drag.png" alt="drag">
        </button>
       <button class="add-btn" @click="$emit('add-note')"> + </button>
      </div>
    </div>

    <ul>
     <li v-for="(note, index) in notes" 
        :key="note.id" 
        :draggable="dragMode"
        @dragstart="dragStart(index)" 
        @dragover.prevent 
        @drop="drop(index)"
       :class="{active: note.id === activeNoteId}">
        
       <div class="note-title-wrapper">
          <input 
            v-if="renamingNoteId === note.id"
            v-model="renameInput"
            @keyup.enter="confirmRename(note.id)"
            @blur="cancelRename"
            class="rename-input"
            autofocus
          />
          <span 
             v-else 
              @click="$emit('select-note', note.id)" 
              @dblclick="startRename(note)" >
              {{ note.title || 'Untitled Note' }}
              <img v-if="note.pinned" src="/icons/pin.png" alt="Pinned" class="pin-icon"/>
          </span>
        </div>
        
        <div class="menu-wrapper">
          <button class="menu-btn" @click="toggleMenu(note.id)">⋮</button>
          <div v-if="openMenuId === note.id" class="menu-dropdown">
            <button @click="$emit('delete-note', note.id)">Delete</button>
            <button @click="$emit('select-note', note.id)">Edit</button>
            <button @click="startRename(note)">Rename</button>
             <button @click="$emit('pin-note', note.id)">
               {{ note.pinned ? 'Unpin Note' : 'Pin Note' }}
            </button>
          </div>
        </div>
      </li>
    </ul>
  </aside>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const props = defineProps({
  notes: Array,
  activeNoteId: String
})

const emit = defineEmits(['add-note','select-note', 'reorder-notes', 'delete-note'])

const openMenuId = ref(null)

function toggleMenu(noteId) {
  openMenuId.value = openMenuId.value === noteId ? null : noteId
}
function handleClickOutside(event) {
  const menus = document.querySelectorAll('.menu-wrapper')
  let clickedInside = false
  menus.forEach(menu => {
    if (menu.contains(event.target)) clickedInside = true
  })
     if (!clickedInside) openMenuId.value = null
}

onMounted(() => {
  document.addEventListener('click', handleClickOutside)
})

onBeforeUnmount(() => {
  document.removeEventListener('click', handleClickOutside)
})

//Rename 

const renamingNoteId = ref(null)
const renameInput = ref('')

function startRename(note) {
  renamingNoteId.value = note.id
  renameInput.value = note.title || ''
  openMenuId.value = null
}
function confirmRename(noteId) {
  const note = props.notes.find(n => n.id === noteId)
  if (note) {
    note.title = renameInput.value.trim() || 'Untitled Note'
    emit('rename-note', note)
  }
  renamingNoteId.value = null
}
function cancelRename() {
  renamingNoteId.value = null
}


// drag & drop
const dragMode = ref(false)
const dragIndex = ref(null)

function toggleDragMode() {
  dragMode.value = !dragMode.value
}

function dragStart(index) {
  dragIndex.value = index
}

function drop(dropIndex) {
  if (dragIndex.value === null) return
  const newNotes = [...props.notes]
  const moved = newNotes.splice(dragIndex.value, 1)[0]
  newNotes.splice(dropIndex, 0, moved)
  emit('reorder-notes', newNotes)
  dragIndex.value = null
}


</script>