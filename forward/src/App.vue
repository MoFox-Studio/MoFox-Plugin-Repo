<script setup>
import { onMounted, computed } from 'vue'
import { usePlugins } from './composables/usePlugins'
import { useTheme } from './composables/useTheme'
import { useModal } from './composables/useModal'

import TheHeader from './components/TheHeader.vue'
import TheFooter from './components/TheFooter.vue'
import SearchBar from './components/SearchBar.vue'
import FeaturedPlugins from './components/FeaturedPlugins.vue'
import PluginList from './components/PluginList.vue'
import Pagination from './components/Pagination.vue'
import PluginDetailsModal from './components/PluginDetailsModal.vue'
import DownloadModal from './components/DownloadModal.vue'
import NoticeModal from './components/NoticeModal.vue'

const {
  searchQuery,
  currentPage,
  isLoading,
  loadingStatus,
  showPlugins,
  plugins,
  error,
  fetchPluginsData,
  featuredPlugins,
  filteredPlugins,
  paginatedPlugins,
  totalPages,
  goToPage,
  goToRepository,
  sortOrder
} = usePlugins()

const { isDarkMode, toggleTheme } = useTheme()

const {
  showModal,
  selectedPlugin,
  showPluginDetails,
  closeModal,
  showDownloadModal,
  selectedDownloadPlugin,
  openDownloadModal,
  showNotice,
  canCloseNotice,
  dontShowNoticeAgain,
  closeNotice
} = useModal()

onMounted(() => {
  fetchPluginsData()
})
</script>

<template>
  <div :class="[
    'min-h-screen transition-colors duration-300 bg-grid',
    isDarkMode ? 'dark bg-dark-bg' : 'bg-white'
  ]">
    <TheHeader :isDarkMode="isDarkMode" :toggleTheme="toggleTheme" />

    <main class="pt-24 pb-12">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <SearchBar :isDarkMode="isDarkMode" v-model:searchQuery="searchQuery" />
        <FeaturedPlugins 
          :isDarkMode="isDarkMode" 
          :featuredPlugins="featuredPlugins"
          :showPluginDetails="showPluginDetails"
          :openDownloadModal="openDownloadModal"
          :goToRepository="goToRepository"
        />
       <PluginList
         :isDarkMode="isDarkMode"
          :isLoading="isLoading"
          :loadingStatus="loadingStatus"
          :error="error"
          :plugins="plugins"
          :paginatedPlugins="paginatedPlugins"
          :filteredPlugins="filteredPlugins"
          :showPlugins="showPlugins"
          :fetchPlugins="fetchPluginsData"
          :showPluginDetails="showPluginDetails"
          :openDownloadModal="openDownloadModal"
          :goToRepository="goToRepository"
          v-model:sortOrder="sortOrder"
        />
       <Pagination
         :isDarkMode="isDarkMode"
          :currentPage="currentPage"
          :totalPages="totalPages"
          :goToPage="goToPage"
        />
      </div>
    </main>

    <TheFooter :isDarkMode="isDarkMode" />

    <PluginDetailsModal
      :isDarkMode="isDarkMode"
      :showModal="showModal"
      :selectedPlugin="selectedPlugin"
      :closeModal="closeModal"
      :goToRepository="goToRepository"
    />

    <DownloadModal
      :isDarkMode="isDarkMode"
      :showModal="showDownloadModal"
      :plugin="selectedDownloadPlugin"
      :closeModal="() => showDownloadModal = false"
    />

    <NoticeModal
      :isDarkMode="isDarkMode"
      :showNotice="showNotice"
      :canCloseNotice="canCloseNotice"
      :closeNotice="closeNotice"
      v-model:dontShowNoticeAgain="dontShowNoticeAgain"
    />
  </div>
</template>

<style>
/* 弹窗背景动画 */
.modal-backdrop-enter-active,
.modal-backdrop-leave-active {
  transition: all 0.3s ease;
}

.modal-backdrop-enter-from,
.modal-backdrop-leave-to {
  opacity: 0;
}

/* 弹窗内容动画 */
.modal-content-enter-active,
.modal-content-leave-active {
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}

.modal-content-enter-from {
  opacity: 0;
  transform: scale(0.95) translateY(10px);
}

.modal-content-leave-to {
  opacity: 0;
  transform: scale(0.95) translateY(10px);
}

/* 卡片扫描动画 */
.plugin-card {
  opacity: 0;
  transform: translateY(20px);
}

.animate-card-sweep {
  animation: cardSweepReveal 0.6s cubic-bezier(0.16, 1, 0.3, 1) forwards;
  animation-delay: var(--sweep-delay);
}

@keyframes cardSweepReveal {
  0% {
    opacity: 0;
    transform: translateY(20px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 列表扫描动画 */
.plugin-list-item {
  opacity: 0;
  transform: translateX(-20px);
}

.animate-list-sweep {
  animation: listSweepReveal 0.5s cubic-bezier(0.16, 1, 0.3, 1) forwards;
  animation-delay: var(--sweep-delay);
}

@keyframes listSweepReveal {
  0% {
    opacity: 0;
    transform: translateX(-20px);
  }
  100% {
    opacity: 1;
    transform: translateX(0);
  }
}

/* 加载动画 */
.animate-spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* 脉冲动画 */
@keyframes pulse-ring {
  0% {
    transform: scale(0.8);
    opacity: 0.8;
  }
  50% {
    transform: scale(1.2);
    opacity: 0.4;
  }
  100% {
    transform: scale(0.8);
    opacity: 0.8;
  }
}

.animate-pulse-ring {
  animation: pulse-ring 2s ease-in-out infinite;
}

/* 渐变边框动画 */
@keyframes gradient-border {
  0%, 100% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
}

.animate-gradient-border {
  background-size: 200% 200%;
  animation: gradient-border 3s ease infinite;
}
</style>
