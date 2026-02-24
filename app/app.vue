<template>
  <div class="min-h-screen bg-[#f0f0f0] p-4 font-sans text-sm text-black">
    <div class="max-w-[1100px] mx-auto bg-[#f0f0f0] border-2 border-gray-400 shadow-md">
      
      <fieldset class="border border-gray-400 m-2 p-2 rounded flex items-center justify-between">
        <legend class="text-xs px-1">系統狀態</legend>
        <div class="flex items-center gap-4">
          <span class="font-bold text-lg ml-2">本學期最新課表</span>
          <span class="text-blue-700">{{ statusMessage }}</span>
        </div>
        <button class="tk-btn mr-2" @click="loadScheduleData">重新載入</button>
      </fieldset>

      <div class="m-2 mt-4">
        <div class="flex border-b border-gray-400">
          <button 
            v-for="tab in tabs" :key="tab.id" @click="activeTab = tab.id"
            :class="['px-4 py-1 border border-gray-400 border-b-0 rounded-t-sm -mb-[1px]', activeTab === tab.id ? 'bg-white z-10 border-b-white font-bold' : 'bg-[#e0e0e0] text-gray-600 hover:bg-[#eaeaea]']">
            {{ tab.name }}
          </button>
        </div>

        <div class="border border-gray-400 bg-white p-4 h-[600px] overflow-auto">
          
          <div v-show="activeTab === 'teacher'">
            <div class="flex items-center gap-2 mb-4">
              <label>選擇老師:</label>
              <select v-model="selectedTeacher" class="tk-select w-40">
                <option v-for="name in teacherNames" :key="name" :value="name">{{ name }}</option>
              </select>
            </div>
            <table class="w-full border-collapse border border-gray-400 text-center">
              <thead>
                <tr class="bg-[#f0f0f0]">
                  <th class="border border-gray-400 w-20"></th>
                  <th v-for="d in daysOptions" :key="d" class="border border-gray-400 py-1">{{ d.label }}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="p in 8" :key="p">
                  <td class="border border-gray-400 bg-[#f0f0f0] py-4">第{{ p }}節</td>
                  <td v-for="d in 5" :key="d" class="border border-gray-400 h-16" :class="getTeacherCell(d, p) ? 'bg-[#e1f5fe]' : 'bg-white'">
                    <div class="whitespace-pre-line">{{ getTeacherCell(d, p) }}</div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <div v-show="activeTab === 'class'">
            <div class="flex items-center gap-2 mb-4">
              <label>選擇班級:</label>
              <select v-model="selectedClass" class="tk-select w-40">
                <option v-for="cls in classesList" :key="cls" :value="cls">{{ cls }}</option>
              </select>
            </div>
            <table class="w-full border-collapse border border-gray-400 text-center">
              <thead>
                <tr class="bg-[#f0f0f0]">
                  <th class="border border-gray-400 w-20"></th>
                  <th v-for="d in daysOptions" :key="d" class="border border-gray-400 py-1">{{ d.label }}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="p in 8" :key="p">
                  <td class="border border-gray-400 bg-[#f0f0f0] py-4">第{{ p }}節</td>
                  <td v-for="d in 5" :key="d" class="border border-gray-400 h-16" :class="getClassCell(d, p) ? 'bg-[#fff9c4]' : 'bg-white'">
                    <div class="whitespace-pre-line">{{ getClassCell(d, p) }}</div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <div v-show="activeTab === 'free'">
            <div class="flex items-center gap-2 mb-4">
              <label>星期:</label>
              <select v-model="searchDay" class="tk-select w-24"><option v-for="d in daysOptions" :value="d.val">{{ d.label }}</option></select>
              <label class="ml-4">節次:</label>
              <select v-model="searchPeriod" class="tk-select w-20"><option v-for="p in 8" :value="p">{{ p }}</option></select>
            </div>
            <table class="w-full border-collapse border border-gray-400 text-left">
              <thead class="bg-[#f0f0f0]">
                <tr><th class="border border-gray-400 px-2 py-1">姓名</th><th class="border border-gray-400 px-2 py-1">職稱</th><th class="border border-gray-400 px-2 py-1">基本節數</th></tr>
              </thead>
              <tbody>
                <tr v-for="t in freeTeachers" :key="t.name" class="hover:bg-gray-100">
                  <td class="border border-gray-400 px-2 py-1">{{ t.name }}</td>
                  <td class="border border-gray-400 px-2 py-1">{{ t.title }}</td>
                  <td class="border border-gray-400 px-2 py-1">{{ t.base_load }}</td>
                </tr>
              </tbody>
            </table>
          </div>

          <div v-show="activeTab === 'overview'">
             <div class="flex items-center gap-2 mb-4">
              <label>星期:</label>
              <select v-model="searchDay" class="tk-select w-24"><option v-for="d in daysOptions" :value="d.val">{{ d.label }}</option></select>
              <label class="ml-4">節次:</label>
              <select v-model="searchPeriod" class="tk-select w-20"><option v-for="p in 8" :value="p">{{ p }}</option></select>
            </div>
            <table class="w-full border-collapse border border-gray-400 text-left">
              <thead class="bg-[#f0f0f0]">
                <tr><th class="border border-gray-400 px-2 py-1">年級</th><th class="border border-gray-400 px-2 py-1">班級</th><th class="border border-gray-400 px-2 py-1">科目</th><th class="border border-gray-400 px-2 py-1">授課教師</th></tr>
              </thead>
              <tbody>
                <tr v-for="s in periodOverview" :key="s.teacher + s.class_name" class="hover:bg-gray-100">
                  <td class="border border-gray-400 px-2 py-1">{{ s.grade }}</td>
                  <td class="border border-gray-400 px-2 py-1">{{ s.class_name }}</td>
                  <td class="border border-gray-400 px-2 py-1">{{ s.subject }}</td>
                  <td class="border border-gray-400 px-2 py-1">{{ s.teacher }}</td>
                </tr>
              </tbody>
            </table>
          </div>

          <div v-show="activeTab === 'stats'">
            <table class="w-full border-collapse border border-gray-400 text-center">
              <thead class="bg-[#f0f0f0]">
                <tr>
                  <th class="border border-gray-400 px-2 py-1 text-left">教師姓名</th>
                  <th v-for="d in daysOptions" :key="d" class="border border-gray-400 px-2 py-1">{{ d.label }}</th>
                  <th class="border border-gray-400 px-2 py-1 font-bold text-red-600">總計</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="stat in teacherStats" :key="stat.name" class="hover:bg-gray-100">
                  <td class="border border-gray-400 px-2 py-1 text-left">{{ stat.name }}</td>
                  <td class="border border-gray-400 px-2 py-1">{{ stat[1] }}</td>
                  <td class="border border-gray-400 px-2 py-1">{{ stat[2] }}</td>
                  <td class="border border-gray-400 px-2 py-1">{{ stat[3] }}</td>
                  <td class="border border-gray-400 px-2 py-1">{{ stat[4] }}</td>
                  <td class="border border-gray-400 px-2 py-1">{{ stat[5] }}</td>
                  <td class="border border-gray-400 px-2 py-1 font-bold">{{ stat.total }}</td>
                </tr>
              </tbody>
            </table>
          </div>

          <div v-show="activeTab === 'swap'" class="flex gap-4">
            <fieldset class="border border-gray-400 p-4 w-1/2">
              <legend class="text-xs px-1">步驟1：尋找交換對象</legend>
              <div class="mb-2"><label class="inline-block w-28">發起老師 (A):</label><select v-model="swapA" class="tk-select w-32"><option v-for="name in teacherNames" :value="name">{{ name }}</option></select></div>
              <div class="mb-2"><label class="inline-block w-28">原時段 (A有課):</label>
                <select v-model="swapSrcDay" class="tk-select w-16 mr-1"><option v-for="d in daysOptions" :value="d.val">{{ d.val }}</option></select>
                <select v-model="swapSrcPeriod" class="tk-select w-16"><option v-for="p in 8" :value="p">{{ p }}</option></select>
              </div>
              <div class="mb-4"><label class="inline-block w-28">目標時段 (A空堂):</label>
                <select v-model="swapDstDay" class="tk-select w-16 mr-1"><option v-for="d in daysOptions" :value="d.val">{{ d.val }}</option></select>
                <select v-model="swapDstPeriod" class="tk-select w-16"><option v-for="p in 8" :value="p">{{ p }}</option></select>
              </div>
              <button class="tk-btn w-full mb-4" @click="findSwapCandidates">搜尋同班互調對象</button>
              <p class="text-xs text-gray-500">規則：<br>1. 老師A在原時段必須有課。<br>2. 老師A在目標時段必須空堂。<br>3. 老師B在目標時段必須有「同一班級」的課。<br>4. 老師B在原時段必須空堂。</p>
            </fieldset>

            <fieldset class="border border-gray-400 p-4 w-1/2 flex flex-col">
              <legend class="text-xs px-1">步驟2：選擇對象並執行互調</legend>
              <div class="text-red-600 mb-2 font-bold">{{ swapMessage }}</div>
              <table class="w-full border-collapse border border-gray-400 text-left mb-4 flex-grow">
                <thead class="bg-[#f0f0f0]">
                  <tr><th class="border border-gray-400 px-2 py-1">選擇</th><th class="border border-gray-400 px-2 py-1">老師 B</th><th class="border border-gray-400 px-2 py-1">B 的科目</th></tr>
                </thead>
                <tbody>
                  <tr v-for="c in swapCandidates" :key="c.name" class="hover:bg-gray-100">
                    <td class="border border-gray-400 px-2 py-1 text-center"><input type="radio" :value="c.name" v-model="selectedSwapB"></td>
                    <td class="border border-gray-400 px-2 py-1">{{ c.name }}</td>
                    <td class="border border-gray-400 px-2 py-1">{{ c.subject }} ({{ c.grade }}{{ c.class_name }})</td>
                  </tr>
                </tbody>
              </table>
              <button class="tk-btn w-full" :disabled="!selectedSwapB" @click="executeSwap">確認執行互調 (僅暫存於網頁)</button>
            </fieldset>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

