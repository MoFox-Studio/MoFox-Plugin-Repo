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
    <div v-if="showModal" class="fixed inset-0 z-50 flex items-center justify-center p-4">
      <!-- Backdrop -->
      <div 
        class="fixed inset-0 bg-black/50 backdrop-blur-sm"
        @click="closeModal"
      ></div>
      
      <!-- Modal Content -->
      <Transition name="modal-content" appear>
        <div v-if="showModal" :class="[
          'relative w-full max-w-2xl max-h-[90vh] overflow-hidden rounded-2xl border shadow-2xl',
          isDarkMode ? 'bg-dark-bg border-dark-border' : 'bg-white border-light-border'
        ]">
          <!-- Close Button -->
          <button 
            @click="closeModal"
            :class="[
              'absolute top-4 right-4 z-10 w-10 h-10 rounded-xl flex items-center justify-center transition-colors',
              isDarkMode 
                ? 'bg-dark-surface hover:bg-dark-border text-gray-400 hover:text-white' 
                : 'bg-light-surface hover:bg-light-muted text-gray-500 hover:text-gray-900'
            ]"
          >
            <Icon icon="mdi:close" class="text-xl" />
          </button>
        
          <div v-if="selectedPlugin" class="overflow-y-auto max-h-[90vh]">
            <!-- Header -->
            <div :class="[
              'p-6 border-b',
              isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
            ]">
              <div class="flex items-start gap-5">
                <div class="w-16 h-16 rounded-2xl bg-gradient-to-br from-primary-500 to-primary-600 flex items-center justify-center shadow-glow flex-shrink-0">
                  <Icon :icon="selectedPlugin.icon" class="text-2xl text-white" />
                </div>
                <div class="flex-1 min-w-0 pr-10">
                  <h2 :class="[
                    'text-2xl font-bold mb-1 truncate',
                    isDarkMode ? 'text-white' : 'text-gray-900'
                  ]">{{ selectedPlugin.name }}</h2>
                  <div class="flex items-center gap-3 flex-wrap">
                    <span :class="[
                      'text-sm flex items-center gap-1',
                      isDarkMode ? 'text-gray-400' : 'text-gray-500'
                    ]">
                      <Icon icon="mdi:account" />
                      {{ selectedPlugin.author }}
                    </span>
                    <a 
                      v-if="selectedPlugin.authorUrl" 
                      :href="selectedPlugin.authorUrl" 
                      target="_blank" 
                      rel="noopener noreferrer"
                      class="text-primary-500 hover:text-primary-600 text-sm flex items-center gap-1 transition-colors"
                    >
                      <Icon icon="mdi:open-in-new" class="text-xs" />
                      主页
                    </a>
                  </div>
                  <div class="flex items-center gap-4 mt-2">
                    <span :class="[
                      'px-2.5 py-1 text-xs font-medium rounded-lg',
                      isDarkMode ? 'bg-dark-border text-gray-400' : 'bg-light-muted text-gray-500'
                    ]">v{{ selectedPlugin.version }}</span>
                    <span :class="[
                      'text-xs flex items-center gap-1',
                      isDarkMode ? 'text-gray-500' : 'text-gray-400'
                    ]">
                      <Icon icon="mdi:license" />
                      {{ selectedPlugin.license }}
                    </span>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- Body -->
            <div class="p-6 space-y-6">
              <!-- Description -->
              <div>
                <h3 :class="[
                  'text-sm font-semibold uppercase tracking-wider mb-3 flex items-center gap-2',
                  isDarkMode ? 'text-gray-400' : 'text-gray-500'
                ]">
                  <Icon icon="mdi:information" />
                  插件描述
                </h3>
                <p :class="[
                  'text-sm leading-relaxed',
                  isDarkMode ? 'text-gray-300' : 'text-gray-600'
                ]">{{ selectedPlugin.description }}</p>
              </div>
              
              <!-- Categories & Keywords -->
              <div class="grid grid-cols-1 sm:grid-cols-2 gap-6">
                <div>
                  <h3 :class="[
                    'text-sm font-semibold uppercase tracking-wider mb-3 flex items-center gap-2',
                    isDarkMode ? 'text-gray-400' : 'text-gray-500'
                  ]">
                    <Icon icon="mdi:folder" />
                    分类
                  </h3>
                  <div v-if="selectedPlugin.categories && selectedPlugin.categories.length > 0" class="flex flex-wrap gap-2">
                    <span
                      v-for="category in selectedPlugin.categories"
                      :key="category"
                      :class="[
                        'px-3 py-1.5 text-xs font-medium rounded-lg',
                        isDarkMode ? 'bg-purple-900/30 text-purple-400' : 'bg-purple-50 text-purple-600'
                      ]"
                    >
                      {{ category }}
                    </span>
                  </div>
                  <p v-else :class="[
                    'text-sm',
                    isDarkMode ? 'text-gray-500' : 'text-gray-400'
                  ]">无</p>
                </div>
                
                <div>
                  <h3 :class="[
                    'text-sm font-semibold uppercase tracking-wider mb-3 flex items-center gap-2',
                    isDarkMode ? 'text-gray-400' : 'text-gray-500'
                  ]">
                    <Icon icon="mdi:tag-multiple" />
                    关键词
                  </h3>
                  <div v-if="selectedPlugin.keywords && selectedPlugin.keywords.length > 0" class="flex flex-wrap gap-2">
                    <span
                      v-for="keyword in selectedPlugin.keywords"
                      :key="keyword"
                      class="tag"
                    >
                      {{ keyword }}
                    </span>
                  </div>
                  <p v-else :class="[
                    'text-sm',
                    isDarkMode ? 'text-gray-500' : 'text-gray-400'
                  ]">无</p>
                </div>
              </div>
              
              <!-- Technical Info -->
              <div>
                <h3 :class="[
                  'text-sm font-semibold uppercase tracking-wider mb-3 flex items-center gap-2',
                  isDarkMode ? 'text-gray-400' : 'text-gray-500'
                ]">
                  <Icon icon="mdi:cog" />
                  技术信息
                </h3>
                <div class="grid grid-cols-2 gap-3">
                  <div :class="[
                    'rounded-xl p-4 border',
                    isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
                  ]">
                    <div :class="[
                      'text-xs mb-1.5 flex items-center gap-1.5',
                      isDarkMode ? 'text-gray-500' : 'text-gray-400'
                    ]">
                      <Icon icon="mdi:check-circle" />
                      状态
                    </div>
                    <div class="text-sm font-medium text-green-500 flex items-center gap-1.5">
                      <Icon icon="mdi:check" />
                      可用
                    </div>
                  </div>
                  <div :class="[
                    'rounded-xl p-4 border',
                    isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
                  ]">
                    <div :class="[
                      'text-xs mb-1.5 flex items-center gap-1.5',
                      isDarkMode ? 'text-gray-500' : 'text-gray-400'
                    ]">
                      <Icon icon="mdi:identifier" />
                      插件 ID
                    </div>
                    <div :class="[
                      'text-sm font-mono font-medium truncate',
                      isDarkMode ? 'text-gray-300' : 'text-gray-700'
                    ]">{{ selectedPlugin.id }}</div>
                  </div>
                  <div :class="[
                    'rounded-xl p-4 border col-span-2',
                    isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
                  ]">
                    <div :class="[
                      'text-xs mb-1.5 flex items-center gap-1.5',
                      isDarkMode ? 'text-gray-500' : 'text-gray-400'
                    ]">
                      <Icon icon="mdi:clock-outline" />
                      上传时间
                    </div>
                    <div :class="[
                      'text-sm font-medium',
                      isDarkMode ? 'text-gray-300' : 'text-gray-700'
                    ]">{{ selectedPlugin.createdAt || '未知' }}</div>
                  </div>
                  <div v-if="selectedPlugin.homepageUrl" :class="[
                    'rounded-xl p-4 border col-span-2',
                    isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
                  ]">
                    <div :class="[
                      'text-xs mb-1.5 flex items-center gap-1.5',
                      isDarkMode ? 'text-gray-500' : 'text-gray-400'
                    ]">
                      <Icon icon="mdi:home" />
                      官方网站
                    </div>
                    <a 
                      :href="selectedPlugin.homepageUrl" 
                      target="_blank" 
                      rel="noopener noreferrer"
                      class="text-sm font-medium text-primary-500 hover:text-primary-600 transition-colors flex items-center gap-1"
                    >
                      访问网站
                      <Icon icon="mdi:open-in-new" class="text-xs" />
                    </a>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- Footer -->
            <div :class="[
              'p-6 border-t flex gap-3',
              isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
            ]">
              <button 
                @click="goToRepository(selectedPlugin)"
                :disabled="!selectedPlugin.repositoryUrl || selectedPlugin.repositoryUrl.trim() === ''"
                :class="[
                  'flex-1 px-6 py-3 rounded-xl font-medium transition-all duration-200 flex items-center justify-center gap-2',
                  selectedPlugin.repositoryUrl && selectedPlugin.repositoryUrl.trim() !== '' 
                    ? 'bg-primary-500 hover:bg-primary-600 text-white hover:shadow-glow' 
                    : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
                ]"
              >
                <Icon icon="mdi:github" />
                查看仓库
              </button>
              <button 
                @click="closeModal"
                :class="[
                  'px-6 py-3 rounded-xl font-medium transition-all duration-200',
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