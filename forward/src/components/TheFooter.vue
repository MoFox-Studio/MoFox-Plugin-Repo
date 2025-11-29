<script setup>
import { Icon } from '@iconify/vue'

defineProps({
  isDarkMode: Boolean
})
</script>

<template>
  <footer :class="[
    'border-t transition-colors duration-300',
    isDarkMode ? 'bg-dark-bg border-dark-border' : 'bg-white border-light-border'
  ]">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8 sm:py-12">
      <div class="grid grid-cols-2 sm:grid-cols-3 gap-6 sm:gap-8">
        <!-- Brand -->
        <div class="col-span-2 sm:col-span-1 space-y-3 sm:space-y-4">
          <div class="flex items-center gap-2 sm:gap-3">
            <div class="w-8 h-8 sm:w-10 sm:h-10 rounded-lg sm:rounded-xl bg-gradient-to-br from-primary-500 to-primary-600 flex items-center justify-center">
              <Icon icon="mdi:puzzle" class="text-base sm:text-xl text-white" />
            </div>
            <span :class="[
              'text-lg sm:text-xl font-bold transition-colors',
              isDarkMode ? 'text-white' : 'text-gray-900'
            ]">MoFox</span>
          </div>
          <p :class="[
            'text-xs sm:text-sm leading-relaxed max-w-xs transition-colors',
            isDarkMode ? 'text-gray-500' : 'text-gray-500'
          ]">
            为您的 MoFox Bot 提供强大的插件支持，探索无限可能。
          </p>
        </div>

        <!-- Links -->
        <div class="space-y-3 sm:space-y-4">
          <h4 :class="[
            'text-xs sm:text-sm font-semibold uppercase tracking-wider transition-colors',
            isDarkMode ? 'text-gray-400' : 'text-gray-500'
          ]">快速链接</h4>
          <ul class="space-y-2 sm:space-y-3">
            <li>
              <a 
                href="https://github.com/MoFox-Studio/MoFox-Plugin-Repo" 
                target="_blank"
                :class="[
                  'text-xs sm:text-sm flex items-center gap-1.5 sm:gap-2 transition-colors',
                  isDarkMode 
                    ? 'text-gray-400 hover:text-primary-400' 
                    : 'text-gray-600 hover:text-primary-600'
                ]"
              >
                <Icon icon="mdi:github" class="text-sm sm:text-base" />
                GitHub 仓库
              </a>
            </li>
            <li>
              <a 
                href="#" 
                :class="[
                  'text-xs sm:text-sm flex items-center gap-1.5 sm:gap-2 transition-colors',
                  isDarkMode 
                    ? 'text-gray-400 hover:text-primary-400' 
                    : 'text-gray-600 hover:text-primary-600'
                ]"
              >
                <Icon icon="mdi:file-document-outline" class="text-sm sm:text-base" />
                开发文档
              </a>
            </li>
            <li>
              <a 
                href="#" 
                :class="[
                  'text-xs sm:text-sm flex items-center gap-1.5 sm:gap-2 transition-colors',
                  isDarkMode 
                    ? 'text-gray-400 hover:text-primary-400' 
                    : 'text-gray-600 hover:text-primary-600'
                ]"
              >
                <Icon icon="mdi:help-circle-outline" class="text-sm sm:text-base" />
                帮助中心
              </a>
            </li>
          </ul>
        </div>

        <!-- Social -->
        <div class="space-y-3 sm:space-y-4">
          <h4 :class="[
            'text-xs sm:text-sm font-semibold uppercase tracking-wider transition-colors',
            isDarkMode ? 'text-gray-400' : 'text-gray-500'
          ]">联系我们</h4>
          <div class="flex items-center gap-2 sm:gap-3">
            <a 
              href="https://github.com/MoFox-Studio" 
              target="_blank"
              :class="[
                'w-8 h-8 sm:w-10 sm:h-10 rounded-lg sm:rounded-xl flex items-center justify-center transition-all duration-200',
                'hover:scale-105 active:scale-95',
                isDarkMode 
                  ? 'bg-dark-surface hover:bg-dark-border text-gray-400 hover:text-white' 
                  : 'bg-light-surface hover:bg-light-muted text-gray-500 hover:text-gray-700'
              ]"
            >
              <Icon icon="mdi:github" class="text-base sm:text-xl" />
            </a>
          </div>
        </div>
      </div>

      <!-- Divider -->
      <div class="divider my-6 sm:my-8"></div>

      <!-- Copyright -->
      <div class="flex flex-col sm:flex-row items-center justify-between gap-2 sm:gap-4">
        <p :class="[
          'text-[10px] sm:text-sm transition-colors text-center sm:text-left',
          isDarkMode ? 'text-gray-500' : 'text-gray-500'
        ]">
          © 2025 MoFox Plugin Repository. All rights reserved.
        </p>
        <p :class="[
          'text-[10px] sm:text-sm transition-colors',
          isDarkMode ? 'text-gray-600' : 'text-gray-400'
        ]">
          Made with <Icon icon="mdi:heart" class="text-primary-500 inline text-xs sm:text-sm" /> by MoFox Studio
        </p>
      </div>
    </div>
  </footer>
</template>