// --- 介面狀態 ---
const tabs = [
  { id: 'teacher', name: '教師課表' }, { id: 'class', name: '班級課表' },
  { id: 'free', name: '空堂查詢' }, { id: 'overview', name: '該節總覽' },
  { id: 'stats', name: '課數排行' }, { id: 'swap', name: '同班互調' }
]
const activeTab = ref('teacher')
const statusMessage = ref('初始化中...')
const daysOptions = [
  { val: 1, label: '一' }, { val: 2, label: '二' }, 
  { val: 3, label: '三' }, { val: 4, label: '四' }, { val: 5, label: '五' }
]

// --- 核心資料庫 ---
const dbTeachers = ref({}) 
const dbSchedules = ref([])

// --- 自動載入 public/data.json ---
onMounted(() => {
  loadScheduleData()
})

const loadScheduleData = async () => {
  statusMessage.value = '載入中...'
  try {
    // 從 public 資料夾抓取靜態 JSON 檔案
    const res = await fetch('/data.json')
    if (!res.ok) throw new Error('找不到檔案')
    
    const data = await res.json()
    dbTeachers.value = data.teachers || {}
    dbSchedules.value = data.schedules || []
    statusMessage.value = `載入成功！共 ${Object.keys(dbTeachers.value).length} 位教師，${dbSchedules.value.length} 堂課。`
    
    // 初始化下拉選單第一筆
    if (teacherNames.value.length > 0) selectedTeacher.value = teacherNames.value[0]
    if (classesList.value.length > 0) selectedClass.value = classesList.value[0]
  } catch (err) {
    statusMessage.value = '讀取失敗，請確認 public/data.json 是否存在！'
    console.error(err)
  }
}

