<template>
  <button
    ref="_ref"
    :class="[
      ns.b(),
      ns.m(_type),
      ns.m(_size),
      ns.is('disabled', _disabled),
      ns.is('loading', loading),
      ns.is('plain', plain),
      ns.is('round', round),
      ns.is('circle', circle),
    ]"
    :disabled="_disabled || loading"
    :autofocus="autofocus"
    :type="nativeType"
    :style="buttonStyle"
    @click="handleClick"
  >
    <template v-if="loading">
      <slot v-if="$slots.loading" name="loading" />
      <el-icon v-else :class="ns.is('loading')">
        <component :is="loadingIcon" />
      </el-icon>
    </template>
    <el-icon v-else-if="icon || $slots.icon">
      <component :is="icon" v-if="icon" />
      <slot v-else name="icon" />
    </el-icon>
    <span
      v-if="$slots.default"
      :class="{ [ns.em('text', 'expand')]: shouldAddSpace }"
    >
      <slot />
    </span>
  </button>
</template>

<script lang="ts" setup>
import { computed, inject, Text, ref, useSlots } from 'vue'
import { TinyColor } from '@ctrl/tinycolor'
import { ElIcon } from '@element-plus/components/icon'
import {
  useDisabled,
  useFormItem,
  useGlobalConfig,
  useNamespace,
  useSize,
} from '@element-plus/hooks'
import { buttonGroupContextKey } from '@element-plus/tokens'
import { buttonEmits, buttonProps } from './button'

defineOptions({
  name: 'ElButton',
})

const props = defineProps(buttonProps)
const emit = defineEmits(buttonEmits)
const slots = useSlots()

const buttonGroupContext = inject(buttonGroupContextKey, undefined)
const globalConfig = useGlobalConfig('button')
const ns = useNamespace('button')
const { form } = useFormItem()
const _size = useSize(computed(() => buttonGroupContext?.size))
const _disabled = useDisabled()
const _ref = ref<HTMLButtonElement>()

const _type = computed(() => props.type || buttonGroupContext?.type || '')
const autoInsertSpace = computed(
  () => props.autoInsertSpace ?? globalConfig.value?.autoInsertSpace ?? false
)

// add space between two characters in Chinese
const shouldAddSpace = computed(() => {
  const defaultSlot = slots.default?.()
  if (autoInsertSpace.value && defaultSlot?.length === 1) {
    const slot = defaultSlot[0]
    if (slot?.type === Text) {
      const text = slot.children as string
      return /^\p{Unified_Ideograph}{2}$/u.test(text.trim())
    }
  }
  return false
})

// calculate hover & active color by custom color
// only work when custom color
const buttonStyle = computed(() => {
  let styles: Record<string, string> = {}

  const buttonColor = props.color

  if (buttonColor) {
    const color = new TinyColor(buttonColor)
    const shadeBgColor = color.shade(20).toString()
    if (props.plain) {
      styles = {
        '--el-button-bg-color': color.tint(90).toString(),
        '--el-button-text-color': buttonColor,
        '--el-button-hover-text-color': 'var(--el-color-white)',
        '--el-button-hover-bg-color': buttonColor,
        '--el-button-hover-border-color': buttonColor,
        '--el-button-active-bg-color': shadeBgColor,
        '--el-button-active-text-color': 'var(--el-color-white)',
        '--el-button-active-border-color': shadeBgColor,
      }
    } else {
      const tintBgColor = color.tint(30).toString()
      styles = {
        '--el-button-bg-color': buttonColor,
        '--el-button-border-color': buttonColor,
        '--el-button-hover-bg-color': tintBgColor,
        '--el-button-hover-border-color': tintBgColor,
        '--el-button-active-bg-color': shadeBgColor,
        '--el-button-active-border-color': shadeBgColor,
      }
    }

    if (_disabled.value) {
      const disabledButtonColor = color.tint(50).toString()
      styles['--el-button-disabled-bg-color'] = disabledButtonColor
      styles['--el-button-disabled-border-color'] = disabledButtonColor
    }
  }

  return styles
})

const handleClick = (evt: MouseEvent) => {
  if (props.nativeType === 'reset') {
    form?.resetFields()
  }
  emit('click', evt)
}

defineExpose({
  /** @description button html element */
  ref: _ref,
  /** @description button size */
  size: _size,
  /** @description button type */
  type: _type,
  /** @description button disabled */
  disabled: _disabled,
  /** @description whether adding space */
  shouldAddSpace,
})
</script>
