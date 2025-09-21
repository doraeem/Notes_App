<template>
  <aside class="sidebar">
    <div class="sidebar-header">
      <h3>Notes</h3>
      <div class="header-actions">
        <button class="fav-btn"
                :class="{ active: showFavorites }"
                @click="toggleFavorites"
                title="Show Favorites">
          <img src="/icons/fav.png" alt="fav">
        </button>
        <button class="drag-btn" @click="toggleDragMode">
          <img src="/icons/drag.png" alt="drag">
        </button>
        <button class="add-btn" @click="$emit('add-note')"> + </button>
      </div>
    </div>

    <ul>
      <li v-for="(note, index) in filteredNotes"
          :key="note.id"
          :draggable="dragMode"
          @dragstart="dragStart(index)"
          @dragover.prevent
          @drop="drop(index)"
          :class="{active: note.id === activeNoteId}"
           @click="$emit('select-note', note.id)" >
        
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
            @dblclick="startRename(note)">
            {{ note.sidebarTitle || note.title || 'Untitled Note' }}
            <img v-if="note.pinned" src="/icons/pin.png" alt="Pinned" class="pin-icon"/>
            <img v-if="note.favorite" src="/icons/fav-filled.png" alt="Favorite" class="fav-icon"/>
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
import { ref, computed, onMounted, onBeforeUnmount, nextTick } from 'vue'

const props = defineProps({
  notes: { 
    type: Array, 
    required: true
   },
  activeNoteId: { 
    type: [String, Number], 
    default: null }
})

const emit = defineEmits([
  'add-note',
  'select-note',
  'reorder-notes',
  'delete-note',
  'rename-note',
  'pin-note'
])

const openMenuId = ref(null)
const renamingNoteId = ref(null)
const renameInput = ref('')
const dragMode = ref(false)
const dragIndex = ref(null)
const showFavorites = ref(false)

function toggleMenu(noteId) {
  openMenuId.value = openMenuId.value === noteId ? null : noteId
}

function handleClickOutside(event) {
  const menus = document.querySelectorAll('.menu-wrapper')
  let clickedInside = false
  menus.forEach(menu => { 
    if(menu.contains(event.target)) 
    clickedInside = true 
  })
  if(!clickedInside) openMenuId.value = null
}
onMounted(() => 
  document.addEventListener('click', handleClickOutside))

onBeforeUnmount(() =>
 document.removeEventListener('click', handleClickOutside))

function startRename(note) {
  renamingNoteId.value = note.id
  renameInput.value = note.sidebarTitle || note.title || ''
  openMenuId.value = null
}
function confirmRename(noteId) {
  const newTitle = renameInput.value.trim() || 'Untitled Note'
  emit('rename-note', { id: noteId, title: newTitle })
  renamingNoteId.value = null
}
function cancelRename() { 
  renamingNoteId.value = null
 }

function toggleDragMode() { 
  dragMode.value = !dragMode.value 
}
function dragStart(index) { 
  dragIndex.value = index 
}
function drop(dropIndex) {
  if(dragIndex.value === null) return
  const newNotes = [...props.notes]
  const moved = newNotes.splice(dragIndex.value,1)[0]
  newNotes.splice(dropIndex,0,moved)
  emit('reorder-notes', newNotes)
  dragIndex.value = null
}

function toggleFavorites() { 
  showFavorites.value = !showFavorites.value
 }

const filteredNotes = computed(() => {
  let notesList = props.notes
  if (showFavorites.value) 
   notesList = notesList.filter(n => n.favorite)
   return notesList
})


</script>
