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
      'plugin-list-item group rounded-xl sm:rounded-2xl border transition-all duration-300 cursor-pointer overflow-hidden',
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
    <div class="p-4 sm:p-5 md:p-6">
      <!-- Mobile: Stack layout / Desktop: Flex row -->
      <div class="flex flex-col sm:flex-row sm:items-center gap-3 sm:gap-5">
        <!-- Top Row (Mobile) / Left Section (Desktop) -->
        <div class="flex items-center gap-3 sm:gap-4 md:gap-5">
          <!-- Icon -->
          <div class="relative flex-shrink-0">
            <div class="w-11 h-11 sm:w-12 sm:h-12 md:w-14 md:h-14 rounded-lg sm:rounded-xl bg-gradient-to-br from-primary-500 to-primary-600 flex items-center justify-center shadow-glow">
              <Icon :icon="plugin.icon" class="text-lg sm:text-xl md:text-2xl text-white" />
            </div>
          </div>
          
          <!-- Basic Info (Mobile only shows title & author inline) -->
          <div class="flex-1 min-w-0 sm:hidden">
            <div class="flex items-center justify-between gap-2">
              <h3 :class="[
                'text-base font-bold transition-colors group-hover:text-primary-500 truncate',
                isDarkMode ? 'text-white' : 'text-gray-900'
              ]">{{ plugin.name }}</h3>
              <span :class="[
                'px-1.5 py-0.5 text-[10px] font-medium rounded-md flex-shrink-0',
                isDarkMode ? 'bg-dark-muted text-gray-400' : 'bg-light-muted text-gray-500'
              ]">v{{ plugin.version }}</span>
            </div>
            <p :class="[
              'text-xs',
              isDarkMode ? 'text-gray-500' : 'text-gray-500'
            ]">{{ plugin.author }} · {{ formattedDate }}</p>
          </div>
        </div>
        
        <!-- Info (Desktop) -->
        <div class="flex-1 min-w-0 hidden sm:block">
          <div class="flex items-start justify-between gap-4 mb-1.5 md:mb-2">
            <div class="min-w-0">
              <h3 :class="[
                'text-base md:text-lg font-bold transition-colors group-hover:text-primary-500 truncate',
                isDarkMode ? 'text-white' : 'text-gray-900'
              ]">{{ plugin.name }}</h3>
              <p :class="[
                'text-xs md:text-sm',
                isDarkMode ? 'text-gray-500' : 'text-gray-500'
              ]">{{ plugin.author }} · {{ formattedDate }}</p>
            </div>
            
            <!-- Version Badge -->
            <span :class="[
              'px-2 sm:px-2.5 py-0.5 sm:py-1 text-[10px] sm:text-xs font-medium rounded-md sm:rounded-lg flex-shrink-0',
              isDarkMode ? 'bg-dark-muted text-gray-400' : 'bg-light-muted text-gray-500'
            ]">v{{ plugin.version }}</span>
          </div>
          
          <!-- Description -->
          <p :class="[
            'text-xs md:text-sm line-clamp-2 mb-2.5 md:mb-3 leading-relaxed',
            isDarkMode ? 'text-gray-400' : 'text-gray-600'
          ]">{{ plugin.description }}</p>
          
          <!-- Tags & Actions Row -->
          <div class="flex items-center justify-between gap-4">
            <!-- Tags -->
            <div class="flex flex-wrap gap-1 md:gap-1.5 flex-1 min-w-0">
              <span
                v-for="tag in plugin.tags.slice(0, 4)"
                :key="tag"
                class="tag text-[10px] md:text-xs"
              >
                {{ tag }}
              </span>
            </div>
            
            <!-- Actions (Desktop) -->
            <div class="hidden md:flex items-center gap-2 flex-shrink-0" @click.stop>
              <button
                @click="openDownloadModal(plugin)"
                :disabled="!plugin.repositoryUrl || plugin.repositoryUrl.trim() === ''"
                :class="[
                  'px-3 md:px-4 py-1.5 md:py-2 rounded-lg md:rounded-xl text-xs md:text-sm font-medium transition-all duration-200',
                  'flex items-center gap-1 md:gap-1.5',
                  plugin.repositoryUrl && plugin.repositoryUrl.trim() !== ''
                    ? 'bg-primary-500 hover:bg-primary-600 text-white hover:shadow-glow'
                    : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
                ]"
              >
                <Icon icon="mdi:download" class="text-sm md:text-base" />
                <span>下载</span>
              </button>
              <button
                @click="goToRepository(plugin)"
                :disabled="!plugin.repositoryUrl || plugin.repositoryUrl.trim() === ''"
                :class="[
                  'px-3 md:px-4 py-1.5 md:py-2 rounded-lg md:rounded-xl text-xs md:text-sm font-medium transition-all duration-200',
                  'flex items-center gap-1 md:gap-1.5',
                  plugin.repositoryUrl && plugin.repositoryUrl.trim() !== ''
                    ? isDarkMode 
                      ? 'bg-dark-border hover:bg-dark-muted text-gray-300 hover:text-white'
                      : 'bg-light-muted hover:bg-gray-200 text-gray-600 hover:text-gray-900'
                    : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
                ]"
              >
                <Icon icon="mdi:github" class="text-sm md:text-base" />
                <span>仓库</span>
              </button>
            </div>
          </div>
        </div>
        
        <!-- Mobile Only: Description, Tags, Actions -->
        <div class="sm:hidden space-y-2.5">
          <!-- Description -->
          <p :class="[
            'text-xs line-clamp-2 leading-relaxed',
            isDarkMode ? 'text-gray-400' : 'text-gray-600'
          ]">{{ plugin.description }}</p>
          
          <!-- Tags -->
          <div class="flex flex-wrap gap-1">
            <span
              v-for="tag in plugin.tags.slice(0, 3)"
              :key="tag"
              class="tag text-[10px]"
            >
              {{ tag }}
            </span>
          </div>
          
          <!-- Actions -->
          <div class="flex items-center gap-2 pt-1" @click.stop>
            <button
              @click="openDownloadModal(plugin)"
              :disabled="!plugin.repositoryUrl || plugin.repositoryUrl.trim() === ''"
              :class="[
                'flex-1 px-3 py-2 rounded-lg text-xs font-medium transition-all duration-200',
                'flex items-center justify-center gap-1.5',
                plugin.repositoryUrl && plugin.repositoryUrl.trim() !== ''
                  ? 'bg-primary-500 hover:bg-primary-600 text-white'
                  : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
              ]"
            >
              <Icon icon="mdi:download" class="text-sm" />
              下载
            </button>
            <button
              @click="goToRepository(plugin)"
              :disabled="!plugin.repositoryUrl || plugin.repositoryUrl.trim() === ''"
              :class="[
                'px-3 py-2 rounded-lg text-xs font-medium transition-all duration-200',
                'flex items-center justify-center gap-1.5',
                plugin.repositoryUrl && plugin.repositoryUrl.trim() !== ''
                  ? isDarkMode 
                    ? 'bg-dark-border hover:bg-dark-muted text-gray-300'
                    : 'bg-light-muted hover:bg-gray-200 text-gray-600'
                  : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
              ]"
            >
              <Icon icon="mdi:github" class="text-sm" />
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>