<template>
  <div class="min-h-screen bg-[#f0f0f0] p-4 font-sans text-sm text-black">
    <div class="max-w-[1100px] mx-auto bg-[#f0f0f0] border-2 border-gray-400 shadow-md">
      
      <fieldset class="border border-gray-400 m-2 p-2 rounded flex items-center gap-2">
        <legend class="text-xs px-1">檔案操作</legend>
        <button class="tk-btn" @click="triggerFileInput">載入 Excel 課表</button>
        <input type="file" ref="fileInput" class="hidden" accept=".xlsx" @change="handleFileUpload" />
        <button class="tk-btn">匯入 JSON</button>
        <button class="tk-btn ml-4">匯出 CSV</button>
        <button class="tk-btn">匯出 JSON</button>
        <span class="ml-4 text-blue-700">{{ statusMessage }}</span>
      </fieldset>

      <div class="m-2 mt-4">
        <div class="flex border-b border-gray-400">
          <button 
            v-for="tab in tabs" 
            :key="tab.id"
            @click="activeTab = tab.id"
            :class="[
              'px-4 py-1 border border-gray-400 border-b-0 rounded-t-sm -mb-[1px] bg-[#f0f0f0]',
              activeTab === tab.id ? 'bg-white z-10 border-b-white' : 'bg-[#e0e0e0] text-gray-600 hover:bg-[#eaeaea]'
            ]"
          >
            {{ tab.name }}
          </button>
        </div>

        <div class="border border-gray-400 bg-white p-4 h-[550px] overflow-auto">
          
          <div v-show="activeTab === 'teacher'">
            <div class="flex items-center gap-2 mb-4">
              <label>選擇老師:</label>
              <select v-model="selectedTeacher" class="tk-select">
                <option v-for="t in teachers" :key="t" :value="t">{{ t }}</option>
              </select>
            </div>
            
            <table class="w-full border-collapse border border-gray-400 text-center">
              <thead>
                <tr class="bg-[#f0f0f0]">
                  <th class="border border-gray-400 w-20"></th>
                  <th v-for="d in days" :key="d" class="border border-gray-400 py-1">{{ d }}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="p in 8" :key="p">
                  <td class="border border-gray-400 bg-[#f0f0f0] py-4">第{{ p }}節</td>
                  <td v-for="d in 5" :key="d" class="border border-gray-400 h-16 bg-[#e1f5fe]">
                    </td>
                </tr>
              </tbody>
            </table>
          </div>

          <div v-show="activeTab === 'class'">班級課表開發中...</div>
          <div v-show="activeTab === 'swap'">同班互調開發中...</div>

        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import * as XLSX from 'xlsx'

const supabase = useSupabaseClient()

const tabs = [
  { id: 'teacher', name: '教師課表' },
  { id: 'class', name: '班級課表' },
  { id: 'free', name: '空堂查詢' },
  { id: 'overview', name: '該節總覽' },
  { id: 'stats', name: '課數排行' },
  { id: 'swap', name: '同班互調' }
]
const activeTab = ref('teacher')
const statusMessage = ref('尚未載入檔案')
const days = ['一', '二', '三', '四', '五']

const fileInput = ref(null)
const teachers = ref(['王小明', '李大華']) // 測試資料
const selectedTeacher = ref('')

const triggerFileInput = () => fileInput.value.click()

// 讀取 Excel 的核心邏輯 (取代原本的 openpyxl)
const handleFileUpload = async (event) => {
  const file = event.target.files[0]
  if (!file) return
  statusMessage.value = '解析中...'
  
  const reader = new FileReader()
  reader.onload = async (e) => {
    const data = new Uint8Array(e.target.result)
    const workbook = XLSX.read(data, { type: 'array' })
    
    // 這裡放入你原本 _analyze_sheet_type 等解析邏輯的 JavaScript 版
    // 解析完成後寫入 Supabase：
    // await supabase.from('teachers').insert(parsedTeachers)
    // await supabase.from('schedules').insert(parsedSchedules)
    
    statusMessage.value = '解析並上傳至資料庫完成！'
  }
  reader.readAsArrayBuffer(file)
}
</script>

<style>
/* 模擬 Tkinter 元件的 CSS */
.tk-btn {
  @apply px-3 py-1 bg-[#f0f0f0] border border-gray-400 shadow-[1px_1px_2px_rgba(0,0,0,0.2)] active:shadow-none active:translate-y-[1px] text-black;
}
.tk-select {
  @apply border border-gray-400 bg-white px-2 py-1 outline-none focus:border-blue-500;
}
</style>