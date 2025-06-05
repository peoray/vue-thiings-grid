<template>
  <div class="flex-1 relative w-full h-full overscroll-none">
    <component :is="CurrentExampleComponent" />
  </div>
</template>

<script lang="ts">
import { computed } from 'vue'
// component imports
import ThiingsIconsExample from '../examples/ThiingsIcons.vue'
import SimpleNumbersExample from '../examples/SimpleNumbers.vue'
import EmojiFunExample from '../examples/EmojiFun.vue'
// source code imports
import thiingsIconsSource from '../examples/ThiingsIcons.vue?raw'
import simpleNumbersSource from '../examples/SimpleNumbers.vue?raw'
import emojiFunSource from '../examples/EmojiFun.vue?raw'

export const exampleComponents = [
  ThiingsIconsExample,
  SimpleNumbersExample,
  EmojiFunExample,
]

export const sourceCodes = [
  thiingsIconsSource,
  simpleNumbersSource,
  emojiFunSource,
]

export const componentNames = computed(() =>
  sourceCodes.map((code) => {
    // Updated regex to match `name: 'ComponentName'` inside `export default`
    const match = code.match(/export\s+default\s+\{\s*name:\s*['"]([^'"]+)['"]/)
    return match?.[1] || 'Unknown'
  })
)
</script>

<script setup lang="ts">
import { computed } from 'vue'

interface PlaygroundProps {
  currentExample: number
}

const props = defineProps<PlaygroundProps>()

const CurrentExampleComponent = computed(
  () => exampleComponents[props.currentExample]
)
</script>
