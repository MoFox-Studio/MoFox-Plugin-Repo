<script setup>
import { Icon } from '@iconify/vue'

defineProps({
  isDarkMode: Boolean,
  showModal: Boolean,
  selectedPlugin: Object,
  closeModal: Function,
  goToRepository: Function
})
</script>

<template>
  <Transition name="modal-backdrop">
    <div v-if="showModal && selectedPlugin" class="fixed inset-0 z-50 flex items-center justify-center p-2 sm:p-4">
      <!-- Backdrop -->
      <div 
        class="fixed inset-0 bg-black/50 backdrop-blur-sm"
        @click="closeModal"
      ></div>
      
      <!-- Modal Content -->
      <Transition name="modal-content" appear>
        <div v-if="showModal && selectedPlugin" :class="[
          'relative w-full max-w-2xl max-h-[95vh] sm:max-h-[90vh] overflow-hidden rounded-xl sm:rounded-2xl border shadow-2xl',
          isDarkMode ? 'bg-dark-bg border-dark-border' : 'bg-white border-light-border'
        ]">
          <!-- Close Button -->
          <button 
            @click="closeModal"
            :class="[
              'absolute top-2 right-2 sm:top-4 sm:right-4 z-10 w-8 h-8 sm:w-10 sm:h-10 rounded-lg sm:rounded-xl flex items-center justify-center transition-colors',
              isDarkMode 
                ? 'bg-dark-surface hover:bg-dark-border text-gray-400 hover:text-white' 
                : 'bg-light-surface hover:bg-light-muted text-gray-500 hover:text-gray-900'
            ]"
          >
            <Icon icon="mdi:close" class="text-lg sm:text-xl" />
          </button>
        
          <div class="overflow-y-auto max-h-[95vh] sm:max-h-[90vh]">
            <!-- Header -->
            <div :class="[
              'p-4 sm:p-6 border-b',
              isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
            ]">
              <div class="flex items-start gap-3 sm:gap-5">
                <div class="w-12 h-12 sm:w-16 sm:h-16 rounded-xl sm:rounded-2xl bg-gradient-to-br from-primary-500 to-primary-600 flex items-center justify-center shadow-glow flex-shrink-0">
                  <Icon :icon="selectedPlugin.icon" class="text-xl sm:text-2xl text-white" />
                </div>
                <div class="flex-1 min-w-0 pr-8 sm:pr-10">
                  <h2 :class="[
                    'text-lg sm:text-2xl font-bold mb-0.5 sm:mb-1 truncate',
                    isDarkMode ? 'text-white' : 'text-gray-900'
                  ]">{{ selectedPlugin.name }}</h2>
                  <div class="flex items-center gap-2 sm:gap-3 flex-wrap">
                    <span :class="[
                      'text-xs sm:text-sm flex items-center gap-1',
                      isDarkMode ? 'text-gray-400' : 'text-gray-500'
                    ]">
                      <Icon icon="mdi:account" class="text-xs sm:text-sm" />
                      {{ selectedPlugin.author }}
                    </span>
                    <a 
                      v-if="selectedPlugin.authorUrl" 
                      :href="selectedPlugin.authorUrl" 
                      target="_blank" 
                      rel="noopener noreferrer"
                      class="text-primary-500 hover:text-primary-600 text-xs sm:text-sm flex items-center gap-1 transition-colors"
                    >
                      <Icon icon="mdi:open-in-new" class="text-[10px] sm:text-xs" />
                      主页
                    </a>
                  </div>
                  <div class="flex items-center gap-3 sm:gap-4 mt-1.5 sm:mt-2">
                    <span :class="[
                      'px-2 py-0.5 sm:px-2.5 sm:py-1 text-[10px] sm:text-xs font-medium rounded-md sm:rounded-lg',
                      isDarkMode ? 'bg-dark-border text-gray-400' : 'bg-light-muted text-gray-500'
                    ]">v{{ selectedPlugin.version }}</span>
                    <span :class="[
                      'text-[10px] sm:text-xs flex items-center gap-1',
                      isDarkMode ? 'text-gray-500' : 'text-gray-400'
                    ]">
                      <Icon icon="mdi:license" class="text-xs sm:text-sm" />
                      {{ selectedPlugin.license }}
                    </span>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- Body -->
            <div class="p-4 sm:p-6 space-y-4 sm:space-y-6">
              <!-- Description -->
              <div>
                <h3 :class="[
                  'text-xs sm:text-sm font-semibold uppercase tracking-wider mb-2 sm:mb-3 flex items-center gap-1.5 sm:gap-2',
                  isDarkMode ? 'text-gray-400' : 'text-gray-500'
                ]">
                  <Icon icon="mdi:information" class="text-sm sm:text-base" />
                  插件描述
                </h3>
                <p :class="[
                  'text-xs sm:text-sm leading-relaxed',
                  isDarkMode ? 'text-gray-300' : 'text-gray-600'
                ]">{{ selectedPlugin.description }}</p>
              </div>
              
              <!-- Categories & Keywords -->
              <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 sm:gap-6">
                <div>
                  <h3 :class="[
                    'text-xs sm:text-sm font-semibold uppercase tracking-wider mb-2 sm:mb-3 flex items-center gap-1.5 sm:gap-2',
                    isDarkMode ? 'text-gray-400' : 'text-gray-500'
                  ]">
                    <Icon icon="mdi:folder" class="text-sm sm:text-base" />
                    分类
                  </h3>
                  <div v-if="selectedPlugin.categories && selectedPlugin.categories.length > 0" class="flex flex-wrap gap-1.5 sm:gap-2">
                    <span
                      v-for="category in selectedPlugin.categories"
                      :key="category"
                      :class="[
                        'px-2 sm:px-3 py-1 sm:py-1.5 text-[10px] sm:text-xs font-medium rounded-md sm:rounded-lg',
                        isDarkMode ? 'bg-purple-900/30 text-purple-400' : 'bg-purple-50 text-purple-600'
                      ]"
                    >
                      {{ category }}
                    </span>
                  </div>
                  <p v-else :class="[
                    'text-xs sm:text-sm',
                    isDarkMode ? 'text-gray-500' : 'text-gray-400'
                  ]">无</p>
                </div>
                
                <div>
                  <h3 :class="[
                    'text-xs sm:text-sm font-semibold uppercase tracking-wider mb-2 sm:mb-3 flex items-center gap-1.5 sm:gap-2',
                    isDarkMode ? 'text-gray-400' : 'text-gray-500'
                  ]">
                    <Icon icon="mdi:tag-multiple" class="text-sm sm:text-base" />
                    关键词
                  </h3>
                  <div v-if="selectedPlugin.keywords && selectedPlugin.keywords.length > 0" class="flex flex-wrap gap-1.5 sm:gap-2">
                    <span
                      v-for="keyword in selectedPlugin.keywords"
                      :key="keyword"
                      class="tag text-[10px] sm:text-xs"
                    >
                      {{ keyword }}
                    </span>
                  </div>
                  <p v-else :class="[
                    'text-xs sm:text-sm',
                    isDarkMode ? 'text-gray-500' : 'text-gray-400'
                  ]">无</p>
                </div>
              </div>
              
              <!-- Technical Info -->
              <div>
                <h3 :class="[
                  'text-xs sm:text-sm font-semibold uppercase tracking-wider mb-2 sm:mb-3 flex items-center gap-1.5 sm:gap-2',
                  isDarkMode ? 'text-gray-400' : 'text-gray-500'
                ]">
                  <Icon icon="mdi:cog" class="text-sm sm:text-base" />
                  技术信息
                </h3>
                <div class="grid grid-cols-2 gap-2 sm:gap-3">
                  <div :class="[
                    'rounded-lg sm:rounded-xl p-3 sm:p-4 border',
                    isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
                  ]">
                    <div :class="[
                      'text-[10px] sm:text-xs mb-1 sm:mb-1.5 flex items-center gap-1 sm:gap-1.5',
                      isDarkMode ? 'text-gray-500' : 'text-gray-400'
                    ]">
                      <Icon icon="mdi:check-circle" class="text-xs sm:text-sm" />
                      状态
                    </div>
                    <div class="text-xs sm:text-sm font-medium text-green-500 flex items-center gap-1 sm:gap-1.5">
                      <Icon icon="mdi:check" class="text-xs sm:text-sm" />
                      可用
                    </div>
                  </div>
                  <div :class="[
                    'rounded-lg sm:rounded-xl p-3 sm:p-4 border',
                    isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
                  ]">
                    <div :class="[
                      'text-[10px] sm:text-xs mb-1 sm:mb-1.5 flex items-center gap-1 sm:gap-1.5',
                      isDarkMode ? 'text-gray-500' : 'text-gray-400'
                    ]">
                      <Icon icon="mdi:identifier" class="text-xs sm:text-sm" />
                      插件 ID
                    </div>
                    <div :class="[
                      'text-xs sm:text-sm font-mono font-medium truncate',
                      isDarkMode ? 'text-gray-300' : 'text-gray-700'
                    ]">{{ selectedPlugin.id }}</div>
                  </div>
                  <div :class="[
                    'rounded-lg sm:rounded-xl p-3 sm:p-4 border col-span-2',
                    isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
                  ]">
                    <div :class="[
                      'text-[10px] sm:text-xs mb-1 sm:mb-1.5 flex items-center gap-1 sm:gap-1.5',
                      isDarkMode ? 'text-gray-500' : 'text-gray-400'
                    ]">
                      <Icon icon="mdi:clock-outline" class="text-xs sm:text-sm" />
                      上传时间
                    </div>
                    <div :class="[
                      'text-xs sm:text-sm font-medium',
                      isDarkMode ? 'text-gray-300' : 'text-gray-700'
                    ]">{{ selectedPlugin.createdAt || '未知' }}</div>
                  </div>
                  <div v-if="selectedPlugin.homepageUrl" :class="[
                    'rounded-lg sm:rounded-xl p-3 sm:p-4 border col-span-2',
                    isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
                  ]">
                    <div :class="[
                      'text-[10px] sm:text-xs mb-1 sm:mb-1.5 flex items-center gap-1 sm:gap-1.5',
                      isDarkMode ? 'text-gray-500' : 'text-gray-400'
                    ]">
                      <Icon icon="mdi:home" class="text-xs sm:text-sm" />
                      官方网站
                    </div>
                    <a 
                      :href="selectedPlugin.homepageUrl" 
                      target="_blank" 
                      rel="noopener noreferrer"
                      class="text-xs sm:text-sm font-medium text-primary-500 hover:text-primary-600 transition-colors flex items-center gap-1"
                    >
                      访问网站
                      <Icon icon="mdi:open-in-new" class="text-[10px] sm:text-xs" />
                    </a>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- Footer -->
            <div :class="[
              'p-4 sm:p-6 border-t flex flex-col sm:flex-row gap-2 sm:gap-3',
              isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
            ]">
              <button 
                @click="goToRepository(selectedPlugin)"
                :disabled="!selectedPlugin.repositoryUrl || selectedPlugin.repositoryUrl.trim() === ''"
                :class="[
                  'flex-1 px-4 sm:px-6 py-2.5 sm:py-3 rounded-lg sm:rounded-xl text-sm font-medium transition-all duration-200 flex items-center justify-center gap-2',
                  selectedPlugin.repositoryUrl && selectedPlugin.repositoryUrl.trim() !== '' 
                    ? 'bg-primary-500 hover:bg-primary-600 text-white hover:shadow-glow' 
                    : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
                ]"
              >
                <Icon icon="mdi:github" class="text-base sm:text-lg" />
                查看仓库
              </button>
              <button 
                @click="closeModal"
                :class="[
                  'px-4 sm:px-6 py-2.5 sm:py-3 rounded-lg sm:rounded-xl text-sm font-medium transition-all duration-200',
                  isDarkMode 
                    ? 'bg-dark-border hover:bg-dark-muted text-gray-300' 
                    : 'bg-light-muted hover:bg-gray-200 text-gray-700'
                ]"
              >
                关闭
              </button>
            </div>
          </div>
        </div>
      </Transition>
    </div>
  </Transition>
</template>