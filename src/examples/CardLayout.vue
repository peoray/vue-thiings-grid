<template>
  <ThiingsGrid :grid-size="150">
    <template #default="{ gridIndex, position, isMoving }">
      <div
        :class="[
          'absolute',
          'inset-1',
          'flex',
          'flex-col',
          'items-center',
          'justify-center',
          'bg-white',
          'border',
          'border-gray-200',
          'rounded-xl',
          'p-2',
          'text-xs',
          'text-gray-800',
          'transition-shadow',
          isMoving ? 'shadow-xl' : 'shadow-md',
        ]"
      >
        <div class="text-base font-bold mb-1">#{{ gridIndex }}</div>
        <div class="text-[10px] text-gray-500">
          {{ position.x }}, {{ position.y }}
        </div>
      </div>
    </template>
  </ThiingsGrid>
</template>

<script setup lang="ts">
import ThiingsGrid, { type ItemConfig } from '../../lib/ThiingsGrid.vue'
</script>

<script lang="ts">
export default {
  name: 'CardLayout',
}
</script>
