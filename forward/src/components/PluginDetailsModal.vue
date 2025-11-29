<script setup>
import { ref, watch, computed } from 'vue'
import { Icon } from '@iconify/vue'
import { marked } from 'marked'
import { usePlugins } from '../composables/usePlugins'

const { plugins } = usePlugins()

const props = defineProps({
  isDarkMode: Boolean,
  showModal: Boolean,
  selectedPlugin: Object,
  closeModal: Function,
  goToRepository: Function,
  openDownloadModal: Function
})

// 当前 Tab：info | readme
const activeTab = ref('info')

// 分支相关状态
const branches = ref([])
const selectedBranch = ref('')
const readmeContent = ref('')
const isLoadingBranches = ref(false)
const isLoadingReadme = ref(false)
const errorBranches = ref(null)
const errorReadme = ref(null)

const pluginRepoUrl = computed(() => {
  if (!props.selectedPlugin || !props.selectedPlugin.id) return null
  const foundPlugin = plugins.value.find(p => p.id === props.selectedPlugin.id)
  return foundPlugin ? foundPlugin.repositoryUrl : null
})

// 获取分支
async function fetchBranches() {
  if (!pluginRepoUrl.value) {
    errorBranches.value = '未找到仓库地址。'
    readmeContent.value = '无法加载分支，因为插件数据不包含有效的仓库地址。'
    return
  }

  isLoadingBranches.value = true
  errorBranches.value = null
  branches.value = []
  readmeContent.value = ''
  selectedBranch.value = ''

  try {
    const url = new URL(pluginRepoUrl.value)
    const pathParts = url.pathname.split('/').filter(Boolean)
    if (pathParts.length < 2) throw new Error('无效的仓库地址')
    const [owner, repo] = pathParts.slice(-2)
    
    const response = await fetch(`https://api.github.com/repos/${owner}/${repo}/branches`)
    if (!response.ok) {
      throw new Error('无法获取分支列表')
    }
    const data = await response.json()
    branches.value = data
    if (branches.value.length > 0) {
      selectedBranch.value = branches.value[0].name
      await fetchReadme()
    } else {
      readmeContent.value = '该仓库没有分支。'
    }
  } catch (err) {
    errorBranches.value = err.message
    readmeContent.value = '加载分支信息失败。'
  } finally {
    isLoadingBranches.value = false
  }
}

// 获取README
async function fetchReadme() {
  if (!selectedBranch.value || !pluginRepoUrl.value) return

  isLoadingReadme.value = true
  errorReadme.value = null
  readmeContent.value = ''

  try {
    const url = new URL(pluginRepoUrl.value)
    const pathParts = url.pathname.split('/').filter(Boolean)
    if (pathParts.length < 2) throw new Error('无效的仓库地址')
    const [owner, repo] = pathParts.slice(-2)

    const response = await fetch(`https://raw.githubusercontent.com/${owner}/${repo}/${selectedBranch.value}/README.md`)
    if (!response.ok) {
      if (response.status === 404) {
        throw new Error('该分支下没有找到 README.md 文件。')
      }
      throw new Error('无法获取 README.md')
    }
    const text = await response.text()
    readmeContent.value = marked(text)
  } catch (err) {
    errorReadme.value = err.message
    readmeContent.value = ''
  } finally {
    isLoadingReadme.value = false
  }
}

// 当选择的分支变化时，重新获取README
watch(selectedBranch, (newBranch, oldBranch) => {
  if (newBranch && newBranch !== oldBranch) {
    fetchReadme()
  }
})

// 当模态框显示或插件变化时，获取分支
watch(
  () => props.showModal,
  (newShowModal) => {
    if (newShowModal && props.selectedPlugin) {
      activeTab.value = 'info'
      fetchBranches()
    }
  }
)

watch(
  () => props.selectedPlugin,
  (newPlugin) => {
    if (newPlugin && props.showModal) {
      activeTab.value = 'info'
      fetchBranches()
    }
  }
)

