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
        
        <button class="delete-btn" @click="$emit('delete-note', note.id)">
          🗑
        </button>
      </li>
    </ul>
  </aside>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  notes: Array,
  activeNoteId: String
})

const emit = defineEmits(['add-note','select-note', 'reorder-notes', 'delete-note'])

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