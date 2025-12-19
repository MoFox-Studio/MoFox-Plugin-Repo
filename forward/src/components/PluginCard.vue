<script setup>
import { Icon } from '@iconify/vue'
import { computed } from 'vue'

const props = defineProps({
  isDarkMode: Boolean,
  plugin: Object,
  index: Number,
  showPlugins: Boolean,
  showPluginDetails: Function
})

const formattedDate = computed(() => {
  if (!props.plugin.createdAt) {
    return '未知'
  }
  return new Date(props.plugin.createdAt).toLocaleDateString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit'
  })
})
</script>

<template>
  <div
    :class="[
      'plugin-card group rounded-xl sm:rounded-2xl border transition-all duration-300 cursor-pointer overflow-hidden flex flex-col h-full',
      isDarkMode
        ? 'bg-dark-surface border-dark-border hover:border-primary-800'
        : 'bg-white border-light-border hover:border-primary-200',
      'hover:shadow-soft-lg sm:hover:-translate-y-1',
      { 'animate-card-sweep': showPlugins }
    ]"
    :style="{
      '--sweep-delay': `${Math.floor(index / 3) * 0.1 + (index % 3) * 0.05}s`
    }"
    @click="showPluginDetails(plugin)"
  >
    <!-- Card Content -->
    <div class="p-4 sm:p-6 flex-1">
      <!-- Header -->
      <div class="flex items-start gap-3 sm:gap-4 mb-3 sm:mb-4">
        <div class="relative flex-shrink-0">
          <div class="w-10 h-10 sm:w-12 sm:h-12 rounded-lg sm:rounded-xl bg-gradient-to-br from-primary-500 to-primary-600 flex items-center justify-center shadow-glow">
            <Icon :icon="plugin.icon" class="text-lg sm:text-xl text-white" />
          </div>
        </div>
        
        <div class="flex-1 min-w-0">
          <div class="flex items-start justify-between gap-2">
            <h3 :class="[
              'font-bold text-base sm:text-lg truncate transition-colors group-hover:text-primary-500',
              isDarkMode ? 'text-white' : 'text-gray-900'
            ]">{{ plugin.name }}</h3>
            <span :class="[
              'px-1.5 sm:px-2 py-0.5 text-[10px] sm:text-xs font-medium rounded-md flex-shrink-0',
              isDarkMode ? 'bg-dark-muted text-gray-400' : 'bg-light-muted text-gray-500'
            ]">v{{ plugin.version }}</span>
          </div>
          <p :class="[
            'text-xs sm:text-sm mt-0.5 transition-colors',
            isDarkMode ? 'text-gray-500' : 'text-gray-500'
          ]">{{ plugin.author }}</p>
        </div>
      </div>
      
      <!-- Description -->
      <p :class="[
        'text-xs sm:text-sm line-clamp-2 mb-3 sm:mb-4 leading-relaxed transition-colors',
        isDarkMode ? 'text-gray-400' : 'text-gray-600'
      ]">{{ plugin.description }}</p>
      
      <!-- Tags -->
      <div class="flex flex-wrap gap-1 sm:gap-1.5 mb-3 sm:mb-4">
        <span
          v-for="tag in plugin.tags.slice(0, 3)"
          :key="tag"
          class="tag text-[10px] sm:text-xs"
        >
          {{ tag }}
        </span>
      </div>
      
      <!-- Meta Info -->
      <div class="flex items-center text-[10px] sm:text-xs" :class="isDarkMode ? 'text-gray-500' : 'text-gray-400'">
        <Icon icon="mdi:clock-outline" class="mr-1 sm:mr-1.5 text-xs sm:text-sm" />
        <span>{{ formattedDate }}</span>
      </div>
    </div>
    
    <!-- Action Bar -->
    <div :class="[
      'flex items-center justify-center px-4 sm:px-6 py-3 sm:py-4 border-t transition-colors',
      isDarkMode ? 'bg-dark-muted/50 border-dark-border' : 'bg-light-surface/50 border-light-border'
    ]">
      <span :class="[
        'text-xs sm:text-sm font-medium flex items-center gap-1.5 transition-colors group-hover:text-primary-500',
        isDarkMode ? 'text-gray-400' : 'text-gray-500'
      ]">
        <Icon icon="mdi:information-outline" class="text-sm sm:text-base" />
        点击查看详情
      </span>
    </div>
  </div>
</template>