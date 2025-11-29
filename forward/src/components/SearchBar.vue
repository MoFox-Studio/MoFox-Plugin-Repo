<script setup>
import { Icon } from '@iconify/vue'

defineProps({
  isDarkMode: Boolean,
  searchQuery: String
})

const emit = defineEmits(['update:searchQuery'])

const updateSearchQuery = (event) => {
  emit('update:searchQuery', event.target.value)
}
</script>

<template>
  <section class="text-center mb-20">
    <!-- Hero Section -->
    <div class="relative mb-12">
      <!-- Decorative Elements -->
      <div class="absolute -top-10 left-1/2 -translate-x-1/2 w-96 h-96 bg-primary-500/10 rounded-full blur-3xl -z-10"></div>
      
      <div class="inline-flex items-center gap-2 px-4 py-2 rounded-full mb-6 transition-colors"
           :class="isDarkMode ? 'bg-primary-900/30 text-primary-400' : 'bg-primary-50 text-primary-600'">
        <Icon icon="mdi:sparkles" class="text-sm" />
        <span class="text-sm font-medium">发现优质插件</span>
      </div>
      
      <h1 :class="[
        'text-5xl sm:text-6xl font-extrabold tracking-tight mb-6',
        isDarkMode ? 'text-white' : 'text-gray-900'
      ]">
        探索<span class="gradient-text">无限可能</span>
      </h1>
      
      <p :class="[
        'text-lg sm:text-xl max-w-2xl mx-auto leading-relaxed',
        isDarkMode ? 'text-gray-400' : 'text-gray-600'
      ]">
        在插件市场中寻找能增强您 MoFox Bot 体验的强大工具
      </p>
    </div>
    
    <!-- Search Box -->
    <div class="max-w-2xl mx-auto">
      <div class="relative group">
        <!-- Glow Effect -->
        <div :class="[
          'absolute -inset-0.5 rounded-2xl opacity-0 group-focus-within:opacity-100 transition-opacity duration-300 blur',
          'bg-gradient-to-r from-primary-500 to-primary-600'
        ]"></div>
        
        <!-- Search Input Container -->
        <div :class="[
          'relative flex items-center gap-3 px-5 py-4 rounded-2xl border transition-all duration-300',
          isDarkMode
            ? 'bg-dark-surface border-dark-border focus-within:border-primary-500'
            : 'bg-white border-light-border focus-within:border-primary-500 shadow-soft'
        ]">
          <Icon icon="mdi:magnify" :class="[
            'text-2xl flex-shrink-0 transition-colors',
            isDarkMode ? 'text-gray-500' : 'text-gray-400'
          ]" />
          
          <input
            :value="searchQuery"
            @input="updateSearchQuery"
            type="text"
            placeholder="搜索插件名称、作者或关键词..."
            :class="[
              'flex-1 bg-transparent text-lg outline-none transition-colors',
              isDarkMode
                ? 'text-white placeholder-gray-500'
                : 'text-gray-900 placeholder-gray-400'
            ]"
          />
          
          <Transition name="fade">
            <button 
              v-if="searchQuery"
              @click="emit('update:searchQuery', '')"
              :class="[
                'p-1.5 rounded-lg transition-colors flex-shrink-0',
                isDarkMode 
                  ? 'hover:bg-dark-border text-gray-500 hover:text-gray-300' 
                  : 'hover:bg-light-muted text-gray-400 hover:text-gray-600'
              ]"
            >
              <Icon icon="mdi:close" class="text-lg" />
            </button>
          </Transition>
        </div>
      </div>
      
      <!-- Quick Tags -->
      <div class="flex flex-wrap items-center justify-center gap-2 mt-6">
        <span :class="[
          'text-sm',
          isDarkMode ? 'text-gray-500' : 'text-gray-400'
        ]">热门：</span>
        <button 
          v-for="tag in ['工具', '娱乐', '管理', 'AI']" 
          :key="tag"
          @click="emit('update:searchQuery', tag)"
          :class="[
            'px-3 py-1.5 text-sm font-medium rounded-lg transition-all duration-200',
            'hover:scale-105 active:scale-95',
            isDarkMode 
              ? 'bg-dark-surface hover:bg-dark-border text-gray-400 hover:text-white' 
              : 'bg-light-surface hover:bg-light-muted text-gray-600 hover:text-gray-900'
          ]"
        >
          {{ tag }}
        </button>
      </div>
    </div>
  </section>
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>