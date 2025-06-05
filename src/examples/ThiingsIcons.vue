<template>
  <ThiingsGrid :grid-size="160">
    <template #default="{ gridIndex }">
      <div class="absolute inset-1 flex items-center justify-center">
        <img
          :draggable="false"
          :src="`/thiings/${images[gridIndex % images.length]}.png`"
        />
      </div>
    </template>
  </ThiingsGrid>
</template>

<script setup lang="ts">
import ThiingsGrid from '../../lib/ThiingsGrid.vue'

const images = ['1', '2', '3', '4', '5', '6', '7', '8', '9']
</script>

<script lang="ts">
export default {
  name: 'ThiingsIcons',
}
</script>
