<template>
  <div>
    <h2>キャラクターアイコン画像設定</h2>
    <button @click="emitBack" style="margin-left:1em;">ログアップロード画面に戻る</button>
    <div style="margin-bottom: 32px;"></div>
    <div v-for="char in localCharacters" :key="char.name" class="char-row">
      <div class="char-col icon-col">
        <div class="icon-box">
          <img v-if="icons[char.name]" :src="icons[char.name]" />
        </div>
      </div>
      <div class="char-col name-col">
        <span :style="{color: char.color, fontWeight:'bold'}">{{ char.name }}</span>
      </div>
      <div class="char-col fontcolor-col">
        <input type="color" v-model="char.color" style="margin-right:0.5em;" />
        <input type="text" v-model="char.color" style="width:90px;" />
      </div>
      <div class="char-col file-col">
        <input type="file" accept="image/*" @change="e=>onIconChange(e, char.name)" />
      </div>
    </div>
/* キャラクター設定行のレイアウト調整 */
.char-row {
  display: flex;
  align-items: center;
  margin-bottom: 1em;
  gap: 0.5em;
}
.char-col {
  display: flex;
  align-items: center;
  justify-content: flex-start;
}
.icon-col {
  width: 60px;
}
.icon-box {
  width: 60px;
  height: 60px;
  background: #000;
  display: flex;
  align-items: center;
  justify-content: center;
}
.icon-box img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.name-col {
  min-width: 80px;
  margin-right: 1em;
}
.fontcolor-col {
  min-width: 160px;
  margin-right: 1em;
}
.file-col {
  min-width: 140px;
}
    <button @click="emitDone">設定完了</button>
  </div>
</template>

<script setup lang="ts">
import { ref, defineProps, defineEmits } from 'vue'

const props = defineProps<{ characters: { name: string; color: string }[] }>()
const emit = defineEmits(['done', 'back'])
const icons = ref<Record<string, string>>({})
// キャラクターごとにローカルで色を編集できるようにする
const localCharacters = ref(props.characters.map(c => ({ ...c })))

function onIconChange(e: Event, name: string) {
  const file = (e.target as HTMLInputElement).files?.[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = ev => {
    icons.value[name] = ev.target?.result as string
  }
  reader.readAsDataURL(file)
}

function emitDone() {
  // 色変更も反映してemit
  emit('done', { icons: icons.value, characters: localCharacters.value })
  window.scrollTo({ top: 0, behavior: 'auto' })
}
function emitBack() {
  emit('back')
  window.scrollTo({ top: 0, behavior: 'auto' })
}
</script>
