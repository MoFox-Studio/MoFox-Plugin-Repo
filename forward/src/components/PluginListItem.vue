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
      'plugin-list-item group rounded-2xl border transition-all duration-300 cursor-pointer overflow-hidden',
      isDarkMode
        ? 'bg-dark-surface border-dark-border hover:border-primary-800'
        : 'bg-white border-light-border hover:border-primary-200',
      'hover:shadow-soft-lg',
      { 'animate-list-sweep': showPlugins }
    ]"
    :style="{
      '--sweep-delay': `${index * 0.05}s`
    }"
    @click="showPluginDetails(plugin)"
  >
    <div class="p-5 sm:p-6">
      <div class="flex items-center gap-5">
        <!-- Icon -->
        <div class="relative flex-shrink-0">
          <div class="w-14 h-14 rounded-xl bg-gradient-to-br from-primary-500 to-primary-600 flex items-center justify-center shadow-glow">
            <Icon :icon="plugin.icon" class="text-2xl text-white" />
          </div>
        </div>
        
        <!-- Info -->
        <div class="flex-1 min-w-0">
          <div class="flex items-start justify-between gap-4 mb-2">
            <div class="min-w-0">
              <h3 :class="[
                'text-lg font-bold transition-colors group-hover:text-primary-500 truncate',
                isDarkMode ? 'text-white' : 'text-gray-900'
              ]">{{ plugin.name }}</h3>
              <p :class="[
                'text-sm',
                isDarkMode ? 'text-gray-500' : 'text-gray-500'
              ]">{{ plugin.author }} · {{ formattedDate }}</p>
            </div>
            
            <!-- Version Badge -->
            <span :class="[
              'px-2.5 py-1 text-xs font-medium rounded-lg flex-shrink-0',
              isDarkMode ? 'bg-dark-muted text-gray-400' : 'bg-light-muted text-gray-500'
            ]">v{{ plugin.version }}</span>
          </div>
          
          <!-- Description -->
          <p :class="[
            'text-sm line-clamp-2 mb-3 leading-relaxed',
            isDarkMode ? 'text-gray-400' : 'text-gray-600'
          ]">{{ plugin.description }}</p>
          
          <!-- Tags & Actions Row -->
          <div class="flex items-center justify-between gap-4">
            <!-- Tags -->
            <div class="flex flex-wrap gap-1.5 flex-1 min-w-0">
              <span
                v-for="tag in plugin.tags.slice(0, 4)"
                :key="tag"
                class="tag"
              >
                {{ tag }}
              </span>
            </div>
            
            <!-- Actions -->
            <div class="flex items-center gap-2 flex-shrink-0" @click.stop>
              <button
                @click="openDownloadModal(plugin)"
                :disabled="!plugin.repositoryUrl || plugin.repositoryUrl.trim() === ''"
                :class="[
                  'px-4 py-2 rounded-xl text-sm font-medium transition-all duration-200',
                  'flex items-center gap-1.5',
                  plugin.repositoryUrl && plugin.repositoryUrl.trim() !== ''
                    ? 'bg-primary-500 hover:bg-primary-600 text-white hover:shadow-glow'
                    : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
                ]"
              >
                <Icon icon="mdi:download" />
                <span class="hidden sm:inline">下载</span>
              </button>
              <button
                @click="goToRepository(plugin)"
                :disabled="!plugin.repositoryUrl || plugin.repositoryUrl.trim() === ''"
                :class="[
                  'px-4 py-2 rounded-xl text-sm font-medium transition-all duration-200',
                  'flex items-center gap-1.5',
                  plugin.repositoryUrl && plugin.repositoryUrl.trim() !== ''
                    ? isDarkMode 
                      ? 'bg-dark-border hover:bg-dark-muted text-gray-300 hover:text-white'
                      : 'bg-light-muted hover:bg-gray-200 text-gray-600 hover:text-gray-900'
                    : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
                ]"
              >
                <Icon icon="mdi:github" />
                <span class="hidden sm:inline">仓库</span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>