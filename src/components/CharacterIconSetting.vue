<template>
  <div>
    <h2>キャラクターアイコン画像設定</h2>
    <div v-for="char in characters" :key="char.name" style="display:flex;align-items:center;margin-bottom:1em">
      <div style="width:60px;height:60px;background:#000;display:flex;align-items:center;justify-content:center;margin-right:1em;">
        <img v-if="icons[char.name]" :src="icons[char.name]" style="width:100%;height:100%;object-fit:cover;" />
      </div>
      <span :style="{color: char.color, fontWeight:'bold', marginRight:'1em'}">{{ char.name }}</span>
      <input type="file" accept="image/*" @change="e=>onIconChange(e, char.name)" />
    </div>
    <button @click="emitDone">設定完了</button>
  </div>
</template>

<script setup lang="ts">
import { ref, defineProps, defineEmits } from 'vue'

defineProps<{ characters: { name: string; color: string }[] }>()
const emit = defineEmits(['done'])
const icons = ref<Record<string, string>>({})

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
  emit('done', icons.value)
}
</script>
