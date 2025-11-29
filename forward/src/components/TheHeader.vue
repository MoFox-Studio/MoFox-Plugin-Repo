<script setup>
import { Icon } from '@iconify/vue'

defineProps({
  isDarkMode: Boolean,
  toggleTheme: Function
})
</script>

<template>
  <header :class="[
    'fixed top-0 left-0 right-0 z-50 transition-all duration-300',
    isDarkMode 
      ? 'bg-dark-bg/80 border-dark-border' 
      : 'bg-white/80 border-light-border',
    'backdrop-blur-xl border-b'
  ]">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="flex items-center justify-between h-16">
        <!-- Logo & Brand -->
        <div class="flex items-center gap-3">
          <div class="relative">
            <div class="w-9 h-9 rounded-xl bg-gradient-to-br from-primary-500 to-primary-600 flex items-center justify-center shadow-glow">
              <Icon icon="mdi:puzzle" class="text-lg text-white" />
            </div>
            <div class="absolute -inset-1 rounded-xl bg-primary-500/20 blur-sm -z-10"></div>
          </div>
          <div class="flex flex-col">
            <span :class="[
              'text-lg font-bold tracking-tight transition-colors',
              isDarkMode ? 'text-white' : 'text-gray-900'
            ]">MoFox</span>
            <span :class="[
              'text-xs font-medium -mt-0.5 transition-colors',
              isDarkMode ? 'text-gray-500' : 'text-gray-400'
            ]">Plugin Repository</span>
          </div>
        </div>
        
        <!-- Right Actions -->
        <div class="flex items-center gap-2">
          <!-- Theme Toggle -->
          <button 
            @click="toggleTheme"
            :class="[
              'relative w-10 h-10 rounded-xl flex items-center justify-center transition-all duration-200',
              'hover:scale-105 active:scale-95',
              isDarkMode 
                ? 'bg-dark-surface hover:bg-dark-border text-yellow-400' 
                : 'bg-light-surface hover:bg-light-muted text-gray-600'
            ]"
            :title="isDarkMode ? '切换到浅色模式' : '切换到深色模式'"
          >
            <Transition name="icon-switch" mode="out-in">
              <Icon v-if="isDarkMode" key="sun" icon="mdi:white-balance-sunny" class="text-xl" />
              <Icon v-else key="moon" icon="mdi:moon-waning-crescent" class="text-xl" />
            </Transition>
          </button>
          
          <!-- GitHub Link -->
          <a 
            href="https://github.com/MoFox-Studio/MoFox-Plugin-Repo" 
            target="_blank" 
            rel="noopener noreferrer"
            :class="[
              'w-10 h-10 rounded-xl flex items-center justify-center transition-all duration-200',
              'hover:scale-105 active:scale-95',
              isDarkMode 
                ? 'bg-white text-black hover:bg-gray-100' 
                : 'bg-gray-900 text-white hover:bg-gray-800'
            ]"
            title="查看 GitHub 仓库"
          >
            <Icon icon="mdi:github" class="text-xl" />
          </a>
        </div>
      </div>
    </div>
  </header>
</template>

<style scoped>
.icon-switch-enter-active,
.icon-switch-leave-active {
  transition: all 0.2s ease;
}

.icon-switch-enter-from {
  opacity: 0;
  transform: rotate(-90deg) scale(0.8);
}

.icon-switch-leave-to {
  opacity: 0;
  transform: rotate(90deg) scale(0.8);
}
</style>