<template>
  <Teleport to="body">
    <div class="modal fixed inset-0 z-50 flex items-center justify-center p-4">
      <div class="modal-content mx-auto flex max-h-[90vh] w-full max-w-md flex-col p-8">
        <div class="mb-6 flex items-center justify-between">
          <div class="flex items-center gap-3">
            <div
              class="flex h-10 w-10 items-center justify-center rounded-xl bg-gradient-to-br from-amber-500 to-amber-600"
            >
              <i class="fas fa-sync-alt text-white" />
            </div>
            <h3 class="text-xl font-bold text-gray-900 dark:text-gray-100">
              {{ showResult ? '新密钥已生成' : '重置密钥' }}
            </h3>
          </div>
          <button
            class="text-gray-400 transition-colors hover:text-gray-600 dark:text-gray-500 dark:hover:text-gray-300"
            @click="handleClose"
          >
            <i class="fas fa-times text-xl" />
          </button>
        </div>

        <!-- 生成模式选择 -->
        <div v-if="!showResult" class="modal-scroll-content custom-scrollbar flex-1 space-y-6">
          <!-- 警告提示 -->
          <div
            class="rounded-lg border border-amber-200 bg-amber-50 p-4 dark:border-amber-700 dark:bg-amber-900/20"
          >
            <div class="flex items-start gap-3">
              <div
                class="flex h-8 w-8 flex-shrink-0 items-center justify-center rounded-lg bg-amber-500"
              >
                <i class="fas fa-exclamation-triangle text-sm text-white" />
              </div>
              <div>
                <h4 class="mb-1 font-semibold text-gray-800 dark:text-amber-400">API Key 信息</h4>
                <p class="text-sm text-gray-700 dark:text-gray-300">
                  {{ apiKey.name }}
                </p>
                <p class="mt-1 text-xs text-red-600 dark:text-red-400">
                  重置后旧密钥将立即失效，使用该密钥的所有客户端需要更新配置。
                </p>
              </div>
            </div>
          </div>

          <!-- 模式选择 -->
          <div>
            <label class="mb-3 block text-sm font-semibold text-gray-700 dark:text-gray-300"
              >重置方式</label
            >
            <div class="space-y-2">
              <label
                class="flex cursor-pointer items-center gap-3 rounded-lg border border-gray-200 p-3 transition-colors hover:bg-gray-50 dark:border-gray-600 dark:hover:bg-gray-700/50"
                :class="{
                  'border-blue-500 bg-blue-50 dark:border-blue-400 dark:bg-blue-900/20':
                    mode === 'random'
                }"
              >
                <input
                  v-model="mode"
                  class="text-blue-500"
                  name="regenerateMode"
                  type="radio"
                  value="random"
                />
                <div>
                  <p class="font-medium text-gray-900 dark:text-gray-100">随机生成</p>
                  <p class="text-xs text-gray-500 dark:text-gray-400">系统自动生成安全的随机密钥</p>
                </div>
              </label>
              <label
                class="flex cursor-pointer items-center gap-3 rounded-lg border border-gray-200 p-3 transition-colors hover:bg-gray-50 dark:border-gray-600 dark:hover:bg-gray-700/50"
                :class="{
                  'border-blue-500 bg-blue-50 dark:border-blue-400 dark:bg-blue-900/20':
                    mode === 'custom'
                }"
              >
                <input
                  v-model="mode"
                  class="text-blue-500"
                  name="regenerateMode"
                  type="radio"
                  value="custom"
                />
                <div>
                  <p class="font-medium text-gray-900 dark:text-gray-100">自定义密钥</p>
                  <p class="text-xs text-gray-500 dark:text-gray-400">手动输入自定义的密钥字符串</p>
                </div>
              </label>
            </div>
          </div>

          <!-- 自定义输入框 -->
          <div v-if="mode === 'custom'">
            <label class="mb-2 block text-sm font-semibold text-gray-700 dark:text-gray-300"
              >自定义密钥</label
            >
            <div class="flex items-center gap-2">
              <span
                class="rounded-lg bg-gray-100 px-3 py-2.5 font-mono text-sm text-gray-500 dark:bg-gray-700 dark:text-gray-400"
                >{{ prefix }}</span
              >
              <input
                v-model="customKeyBody"
                class="form-input flex-1 font-mono"
                placeholder="输入至少16位字符"
                type="text"
              />
            </div>
            <p v-if="customKeyError" class="mt-1 text-xs text-red-500">{{ customKeyError }}</p>
            <p v-else class="mt-1 text-xs text-gray-500 dark:text-gray-400">
              密钥主体至少需要 16 个字符，系统会自动添加 "{{ prefix }}" 前缀
            </p>
          </div>
        </div>

        <!-- 结果显示 -->
        <div v-else class="modal-scroll-content custom-scrollbar flex-1 space-y-6">
          <div
            class="border-l-4 border-amber-400 bg-amber-50 p-4 dark:border-amber-500 dark:bg-amber-900/20"
          >
            <div class="flex items-start">
              <div
                class="mt-0.5 flex h-6 w-6 flex-shrink-0 items-center justify-center rounded-lg bg-amber-400 dark:bg-amber-500"
              >
                <i class="fas fa-exclamation-triangle text-sm text-white" />
              </div>
              <div class="ml-3">
                <h5 class="mb-1 font-semibold text-amber-900 dark:text-amber-400">重要提醒</h5>
                <p class="text-sm text-amber-800 dark:text-amber-300">
                  这是您唯一能看到完整新密钥的机会。关闭此窗口后将不再显示。请立即复制并妥善保存。
                </p>
              </div>
            </div>
          </div>

          <div>
            <label class="mb-2 block text-sm font-semibold text-gray-700 dark:text-gray-300"
              >新的 API Key</label
            >
            <div class="relative">
              <div
                class="flex min-h-[60px] items-center break-all rounded-lg border border-gray-700 bg-gray-900 p-4 pr-14 font-mono text-sm text-white dark:border-gray-600 dark:bg-gray-900"
              >
                {{ showFullKey ? newKey : maskKey(newKey) }}
              </div>
              <div class="absolute right-3 top-3">
                <button
                  class="btn-icon-sm bg-gray-700 hover:bg-gray-800 dark:bg-gray-700 dark:hover:bg-gray-600"
                  :title="showFullKey ? '隐藏API Key' : '显示完整API Key'"
                  type="button"
                  @click="showFullKey = !showFullKey"
                >
                  <i :class="['fas', showFullKey ? 'fa-eye-slash' : 'fa-eye', 'text-gray-300']" />
                </button>
              </div>
            </div>
          </div>

          <div class="flex flex-col gap-3 sm:flex-row sm:gap-4">
            <button
              class="flex w-full items-center justify-center gap-2 rounded-xl border border-blue-200 bg-blue-50 px-5 py-3 text-sm font-semibold text-blue-700 transition-colors hover:border-blue-300 hover:bg-blue-100 dark:border-blue-500/50 dark:bg-blue-500/10 dark:text-blue-200 dark:hover:bg-blue-500/20 sm:flex-1 sm:text-base"
              @click="copyKeyOnly"
            >
              <i class="fas fa-key" />
              仅复制密钥
            </button>
            <button
              class="btn btn-primary flex w-full items-center justify-center gap-2 px-5 py-3 text-sm font-semibold sm:flex-1 sm:text-base"
              @click="copyFullConfig"
            >
              <i class="fas fa-copy" />
              复制Claude配置
            </button>
          </div>
        </div>

        <!-- 操作按钮（生成前） -->
        <div v-if="!showResult" class="flex gap-3 pt-4">
          <button
            class="flex-1 rounded-xl bg-gray-100 px-6 py-3 font-semibold text-gray-700 transition-colors hover:bg-gray-200 dark:bg-gray-700 dark:text-gray-300 dark:hover:bg-gray-600"
            type="button"
            @click="handleClose"
          >
            取消
          </button>
          <button
            class="btn btn-primary flex-1 px-6 py-3 font-semibold"
            :disabled="loading || (mode === 'custom' && !!customKeyError)"
            type="button"
            @click="handleRegenerate"
          >
            <div v-if="loading" class="loading-spinner mr-2" />
            <i v-else class="fas fa-sync-alt mr-2" />
            {{ loading ? '处理中...' : '确认重置' }}
          </button>
        </div>

        <!-- 关闭按钮（生成后） -->
        <div v-else class="pt-4">
          <button
            class="flex w-full items-center justify-center gap-2 rounded-xl border border-gray-300 bg-gray-200 px-5 py-3 text-sm font-semibold text-gray-800 transition-colors hover:border-gray-400 hover:bg-gray-300 dark:border-gray-600 dark:bg-gray-700 dark:text-gray-200 dark:hover:bg-gray-600 sm:text-base"
            @click="handleResultClose"
          >
            <i class="fas fa-check-circle" />
            我已保存
          </button>
        </div>
      </div>
    </div>

    <ConfirmModal
      :cancel-text="confirmModalConfig.cancelText"
      :confirm-text="confirmModalConfig.confirmText"
      :message="confirmModalConfig.message"
      :show="showConfirmModal"
      :title="confirmModalConfig.title"
      :type="confirmModalConfig.type"
      @cancel="handleCancelModal"
      @confirm="handleConfirmModal"
    />
  </Teleport>
