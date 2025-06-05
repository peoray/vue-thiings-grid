<template>
  <ThiingsGrid :grid-size="100">
    <template #default="{ gridIndex }">
      <div
        :class="`absolute inset-0 flex items-center justify-center ${getColorClass(
          gridIndex
        )} text-xs font-bold text-gray-800 shadow-sm`"
      >
        {{ gridIndex }}
      </div>
    </template>
  </ThiingsGrid>
</template>

<script setup lang="ts">
import ThiingsGrid, { type ItemConfig } from '../../lib/ThiingsGrid.vue'

const colors = [
  'bg-red-300',
  'bg-green-300',
  'bg-blue-300',
  'bg-yellow-300',
  'bg-pink-300',
  'bg-cyan-300',
]

const getColorClass = (gridIndex: number): string => {
  return colors[gridIndex % colors.length]
}
</script>

<script lang="ts">
export default {
  name: 'ColorfulGrid',
}
</script>
