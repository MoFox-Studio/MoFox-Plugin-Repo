<script setup>
import { ref, watch, computed } from 'vue'
import { Icon } from '@iconify/vue'
import { marked } from 'marked'
import { usePlugins } from '../composables/usePlugins'

const { plugins } = usePlugins()

const props = defineProps({
  isDarkMode: Boolean,
  showModal: Boolean,
  plugin: Object,
  closeModal: Function
})

const branches = ref([])
const selectedBranch = ref('')
const readmeContent = ref('')
const isLoadingBranches = ref(false)
const isLoadingReadme = ref(false)
const errorBranches = ref(null)
const errorReadme = ref(null)

const pluginRepoUrl = computed(() => {
  if (!props.plugin || !props.plugin.id) return null
  const foundPlugin = plugins.value.find(p => p.id === props.plugin.id)
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
    readmeContent.value = err.message
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
    if (newShowModal && props.plugin) {
      fetchBranches()
    }
  }
)

watch(
  () => props.plugin,
  (newPlugin) => {
    if (newPlugin && props.showModal) {
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
    <div v-if="showModal && plugin" class="fixed inset-0 z-50 flex items-center justify-center p-2 sm:p-4">
      <!-- Backdrop -->
      <div
        class="fixed inset-0 bg-black/50 backdrop-blur-sm"
        @click="closeModal"
      ></div>

      <!-- Modal Content -->
      <Transition name="modal-content" appear>
        <div v-if="showModal && plugin" :class="[
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
              <div class="flex items-start gap-3 sm:gap-5 pr-8 sm:pr-10">
                <div class="w-11 h-11 sm:w-14 sm:h-14 rounded-xl sm:rounded-2xl bg-gradient-to-br from-green-400 to-emerald-500 flex items-center justify-center shadow-lg flex-shrink-0">
                  <Icon icon="mdi:download" class="text-xl sm:text-2xl text-white" />
                </div>
                <div class="flex-1 min-w-0">
                  <h2 :class="[
                    'text-lg sm:text-xl font-bold mb-0.5 sm:mb-1 truncate',
                    isDarkMode ? 'text-white' : 'text-gray-900'
                  ]">下载 {{ plugin.name }}</h2>
                  <p :class="[
                    'text-xs sm:text-sm',
                    isDarkMode ? 'text-gray-400' : 'text-gray-500'
                  ]">选择分支并下载插件</p>
                </div>
              </div>
            </div>

            <!-- Body -->
            <div class="p-4 sm:p-6 space-y-4 sm:space-y-6">
              <!-- Branch Selection -->
              <div>
                <h3 :class="[
                  'text-xs sm:text-sm font-semibold uppercase tracking-wider mb-2 sm:mb-3 flex items-center gap-1.5 sm:gap-2',
                  isDarkMode ? 'text-gray-400' : 'text-gray-500'
                ]">
                  <Icon icon="mdi:source-branch" class="text-sm sm:text-base" />
                  选择分支
                </h3>
                <div class="relative">
                  <select
                    v-model="selectedBranch"
                    :disabled="isLoadingBranches || branches.length === 0"
                    :class="[
                      'w-full px-3 sm:px-4 py-2.5 sm:py-3 rounded-lg sm:rounded-xl border appearance-none transition-all cursor-pointer text-sm',
                      isDarkMode 
                        ? 'bg-dark-surface border-dark-border text-white focus:border-primary-500' 
                        : 'bg-light-surface border-light-border text-gray-900 focus:border-primary-500',
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
                    'absolute right-3 sm:right-4 top-1/2 -translate-y-1/2 pointer-events-none text-sm sm:text-base',
                    isDarkMode ? 'text-gray-500' : 'text-gray-400'
                  ]" />
                </div>
              </div>

              <!-- README Preview -->
              <div>
                <h3 :class="[
                  'text-xs sm:text-sm font-semibold uppercase tracking-wider mb-2 sm:mb-3 flex items-center gap-1.5 sm:gap-2',
                  isDarkMode ? 'text-gray-400' : 'text-gray-500'
                ]">
                  <Icon icon="mdi:book-open-variant" class="text-sm sm:text-base" />
                  README.md
                </h3>
                <div :class="[
                  'prose prose-sm max-w-none p-3 sm:p-4 rounded-lg sm:rounded-xl border h-48 sm:h-64 overflow-y-auto relative',
                  isDarkMode 
                    ? 'prose-invert bg-dark-surface border-dark-border' 
                    : 'bg-light-surface border-light-border'
                ]">
                  <div v-if="isLoadingReadme" class="absolute inset-0 flex items-center justify-center">
                    <div class="flex items-center gap-2 sm:gap-3">
                      <div class="w-4 h-4 sm:w-5 sm:h-5 border-2 border-primary-500/30 border-t-primary-500 rounded-full animate-spin"></div>
                      <span :class="['text-xs sm:text-sm', isDarkMode ? 'text-gray-400' : 'text-gray-500']">正在加载...</span>
                    </div>
                  </div>
                  <div v-else-if="errorReadme" :class="[
                    'text-xs sm:text-sm',
                    isDarkMode ? 'text-red-400' : 'text-red-500'
                  ]">
                    <p>{{ errorReadme }}</p>
                  </div>
                  <div v-else-if="readmeContent" v-html="readmeContent" class="text-xs sm:text-sm"></div>
                  <div v-else :class="[
                    'text-xs sm:text-sm',
                    isDarkMode ? 'text-gray-500' : 'text-gray-400'
                  ]">
                    <p>没有可预览的 README 内容。</p>
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
                    ? 'bg-primary-500 hover:bg-primary-600 text-white hover:shadow-glow'
                    : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
                ]"
              >
                <Icon icon="mdi:download" class="text-base sm:text-lg" />
                下载插件
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