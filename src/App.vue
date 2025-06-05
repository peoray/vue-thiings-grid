<template>
  <div class="flex h-screen w-screen font-sans">
    <Sidebar
      :current-example="currentExample"
      :on-example-change="setCurrentExample"
    />
    <main class="flex flex-1 flex-col h-screen">
      <div class="flex-[3] relative bg-white border-b border-gray-200">
        <Playground :current-example="currentExample" />
      </div>
      <div class="flex flex-1 min-h-[300px]">
        <SourceCode :current-example="currentExample" />
      </div>
    </main>
    <Toaster position="top-right" rich-colors />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import 'vue-sonner/style.css'
import { Toaster } from 'vue-sonner'
import Sidebar from './components/Sidebar.vue'
import Playground from './components/Playground.vue'
import SourceCode from './components/SourceCode.vue'

const currentExample = ref(0)

const setCurrentExample = (index: number) => {
  currentExample.value = index
}
</script>
