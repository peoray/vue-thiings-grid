<template>
  <button @click="getCurrentPosition">Log Current Position</button>

  <ThiingsGrid :gridSize="80" ref="gridRef" class="bg-gray-100">
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
import { onMounted, ref } from 'vue'
import ThiingsGrid from '../../lib/ThiingsGrid.vue'

const gridRef = ref<InstanceType<typeof ThiingsGrid> | null>(null)

const getCurrentPosition = () => {
  if (gridRef.value) {
    const position = gridRef.value.publicGetCurrentPosition()
    console.log('Current position:', position)
  }
}

onMounted(() => {
  console.log(getCurrentPosition())
})
</script>

<script lang="ts">
export default {
  name: 'SimpleNumbers',
}
</script>