// ==========================================
// 1. 教師課表 (Tab 1)
// ==========================================
const teacherNames = computed(() => Object.keys(dbTeachers.value).sort())
const selectedTeacher = ref('')

const getTeacherCell = (d, p) => {
  if (!selectedTeacher.value) return ""
  const course = dbSchedules.value.find(s => s.teacher === selectedTeacher.value && s.day === d && s.period === p)
  if (!course) return ""
  return `${course.grade === 'Unknown' ? '' : course.grade}${course.class_name}\n${course.subject}`
}

// ==========================================
// 2. 班級課表 (Tab 2)
// ==========================================
const classesList = computed(() => {
  const clsSet = new Set()
  dbSchedules.value.forEach(s => {
    if (s.grade && s.grade !== "Unknown" && s.class_name) {
      clsSet.add(`${s.grade} ${s.class_name}`)
    }
  })
  return Array.from(clsSet).sort()
})
const selectedClass = ref('')

const getClassCell = (d, p) => {
  if (!selectedClass.value) return ""
  const [grade, cls] = selectedClass.value.split(" ")
  const course = dbSchedules.value.find(s => s.grade === grade && s.class_name === cls && s.day === d && s.period === p)
  if (!course) return ""
  return `${course.subject}\n${course.teacher}`
}

// ==========================================
// 3. 空堂查詢 & 4. 該節總覽 (Tab 3 & 4)
// ==========================================
const searchDay = ref(1)
const searchPeriod = ref(1)

const freeTeachers = computed(() => {
  const busyTeachers = new Set(
    dbSchedules.value.filter(s => s.day === searchDay.value && s.period === searchPeriod.value).map(s => s.teacher)
  )
  return Object.values(dbTeachers.value).filter(t => !busyTeachers.has(t.name))
})

