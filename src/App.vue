<script setup lang="ts">
import Sidebar from './components/Sidebar.vue'
import Playground from './components/Playground.vue'
import SourceCode from './components/SourceCode.vue'

import ThiingsGrid, { type ItemConfig } from '../lib/ThiingsGrid.vue'
</script>

<template>
  <div>
    <h1 class="text-4xl">Hello world</h1>

    <Sidebar />
    <Playground />
    <SourceCode />

    <!-- <ThiingsGrid
      :grid-size="100"
      class="my-grid"
      :initial-position="{ x: 0, y: 0 }"
    >
      <template #default="{ itemConfig }">
        <div class="grid-item">
          {{ itemConfig.gridIndex }}
        </div>
      </template>
    </ThiingsGrid> -->

    <div :style="{ width: '100vw', height: '100vh' }">
      <ThiingsGrid :grid-size="80">
        <template #default="{ itemConfig }">
          <!-- Inline cell rendering -->
          <div class="absolute inset-1 flex items-center justify-center">
            {{ itemConfig.gridIndex }}
          </div>
        </template>
      </ThiingsGrid>
    </div>
  </div>
</template>
