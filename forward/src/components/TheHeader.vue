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
    <div class="max-w-7xl mx-auto px-3 sm:px-6 lg:px-8">
      <div class="flex items-center justify-between h-14 sm:h-16">
        <!-- Logo & Brand -->
        <div class="flex items-center gap-2 sm:gap-3">
          <div class="relative">
            <div class="w-8 h-8 sm:w-9 sm:h-9 rounded-lg sm:rounded-xl bg-gradient-to-br from-primary-500 to-primary-600 flex items-center justify-center shadow-glow">
              <Icon icon="mdi:puzzle" class="text-base sm:text-lg text-white" />
            </div>
            <div class="absolute -inset-1 rounded-xl bg-primary-500/20 blur-sm -z-10 hidden sm:block"></div>
          </div>
          <div class="flex flex-col">
            <span :class="[
              'text-base sm:text-lg font-bold tracking-tight transition-colors',
              isDarkMode ? 'text-white' : 'text-gray-900'
            ]">MoFox</span>
            <span :class="[
              'text-[10px] sm:text-xs font-medium -mt-0.5 transition-colors hidden xs:block',
              isDarkMode ? 'text-gray-500' : 'text-gray-400'
            ]">Plugin Repository</span>
          </div>
        </div>
        
        <!-- Right Actions -->
        <div class="flex items-center gap-1.5 sm:gap-2">
          <!-- Theme Toggle -->
          <button 
            @click="toggleTheme"
            :class="[
              'relative w-9 h-9 sm:w-10 sm:h-10 rounded-lg sm:rounded-xl flex items-center justify-center transition-all duration-200',
              'hover:scale-105 active:scale-95',
              isDarkMode 
                ? 'bg-dark-surface hover:bg-dark-border text-yellow-400' 
                : 'bg-light-surface hover:bg-light-muted text-gray-600'
            ]"
            :title="isDarkMode ? '切换到浅色模式' : '切换到深色模式'"
          >
            <Transition name="icon-switch" mode="out-in">
              <Icon v-if="isDarkMode" key="sun" icon="mdi:white-balance-sunny" class="text-lg sm:text-xl" />
              <Icon v-else key="moon" icon="mdi:moon-waning-crescent" class="text-lg sm:text-xl" />
            </Transition>
          </button>
          
          <!-- GitHub Link -->
          <a 
            href="https://github.com/MoFox-Studio/MoFox-Plugin-Repo" 
            target="_blank" 
            rel="noopener noreferrer"
            :class="[
              'w-9 h-9 sm:w-10 sm:h-10 rounded-lg sm:rounded-xl flex items-center justify-center transition-all duration-200',
              'hover:scale-105 active:scale-95',
              isDarkMode 
                ? 'bg-white text-black hover:bg-gray-100' 
                : 'bg-gray-900 text-white hover:bg-gray-800'
            ]"
            title="查看 GitHub 仓库"
          >
            <Icon icon="mdi:github" class="text-lg sm:text-xl" />
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