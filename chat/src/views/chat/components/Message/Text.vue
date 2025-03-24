<script lang="ts" setup>
import { computed, ref } from 'vue'
import MarkdownIt from 'markdown-it'
import mdKatex from '@traptitech/markdown-it-katex'
import mila from 'markdown-it-link-attributes'
import hljs from 'highlight.js'
import { NButton, NIcon } from 'naive-ui'
import { Copy, Delete, Refresh } from '@icon-park/vue-next'
import { SvgIcon } from '@/components/common'

import { useBasicLayout } from '@/hooks/useBasicLayout'
import { t } from '@/locales'
interface Props {
  inversion?: boolean
  error?: boolean
  text?: string
  loading?: boolean
  asRawText?: boolean
}

interface Emit {
  (ev: 'regenerate'): void
  (ev: 'delete'): void
  (ev: 'copy'): void
}

const props = defineProps<Props>()

const emit = defineEmits<Emit>()

const { isMobile } = useBasicLayout()

const textRef = ref<HTMLElement>()

const mdi = new MarkdownIt({
  linkify: true,
  highlight(code, language) {
    const validLang = !!(language && hljs.getLanguage(language))
    if (validLang) {
      const lang = language ?? ''
      return highlightBlock(hljs.highlight(code, { language: lang }).value, lang)
    }
    return highlightBlock(hljs.highlightAuto(code).value, '')
  },
})

mdi.use(mila, { attrs: { target: '_blank', rel: 'noopener' } })
mdi.use(mdKatex, { blockClass: 'katexmath-block rounded-md p-[10px]', errorColor: ' #cc0000' })

const wrapClass = computed(() => {
  return [
    'text-wrap',
    'min-w-[20px]',
    'rounded-md',
    isMobile.value ? 'p-2' : 'px-3 py-2',
    props.inversion ? 'bg-[#5A91fc]' : 'bg-[#f4f6f8]',
    props.inversion ? 'text-[#fff]' : 'text-[#000]',
    props.inversion ? 'dark:bg-[#69af6b]' : 'dark:bg-[#1e1e20]',
    props.inversion ? 'message-request' : 'message-reply',
    { 'text-red-500': props.error },
  ]
})

const text = computed(() => {
  const value = props.text ?? ''
  if (!props.asRawText)
    return mdi.render(value)
  return value
})

function highlightBlock(str: string, lang?: string) {
  return `<pre class="code-block-wrapper"><div class="code-block-header"><span class="code-block-header__lang">${lang}</span><span class="code-block-header__copy">${t('chat.copyCode')}</span></div><code class="hljs code-block-body ${lang}">${str}</code></pre>`
}
function handleRegenerate() {
  emit('regenerate')
}

function handleCopy() {
  emit('copy')
}

function handleDelete() {
  emit('delete')
}

defineExpose({ textRef })
</script>

<template>
  <div :class="wrapClass" class="w-full">
    <div ref="textRef" class="leading-relaxed break-words">
      <div v-if="!inversion" class="flex flex-col items-start">
        <div class="w-full">
          <div v-if="!asRawText" class="w-full markdown-body" :class="[{ 'markdown-body-generate': loading }]" v-html="text" />
          <div v-else class="w-full whitespace-pre-wrap" v-text="text" />
          <!-- <span v-if="loading" class="dark:text-white w-[4px] h-[20px] block animate-blink" /> -->
        </div>
        <div class="mt-3">
          <NButton class="ml-2" text type="primary" @click="handleCopy">
            <template #icon>
              <NIcon :size="10" :component="Copy" />
            </template>
            <span class="text-xs">复制</span>
          </NButton>

          <span class="ml-4" />
          <NButton class="ml-2" text type="primary" @click="asRawText = !asRawText">
            <template #icon>
              <SvgIcon class="text-xs" :icon=" asRawText ? 'ic:outline-code-off' : 'ic:outline-code'" />
            </template>
            <span class="text-xs">{{ asRawText ? t('chat.preview') : t('chat.showRawText') }}</span>
          </NButton>

          <span class="ml-4" />
          <NButton text type="primary" @click="handleDelete">
            <template #icon>
              <NIcon :size="10" :component="Delete" />
            </template>
            <span class="text-xs">删除</span>
          </NButton>

          <span class="ml-4" />
          <NButton text type="primary" @click="handleRegenerate">
            <template #icon>
              <NIcon :size="10" :component="Refresh" />
            </template>
            <span class="text-xs">重新回答</span>
          </NButton>
        </div>
      </div>
      <div v-else>
        <div class="whitespace-pre-wrap" v-text="text" />
        <div v-if="false" class="mt-1">
          <NButton class="ml-2" text color="#FFF" @click="handleCopy">
            <template #icon>
              <NIcon :size="10" :component="Copy" />
            </template>

            <span class="text-xs">复制</span>
          </NButton>
          <span class="ml-3" />
          <NButton text color="#FFF" @click="handleDelete">
            <template #icon>
              <NIcon :size="10" :component="Delete" />
            </template>
            <span class="text-xs">删除</span>
          </NButton>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="less">
@import url(./style.less);
</style>
