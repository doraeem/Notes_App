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

      <button v-show="true" class="action-btn" @click="saveAsFile" title="Save as File">
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
         
        <textarea 
          v-model="note.content" 
          placeholder="Write your note here..." 
          class="note-content" 
          @input="onEdit"
        ></textarea>

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