// 下载插件
function downloadPlugin() {
  if (!pluginRepoUrl.value || !selectedBranch.value) {
    alert('缺少必要信息，无法下载。')
    return
  }
  const downloadUrl = `${pluginRepoUrl.value}/archive/refs/heads/${selectedBranch.value}.zip`
  const repoName = pluginRepoUrl.value.split('/').pop()
  
  const a = document.createElement('a')
  a.href = downloadUrl
  a.download = `${repoName}-${selectedBranch.value}.zip`
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
}
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
          'relative w-full max-w-3xl max-h-[95vh] sm:max-h-[90vh] overflow-hidden rounded-xl sm:rounded-2xl border shadow-2xl',
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
              
              <!-- Tab Navigation -->
              <div class="flex items-center gap-1 mt-4 sm:mt-5 p-1 rounded-lg sm:rounded-xl" :class="isDarkMode ? 'bg-dark-muted' : 'bg-light-muted'">
                <button
                  @click="activeTab = 'info'"
                  :class="[
                    'flex-1 px-3 sm:px-4 py-2 sm:py-2.5 rounded-md sm:rounded-lg text-xs sm:text-sm font-medium transition-all duration-200 flex items-center justify-center gap-1.5',
                    activeTab === 'info'
                      ? 'bg-white shadow-sm text-primary-600 dark:bg-dark-surface dark:text-primary-400'
                      : isDarkMode ? 'text-gray-400 hover:text-gray-200' : 'text-gray-500 hover:text-gray-700'
                  ]"
                >
                  <Icon icon="mdi:information-outline" class="text-sm sm:text-base" />
                  详细信息
                </button>
                <button
                  @click="activeTab = 'readme'"
                  :class="[
                    'flex-1 px-3 sm:px-4 py-2 sm:py-2.5 rounded-md sm:rounded-lg text-xs sm:text-sm font-medium transition-all duration-200 flex items-center justify-center gap-1.5',
                    activeTab === 'readme'
                      ? 'bg-white shadow-sm text-primary-600 dark:bg-dark-surface dark:text-primary-400'
                      : isDarkMode ? 'text-gray-400 hover:text-gray-200' : 'text-gray-500 hover:text-gray-700'
                  ]"
                >
                  <Icon icon="mdi:file-document-outline" class="text-sm sm:text-base" />
                  README
                </button>
              </div>
            </div>
            
            <!-- Body - Info Tab -->
            <div v-show="activeTab === 'info'" class="p-4 sm:p-6 space-y-4 sm:space-y-6">
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
            
            <!-- Body - README Tab -->
            <div v-show="activeTab === 'readme'" class="p-4 sm:p-6 space-y-4 sm:space-y-5">
              <!-- Branch Selection -->
              <div :class="[
                'rounded-xl p-4 border',
                isDarkMode ? 'bg-dark-surface/50 border-dark-border' : 'bg-light-surface/50 border-light-border'
              ]">
                <div class="flex flex-col sm:flex-row sm:items-center gap-3">
                  <div class="flex items-center gap-2">
                    <Icon icon="mdi:source-branch" :class="['text-lg', isDarkMode ? 'text-primary-400' : 'text-primary-500']" />
                    <span :class="['text-sm font-medium', isDarkMode ? 'text-gray-300' : 'text-gray-700']">分支选择</span>
                  </div>
                  <div class="relative flex-1">
                    <select
                      v-model="selectedBranch"
                      :disabled="isLoadingBranches || branches.length === 0"
                      :class="[
                        'w-full px-3 py-2 rounded-lg border appearance-none transition-all cursor-pointer text-sm pr-8',
                        isDarkMode 
                          ? 'bg-dark-bg border-dark-border text-white focus:border-primary-500' 
                          : 'bg-white border-light-border text-gray-900 focus:border-primary-500',
                        (isLoadingBranches || branches.length === 0) ? 'opacity-50 cursor-not-allowed' : ''
                      ]"
                    >
                      <option v-if="isLoadingBranches" value="" disabled>正在加载分支...</option>
                      <option v-else-if="errorBranches" value="" disabled>{{ errorBranches }}</option>
                      <option v-else-if="branches.length === 0" value="" disabled>没有可用的分支</option>
                      <option v-for="branch in branches" :key="branch.name" :value="branch.name">
                        {{ branch.name }}
                      </option>
                    </select>
                    <Icon icon="mdi:chevron-down" :class="[
                      'absolute right-3 top-1/2 -translate-y-1/2 pointer-events-none text-sm',
                      isDarkMode ? 'text-gray-500' : 'text-gray-400'
                    ]" />
                  </div>
                </div>
              </div>

              <!-- README Content -->
              <div :class="[
                'readme-container rounded-xl border overflow-hidden',
                isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-white border-light-border'
              ]">
                <!-- README Header -->
                <div :class="[
                  'flex items-center gap-2 px-4 py-3 border-b',
                  isDarkMode ? 'bg-dark-muted border-dark-border' : 'bg-light-muted border-light-border'
                ]">
                  <Icon icon="mdi:file-document" :class="['text-base', isDarkMode ? 'text-gray-400' : 'text-gray-500']" />
                  <span :class="['text-sm font-medium', isDarkMode ? 'text-gray-300' : 'text-gray-700']">README.md</span>
                </div>
                
                <!-- README Body -->
                <div :class="[
                  'readme-content p-4 sm:p-6 h-72 sm:h-96 overflow-y-auto',
                  isDarkMode ? 'readme-dark' : 'readme-light'
                ]">
                  <!-- Loading State -->
                  <div v-if="isLoadingReadme" class="flex flex-col items-center justify-center h-full gap-3">
                    <div class="w-8 h-8 border-3 border-primary-500/30 border-t-primary-500 rounded-full animate-spin"></div>
                    <span :class="['text-sm', isDarkMode ? 'text-gray-400' : 'text-gray-500']">正在加载 README...</span>
                  </div>
                  
                  <!-- Error State -->
                  <div v-else-if="errorReadme" class="flex flex-col items-center justify-center h-full gap-3 text-center">
                    <div :class="[
                      'w-12 h-12 rounded-xl flex items-center justify-center',
                      isDarkMode ? 'bg-red-900/20' : 'bg-red-50'
                    ]">
                      <Icon icon="mdi:file-alert" :class="['text-2xl', isDarkMode ? 'text-red-400' : 'text-red-500']" />
                    </div>
                    <div>
                      <p :class="['text-sm font-medium mb-1', isDarkMode ? 'text-gray-300' : 'text-gray-700']">无法加载 README</p>
                      <p :class="['text-xs', isDarkMode ? 'text-gray-500' : 'text-gray-400']">{{ errorReadme }}</p>
                    </div>
                  </div>
                  
                  <!-- README Content -->
                  <div v-else-if="readmeContent" v-html="readmeContent" class="markdown-body"></div>
                  
                  <!-- Empty State -->
                  <div v-else class="flex flex-col items-center justify-center h-full gap-3 text-center">
                    <div :class="[
                      'w-12 h-12 rounded-xl flex items-center justify-center',
                      isDarkMode ? 'bg-dark-muted' : 'bg-light-muted'
                    ]">
                      <Icon icon="mdi:file-question" :class="['text-2xl', isDarkMode ? 'text-gray-500' : 'text-gray-400']" />
                    </div>
                    <p :class="['text-sm', isDarkMode ? 'text-gray-500' : 'text-gray-400']">没有可预览的 README 内容</p>
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
                @click="downloadPlugin"
                :disabled="!selectedBranch || !pluginRepoUrl"
                :class="[
                  'flex-1 px-4 sm:px-6 py-2.5 sm:py-3 rounded-lg sm:rounded-xl text-sm font-medium transition-all duration-200 flex items-center justify-center gap-2',
                  selectedBranch && pluginRepoUrl
                    ? 'bg-gradient-to-r from-green-500 to-emerald-500 hover:from-green-600 hover:to-emerald-600 text-white hover:shadow-lg'
                    : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
                ]"
              >
                <Icon icon="mdi:download" class="text-base sm:text-lg" />
                下载插件
              </button>
              <button 
                @click="goToRepository(selectedPlugin)"
                :disabled="!selectedPlugin.repositoryUrl || selectedPlugin.repositoryUrl.trim() === ''"
                :class="[
                  'flex-1 px-4 sm:px-6 py-2.5 sm:py-3 rounded-lg sm:rounded-xl text-sm font-medium transition-all duration-200 flex items-center justify-center gap-2',
                  selectedPlugin.repositoryUrl && selectedPlugin.repositoryUrl.trim() !== '' 
                    ? isDarkMode
                      ? 'bg-dark-border hover:bg-dark-muted text-gray-300 hover:text-white'
                      : 'bg-light-muted hover:bg-gray-200 text-gray-700 hover:text-gray-900'
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