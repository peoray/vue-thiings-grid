<template>
  <ThiingsGrid :gridSize="80" class="bg-gray-100">
    <template #default="{ gridIndex }">
      <div
        class="absolute inset-1 flex items-center justify-center bg-blue-50 border border-blue-500 rounded text-sm font-bold text-blue-800"
      >
        {{ gridIndex }}
      </div>
    </template>
  </ThiingsGrid>
</template>

<script setup lang="ts">
import ThiingsGrid from '../../lib/ThiingsGrid.vue'
</script>

<script lang="ts">
export default {
  name: 'SimpleNumbers',
}
</script>
