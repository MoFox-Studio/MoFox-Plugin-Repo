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

// 当前 Tab：info | readme | download
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
                <button
                  @click="activeTab = 'download'"
                  :class="[
                    'flex-1 px-3 sm:px-4 py-2 sm:py-2.5 rounded-md sm:rounded-lg text-xs sm:text-sm font-medium transition-all duration-200 flex items-center justify-center gap-1.5',
                    activeTab === 'download'
                      ? 'bg-white shadow-sm text-primary-600 dark:bg-dark-surface dark:text-primary-400'
                      : isDarkMode ? 'text-gray-400 hover:text-gray-200' : 'text-gray-500 hover:text-gray-700'
                  ]"
                >
                  <Icon icon="mdi:download-outline" class="text-sm sm:text-base" />
                  下载
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
              <!-- README Content -->
              <div :class="[
                'readme-container rounded-xl border overflow-hidden',
                isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-white border-light-border'
              ]">
                <!-- README Header -->
                <div :class="[
                  'flex items-center justify-between gap-2 px-4 py-3 border-b',
                  isDarkMode ? 'bg-dark-muted border-dark-border' : 'bg-light-muted border-light-border'
                ]">
                  <div class="flex items-center gap-2">
                    <Icon icon="mdi:file-document" :class="['text-base', isDarkMode ? 'text-gray-400' : 'text-gray-500']" />
                    <span :class="['text-sm font-medium', isDarkMode ? 'text-gray-300' : 'text-gray-700']">README.md</span>
                  </div>
                  <span v-if="selectedBranch" :class="['text-xs px-2 py-1 rounded-md', isDarkMode ? 'bg-dark-bg text-gray-400' : 'bg-light-surface text-gray-500']">
                    <Icon icon="mdi:source-branch" class="inline text-xs mr-1" />{{ selectedBranch }}
                  </span>
                </div>
                
                <!-- README Body -->
                <div :class="[
                  'readme-content p-4 sm:p-6 h-80 sm:h-[28rem] overflow-y-auto',
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
            
            <!-- Body - Download Tab -->
            <div v-show="activeTab === 'download'" class="p-4 sm:p-6 space-y-4 sm:space-y-5">
              <!-- Download Header Card -->
              <div :class="[
                'rounded-xl p-5 sm:p-6 border text-center',
                isDarkMode ? 'bg-gradient-to-br from-dark-surface to-dark-muted border-dark-border' : 'bg-gradient-to-br from-white to-light-surface border-light-border'
              ]">
                <div class="w-16 h-16 sm:w-20 sm:h-20 mx-auto mb-4 rounded-2xl bg-gradient-to-br from-green-400 to-emerald-500 flex items-center justify-center shadow-lg">
                  <Icon icon="mdi:download" class="text-3xl sm:text-4xl text-white" />
                </div>
                <h3 :class="['text-lg sm:text-xl font-bold mb-2', isDarkMode ? 'text-white' : 'text-gray-900']">
                  下载 {{ selectedPlugin.name }}
                </h3>
                <p :class="['text-sm', isDarkMode ? 'text-gray-400' : 'text-gray-500']">
                  在下方选择分支后点击下载按钮获取插件源码
                </p>
              </div>
              
              <!-- Download Info -->
              <div v-if="selectedBranch" :class="[
                'rounded-xl border overflow-hidden',
                isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-white border-light-border'
              ]">
                <div :class="[
                  'flex items-center gap-2 px-4 py-3 border-b',
                  isDarkMode ? 'bg-dark-muted border-dark-border' : 'bg-light-muted border-light-border'
                ]">
                  <Icon icon="mdi:package-variant" :class="['text-base', isDarkMode ? 'text-green-400' : 'text-green-500']" />
                  <span :class="['text-sm font-medium', isDarkMode ? 'text-gray-300' : 'text-gray-700']">下载信息</span>
                </div>
                <div class="p-4 space-y-3">
                  <div class="flex items-center justify-between">
                    <span :class="['text-xs', isDarkMode ? 'text-gray-500' : 'text-gray-400']">当前分支</span>
                    <span :class="['text-sm font-medium flex items-center gap-1.5', isDarkMode ? 'text-primary-400' : 'text-primary-600']">
                      <Icon icon="mdi:source-branch" class="text-sm" />
                      {{ selectedBranch }}
                    </span>
                  </div>
                  <div class="flex items-center justify-between">
                    <span :class="['text-xs', isDarkMode ? 'text-gray-500' : 'text-gray-400']">文件名</span>
                    <span :class="['text-sm font-mono', isDarkMode ? 'text-gray-300' : 'text-gray-700']">
                      {{ selectedPlugin.name }}-{{ selectedBranch }}.zip
                    </span>
                  </div>
                  <div class="flex items-center justify-between">
                    <span :class="['text-xs', isDarkMode ? 'text-gray-500' : 'text-gray-400']">格式</span>
                    <span :class="['text-sm', isDarkMode ? 'text-gray-300' : 'text-gray-700']">ZIP 压缩包</span>
                  </div>
                </div>
              </div>
              
              <!-- Download Button -->
              <button 
                @click="downloadPlugin"
                :disabled="!selectedBranch || !pluginRepoUrl"
                :class="[
                  'w-full px-6 py-4 rounded-xl text-base font-semibold transition-all duration-200 flex items-center justify-center gap-3',
                  selectedBranch && pluginRepoUrl
                    ? 'bg-gradient-to-r from-green-500 to-emerald-500 hover:from-green-600 hover:to-emerald-600 text-white hover:shadow-xl hover:scale-[1.02] active:scale-[0.98]'
                    : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
                ]"
              >
                <Icon icon="mdi:download" class="text-xl" />
                <span v-if="selectedBranch">下载 {{ selectedBranch }} 分支</span>
                <span v-else>请先选择分支</span>
              </button>
              
              <!-- Tips -->
              <div :class="[
                'rounded-xl p-4 border flex items-start gap-3',
                isDarkMode ? 'bg-primary-900/10 border-primary-800/30' : 'bg-primary-50 border-primary-200'
              ]">
                <Icon icon="mdi:information-outline" :class="['text-xl flex-shrink-0 mt-0.5', isDarkMode ? 'text-primary-400' : 'text-primary-500']" />
                <div>
                  <p :class="['text-sm font-medium mb-1', isDarkMode ? 'text-primary-300' : 'text-primary-700']">关于分支选择</p>
                  <p :class="['text-xs leading-relaxed', isDarkMode ? 'text-primary-400/80' : 'text-primary-600']">
                    你可以在底部左下角的分支选择器中切换分支。README 预览和下载会同步使用所选分支。
                  </p>
                </div>
              </div>
            </div>
            
            <!-- Footer -->
            <div :class="[
              'p-4 sm:p-6 border-t flex flex-col sm:flex-row items-stretch sm:items-center gap-3',
              isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
            ]">
              <!-- Branch Selector (Left) -->
              <div class="flex items-center gap-2 flex-1 min-w-0">
                <Icon icon="mdi:source-branch" :class="['text-lg flex-shrink-0', isDarkMode ? 'text-primary-400' : 'text-primary-500']" />
                <div class="relative flex-1 min-w-0 max-w-[200px]">
                  <select
                    v-model="selectedBranch"
                    :disabled="isLoadingBranches || branches.length === 0"
                    :class="[
                      'w-full pl-3 pr-8 py-2 rounded-lg border appearance-none transition-all cursor-pointer text-sm',
                      isDarkMode 
                        ? 'bg-dark-muted border-dark-border text-white focus:border-primary-500' 
                        : 'bg-white border-light-border text-gray-900 focus:border-primary-500',
                      (isLoadingBranches || branches.length === 0) ? 'opacity-50 cursor-not-allowed' : ''
                    ]"
                  >
                    <option v-if="isLoadingBranches" value="" disabled>加载中...</option>
                    <option v-else-if="errorBranches" value="" disabled>加载失败</option>
                    <option v-else-if="branches.length === 0" value="" disabled>无分支</option>
                    <option v-for="branch in branches" :key="branch.name" :value="branch.name">
                      {{ branch.name }}
                    </option>
                  </select>
                  <Icon icon="mdi:chevron-down" :class="[
                    'absolute right-2 top-1/2 -translate-y-1/2 pointer-events-none text-sm',
                    isDarkMode ? 'text-gray-500' : 'text-gray-400'
                  ]" />
                </div>
              </div>
              
              <!-- Action Buttons (Right) -->
              <div class="flex items-center gap-2 sm:gap-3">
                <button 
                  @click="goToRepository(selectedPlugin)"
                  :disabled="!selectedPlugin.repositoryUrl || selectedPlugin.repositoryUrl.trim() === ''"
                  :class="[
                    'px-3 sm:px-4 py-2 sm:py-2.5 rounded-lg sm:rounded-xl text-sm font-medium transition-all duration-200 flex items-center justify-center gap-1.5',
                    selectedPlugin.repositoryUrl && selectedPlugin.repositoryUrl.trim() !== '' 
                      ? isDarkMode
                        ? 'bg-dark-border hover:bg-dark-muted text-gray-300 hover:text-white'
                        : 'bg-light-muted hover:bg-gray-200 text-gray-700 hover:text-gray-900'
                      : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
                  ]"
                >
                  <Icon icon="mdi:github" class="text-base" />
                  <span class="hidden sm:inline">仓库</span>
                </button>
                <button 
                  @click="closeModal"
                  :class="[
                    'px-3 sm:px-4 py-2 sm:py-2.5 rounded-lg sm:rounded-xl text-sm font-medium transition-all duration-200',
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
        </div>
      </Transition>
    </div>
  </Transition>
</template>