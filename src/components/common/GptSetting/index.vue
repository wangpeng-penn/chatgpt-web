<script lang="ts" setup>
import { computed, ref } from 'vue'
import { NButton, NInputNumber, NModal } from 'naive-ui'

interface Props {
  visible: boolean
  gtpModel: string
  gtpModelTitle: string
}

interface Emit {
  (e: 'update:visible', visible: boolean): void
}

const props = defineProps<Props>()

const emit = defineEmits<Emit>()

const show = computed({
  get() {
    return props.visible
  },
  set(visible: boolean) {
    emit('update:visible', visible)
  },
})

let gptSettings = {
  max_token: props.gtpModel === '' ? 7680 : 3584,
  temperature: 0.9,
  presence_penalty: 0.6,
  frequency_penalty: 0.0,
}
const strGptSettings = localStorage.getItem(`setting.gpt.${props.gtpModel}`)
if (strGptSettings)
  gptSettings = JSON.parse(strGptSettings)

const gptMaxToken = ref(gptSettings.max_token)
const gptTemperature = ref(gptSettings.temperature)
const gptPresencePenalty = ref(gptSettings.presence_penalty)
const gptFrequencyPenalty = ref(gptSettings.frequency_penalty)

function onReset() {
  gptMaxToken.value = 3584
  gptTemperature.value = 0.9
  gptPresencePenalty.value = 0.6
  gptFrequencyPenalty.value = 0.0
}

function onSaveClick() {
  localStorage.setItem(`setting.gpt.${props.gtpModel}`, JSON.stringify({
    max_token: gptMaxToken.value,
    temperature: gptTemperature.value,
    presence_penalty: gptPresencePenalty.value,
    frequency_penalty: gptFrequencyPenalty.value,
  }))
}
</script>

<template>
  <NModal v-model:show="show" :auto-focus="false" preset="dialog" :mask-closable="false" :closable="false" style="width: 95%; max-width: 640px" negative-text="取消" positive-text="保存" @positive-click="onSaveClick">
    <template #header>
      <div>【{{ props.gtpModelTitle }}】参数设置</div>
    </template>
    <div class="p-4 space-y-5 min-h-[200px]">
      <div class="space-y-6">
        <div class="flex items-center space-x-4">
          <span class="flex-shrink-0 w-[90px]">最大标记数</span>
          <div class="w-[100px]">
            <NInputNumber v-model:value="gptMaxToken" placeholder="" :step="1" :min="1024" :max="8192" />
          </div>
          <span class="text-xs text-neutral-400 w-[320px]">分析内容最多支持的标记数</span>
        </div>
        <div class="flex items-center space-x-4">
          <span class="flex-shrink-0 w-[90px]">采样温度系数</span>
          <div class="w-[100px]">
            <NInputNumber v-model:value="gptTemperature" placeholder="" :step="0.1" :min="0.0" :max="2.0" />
          </div>
          <span class="text-xs text-neutral-400 w-[320px]">取值[0.0~2.0]。值越高（如0.8），输出越随机，而值越低（如0.2），输出就越集中，确定性更强。</span>
        </div>
        <div class="flex items-center space-x-4">
          <span class="flex-shrink-0 w-[90px]">推新判定值</span>
          <div class="w-[100px]">
            <NInputNumber v-model:value="gptPresencePenalty" placeholder="" :step="0.1" :min="-2.0" :max="2.0" />
          </div>
          <span class="text-xs text-neutral-400 w-[320px]">取值[-2.0~2.0]。正值会根据到目前为止新标记是否出现在文本中来判定，从而增加模型谈论新主题的可能性。</span>
        </div>
        <div class="flex items-center space-x-4">
          <span class="flex-shrink-0 w-[90px]">频率判定值</span>
          <div class="w-[100px]">
            <NInputNumber v-model:value="gptFrequencyPenalty" placeholder="" :step="0.1" :min="-2.0" :max="2.0" />
          </div>
          <span class="text-xs text-neutral-400 w-[320px]">取值[-2.0~2.0]。正值会根据新标记在文本中的现有频率对其进行判定，从而降低模型逐字重复同一行的可能性。</span>
        </div>
        <div class="flex items-center space-x-4">
          <NButton size="small" @click="onReset">
            重置
          </NButton>
        </div>
      </div>
    </div>
  </NModal>
</template>
