<template>
  <section
    class="flex-[2] p-4 border-r border-gray-200 bg-gray-50 flex flex-col"
  >
    <div class="mb-4">
      <h3 class="m-0 text-lg font-semibold">{{ currentComponentName }}</h3>
    </div>
    <pre
      class="bg-gray-100 p-4 rounded-lg text-xs leading-relaxed overflow-auto flex-1 border border-gray-200 font-mono mb-4"
    >
      <code>{{ currentSourceCode }}</code>
    </pre>
    <div class="flex gap-3">
      <button
        @click="handleCopy"
        class="flex-1 px-4 py-2.5 bg-blue-600 text-white border-none rounded-lg cursor-pointer text-sm font-medium hover:bg-blue-700 transition-colors duration-200"
      >
        Copy Example
      </button>
      <button
        @click="handleCopyThiingsGrid"
        class="flex-1 px-4 py-2.5 bg-gray-600 text-white border-none rounded-lg cursor-pointer text-sm font-medium hover:bg-gray-700 transition-colors duration-200"
      >
        Copy ThiingsGrid.vue
      </button>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { componentNames, sourceCodes } from './Playground.vue'
import { toast } from 'vue-sonner'
import thingsGridSource from '../../lib/ThiingsGrid.vue?raw'

interface SourceCodeProps {
  currentExample: number
}

const props = defineProps<SourceCodeProps>()

const currentComponentName = computed(
  () => componentNames.value[props.currentExample]
)

const currentSourceCode = computed(() => sourceCodes[props.currentExample])

const handleCopy = async () => {
  try {
    await navigator.clipboard.writeText(currentSourceCode.value)
    toast.success('Example code copied to clipboard!')
  } catch (error) {
    toast.error('Failed to copy example code.')
    console.error('Failed to copy:', error)
  }
}

const handleCopyThiingsGrid = async () => {
  try {
    await navigator.clipboard.writeText(thingsGridSource)
    toast.success('ThiingsGrid component copied to clipboard!')
  } catch (error) {
    console.error('Failed to copy:', error)
  }
}
</script>
