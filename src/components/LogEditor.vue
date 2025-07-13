<template>
  <div>
    <h2>ログ編集画面</h2>
    <div class="button-row">
      <button @click="emitBack">キャラクターアイコン画像設定画面に戻る</button>
      <button @click="exportHtml" style="margin-left:1em;">エクスポート</button>
    </div>
      <div style="margin-bottom: 32px;"></div>
    <div v-if="tabs.length">
      <div class="tab-select">
        <label v-for="tab in tabs" :key="tab">
          <input type="radio" name="tab" :value="tab" v-model="selectedTab" />
          {{ tab }}
        </label>
      </div>
      <div class="color-controls">
        <label>
          ページ背景色: <input type="color" v-model="pageBgColor" />
          <input type="text" v-model="pageBgColor" style="width:90px;" />
        </label>
        <label>
          セリフ枠背景色: <input type="color" v-model="bubbleBgColor" />
          <input type="text" v-model="bubbleBgColor" style="width:90px;" />
        </label>
        <label>
          セリフ枠線色: <input type="color" v-model="bubbleBorderColor" />
          <input type="text" v-model="bubbleBorderColor" style="width:90px;" />
        </label>
        <label>
          セリフ文字色: <input type="color" v-model="bubbleTextColor" />
          <input type="text" v-model="bubbleTextColor" style="width:90px;" />
        </label>
      </div>
      <draggable v-model="filteredLogs" item-key="id" class="log-list">
        <template #item="{ element }">
          <div class="log-row">
            <div class="icon-area">
              <img :src="iconMap[element.name] || defaultIcon" alt="icon" />
            </div>
            <div class="log-content left-align">
              <button class="delete-btn" @click="deleteLog(element.id)" title="この行を削除" type="button">×</button>
              <div class="char-name" :style="{ color: charColorMap[element.name] || element.color }">{{ element.name }}</div>
              <div class="log-text">{{ element.text }}</div>
            </div>
          </div>
        </template>
      </draggable>
    </div>
    <div v-else>
      <p>ログデータがありません。</p>
    </div>
  </div>
</template>

<script setup lang="ts">
// ログ行の削除
function deleteLog(id: number) {
  const idx = filteredLogs.value.findIndex(l => l.id === id)
  if (idx !== -1) {
    filteredLogs.value.splice(idx, 1)
  }
}
import { ref, computed, watch, onMounted } from 'vue'
import draggable from 'vuedraggable'

const props = defineProps<{
  logs: { tab: string; name: string; text: string; color: string; id: number }[]
  iconMap: Record<string, string>
  characters: { name: string; color: string }[]
}>()
const emit = defineEmits(['update', 'back'])
function emitBack() {
  emit('back')
}


const tabs = computed(() => Array.from(new Set(props.logs.map(l => l.tab))))
const selectedTab = ref('')
const defaultIcon = 'data:image/svg+xml;base64,' + btoa('<svg width="200" height="200" xmlns="http://www.w3.org/2000/svg"><rect width="200" height="200" fill="#222"/></svg>')

const filteredLogs = ref(props.logs.filter(l => l.tab === tabs.value[0]))


// 色設定（デフォルトは現状のCSS値）
const pageBgColor = ref('#242424')
const bubbleBgColor = ref('#ffffff')
const bubbleBorderColor = ref('#000000')
const bubbleTextColor = ref('#333333')

onMounted(() => {
  selectedTab.value = tabs.value[0] || ''
})

watch(selectedTab, (tab) => {
  filteredLogs.value = props.logs.filter(l => l.tab === tab)
})

watch(pageBgColor, (val) => {
  document.body.style.backgroundColor = val
})
watch(bubbleBgColor, (val) => {
  document.documentElement.style.setProperty('--bubble-bg', val)
})
watch(bubbleBorderColor, (val) => {
  document.documentElement.style.setProperty('--bubble-border', val)
})
watch(bubbleTextColor, (val) => {
  document.documentElement.style.setProperty('--bubble-text', val)
})

function exportHtml() {
  // 画像をbase64で埋め込み、CSS・背景色・バブル色・画像サイズを現状に合わせて1ファイルのhtmlを生成
  const css = `<style>
    body { background: ${pageBgColor.value}; }
    .log-row { display: flex; align-items: center; margin-bottom: 16px; }
    .icon-area { width: 100px; height: 100px; flex-shrink: 0; display: flex; align-items: center; justify-content: center; }
    .icon-area img { width: 100px; height: 100px; object-fit: cover; border-radius: 8px; background: #222; }
    .log-content { border: 1px solid ${bubbleBorderColor.value}; border-radius: 8px; padding: 16px; margin-left: 16px; background: ${bubbleBgColor.value}; min-width: 0; flex: 1; text-align: left; }
    .char-name { font-weight: bold; font-size: 1.2em; margin-bottom: 8px; text-align: left; }
    .log-text { color: ${bubbleTextColor.value}; text-align: left; }
  </style>`
  const html = `<!DOCTYPE html><html><head><meta charset='utf-8'><title>cocologtool export</title>${css}</head><body>` +
    filteredLogs.value.map(l => {
      const icon = props.iconMap[l.name] || defaultIcon
      return `<div class='log-row'><div class='icon-area'><img src='${icon}' /></div><div class='log-content'><div class='char-name' style='color:${l.color}'>${l.name}</div><div class='log-text'>${l.text}</div></div></div>`
    }).join('') + '</body></html>'
  const blob = new Blob([html], { type: 'text/html' })
  const a = document.createElement('a')
  a.href = URL.createObjectURL(blob)
  a.download = 'cocologtool_export.html'
  a.click()
  URL.revokeObjectURL(a.href)
}

const charColorMap = computed(() => {
  const map: Record<string, string> = {};
  for (const c of props.characters) {
    map[c.name] = c.color;
  }
  return map;
});
</script>

<style scoped>
.tab-select {
  margin-bottom: 16px;
}
.tab-select label {
  margin-right: 16px;
  font-weight: bold;
}
.log-list {
  margin-bottom: 24px;
}
.log-row {
  display: flex;
  align-items: center;
  margin-bottom: 16px;
}
.icon-area {
  width: 100px;
  height: 100px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}
.icon-area img {
  width: 100px;
  height: 100px;
  object-fit: cover;
  border-radius: 8px;
  background: #222;
}

.log-content {
  border: 1px solid var(--bubble-border, #ccc);
  border-radius: 8px;
  padding: 16px;
  margin-left: 16px;
  background: var(--bubble-bg, #fff);
  min-width: 0;
  flex: 1;
}

.color-controls {
  margin-bottom: 16px;
  display: flex;
  align-items: center;
  gap: 1em;
  justify-content: center;
}
.log-content.left-align {
  text-align: left;
}
.delete-btn {
  position: absolute;
  top: 4px;
  right: 8px;
  background: transparent;
  border: none;
  color: #888;
  font-size: 1.1em;
  cursor: pointer;
  padding: 0;
  line-height: 1;
  z-index: 2;
  transition: color 0.2s;
}
.delete-btn:hover {
  color: #e00;
}
.log-content {
  position: relative;
}
.char-name {
  font-weight: bold;
  font-size: 1.0em;
  margin-bottom: 8px;
  text-align: left;
}
.log-text {
  color: var(--bubble-text, #222);
  text-align: left;
}
.button-row {
  margin-bottom: 24px;
}
</style>
