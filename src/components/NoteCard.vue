<template>
  <div class="note-card">
    <div class="note-card-actions">

      <button class="action-btn" @click="goBack" title="Go Back">
        <img src="/icons/back.png" alt="back" class="icon" /> 
      </button>
      <button class="action-btn" @click="addImage" title="Add Image">
        <img src="/icons/image.png" alt="Add" class="icon" /> 
      </button>
      <button class="action-btn" @click="makeSticky" title="Sticky Note">
        <img src="/icons/sticky.png" alt="Sticky" class="icon" /> 
      </button>

      <button v-show="true" class="action-btn" @click="saveAsPDF" title="Save as PDF">
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
    

     <div class="note-card-body">
         
     <div
       class="note-content"
        contenteditable="true"
        v-html="note.content"
        @input="updateContent"
         placeholder="Write your note here..."
></div>


     <div class="sticky-container">
       <div 
         v-for="(sticky, index) in note.stickies" 
         :key="index" 
         class="sticky-note" 
         :style="{ backgroundColor: sticky.color }"
    >
      <textarea 
        v-model="sticky.text"
        placeholder="Write something..."
        class="sticky-text"
        @input="onEdit"
      ></textarea>
    </div>
  </div>
</div>

    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import jsPDF from 'jspdf'

const props= defineProps({
    note:Object
})
const emit = defineEmits(['save-note'])

const firstOpen = ref(true)
const editing = ref(false)
const noteSaved = ref(false)

function addImage() {
  
  const fileInput = document.createElement('input')
  fileInput.type = 'file'
  fileInput.accept = 'image/*' 

  fileInput.onchange = () => {
    const file = fileInput.files[0]
    if (!file) return

    const reader = new FileReader()
    reader.onload = (e) => {
      const imageUrl = e.target.result

      props.note.content += `\n<img src="${imageUrl}" alt="Image" style="max-width:100%; margin-top:10px;" />\n`
    }
    reader.readAsDataURL(file)
  }
  fileInput.click()
}
//buttons change
watch(() => props.note.id, (id) => {
  if (id) {
    noteSaved.value = true
    firstOpen.value = false
    editing.value = false
  } else {
    noteSaved.value = false
    firstOpen.value = true
    editing.value = false
  }
})

function updateContent(event) {
  props.note.content = event.target.innerHTML
}

// Show Save button
const showSave = computed(() => firstOpen.value || editing.value)
const showRefresh = computed(() => {
  if (!noteSaved.value && firstOpen.value) return true
  if (noteSaved.value && editing.value) return true
  return false
})

// Triggered on any input edit
function onEdit() { 
  editing.value = true
 }


const stickyColors = ['#fffa65', '#ff9ff3', '#feca57', '#48dbfb', '#1dd1a1']

function makeSticky() {
  if (!props.note.stickies) props.note.stickies = []

  props.note.stickies.push({
    text: '',
    color: stickyColors[Math.floor(Math.random() * stickyColors.length)]
  })
  editing.value = true
}

function refreshNote() {
  props.note.title = ''
  props.note.content = ''
  if (props.note.stickies) props.note.stickies = []
  editing.value = false
  firstOpen.value = false
}

// Save as PDF


function saveAsPDF() {
  const doc = new jsPDF()

  doc.setFontSize(16)
  doc.text(`Title: ${props.note.title}`, 10, 20)

  doc.setFontSize(12)
  doc.text(`Content:`, 10, 40)
  doc.text(props.note.content, 10, 50)

  if (props.note.stickies && props.note.stickies.length > 0) {
    let y = 70
    doc.setFontSize(12)
    doc.text('Sticky Notes:', 10, y)
    y += 10

    props.note.stickies.forEach((s, i) => {
      doc.setFillColor(s.color || '#fffa65') 
      doc.rect(10, y, 80, 30, 'F') 

      doc.setTextColor(0, 0, 0) 
      doc.text(s.text || '', 14, y + 10, { maxWidth: 72 })

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
    noteSaved.value = true
    firstOpen.value = false
  } else firstOpen.value = true
})
</script>