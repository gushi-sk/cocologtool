
<script setup lang="ts">
import { ref } from 'vue'

import LogUploader from './components/LogUploader.vue'
import CharacterIconSetting from './components/CharacterIconSetting.vue'
import LogEditor from './components/LogEditor.vue'


const step = ref<'upload' | 'icon' | 'edit' | 'done'>('upload')
const icons = ref<Record<string, string>>({})
const logsWithId = ref<any[]>([])
const characters = ref<{ name: string; color: string }[]>([])

function onParsed(data: any) {
  characters.value = data.characters.map((c: any) => ({ ...c }))
  // 各ログにidを付与
  logsWithId.value = data.logs.map((l: any, i: number) => ({ ...l, id: i }))
  step.value = 'icon'
}

function onIconDone(payload: { icons: Record<string, string>, characters: { name: string; color: string }[] }) {
  icons.value = payload.icons
  characters.value = payload.characters
  step.value = 'edit'
}
</script>



<template>
  <h1>cocologtool</h1>
  <div v-if="step === 'upload'">
    <LogUploader @parsed="onParsed" />
  </div>
  <div v-else-if="step === 'icon'">
    <CharacterIconSetting :characters="characters" @done="onIconDone" @back="step = 'upload'" />
  </div>
  <div v-else-if="step === 'edit'">
    <LogEditor :logs="logsWithId" :iconMap="icons" :characters="characters" @back="step = 'icon'" />
  </div>
  <div v-else>
    <h2>設定完了</h2>
    <p>キャラクターアイコン設定が完了しました。</p>
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
