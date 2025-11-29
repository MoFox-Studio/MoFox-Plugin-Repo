<script setup>
import { ref } from 'vue'
import { Icon } from '@iconify/vue'

const props = defineProps({
  isDarkMode: Boolean,
  showNotice: Boolean,
  canCloseNotice: Boolean,
  closeNotice: Function
})

const dontShowNoticeAgain = ref(false)

const handleClose = () => {
  if (dontShowNoticeAgain.value) {
    localStorage.setItem('dontShowNoticeAgain', 'true')
  }
  props.closeNotice()
}
</script>

<template>
  <Transition name="modal-backdrop">
    <div v-if="showNotice" class="fixed inset-0 z-[9999] flex items-center justify-center p-2 sm:p-4">
      <!-- Backdrop -->
      <div 
        class="fixed inset-0 bg-black/60 backdrop-blur-sm"
        @click="canCloseNotice ? handleClose() : null"
      ></div>

      <Transition name="modal-content" appear>
        <div v-if="showNotice" :class="[
          'relative w-full max-w-2xl max-h-[95vh] sm:max-h-[90vh] overflow-hidden rounded-xl sm:rounded-2xl border shadow-2xl',
          isDarkMode ? 'bg-dark-bg border-dark-border' : 'bg-white border-light-border'
        ]">
          <div class="overflow-y-auto max-h-[95vh] sm:max-h-[90vh]">
            <!-- Header -->
            <div :class="[
              'p-4 sm:p-6 border-b',
              isDarkMode ? 'bg-amber-900/20 border-dark-border' : 'bg-amber-50 border-light-border'
            ]">
              <div class="flex items-center gap-3 sm:gap-4">
                <div class="w-10 h-10 sm:w-12 sm:h-12 rounded-lg sm:rounded-xl bg-gradient-to-br from-amber-400 to-orange-500 flex items-center justify-center flex-shrink-0">
                  <Icon icon="mdi:alert" class="text-xl sm:text-2xl text-white" />
                </div>
                <div>
                  <h2 :class="[
                    'text-lg sm:text-xl font-bold',
                    isDarkMode ? 'text-white' : 'text-gray-900'
                  ]">插件市场重要提示</h2>
                  <p :class="[
                    'text-xs sm:text-sm',
                    isDarkMode ? 'text-gray-400' : 'text-gray-500'
                  ]">使用前请仔细阅读</p>
                </div>
              </div>
            </div>

            <!-- Body -->
            <div class="p-4 sm:p-6 space-y-4 sm:space-y-6">
              <!-- Security Notice -->
              <div :class="[
                'p-3 sm:p-4 rounded-lg sm:rounded-xl border-l-4 border-l-red-500',
                isDarkMode ? 'bg-red-900/10' : 'bg-red-50'
              ]">
                <h3 :class="[
                  'text-sm sm:text-base font-semibold mb-1.5 sm:mb-2 flex items-center gap-1.5 sm:gap-2',
                  isDarkMode ? 'text-red-400' : 'text-red-700'
                ]">
                  <Icon icon="mdi:shield-alert" class="text-base sm:text-lg" />
                  安全须知
                </h3>
                <p :class="[
                  'text-xs sm:text-sm leading-relaxed',
                  isDarkMode ? 'text-gray-300' : 'text-gray-600'
                ]">
                  所有插件均由第三方开发者独立开发，<strong>MoFox 项目团队不承担任何责任</strong>。
                </p>
              </div>

              <!-- Warnings -->
              <div>
                <h3 :class="[
                  'text-sm sm:text-base font-semibold mb-2 sm:mb-3 flex items-center gap-1.5 sm:gap-2',
                  isDarkMode ? 'text-white' : 'text-gray-900'
                ]">
                  <Icon icon="mdi:clipboard-text" class="text-base sm:text-lg" />
                  使用前必读
                </h3>
                <ul class="space-y-1.5 sm:space-y-2">
                  <li v-for="(item, index) in [
                    '第三方插件可能存在安全风险：恶意代码、隐私泄露、系统崩溃',
                    '插件质量无法保证：功能缺陷、兼容性问题、法律风险',
                    '使用风险完全自担：MoFox 团队不提供技术支持或售后服务'
                  ]" :key="index" :class="[
                    'flex items-start gap-2 sm:gap-3 text-xs sm:text-sm',
                    isDarkMode ? 'text-gray-300' : 'text-gray-600'
                  ]">
                    <Icon icon="mdi:alert-circle" class="text-amber-500 flex-shrink-0 mt-0.5 text-sm sm:text-base" />
                    <span>{{ item }}</span>
                  </li>
                </ul>
              </div>

              <!-- Safety Tips -->
              <div>
                <h3 :class="[
                  'text-sm sm:text-base font-semibold mb-2 sm:mb-3 flex items-center gap-1.5 sm:gap-2',
                  isDarkMode ? 'text-white' : 'text-gray-900'
                ]">
                  <Icon icon="mdi:shield-check" class="text-base sm:text-lg" />
                  安全建议
                </h3>
                <div class="grid grid-cols-2 gap-2 sm:gap-3">
                  <div v-for="(item, index) in [
                    { icon: 'mdi:magnify', text: '仔细评估插件来源' },
                    { icon: 'mdi:file-document', text: '详细阅读权限要求' },
                    { icon: 'mdi:test-tube', text: '测试环境先试用' },
                    { icon: 'mdi:close-circle', text: '发现异常立即停用' }
                  ]" :key="index" :class="[
                    'flex items-center gap-1.5 sm:gap-2 p-2 sm:p-3 rounded-lg sm:rounded-xl text-xs sm:text-sm',
                    isDarkMode ? 'bg-dark-surface' : 'bg-light-surface'
                  ]">
                    <Icon :icon="item.icon" class="text-primary-500 text-sm sm:text-base flex-shrink-0" />
                    <span :class="isDarkMode ? 'text-gray-300' : 'text-gray-600'">{{ item.text }}</span>
                  </div>
                </div>
              </div>

              <!-- Contact -->
              <div>
                <h3 :class="[
                  'text-sm sm:text-base font-semibold mb-2 sm:mb-3 flex items-center gap-1.5 sm:gap-2',
                  isDarkMode ? 'text-white' : 'text-gray-900'
                ]">
                  <Icon icon="mdi:phone" class="text-base sm:text-lg" />
                  问题处理
                </h3>
                <ul class="space-y-1.5 sm:space-y-2">
                  <li v-for="(item, index) in [
                    '插件技术问题：请直接联系插件开发者',
                    '违规举报：通过官方渠道举报恶意插件',
                    '平台问题：联系 MoFox Bot 官方支持'
                  ]" :key="index" :class="[
                    'flex items-start gap-2 sm:gap-3 text-xs sm:text-sm',
                    isDarkMode ? 'text-gray-300' : 'text-gray-600'
                  ]">
                    <Icon icon="mdi:chevron-right" class="text-primary-500 flex-shrink-0 mt-0.5 text-sm sm:text-base" />
                    <span>{{ item }}</span>
                  </li>
                </ul>
              </div>

              <!-- Agreement -->
              <p :class="[
                'text-[10px] sm:text-xs leading-relaxed',
                isDarkMode ? 'text-gray-500' : 'text-gray-400'
              ]">
                💡 继续使用即表示您已阅读并同意《<a href="./PLUGIN_MARKET_NOTICE.md" class="text-primary-500 hover:underline" target="_blank">MoFox Bot 插件市场使用公告</a>》的全部条款。
              </p>
            </div>

            <!-- Footer -->
            <div :class="[
              'p-4 sm:p-6 border-t',
              isDarkMode ? 'bg-dark-surface border-dark-border' : 'bg-light-surface border-light-border'
            ]">
              <div class="flex flex-col sm:flex-row items-stretch sm:items-center justify-between gap-3 sm:gap-4">
                <label class="flex items-center gap-2 cursor-pointer order-2 sm:order-1">
                  <input 
                    type="checkbox" 
                    v-model="dontShowNoticeAgain"
                    :class="[
                      'w-4 h-4 rounded border-2 transition-colors cursor-pointer',
                      isDarkMode 
                        ? 'bg-dark-bg border-dark-border checked:bg-primary-500 checked:border-primary-500' 
                        : 'bg-white border-light-border checked:bg-primary-500 checked:border-primary-500'
                    ]"
                  />
                  <span :class="[
                    'text-xs sm:text-sm',
                    isDarkMode ? 'text-gray-400' : 'text-gray-500'
                  ]">我已知悉，下次不再提示</span>
                </label>
                
                <button 
                  @click="handleClose"
                  :disabled="!canCloseNotice"
                  :class="[
                    'w-full sm:w-auto px-6 sm:px-8 py-2.5 rounded-lg sm:rounded-xl text-sm font-medium transition-all duration-200 order-1 sm:order-2',
                    canCloseNotice
                      ? 'bg-primary-500 hover:bg-primary-600 text-white hover:shadow-glow'
                      : 'bg-gray-200 text-gray-400 cursor-not-allowed dark:bg-gray-800 dark:text-gray-600'
                  ]"
                >
                  {{ canCloseNotice ? '我已了解' : '请阅读完毕...' }}
                </button>
              </div>
            </div>
          </div>
        </div>
      </Transition>
    </div>
  </Transition>
</template>