</template>

<script setup>
import { ref, computed } from 'vue'
import { showToast } from '@/utils/tools'
import * as httpApis from '@/utils/http_apis'
import ConfirmModal from '@/components/common/ConfirmModal.vue'

const props = defineProps({
  apiKey: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['close', 'success'])

const loading = ref(false)
const mode = ref('random')
const customKeyBody = ref('')
const showResult = ref(false)
const newKey = ref('')
const showFullKey = ref(false)
const prefix = ref(import.meta.env.VITE_API_KEY_PREFIX || 'cr_')

// ConfirmModal 状态
const showConfirmModal = ref(false)
const confirmModalConfig = ref({
  title: '',
  message: '',
  type: 'primary',
  confirmText: '确认',
  cancelText: '取消'
})
const confirmResolve = ref(null)

const showConfirm = (
  title,
  message,
  confirmText = '确认',
  cancelText = '取消',
  type = 'primary'
) => {
  return new Promise((resolve) => {
    confirmModalConfig.value = { title, message, confirmText, cancelText, type }
    confirmResolve.value = resolve
    showConfirmModal.value = true
  })
}
const handleConfirmModal = () => {
  showConfirmModal.value = false
  confirmResolve.value?.(true)
}
const handleCancelModal = () => {
  showConfirmModal.value = false
  confirmResolve.value?.(false)
}

// 自定义key校验
const customKeyError = computed(() => {
  if (mode.value !== 'custom') return ''
  if (!customKeyBody.value) return '请输入密钥'
  if (customKeyBody.value.length < 16) return '密钥主体至少需要 16 个字符'
  return ''
})

// 密钥脱敏
const maskKey = (key) => {
  if (!key || key.length <= 12) return key
  return (
    key.substring(0, 8) + '●'.repeat(Math.max(0, key.length - 12)) + key.substring(key.length - 4)
  )
}

// 获取 API Base URL
const getBaseUrlPrefix = () => {
  const customPrefix = import.meta.env.VITE_API_BASE_PREFIX
  if (customPrefix) return customPrefix.replace(/\/$/, '')
  if (typeof window !== 'undefined') {
    return window.location.protocol + '//' + window.location.host
  }
  return ''
}

// 复制工具
const copyTextWithFallback = async (text, successMessage) => {
  try {
    await navigator.clipboard.writeText(text)
    showToast(successMessage, 'success')
  } catch (_error) {
    const textArea = document.createElement('textarea')
    textArea.value = text
    document.body.appendChild(textArea)
    textArea.select()
    try {
      document.execCommand('copy')
      showToast(successMessage, 'success')
    } catch (_fallbackError) {
      showToast('复制失败，请手动复制', 'error')
    } finally {
      document.body.removeChild(textArea)
    }
  }
}

const copyKeyOnly = async () => {
  await copyTextWithFallback(newKey.value, 'API Key 已复制')
}

const copyFullConfig = async () => {
  const baseUrl = getBaseUrlPrefix() + '/api'
  const configText = `export ANTHROPIC_BASE_URL="${baseUrl}"\nexport ANTHROPIC_AUTH_TOKEN="${newKey.value}"`
  await copyTextWithFallback(configText, '配置信息已复制到剪贴板')
}

// 执行重置
const handleRegenerate = async () => {
  const confirmed = await showConfirm(
    '确认重置密钥',
    `确定要重置 "${props.apiKey.name}" 的密钥吗？\n\n旧密钥将立即失效，所有使用该密钥的客户端需要更新配置。`,
    '确认重置',
    '取消',
    'warning'
  )
  if (!confirmed) return

  loading.value = true
  try {
    const data = mode.value === 'custom' ? { customKey: customKeyBody.value } : {}
    const result = await httpApis.regenerateApiKeyApi(props.apiKey.id, data)

    if (result.success) {
      newKey.value = result.key
      showResult.value = true
      emit('success')
    } else {
      showToast(result.message || '重置失败', 'error')
    }
  } catch (error) {
    const msg = error?.response?.data?.message || error?.message || '重置失败'
    showToast(msg, 'error')
  } finally {
    loading.value = false
  }
}

const handleClose = () => {
  emit('close')
}

const handleResultClose = async () => {
  const confirmed = await showConfirm(
    '关闭提醒',
    '关闭后将无法再次查看完整的新密钥，请确保已经妥善保存。\n\n确定要关闭吗？',
    '确定关闭',
    '取消',
    'warning'
  )
  if (confirmed) {
    emit('close')
  }
}
</script>
