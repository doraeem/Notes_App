<template>
    <header class="app-header">
        <div class="left">
            <img src="/icons/logo.png" alt="logo" class="logo-img">
            <h2 class="logo">MyNotes</h2>
        </div>
        <div class="right">
            <input type="text" class="search" v-model="search" placeholder="Search notes..">
        </div>
    </header>
</template>

<script setup>
import {ref, watch} from 'vue'
const search = ref('')
watch(search, (val)=>{
    emit('update-search',val)
})
const emit = defineEmits(['update-search'])

</script>