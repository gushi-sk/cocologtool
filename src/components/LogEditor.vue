<template>
  <div>
    <h2>ログ編集画面</h2>
    <div class="button-row">
      <button @click="emitBack">キャラクターアイコン画像設定画面に戻る</button>
      <button @click="exportHtml" style="margin-left:1em;">エクスポート</button>
    </div>
    <div style="margin-bottom: 32px;"></div>
    <div class="log-title-row">
      <label>
        ログタイトル：
        <input type="text" v-model="logTitle" style="width: 60%; font-size: 1.1em; margin-left: 0.5em;"
          placeholder="タイトルを入力" />
      </label>
    </div>
    <div style="margin-bottom: 32px;"></div>
    <div v-if="tabs.length">
      <div class="tab-select-row">
        <div class="tab-select">
          <label v-for="tab in tabs" :key="tab">
            <input type="radio" name="tab" :value="tab" v-model="selectedTab" @change="onTabChange" />
            {{ tab }}
          </label>
        </div>
        <div v-if="tabs.length > 1" class="add-tab-btn-area">
          <button @click="toggleAddTab" type="button">表示するタブを追加</button>
        </div>
      </div>
      <div v-if="showAddTab && tabs.length > 1" class="add-tab-list">
        <label v-for="tab in tabs.filter(t => t !== selectedTab)" :key="tab" class="add-tab-checkbox">
          <input type="checkbox" :value="tab" v-model="addedTabs" /> {{ tab }}
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
      <draggable v-model="filteredLogs" item-key="id" class="log-list" @end="onDragEnd">
        <template #item="{ element }">
          <div>
            <div v-if="element.tab === selectedTab" class="log-row">
              <div class="icon-area">
                <img :src="iconMap[element.name] || defaultIcon" alt="icon" />
              </div>
              <div class="log-content left-align">
                <button class="delete-btn" @click="deleteLog(element.id)" title="この行を削除" type="button">×</button>
                <div class="char-name" :style="{ color: charColorMap[element.name] || element.color }">{{ element.name
                  }}</div>
                <div class="log-text" v-html="formatText(element.text)"></div>
              </div>
            </div>
            <template v-for="tab in addedTabs">
              <div v-if="element.tab === tab" class="log-row added-tab-row">
                <div class="added-tab-empty"></div>
                <div class="log-content added-tab-content">
                  <button class="delete-btn" @click="deleteLog(element.id)" title="この行を削除" type="button">×</button>
                  <div class="char-name" :style="{ color: charColorMap[element.name] || element.color }">
                    {{ element.name }}
                    <span class="added-tab-label" :style="{ color: bubbleTextColor }">{{ tab }}</span>
                  </div>
                  <div class="log-text" v-html="formatText(element.text)"></div>
                </div>
              </div>
            </template>
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
  // filteredLogsから該当idを除く
  const idx = filteredLogs.value.findIndex(l => l.id === id)
  if (idx !== -1) {
    filteredLogs.value.splice(idx, 1)
    emit('update', [...filteredLogs.value])
  }
}

