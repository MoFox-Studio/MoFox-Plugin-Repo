<script setup>
import { Icon } from '@iconify/vue'
import { computed } from 'vue'

const props = defineProps({
  isDarkMode: Boolean,
  plugin: Object,
  index: Number,
  showPlugins: Boolean,
  showPluginDetails: Function,
  openDownloadModal: Function,
  goToRepository: Function
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
      'plugin-card group rounded-2xl border transition-all duration-300 cursor-pointer overflow-hidden',
      isDarkMode
        ? 'bg-dark-surface border-dark-border hover:border-primary-800'
        : 'bg-white border-light-border hover:border-primary-200',
      'hover:shadow-soft-lg hover:-translate-y-1',
      { 'animate-card-sweep': showPlugins }
    ]"
    :style="{
      '--sweep-delay': `${Math.floor(index / 3) * 0.1 + (index % 3) * 0.05}s`
    }"
    @click="showPluginDetails(plugin)"
  >
    <!-- Card Content -->
    <div class="p-6">
      <!-- Header -->
      <div class="flex items-start gap-4 mb-4">
        <div class="relative flex-shrink-0">
          <div class="w-12 h-12 rounded-xl bg-gradient-to-br from-primary-500 to-primary-600 flex items-center justify-center shadow-glow">
            <Icon :icon="plugin.icon" class="text-xl text-white" />
          </div>
        </div>
        
        <div class="flex-1 min-w-0">
          <div class="flex items-start justify-between gap-2">
            <h3 :class="[
              'font-bold text-lg truncate transition-colors group-hover:text-primary-500',
              isDarkMode ? 'text-white' : 'text-gray-900'
            ]">{{ plugin.name }}</h3>
            <span :class="[
              'px-2 py-0.5 text-xs font-medium rounded-md flex-shrink-0',
              isDarkMode ? 'bg-dark-muted text-gray-400' : 'bg-light-muted text-gray-500'
            ]">v{{ plugin.version }}</span>
          </div>
          <p :class="[
            'text-sm mt-0.5 transition-colors',
            isDarkMode ? 'text-gray-500' : 'text-gray-500'
          ]">{{ plugin.author }}</p>
        </div>
      </div>
      
      <!-- Description -->
      <p :class="[
        'text-sm line-clamp-2 mb-4 leading-relaxed transition-colors',
        isDarkMode ? 'text-gray-400' : 'text-gray-600'
      ]">{{ plugin.description }}</p>
      
      <!-- Tags -->
      <div class="flex flex-wrap gap-1.5 mb-4">
        <span
          v-for="tag in plugin.tags.slice(0, 3)"
          :key="tag"
          class="tag"
        >
          {{ tag }}
        </span>
      </div>
      
      <!-- Meta Info -->
      <div class="flex items-center text-xs mb-4" :class="isDarkMode ? 'text-gray-500' : 'text-gray-400'">
        <Icon icon="mdi:clock-outline" class="mr-1.5" />
        <span>{{ formattedDate }}</span>
      </div>
    </div>
    
    <!-- Action Bar -->
    <div :class="[
      'flex items-center gap-2 px-6 py-4 border-t transition-colors',
      isDarkMode ? 'bg-dark-muted/50 border-dark-border' : 'bg-light-surface/50 border-light-border'
    ]">
      <button
        @click.stop="openDownloadModal(plugin)"
        :disabled="!plugin.repositoryUrl || plugin.repositoryUrl.trim() === ''"
        :class="[
          'flex-1 px-4 py-2.5 rounded-xl text-sm font-medium transition-all duration-200',
          'flex items-center justify-center gap-1.5',
          plugin.repositoryUrl && plugin.repositoryUrl.trim() !== ''
            ? 'bg-primary-500 hover:bg-primary-600 text-white hover:shadow-glow'
            : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
        ]"
      >
        <Icon icon="mdi:download" />
        下载
      </button>
      <button
        @click.stop="goToRepository(plugin)"
        :disabled="!plugin.repositoryUrl || plugin.repositoryUrl.trim() === ''"
        :class="[
          'px-4 py-2.5 rounded-xl text-sm font-medium transition-all duration-200',
          'flex items-center justify-center gap-1.5',
          plugin.repositoryUrl && plugin.repositoryUrl.trim() !== ''
            ? isDarkMode 
              ? 'bg-dark-border hover:bg-dark-muted text-gray-300 hover:text-white'
              : 'bg-light-muted hover:bg-gray-200 text-gray-600 hover:text-gray-900'
            : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
        ]"
      >
        <Icon icon="mdi:github" />
      </button>
    </div>
  </div>
</template>