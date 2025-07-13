<template>
  <div>
    <h2>ログ編集画面</h2>
    <div v-if="tabs.length">
      <div class="tab-select">
        <label v-for="tab in tabs" :key="tab">
          <input type="radio" name="tab" :value="tab" v-model="selectedTab" />
          {{ tab }}
        </label>
      </div>
      <draggable v-model="filteredLogs" item-key="id" class="log-list">
        <template #item="{ element }">
          <div class="log-row">
            <div class="icon-area">
              <img :src="iconMap[element.name] || defaultIcon" alt="icon" />
            </div>
            <div class="log-content left-align">
              <div class="char-name" :style="{ color: element.color }">{{ element.name }}</div>
              <div class="log-text">{{ element.text }}</div>
            </div>
          </div>
        </template>
      </draggable>
      <button @click="exportHtml">エクスポート</button>
    </div>
    <div v-else>
      <p>ログデータがありません。</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'
import draggable from 'vuedraggable'

const props = defineProps<{
  logs: { tab: string; name: string; text: string; color: string; id: number }[]
  iconMap: Record<string, string>
}>()
const emit = defineEmits(['update'])

const tabs = computed(() => Array.from(new Set(props.logs.map(l => l.tab))))
const selectedTab = ref('')
const defaultIcon = 'data:image/svg+xml;base64,' + btoa('<svg width="200" height="200" xmlns="http://www.w3.org/2000/svg"><rect width="200" height="200" fill="#222"/></svg>')

const filteredLogs = ref(props.logs.filter(l => l.tab === tabs.value[0]))

watch(selectedTab, (tab) => {
  filteredLogs.value = props.logs.filter(l => l.tab === tab)
})

onMounted(() => {
  selectedTab.value = tabs.value[0] || ''
})

function exportHtml() {
  // 画像をbase64で埋め込み、CSSも含めて1ファイルのhtmlを生成
  const css = `<style>
    .log-row { display: flex; align-items: center; margin-bottom: 16px; }
    .icon-area { width: 200px; height: 200px; flex-shrink: 0; display: flex; align-items: center; justify-content: center; }
    .icon-area img { width: 200px; height: 200px; object-fit: cover; border-radius: 8px; background: #222; }
    .log-content { border: 1px solid #ccc; border-radius: 8px; padding: 16px; margin-left: 16px; background: #fff; min-width: 0; flex: 1; }
    .char-name { font-weight: bold; font-size: 1.2em; margin-bottom: 8px; }
    .log-text { color: #222; }
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
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 16px;
  margin-left: 16px;
  background: #fff;
  min-width: 0;
  flex: 1;
}
.log-content.left-align {
  text-align: left;
}
.char-name {
  font-weight: bold;
  font-size: 1.2em;
  margin-bottom: 8px;
  text-align: left;
}
.log-text {
  color: #222;
  text-align: left;
}
</style>
