<script setup>
import { Icon } from '@iconify/vue'
import { computed } from 'vue'

const props = defineProps({
  isDarkMode: Boolean,
  currentPage: Number,
  totalPages: Number,
  goToPage: Function
})

// 计算显示的页码
const visiblePages = computed(() => {
  const pages = []
  const total = props.totalPages
  const current = props.currentPage
  
  if (total <= 7) {
    for (let i = 1; i <= total; i++) pages.push(i)
  } else {
    pages.push(1)
    
    if (current > 3) pages.push('...')
    
    const start = Math.max(2, current - 1)
    const end = Math.min(total - 1, current + 1)
    
    for (let i = start; i <= end; i++) pages.push(i)
    
    if (current < total - 2) pages.push('...')
    
    pages.push(total)
  }
  
  return pages
})
</script>

<template>
  <div class="flex justify-center mt-12 pb-8" v-if="totalPages > 1">
    <div class="flex items-center gap-2">
      <!-- Previous Button -->
      <button
        :class="[
          'w-10 h-10 rounded-xl flex items-center justify-center transition-all duration-200',
          'disabled:opacity-40 disabled:cursor-not-allowed',
          isDarkMode
            ? 'bg-dark-surface hover:bg-dark-border text-gray-400 hover:text-white'
            : 'bg-light-surface hover:bg-light-muted text-gray-500 hover:text-gray-900'
        ]"
        :disabled="currentPage === 1"
        @click="goToPage(currentPage - 1)"
      >
        <Icon icon="mdi:chevron-left" class="text-xl" />
      </button>
      
      <!-- Page Numbers -->
      <template v-for="page in visiblePages" :key="page">
        <span 
          v-if="page === '...'" 
          :class="[
            'w-10 h-10 flex items-center justify-center text-sm',
            isDarkMode ? 'text-gray-600' : 'text-gray-400'
          ]"
        >...</span>
        <button
          v-else
          @click="goToPage(page)"
          :class="[
            'w-10 h-10 rounded-xl flex items-center justify-center text-sm font-medium transition-all duration-200',
            currentPage === page
              ? 'bg-primary-500 text-white shadow-glow'
              : isDarkMode
                ? 'bg-dark-surface hover:bg-dark-border text-gray-400 hover:text-white'
                : 'bg-light-surface hover:bg-light-muted text-gray-500 hover:text-gray-900'
          ]"
        >
          {{ page }}
        </button>
      </template>
      
      <!-- Next Button -->
      <button
        :class="[
          'w-10 h-10 rounded-xl flex items-center justify-center transition-all duration-200',
          'disabled:opacity-40 disabled:cursor-not-allowed',
          isDarkMode
            ? 'bg-dark-surface hover:bg-dark-border text-gray-400 hover:text-white'
            : 'bg-light-surface hover:bg-light-muted text-gray-500 hover:text-gray-900'
        ]"
        :disabled="currentPage === totalPages"
        @click="goToPage(currentPage + 1)"
      >
        <Icon icon="mdi:chevron-right" class="text-xl" />
      </button>
    </div>
  </div>
</template>