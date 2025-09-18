<template>
  <aside class="sidebar">
    <h3>Notes</h3>
    <ul>
      <li v-for="(note, index) in notes" :key="note.id" draggable="true"
          @dragstart="dragStart(index)" @dragover.prevent @drop="drop(index)"
          :class="{active: note.id === activeNoteId}">
        
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

const emit = defineEmits(['select-note', 'reorder-notes', 'delete-note'])


</script>