function onDragEnd() {
  // 並べ替え後にemit
  emit('update', [...filteredLogs.value])
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
const showAddTab = ref(false)
const addedTabs = ref<string[]>([])
const defaultIcon = 'data:image/svg+xml;base64,' + btoa('<svg width="200" height="200" xmlns="http://www.w3.org/2000/svg"><rect width="200" height="200" fill="#222"/></svg>')


const logTitle = ref('')
const filteredLogs = ref([...props.logs])


// 色設定（デフォルトは現状のCSS値）

function onTabChange() {
  addedTabs.value = [];
  showAddTab.value = false;
}

function toggleAddTab() {
  if (showAddTab.value) {
    // すでに表示中なら閉じる＋選択クリア
    showAddTab.value = false;
    addedTabs.value = [];
  } else {
    showAddTab.value = true;
  }
}
const pageBgColor = ref('#242424')
const bubbleBgColor = ref('#ffffff')
const bubbleBorderColor = ref('#000000')
const bubbleTextColor = ref('#333333')

onMounted(() => {
  selectedTab.value = tabs.value[0] || ''
})


// タブ切り替え時はprops.logsの順序を維持
watch(
  () => [props.logs, selectedTab.value, addedTabs.value],
  () => {
    filteredLogs.value = [...props.logs]
  },
  { deep: true }
)

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


function escapeHtml(str: string) {
  // <br>は一時的にプレースホルダに置換してエスケープ後に戻す
  return String(str)
    .replace(/<br\s*\/?>/gi, '[[BR]]')
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;')
    .replace(/\[\[BR\]\]/g, '<br>');
}

function formatText(text: string) {
  // <br>はそのまま、他はescape
  return escapeHtml(text);
}

function exportHtml() {
  // 画像をbase64で埋め込み、CSS・背景色・バブル色・画像サイズを現状に合わせて1ファイルのhtmlを生成
  const css = `<style>
    body { background: ${pageBgColor.value}; }
    .log-title-export {
      text-align: center;
      font-size: 2em;
      font-weight: bold;
      margin: 32px 0 40px 0;
      color: ${bubbleBgColor.value};
    }
    .log-row { display: flex; align-items: center; margin-bottom: 16px; }
    .icon-area { width: 100px; height: 100px; flex-shrink: 0; display: flex; align-items: center; justify-content: center; }
    .icon-area img { width: 100px; height: 100px; object-fit: cover; border-radius: 8px; background: #222; }
    .log-content { border: 1px solid ${bubbleBorderColor.value}; border-radius: 8px; padding: 16px; margin-left: 16px; background: ${bubbleBgColor.value}; min-width: 0; flex: 1; text-align: left; position: relative; }
    .char-name { font-weight: bold; font-size: 1em; margin-bottom: 8px; text-align: left; }
    .log-text { color: ${bubbleTextColor.value}; text-align: left; font-size: 1em; }
    .added-tab-row { display: flex; }
    .added-tab-empty { width: 50%; }
    .added-tab-content { width: 50%; background: ${bubbleBgColor.value}; border: 1px solid ${bubbleBorderColor.value}; border-radius: 8px; padding: 16px; position: relative; }
    .added-tab-label { margin-left: 8px; font-size: 0.95em; font-weight: normal; color: ${bubbleTextColor.value}; }
  </style>`;

  // メインタブ
  const mainTab = selectedTab.value || (tabs.value.length > 0 ? tabs.value[0] : '');
  // 追加タブ
  const addTabs = addedTabs.value;

  // 編集画面の順序通りに、必要なログのみ出力
  const html = `<!DOCTYPE html><html lang='ja'><head><meta charset='utf-8'><title>cocologtool export</title>${css}</head><body>` +
    (logTitle.value ? `<div class='log-title-export'>${escapeHtml(logTitle.value)}</div>` : '') +
    filteredLogs.value
      .filter(l => l.tab === mainTab || addTabs.includes(l.tab))
      .map(l => {
        if (l.tab === mainTab) {
          const icon = props.iconMap[l.name] || defaultIcon;
          const charColor = charColorMap.value[l.name] || l.color;
          return `<div class='log-row'><div class='icon-area'><img src='${icon}' /></div><div class='log-content'><div class='char-name' style='color:${charColor}'>${escapeHtml(l.name)}</div><div class='log-text'>${formatText(l.text)}</div></div></div>`;
        } else {
          const charColor = charColorMap.value[l.name] || l.color;
          return `<div class='log-row added-tab-row'><div class='added-tab-empty'></div><div class='log-content added-tab-content'><div class='char-name' style='color:${charColor}'>${escapeHtml(l.name)}<span class='added-tab-label'>${escapeHtml(l.tab)}</span></div><div class='log-text'>${formatText(l.text)}</div></div></div>`;
        }
      }).join('') +
    '</body></html>';
  const blob = new Blob([html], { type: 'text/html' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = 'cocologtool_export.html';
  a.click();
  URL.revokeObjectURL(a.href);
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
.tab-select-row {
  display: flex;
  align-items: center;
  margin-bottom: 16px;
  justify-content: center;
}

.tab-select {
  margin-bottom: 0;
  margin-right: 16px;
}

.tab-select label {
  margin-right: 16px;
  font-weight: bold;
}

.add-tab-btn-area {
  margin-left: 8px;
}

.add-tab-list {
  margin-bottom: 16px;
}

.add-tab-checkbox {
  margin-right: 16px;
}

.log-list {
  margin-bottom: 24px;
}

.log-row {
  display: flex;
  align-items: center;
  margin-bottom: 16px;
  width: 100%;
  box-sizing: border-box;
}

.added-tab-row {
  display: flex;
}

.added-tab-empty {
  width: 50%;
  flex-shrink: 0;
}

.added-tab-content {
  width: 50%;
  background: var(--bubble-bg, #fff);
  border: 1px solid var(--bubble-border, #ccc);
  border-radius: 8px;
  padding: 16px;
  position: relative;
  box-sizing: border-box;
}

.added-tab-label {
  margin-left: 8px;
  font-size: 0.95em;
  font-weight: normal;
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
  flex: auto;
  box-sizing: border-box;
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

.log-title-row {
margin-bottom: 18px;
text-align: center;
}
