<template>
  <span :class="ns.e('item')">
    <span ref="link" :class="[ns.e('inner'), ns.is('link', !!to)]" role="link">
      <slot />
    </span>
    <el-icon v-if="separatorIcon" :class="ns.e('separator')">
      <component :is="separatorIcon" />
    </el-icon>
    <span v-else :class="ns.e('separator')" role="presentation">
      {{ separator }}
    </span>
  </span>
</template>

<script lang="ts" setup>
import { inject, ref, onMounted, getCurrentInstance } from 'vue'
import ElIcon from '@element-plus/components/icon'
import { breadcrumbKey } from '@element-plus/tokens'
import { useNamespace } from '@element-plus/hooks'
import { breadcrumbItemProps } from './breadcrumb-item'

import type { Router } from 'vue-router'

defineOptions({
  name: 'ElBreadcrumbItem',
})

const props = defineProps(breadcrumbItemProps)

const instance = getCurrentInstance()!
const router = instance.appContext.config.globalProperties.$router as Router
const parent = inject(breadcrumbKey, undefined)
const ns = useNamespace('breadcrumb')

const { separator, separatorIcon } = parent ?? {}

const link = ref<HTMLSpanElement>()

onMounted(() => {
  link.value!.setAttribute('role', 'link')
  link.value!.addEventListener('click', () => {
    if (!props.to || !router) return
    props.replace ? router.replace(props.to) : router.push(props.to)
  })
})
</script>
