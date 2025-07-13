<template>
  <div>
    <h2>ココフォリアログファイルのアップロード</h2>
    <input type="file" accept=".html" @change="onFileChange" />
    <div v-if="error" style="color:red">{{ error }}</div>
    <div v-if="logParsed">
      <p>ログファイルを解析しました。自動的に次の画面へ進みます…</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const emit = defineEmits(['parsed'])
const error = ref('')
const logParsed = ref(false)
const parsedData = ref({
  characters: [] as { name: string; color: string }[],
  logs: [] as { tab: string; name: string; text: string; color: string }[],
})

function onFileChange(e: Event) {
  error.value = ''
  logParsed.value = false
  const file = (e.target as HTMLInputElement).files?.[0]
  if (!file) return
  if (!file.name.endsWith('.html')) {
    error.value = 'htmlファイルを選択してください。'
    return
  }
  const reader = new FileReader()
  reader.onload = function (ev) {
    const html = ev.target?.result as string
    try {
      const parser = new DOMParser()
      const doc = parser.parseFromString(html, 'text/html')
      // titleチェック
      const title = doc.querySelector('head > title')?.textContent || ''
      if (!title.includes('ccfolia - logs')) {
        error.value = 'ココフォリアのログファイルではありません（titleタグ不一致）'
        return
      }
      // pタグ構成チェック＆抽出
      const ps = Array.from(doc.body.querySelectorAll('p'))
      if (ps.length === 0) {
        error.value = 'pタグが見つかりません。'
        return
      }
      const logs: { tab: string; name: string; text: string; color: string }[] = []
      const charSet = new Map<string, string>()
      for (const p of ps) {
        const style = p.getAttribute('style') || ''
        const colorMatch = style.match(/color\s*:\s*([^;]+);?/)?.[1] || '#000'
        const spans = p.querySelectorAll('span')
        if (spans.length !== 3) continue
        const [tab, name, text] = Array.from(spans).map(s => s.textContent || '')
        logs.push({ tab, name, text, color: colorMatch })
        if (!charSet.has(name)) charSet.set(name, colorMatch)
      }
      parsedData.value = {
        characters: Array.from(charSet, ([name, color]) => ({ name, color })),
        logs,
      }
      logParsed.value = true
      emit('parsed', parsedData.value)
    } catch (e) {
      error.value = 'ファイル解析中にエラーが発生しました。'
    }
  }
  reader.readAsText(file)
}
</script>