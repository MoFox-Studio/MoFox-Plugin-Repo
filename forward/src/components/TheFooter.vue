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
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
      <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
        <!-- Brand -->
        <div class="space-y-4">
          <div class="flex items-center gap-3">
            <div class="w-10 h-10 rounded-xl bg-gradient-to-br from-primary-500 to-primary-600 flex items-center justify-center">
              <Icon icon="mdi:puzzle" class="text-xl text-white" />
            </div>
            <span :class="[
              'text-xl font-bold transition-colors',
              isDarkMode ? 'text-white' : 'text-gray-900'
            ]">MoFox</span>
          </div>
          <p :class="[
            'text-sm leading-relaxed max-w-xs transition-colors',
            isDarkMode ? 'text-gray-500' : 'text-gray-500'
          ]">
            为您的 MoFox Bot 提供强大的插件支持，探索无限可能。
          </p>
        </div>

        <!-- Links -->
        <div class="space-y-4">
          <h4 :class="[
            'text-sm font-semibold uppercase tracking-wider transition-colors',
            isDarkMode ? 'text-gray-400' : 'text-gray-500'
          ]">快速链接</h4>
          <ul class="space-y-3">
            <li>
              <a 
                href="https://github.com/MoFox-Studio/MoFox-Plugin-Repo" 
                target="_blank"
                :class="[
                  'text-sm flex items-center gap-2 transition-colors',
                  isDarkMode 
                    ? 'text-gray-400 hover:text-primary-400' 
                    : 'text-gray-600 hover:text-primary-600'
                ]"
              >
                <Icon icon="mdi:github" />
                GitHub 仓库
              </a>
            </li>
            <li>
              <a 
                href="#" 
                :class="[
                  'text-sm flex items-center gap-2 transition-colors',
                  isDarkMode 
                    ? 'text-gray-400 hover:text-primary-400' 
                    : 'text-gray-600 hover:text-primary-600'
                ]"
              >
                <Icon icon="mdi:file-document-outline" />
                开发文档
              </a>
            </li>
            <li>
              <a 
                href="#" 
                :class="[
                  'text-sm flex items-center gap-2 transition-colors',
                  isDarkMode 
                    ? 'text-gray-400 hover:text-primary-400' 
                    : 'text-gray-600 hover:text-primary-600'
                ]"
              >
                <Icon icon="mdi:help-circle-outline" />
                帮助中心
              </a>
            </li>
          </ul>
        </div>

        <!-- Social -->
        <div class="space-y-4">
          <h4 :class="[
            'text-sm font-semibold uppercase tracking-wider transition-colors',
            isDarkMode ? 'text-gray-400' : 'text-gray-500'
          ]">联系我们</h4>
          <div class="flex items-center gap-3">
            <a 
              href="https://github.com/MoFox-Studio" 
              target="_blank"
              :class="[
                'w-10 h-10 rounded-xl flex items-center justify-center transition-all duration-200',
                'hover:scale-105 active:scale-95',
                isDarkMode 
                  ? 'bg-dark-surface hover:bg-dark-border text-gray-400 hover:text-white' 
                  : 'bg-light-surface hover:bg-light-muted text-gray-500 hover:text-gray-700'
              ]"
            >
              <Icon icon="mdi:github" class="text-xl" />
            </a>
          </div>
        </div>
      </div>

      <!-- Divider -->
      <div class="divider my-8"></div>

      <!-- Copyright -->
      <div class="flex flex-col sm:flex-row items-center justify-between gap-4">
        <p :class="[
          'text-sm transition-colors',
          isDarkMode ? 'text-gray-500' : 'text-gray-500'
        ]">
          © 2025 MoFox Plugin Repository. All rights reserved.
        </p>
        <p :class="[
          'text-sm transition-colors',
          isDarkMode ? 'text-gray-600' : 'text-gray-400'
        ]">
          Made with <Icon icon="mdi:heart" class="text-primary-500 inline" /> by MoFox Studio
        </p>
      </div>
    </div>
  </footer>
</template>