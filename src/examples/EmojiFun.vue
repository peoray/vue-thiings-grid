<template>
  <ThiingsGrid :grid-size="120" :initial-position="{ x: 0, y: 0 }">
    <template #default="{ gridIndex }">
      <div
        class="absolute inset-2 flex items-center justify-center bg-gradient-to-br from-gray-100 to-white border border-gray-300 rounded-full text-2xl shadow-md"
      >
        {{ emojis[gridIndex % emojis.length] }}
      </div>
    </template>
  </ThiingsGrid>
</template>

<script setup lang="ts">
import ThiingsGrid from '../../lib/ThiingsGrid.vue' // Adjust path as needed

const emojis = ['🎨', '🚀', '🌟', '🎪', '🎭', '🎨', '🎸', '🎯', '🎲', '🎳']
</script>

<script lang="ts">
export default {
  name: 'EmojiFun',
}
</script>
