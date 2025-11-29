<script setup>
import { ref, nextTick } from 'vue'
import { Icon } from '@iconify/vue'
import PluginCard from './PluginCard.vue'
import PluginListItem from './PluginListItem.vue'

const props = defineProps({
  isDarkMode: Boolean,
  isLoading: Boolean,
  loadingStatus: String,
  error: String,
  plugins: Array,
  paginatedPlugins: Array,
  filteredPlugins: Array,
  showPlugins: Boolean,
  fetchPlugins: Function,
  showPluginDetails: Function,
  openDownloadModal: Function,
  goToRepository: Function,
  sortOrder: String
})

const viewMode = ref('grid')

const switchViewMode = async (mode) => {
  if (mode === viewMode.value) return
  
  const internalShowPlugins = ref(props.showPlugins)
  internalShowPlugins.value = false
  viewMode.value = mode
  
  await nextTick()
  setTimeout(() => {
    internalShowPlugins.value = true
  }, 50)
}
</script>

<template>
  <section class="mt-10 sm:mt-16">
    <!-- Section Header -->
    <div class="flex flex-col gap-3 sm:gap-4 mb-6 sm:mb-8">
      <!-- Title Row -->
      <div class="flex flex-wrap items-center justify-between gap-3 sm:gap-4">
        <div class="flex items-center gap-3 sm:gap-4">
          <h2 :class="[
            'text-xl sm:text-2xl lg:text-3xl font-bold tracking-tight',
            isDarkMode ? 'text-white' : 'text-gray-900'
          ]">全部插件</h2>
          
          <!-- Sort Dropdown -->
          <div class="relative">
            <select
              :value="sortOrder"
              @change="$emit('update:sortOrder', $event.target.value)"
              :class="[
                'appearance-none pl-3 sm:pl-4 pr-8 sm:pr-10 py-1.5 sm:py-2 text-xs sm:text-sm font-medium rounded-lg sm:rounded-xl border transition-all cursor-pointer',
                isDarkMode
                  ? 'bg-dark-surface border-dark-border text-gray-300 hover:border-primary-700'
                  : 'bg-white border-light-border text-gray-600 hover:border-primary-300'
              ]"
            >
              <option value="newest">从新到旧</option>
              <option value="oldest">从旧到新</option>
            </select>
            <Icon icon="mdi:chevron-down" :class="[
              'absolute right-2 sm:right-3 top-1/2 -translate-y-1/2 pointer-events-none text-sm sm:text-base',
              isDarkMode ? 'text-gray-500' : 'text-gray-400'
            ]" />
          </div>
        </div>
        
        <!-- View Mode Toggle - Desktop visible, Mobile condensed -->
        <div :class="[
          'flex p-0.5 sm:p-1 rounded-lg sm:rounded-xl',
          isDarkMode ? 'bg-dark-surface' : 'bg-light-surface'
        ]">
          <button
            @click="switchViewMode('grid')"
            :class="[
              'px-2.5 sm:px-4 py-1.5 sm:py-2 rounded-md sm:rounded-lg text-xs sm:text-sm font-medium transition-all duration-200 flex items-center gap-1.5 sm:gap-2',
              viewMode === 'grid'
                ? 'bg-primary-500 text-white shadow-glow'
                : isDarkMode
                  ? 'text-gray-400 hover:text-white'
                  : 'text-gray-500 hover:text-gray-900'
            ]"
          >
            <Icon icon="mdi:view-grid" class="text-sm sm:text-base" />
            <span class="hidden sm:inline">卡片</span>
          </button>
          <button
            @click="switchViewMode('list')"
            :class="[
              'px-2.5 sm:px-4 py-1.5 sm:py-2 rounded-md sm:rounded-lg text-xs sm:text-sm font-medium transition-all duration-200 flex items-center gap-1.5 sm:gap-2',
              viewMode === 'list'
                ? 'bg-primary-500 text-white shadow-glow'
                : isDarkMode
                  ? 'text-gray-400 hover:text-white'
                  : 'text-gray-500 hover:text-gray-900'
            ]"
          >
            <Icon icon="mdi:view-list" class="text-sm sm:text-base" />
            <span class="hidden sm:inline">列表</span>
          </button>
        </div>
      </div>
      
      <!-- Plugin Count - Separate row on mobile -->
      <div :class="[
        'flex items-center gap-1.5 sm:gap-2 px-3 sm:px-4 py-1.5 sm:py-2 rounded-lg sm:rounded-xl text-xs sm:text-sm font-medium w-fit',
        isDarkMode ? 'bg-dark-surface text-gray-400' : 'bg-light-surface text-gray-500'
      ]">
        <Icon icon="mdi:package-variant" class="text-sm sm:text-base" />
        <span>共 <span :class="isDarkMode ? 'text-primary-400' : 'text-primary-600'">{{ filteredPlugins.length }}</span> 个插件</span>
      </div>
    </div>
    
    <!-- Loading State -->
    <div v-if="isLoading" class="flex justify-center items-center py-20 sm:py-32">
      <div class="flex flex-col items-center gap-3 sm:gap-4">
        <div class="relative">
          <div class="w-12 h-12 sm:w-16 sm:h-16 rounded-full border-4 border-primary-500/20 border-t-primary-500 animate-spin"></div>
          <div class="absolute inset-0 flex items-center justify-center">
            <Icon icon="mdi:puzzle" :class="[
              'text-base sm:text-xl',
              isDarkMode ? 'text-primary-400' : 'text-primary-500'
            ]" />
          </div>
        </div>
        <p :class="[
          'text-sm sm:text-lg font-medium',
          isDarkMode ? 'text-gray-300' : 'text-gray-600'
        ]">{{ loadingStatus }}</p>
      </div>
    </div>
    
    <!-- Error State -->
    <div v-else-if="error" class="flex justify-center items-center py-20 sm:py-32 px-4">
      <div :class="[
        'flex flex-col items-center gap-4 sm:gap-6 p-5 sm:p-8 rounded-xl sm:rounded-2xl border max-w-md text-center w-full',
        isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
      ]">
        <div class="w-12 h-12 sm:w-16 sm:h-16 rounded-full bg-red-500/10 flex items-center justify-center">
          <Icon icon="mdi:alert-circle" class="text-2xl sm:text-3xl text-red-500" />
        </div>
        <div>
          <h3 :class="[
            'text-base sm:text-lg font-semibold mb-1.5 sm:mb-2',
            isDarkMode ? 'text-white' : 'text-gray-900'
          ]">加载失败</h3>
          <p :class="[
            'text-xs sm:text-sm',
            isDarkMode ? 'text-gray-400' : 'text-gray-500'
          ]">{{ error }}</p>
        </div>
        <button
          @click="fetchPlugins"
          class="btn-primary text-sm sm:text-base"
        >
          <Icon icon="mdi:refresh" class="mr-1.5 sm:mr-2" />
          重新加载
        </button>
      </div>
    </div>
    
    <!-- Empty State -->
    <div v-else-if="!error && plugins.length === 0" class="flex justify-center items-center py-20 sm:py-32 px-4">
      <div :class="[
        'flex flex-col items-center gap-4 sm:gap-6 p-5 sm:p-8 rounded-xl sm:rounded-2xl border max-w-md text-center w-full',
        isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
      ]">
        <div class="w-12 h-12 sm:w-16 sm:h-16 rounded-full bg-primary-500/10 flex items-center justify-center">
          <Icon icon="mdi:package-variant" class="text-2xl sm:text-3xl text-primary-500" />
        </div>
        <div>
          <h3 :class="[
            'text-base sm:text-lg font-semibold mb-1.5 sm:mb-2',
            isDarkMode ? 'text-white' : 'text-gray-900'
          ]">暂无插件</h3>
          <p :class="[
            'text-xs sm:text-sm',
            isDarkMode ? 'text-gray-400' : 'text-gray-500'
          ]">还没有任何插件数据</p>
        </div>
      </div>
    </div>
    
    <!-- Grid View -->
    <div v-else-if="!error && viewMode === 'grid'">
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4 sm:gap-6">
        <PluginCard
          v-for="(plugin, index) in paginatedPlugins"
          :key="plugin.id"
          :isDarkMode="isDarkMode"
          :plugin="plugin"
          :index="index"
          :showPlugins="showPlugins"
          :showPluginDetails="showPluginDetails"
          :openDownloadModal="openDownloadModal"
          :goToRepository="goToRepository"
        />
      </div>
    </div>
    
    <!-- List View -->
    <div v-else-if="!error">
      <div class="space-y-3 sm:space-y-4">
        <PluginListItem
          v-for="(plugin, index) in paginatedPlugins"
          :key="plugin.id"
          :isDarkMode="isDarkMode"
          :plugin="plugin"
          :index="index"
          :showPlugins="showPlugins"
          :showPluginDetails="showPluginDetails"
          :openDownloadModal="openDownloadModal"
          :goToRepository="goToRepository"
        />
      </div>
    </div>
  </section>
</template>