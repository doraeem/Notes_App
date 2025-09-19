<template>
  <aside class="sidebar">
    <div class="sidebar-header">
      <h3>Notes</h3>
      <button class="add-btn" @click="$emit('add-note')"> + </button>
     </div>
    <ul>
      <li v-for="(note, index) in notes" :key="note.id" draggable="true"
          @dragstart="dragStart(index)" @dragover.prevent @drop="drop(index)"
          >
        
        <span @click="$emit('select-note', note.id)">
          {{ note.title || 'Untitled Note' }}
        </span>
        
        <div class="menu-wrapper">
          <button class="menu-btn" @click="toggleMenu(note.id)">⋮</button>
          <div v-if="openMenuId === note.id" class="menu-dropdown">
            <button @click="$emit('delete-note', note.id)">Delete</button>
            <button @click="$emit('select-note', note.id)">Edit</button>
            <button @click="$emit('rename-note', note.id)">Rename</button>
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

const dragIndex = ref(null)
function dragStart(index) { dragIndex.value = index }
function drop(dropIndex) {
  if (dragIndex.value === null) return
  const newNotes = [...props.notes]
  const moved = newNotes.splice(dragIndex.value, 1)[0]
  newNotes.splice(dropIndex, 0, moved)
  emit('reorder-notes', newNotes)
  dragIndex.value = null
}


</script>