<template>
  <aside
    class="flex-shrink-0 bg-gray-50 border-r border-gray-300 px-4 py-6 flex flex-col justify-between"
  >
    <div>
      <div class="mb-8">
        <h2
          class="mt-0 text-2xl font-bold bg-gradient-to-r from-blue-600 to-purple-600 bg-clip-text text-transparent tracking-tight"
        >
          vue-thiings-grid
        </h2>
        <div class="text-xs text-gray-500 font-medium tracking-wide uppercase">
          infinite scroll component
        </div>
      </div>
      <nav>
        <div class="list-none p-0 mb-8">
          <div
            v-for="(exampleName, index) in componentNames"
            :key="index"
            :class="[
              'mb-4 cursor-pointer px-3 py-2 rounded transition-colors',
              currentExample === index
                ? 'bg-blue-50 border border-blue-500 font-bold'
                : 'hover:bg-gray-200',
            ]"
            @click="onExampleChange(index)"
          >
            {{ exampleName }}
          </div>
        </div>
      </nav>
    </div>
    <div class="space-y-3">
      <a
        href="https://nuxtsaaskit.com/?ref=vue-thiings-grid"
        target="_blank"
        rel="noopener noreferrer"
        class="block w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-4 rounded-lg text-center text-sm transition-colors duration-200 no-underline"
      >
        Launch your SaaS with Nuxt SaaS Kit
      </a>
      <!-- <a
        href="https://nuxtsaaskit.com/?ref=vue-thiings-grid"
        target="_blank"
        rel="noopener noreferrer"
        class="block w-full bg-[#11B481] hover:bg-[#11B481]/80 text-white font-bold py-3 px-4 rounded-lg text-center text-sm transition-colors duration-200 no-underline"
      >
        Launch your SaaS with Nuxt SaaS Kit
      </a> -->
      <a
        href="https://github.com/peoray/vue-thiings-grid"
        target="_blank"
        rel="noopener noreferrer"
        class="block w-full bg-gray-600 hover:bg-gray-700 text-white font-bold py-3 px-4 rounded-lg text-center text-sm transition-colors duration-200 no-underline"
      >
        view docs
      </a>
    </div>
  </aside>
</template>

<script setup lang="ts">
import { componentNames } from './Playground.vue'

interface SidebarProps {
  currentExample: number
  onExampleChange: (index: number) => void
}

defineProps<SidebarProps>()
</script>
