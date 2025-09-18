<template>
    <header class="app-header">
        <div class="left">
            <h2 class="logo">MyNotes</h2>
        </div>
        <div class="right">
            <input type="text" class="search" v-model="search" placeholder="Search notes..">
            <button class="add-btn" @click="$emit('add-note')">+Add Note</button>
        </div>
    </header>
</template>

<script setup>
import {ref, watch} from 'vue'
const search = ref('')
watch(search, (val)=>{
    emit('update-search',val)
})
const emit = defineEmits(['add-note','update-search'])

</script>