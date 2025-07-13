
<script setup lang="ts">
import { ref } from 'vue'
import LogUploader from './components/LogUploader.vue'
import CharacterIconSetting from './components/CharacterIconSetting.vue'

const step = ref<'upload' | 'icon' | 'done'>('upload')
const parsed = ref<any>(null)
const icons = ref<Record<string, string>>({})

function onParsed(data: any) {
  parsed.value = data
  step.value = 'icon'
}

function onIconDone(iconMap: Record<string, string>) {
  icons.value = iconMap
  step.value = 'done'
}
</script>



<template>
  <h1>cocologtool</h1>
  <div v-if="step === 'upload'">
    <LogUploader @parsed="onParsed" />
  </div>
  <div v-else-if="step === 'icon'">
    <CharacterIconSetting :characters="parsed.characters" @done="onIconDone" />
  </div>
  <div v-else>
    <h2>設定完了</h2>
    <p>キャラクターアイコン設定が完了しました。</p>
    <!-- ここに次の画面や編集画面への遷移を追加予定 -->
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
