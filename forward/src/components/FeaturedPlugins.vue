<script setup>
import { Icon } from '@iconify/vue'

defineProps({
  isDarkMode: Boolean,
  featuredPlugins: Array,
  showPluginDetails: Function,
  openDownloadModal: Function,
  goToRepository: Function
})
</script>

<template>
  <section class="mb-10 sm:mb-16" v-if="featuredPlugins && featuredPlugins.length > 0">
    <!-- Section Header -->
    <div class="flex items-center gap-2.5 sm:gap-3 mb-6 sm:mb-8">
      <div class="w-9 h-9 sm:w-10 sm:h-10 rounded-lg sm:rounded-xl bg-gradient-to-br from-amber-400 to-orange-500 flex items-center justify-center">
        <Icon icon="mdi:star" class="text-lg sm:text-xl text-white" />
      </div>
      <div>
        <h2 :class="[
          'text-xl sm:text-2xl lg:text-3xl font-bold tracking-tight',
          isDarkMode ? 'text-white' : 'text-gray-900'
        ]">今日推荐</h2>
        <p :class="[
          'text-xs sm:text-sm',
          isDarkMode ? 'text-gray-500' : 'text-gray-500'
        ]">精选优质插件</p>
      </div>
    </div>
    
    <!-- Featured Grid -->
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-4 sm:gap-6">
      <div
        v-for="(plugin, index) in featuredPlugins"
        :key="`featured-${plugin.id}`"
        :class="[
          'group relative rounded-xl sm:rounded-2xl border overflow-hidden transition-all duration-300 cursor-pointer',
          isDarkMode
            ? 'bg-dark-surface border-dark-border hover:border-primary-800'
            : 'bg-white border-light-border hover:border-primary-200',
          'hover:shadow-soft-lg sm:hover:-translate-y-1'
        ]"
        @click="showPluginDetails(plugin)"
      >
        <!-- Gradient Accent -->
        <div class="absolute top-0 left-0 right-0 h-1 bg-gradient-to-r from-primary-500 to-primary-600"></div>
        
        <div class="p-4 sm:p-6">
          <!-- Header -->
          <div class="flex items-start gap-3 sm:gap-5 mb-4 sm:mb-5">
            <div class="relative flex-shrink-0">
              <div class="w-12 h-12 sm:w-16 sm:h-16 rounded-xl sm:rounded-2xl bg-gradient-to-br from-primary-500 to-primary-600 flex items-center justify-center shadow-glow">
                <Icon :icon="plugin.icon" class="text-xl sm:text-2xl text-white" />
              </div>
              <!-- Featured Badge -->
              <div class="absolute -top-1 -right-1 w-4 h-4 sm:w-5 sm:h-5 rounded-full bg-amber-400 flex items-center justify-center">
                <Icon icon="mdi:star" class="text-[8px] sm:text-xs text-white" />
              </div>
            </div>
            
            <div class="flex-1 min-w-0">
              <div class="flex items-start justify-between gap-2 sm:gap-3">
                <h3 :class="[
                  'text-base sm:text-xl font-bold transition-colors group-hover:text-primary-500 truncate',
                  isDarkMode ? 'text-white' : 'text-gray-900'
                ]">{{ plugin.name }}</h3>
                <span :class="[
                  'px-2 py-0.5 sm:px-2.5 sm:py-1 text-[10px] sm:text-xs font-medium rounded-md sm:rounded-lg flex-shrink-0',
                  isDarkMode ? 'bg-dark-muted text-gray-400' : 'bg-light-muted text-gray-500'
                ]">v{{ plugin.version }}</span>
              </div>
              <p :class="[
                'text-xs sm:text-sm mt-0.5 sm:mt-1',
                isDarkMode ? 'text-gray-500' : 'text-gray-500'
              ]">{{ plugin.author }}</p>
            </div>
          </div>
          
          <!-- Description -->
          <p :class="[
            'text-xs sm:text-sm line-clamp-2 mb-4 sm:mb-5 leading-relaxed',
            isDarkMode ? 'text-gray-400' : 'text-gray-600'
          ]">{{ plugin.description }}</p>
          
          <!-- Tags -->
          <div class="flex flex-wrap gap-1.5 sm:gap-2 mb-4 sm:mb-5">
            <span
              v-for="tag in plugin.tags.slice(0, 3)"
              :key="tag"
              class="tag text-[10px] sm:text-xs"
            >
              {{ tag }}
            </span>
          </div>
          
          <!-- Actions -->
          <div class="flex items-center gap-2 sm:gap-3" @click.stop>
            <button
              @click="openDownloadModal(plugin)"
              :disabled="!plugin.repositoryUrl || plugin.repositoryUrl.trim() === ''"
              :class="[
                'flex-1 px-3 sm:px-5 py-2 sm:py-2.5 rounded-lg sm:rounded-xl text-xs sm:text-sm font-medium transition-all duration-200',
                'flex items-center justify-center gap-1.5 sm:gap-2',
                plugin.repositoryUrl && plugin.repositoryUrl.trim() !== ''
                  ? 'bg-primary-500 hover:bg-primary-600 text-white hover:shadow-glow'
                  : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
              ]"
            >
              <Icon icon="mdi:download" class="text-sm sm:text-base" />
              <span class="hidden xs:inline">下载</span><span class="hidden sm:inline">插件</span>
            </button>
            <button
              @click="goToRepository(plugin)"
              :disabled="!plugin.repositoryUrl || plugin.repositoryUrl.trim() === ''"
              :class="[
                'px-3 sm:px-5 py-2 sm:py-2.5 rounded-lg sm:rounded-xl text-xs sm:text-sm font-medium transition-all duration-200',
                'flex items-center justify-center gap-1.5 sm:gap-2',
                plugin.repositoryUrl && plugin.repositoryUrl.trim() !== ''
                  ? isDarkMode 
                    ? 'bg-dark-border hover:bg-dark-muted text-gray-300 hover:text-white'
                    : 'bg-light-muted hover:bg-gray-200 text-gray-600 hover:text-gray-900'
                  : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
              ]"
            >
              <Icon icon="mdi:github" class="text-sm sm:text-base" />
              <span class="hidden sm:inline">仓库</span>
            </button>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>