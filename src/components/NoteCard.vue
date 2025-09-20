<template>
  <div class="note-card">
    <div class="note-card-actions">
      <button class="action-btn" @click="goBack" title="Go Back">
        <img src="/icons/back.png" alt="back" class="icon" /> 
      </button>

      <button class="action-btn" @click="makeSticky" title="Sticky Note">
        <img src="/icons/sticky.png" alt="Sticky" class="icon" /> 
      </button>

      <button class="action-btn" @click="toggleFavorite" title="Favorite Note">
        <img :src="note.favorite ? '/icons/fav-filled.png' : '/icons/fav.png'" class="icon" /> 
      </button>

      <button class="action-btn" @click="saveAsPDF" title="Save as PDF">
        <img src="/icons/saveAs.png" alt="Save" class="icon" /> 
      </button>

      <button v-show="showSave" class="action-btn" @click="saveNote" title="Save">
        <img src="/icons/save.png" alt="save-note" class="icon">
      </button>

      <button v-show="showRefresh" class="action-btn" @click="refreshNote" title="Refresh">
        <img src="/icons/refresh.png" alt="refresh" class="icon">
      </button>
    </div>

    <div class="note-card-body">
      <input type="text" v-model="note.title" placeholder="Note Title" class="note-title" @input="onEdit" />
      <textarea v-model="note.content" placeholder="Write your note here..." class="note-content" @input="onEdit"></textarea>

      <div class="sticky-container">
        <div v-for="(sticky, index) in note.stickies" 
             :key="index" class="sticky-note" 
             :style="{ backgroundColor: sticky.color }">
          <textarea v-model="sticky.text" placeholder="Write something..." class="sticky-text" @input="onEdit"></textarea>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import jsPDF from 'jspdf'

const props = defineProps({ note: Object })
const emit = defineEmits(['save-note','go-back','update-favorite'])

const firstOpen = ref(true)
const editing = ref(false)
const noteSaved = ref(false)

const showSave = computed(() =>
 firstOpen.value || editing.value)
const showRefresh = computed(() =>
 (noteSaved.value && editing.value) || (!noteSaved.value && firstOpen.value))

function onEdit() { editing.value = true }

const stickyColors = ['#fffa65', '#ff9ff3', '#feca57', '#48dbfb', '#1dd1a1']
function makeSticky() {
  if (!props.note.stickies) props.note.stickies = []
  props.note.stickies.push({ text: '', color: stickyColors[Math.floor(Math.random() * stickyColors.length)] })
  editing.value = true
}

function toggleFavorite() {
  props.note.favorite = !props.note.favorite
  emit('update-favorite', props.note)
  editing.value = true
}

function refreshNote() {
  props.note.title = ''
  props.note.content = ''
  if (props.note.stickies) props.note.stickies = []
  editing.value = false
  firstOpen.value = false
}

function saveAsPDF() {
  const doc = new jsPDF()
  doc.setFontSize(16)
  doc.text(`Title: ${props.note.title}`, 10, 20)
  doc.setFontSize(12)
  doc.text(`Content:`, 10, 40)
  doc.text(props.note.content, 10, 50)

  if (props.note.stickies && props.note.stickies.length) {
    let y = 70
    doc.text('Sticky Notes:', 10, y)
    y += 10
    props.note.stickies.forEach(s => {
      doc.setFillColor(s.color || '#fffa65')
      doc.rect(10, y, 80, 30, 'F')
      doc.setTextColor(0,0,0)
      doc.text(s.text || '', 14, y+10, { maxWidth: 72 })
      y += 40
    })
  }
  doc.save(`${props.note.title || 'note'}.pdf`)
}

function saveNote() {
  emit('save-note', props.note)
  editing.value = false
  firstOpen.value = false
  noteSaved.value = true
}

function goBack() { 
  emit('go-back') 
}

onMounted(() => {
  if (props.note.id) { 
    noteSaved.value = true; 
    firstOpen.value = false 
  }
})
</script>
