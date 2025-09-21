<template>
  <header class="app-header">
    <div class="left">
      <img src="/icons/logo.png" alt="logo" class="logo-img">
      <h2 class="logo-text">MyNotes</h2>
    </div>
    <div class="right">
      <input type="text" v-model="search" placeholder="Search notes..." class="search" />
    </div>
  </header>
</template>

<script setup>
import { ref, watch } from 'vue'
const search = ref('')
const emit = defineEmits(['update-search'])

watch(search, (val) => {
  emit('update-search', val)
})
</script>
