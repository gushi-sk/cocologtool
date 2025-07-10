<script setup lang="ts">
import { ref } from 'vue'

const fileInput = ref<HTMLInputElement | null>(null)
const isValidating = ref(false)
const validationResult = ref<{
  isValid: boolean
  message: string
  file?: File
} | null>(null)

const handleFileSelect = async (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  
  if (!file) {
    validationResult.value = null
    return
  }

  await validateCcfoliaLog(file)
}

const validateCcfoliaLog = async (file: File) => {
  isValidating.value = true
  validationResult.value = null

  try {
    // Check if it's an HTML file
    if (!file.name.toLowerCase().endsWith('.html')) {
      validationResult.value = {
        isValid: false,
        message: 'HTMLファイルを選択してください。'
      }
      return
    }

    // Read file content
    const content = await readFileAsText(file)
    
    // Parse HTML content
    const parser = new DOMParser()
    const doc = parser.parseFromString(content, 'text/html')
    
    // Check for parsing errors
    const parseError = doc.querySelector('parsererror')
    if (parseError) {
      validationResult.value = {
        isValid: false,
        message: '有効なHTMLファイルではありません。'
      }
      return
    }

    // Check title tag contains "ccfolia - logs"
    const titleElement = doc.querySelector('head title')
    if (!titleElement || !titleElement.textContent?.toLowerCase().includes('ccfolia - logs')) {
      validationResult.value = {
        isValid: false,
        message: 'ココフォリアのログファイルではありません。タイトルに「ccfolia - logs」が含まれている必要があります。'
      }
      return
    }

    // Check body has p tags
    const bodyPTags = doc.querySelectorAll('body p')
    if (bodyPTags.length === 0) {
      validationResult.value = {
        isValid: false,
        message: 'ココフォリアのログファイルではありません。bodyタグ内にpタグが見つかりません。'
      }
      return
    }

    // Check p tag structure (should have style with color and contain spans)
    let validPTagCount = 0
    bodyPTags.forEach(p => {
      const style = p.getAttribute('style')
      const spans = p.querySelectorAll('span')
      if (style && style.includes('color:') && spans.length >= 2) {
        validPTagCount++
      }
    })

    if (validPTagCount === 0) {
      validationResult.value = {
        isValid: false,
        message: 'ココフォリアのログファイルではありません。適切な構造のpタグが見つかりません。'
      }
      return
    }

    // Validation passed
    validationResult.value = {
      isValid: true,
      message: `有効なココフォリアログファイルです。${bodyPTags.length}行のログが見つかりました。`,
      file: file
    }

  } catch (error) {
    validationResult.value = {
      isValid: false,
      message: 'ファイルの読み込み中にエラーが発生しました。'
    }
  } finally {
    isValidating.value = false
  }
}

const readFileAsText = (file: File): Promise<string> => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.onload = () => resolve(reader.result as string)
    reader.onerror = () => reject(reader.error)
    reader.readAsText(file, 'utf-8')
  })
}

const triggerFileInput = () => {
  fileInput.value?.click()
}

const resetUpload = () => {
  if (fileInput.value) {
    fileInput.value.value = ''
  }
  validationResult.value = null
}
</script>

<template>
  <div class="file-upload-container">
    <h1>ココフォリアログアップロード</h1>
    <p class="description">
      ココフォリアから出力されたログファイル（.html形式）をアップロードしてください。
    </p>

    <div class="upload-area">
      <input
        ref="fileInput"
        type="file"
        accept=".html"
        @change="handleFileSelect"
        class="file-input"
      />
      
      <div v-if="!validationResult" class="upload-prompt">
        <div class="upload-icon">📁</div>
        <button @click="triggerFileInput" class="upload-button" :disabled="isValidating">
          {{ isValidating ? '検証中...' : 'ログファイルを選択' }}
        </button>
        <p class="upload-hint">または、ファイルをここにドラッグ&ドロップ</p>
      </div>

      <div v-if="validationResult" class="validation-result">
        <div :class="['result-icon', validationResult.isValid ? 'success' : 'error']">
          {{ validationResult.isValid ? '✅' : '❌' }}
        </div>
        <p :class="['result-message', validationResult.isValid ? 'success' : 'error']">
          {{ validationResult.message }}
        </p>
        
        <div class="result-actions">
          <button v-if="validationResult.isValid" class="proceed-button">
            次へ進む
          </button>
          <button @click="resetUpload" class="reset-button">
            別のファイルを選択
          </button>
        </div>
      </div>
    </div>

    <div class="validation-info">
      <h3>ココフォリアログファイルの条件</h3>
      <ul>
        <li>拡張子が .html であること</li>
        <li>headタグ内のtitleタグに「ccfolia - logs」の記述があること</li>
        <li>bodyタグ内にpタグが存在し、適切な構造を持っていること</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.file-upload-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

.description {
  text-align: center;
  color: #666;
  margin-bottom: 2rem;
}

.upload-area {
  border: 2px dashed #ccc;
  border-radius: 12px;
  padding: 3rem;
  text-align: center;
  background: #fafafa;
  margin-bottom: 2rem;
  transition: border-color 0.3s ease;
}

.upload-area:hover {
  border-color: #646cff;
}

.file-input {
  display: none;
}

.upload-prompt {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.upload-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
}

.upload-button {
  background: #646cff;
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 8px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.upload-button:hover:not(:disabled) {
  background: #535bf2;
}

.upload-button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.upload-hint {
  color: #888;
  font-size: 0.9rem;
  margin: 0;
}

.validation-result {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.result-icon {
  font-size: 3rem;
}

.result-message {
  font-size: 1.1rem;
  margin: 0;
  text-align: center;
}

.result-message.success {
  color: #28a745;
}

.result-message.error {
  color: #dc3545;
}

.result-actions {
  display: flex;
  gap: 1rem;
  margin-top: 1rem;
}

.proceed-button {
  background: #28a745;
  color: white;
  border: none;
  padding: 0.8rem 1.5rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.proceed-button:hover {
  background: #218838;
}

.reset-button {
  background: #6c757d;
  color: white;
  border: none;
  padding: 0.8rem 1.5rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.reset-button:hover {
  background: #5a6268;
}

.validation-info {
  background: #f8f9fa;
  padding: 1.5rem;
  border-radius: 8px;
  border: 1px solid #e9ecef;
}

.validation-info h3 {
  margin-top: 0;
  color: #495057;
}

.validation-info ul {
  margin-bottom: 0;
}

.validation-info li {
  margin-bottom: 0.5rem;
  color: #6c757d;
}

@media (prefers-color-scheme: dark) {
  .upload-area {
    background: #2d2d2d;
    border-color: #555;
  }
  
  .validation-info {
    background: #2d2d2d;
    border-color: #555;
  }
  
  .upload-hint {
    color: #ccc;
  }
}
</style>