<template>
  <div>
    <h2>キャラクターアイコン画像設定</h2>
    <button @click="emitBack">ログアップロード画面に戻る</button>
    <button @click="emitDone" style="margin-left:1em;">設定完了</button>
    <div style="margin-bottom: 32px;"></div>
    <div class="char-grid">
      <div v-for="char in localCharacters" :key="char.name" class="char-grid-row">
        <div class="char-grid-cell icon-cell">
          <div class="icon-box">
            <img v-if="icons[char.name]" :src="icons[char.name]" />
          </div>
        </div>
        <div class="char-grid-cell name-cell">
          <span :style="{color: char.color, fontWeight:'bold'}">{{ char.name }}</span>
        </div>
        <div class="char-grid-cell color-cell">
          <input type="color" v-model="char.color" style="margin-right:0.5em;vertical-align:middle;" />
          <input type="text" v-model="char.color" style="width:90px;vertical-align:middle;" />
        </div>
        <div class="char-grid-cell file-cell">
          <input type="file" accept="image/*" @change="e=>onIconChange(e, char.name)" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'


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

<style scoped>
.char-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 0.5em;
}
.char-grid-row {
  display: grid;
  grid-template-columns: 80px 240px 180px 1fr;
  align-items: center;
  gap: 0.5em;
  margin-bottom: 0.5em;
}
.char-grid-cell {
  display: flex;
  align-items: center;
}
.icon-cell {
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
.name-cell {
  min-width: 180px;
}
.color-cell {
  min-width: 160px;
}
</style>
