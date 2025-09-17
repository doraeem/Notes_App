<template>
    <div id="app">
        <Header @add-note="addNote" @update-search="searchQuery = $event"></Header>


    </div>
</template>

<script setup>
import {ref} from 'vue'
import Header from './components/Header.vue';
</script>