const periodOverview = computed(() => {
  return dbSchedules.value
    .filter(s => s.day === searchDay.value && s.period === searchPeriod.value)
    .sort((a, b) => a.grade.localeCompare(b.grade) || a.class_name.localeCompare(b.class_name))
})

// ==========================================
// 5. 課數排行 (Tab 5)
// ==========================================
const teacherStats = computed(() => {
  const stats = {}
  Object.keys(dbTeachers.value).forEach(name => {
    stats[name] = { name, 1:0, 2:0, 3:0, 4:0, 5:0, total:0 }
  })
  dbSchedules.value.forEach(s => {
    if (stats[s.teacher] && s.day >= 1 && s.day <= 5) {
      stats[s.teacher][s.day] += 1
      stats[s.teacher].total += 1
    }
  })
  return Object.values(stats).sort((a, b) => b.total - a.total)
})

// ==========================================
// 6. 同班互調 (Tab 6)
// ==========================================
const swapA = ref('')
const swapSrcDay = ref(1)
const swapSrcPeriod = ref(1)
const swapDstDay = ref(1)
const swapDstPeriod = ref(1)
const swapMessage = ref('')
const swapCandidates = ref([])
const selectedSwapB = ref('')

const findSwapCandidates = () => {
  swapMessage.value = ''
  swapCandidates.value = []
  selectedSwapB.value = ''

  if (!swapA.value) { swapMessage.value = "請先選擇發起老師 A"; return; }

  const courseA_src = dbSchedules.value.find(s => s.teacher === swapA.value && s.day === swapSrcDay.value && s.period === swapSrcPeriod.value)
  if (!courseA_src) { swapMessage.value = `老師 ${swapA.value} 在原時段沒有課！`; return; }
  
  const targetGrade = courseA_src.grade
  const targetClass = courseA_src.class_name
  if (targetGrade === 'Unknown' || !targetClass) { swapMessage.value = "無法識別該課程的班級，無法互調。"; return; }

  const hasClassA_dst = dbSchedules.value.some(s => s.teacher === swapA.value && s.day === swapDstDay.value && s.period === swapDstPeriod.value)
  if (hasClassA_dst) { swapMessage.value = `老師 ${swapA.value} 在目標時段已經有課了！`; return; }

  const candidates = []
  Object.keys(dbTeachers.value).forEach(tB => {
    if (tB === swapA.value) return
    const courseB_dst = dbSchedules.value.find(s => s.teacher === tB && s.day === swapDstDay.value && s.period === swapDstPeriod.value)
    if (!courseB_dst || courseB_dst.grade !== targetGrade || courseB_dst.class_name !== targetClass) return

    const hasClassB_src = dbSchedules.value.some(s => s.teacher === tB && s.day === swapSrcDay.value && s.period === swapSrcPeriod.value)
    if (!hasClassB_src) {
      candidates.push({ name: tB, subject: courseB_dst.subject, grade: targetGrade, class_name: targetClass })
    }
  })

  if (candidates.length === 0) {
    swapMessage.value = "沒有找到符合條件的老師。"
  } else {
    swapCandidates.value = candidates
    swapMessage.value = `找到 ${candidates.length} 位可互調老師！`
  }
}

const executeSwap = () => {
  const tA = swapA.value
  const tB = selectedSwapB.value
  
  const idxA = dbSchedules.value.findIndex(s => s.teacher === tA && s.day === swapSrcDay.value && s.period === swapSrcPeriod.value)
  const idxB = dbSchedules.value.findIndex(s => s.teacher === tB && s.day === swapDstDay.value && s.period === swapDstPeriod.value)
  
  if (idxA === -1 || idxB === -1) {
    alert("交換失敗：找不到對應課程。")
    return
  }
  
  dbSchedules.value[idxA].day = swapDstDay.value
  dbSchedules.value[idxA].period = swapDstPeriod.value
  dbSchedules.value[idxB].day = swapSrcDay.value
  dbSchedules.value[idxB].period = swapSrcPeriod.value

  alert("互調成功！\n資料已更新（重整網頁後會還原預設課表）。")
  swapCandidates.value = []
  swapMessage.value = ''
  selectedSwapB.value = ''
}
</script>

<style scoped>
.tk-btn {
  @apply px-3 py-1 bg-[#f0f0f0] border border-gray-400 shadow-[1px_1px_2px_rgba(0,0,0,0.2)] active:shadow-none active:translate-y-[1px] text-black cursor-pointer;
}
.tk-select {
  @apply border border-gray-400 bg-white px-2 py-1 outline-none focus:border-blue-500;
}
</style>