<template>
  <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
    <div class="flex items-center justify-between mb-4">
      <h3 class="text-sm font-bold text-slate-400">实验日历看板</h3>
      <button @click="showClearConfirm = true" class="text-xs text-slate-500 hover:text-red-400 transition-colors">
        清空历史
      </button>
    </div>

    <div class="grid grid-cols-3 gap-3 mb-4">
      <div class="bg-slate-900 rounded p-3 text-center">
        <div class="text-2xl font-bold text-cyan-400">{{ store.totalSimulations }}</div>
        <div class="text-xs text-slate-500 mt-1">总模拟次数</div>
      </div>
      <div class="bg-slate-900 rounded p-3 text-center">
        <div class="text-2xl font-bold text-green-400">{{ store.todayCount }}</div>
        <div class="text-xs text-slate-500 mt-1">今日模拟</div>
      </div>
      <div class="bg-slate-900 rounded p-3 text-center">
        <div class="text-2xl font-bold text-purple-400">{{ store.dateList.length }}</div>
        <div class="text-xs text-slate-500 mt-1">活跃天数</div>
      </div>
    </div>

    <div class="flex items-center justify-between mb-3">
      <button @click="prevMonth" class="p-1 text-slate-400 hover:text-cyan-400 transition-colors">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
        </svg>
      </button>
      <span class="text-sm font-bold text-slate-300">{{ currentMonthLabel }}</span>
      <button @click="nextMonth" class="p-1 text-slate-400 hover:text-cyan-400 transition-colors">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
        </svg>
      </button>
    </div>

    <div class="grid grid-cols-7 gap-1 mb-1">
      <div v-for="day in weekDays" :key="day" class="text-center text-xs text-slate-500 py-1">
        {{ day }}
      </div>
    </div>

    <div class="grid grid-cols-7 gap-1">
      <div
        v-for="(day, idx) in calendarDays"
        :key="idx"
        @click="day.count > 0 && selectDate(day.dateStr)"
        :class="[
          'aspect-square flex flex-col items-center justify-center rounded text-xs transition-all',
          day.isCurrentMonth
            ? day.count > 0
              ? 'cursor-pointer hover:scale-105'
              : 'text-slate-600'
            : 'text-slate-700',
          day.isToday ? 'ring-2 ring-cyan-400' : '',
          selectedDate === day.dateStr ? 'ring-2 ring-yellow-400' : '',
          getDayBgClass(day.count, day.isCurrentMonth)
        ]"
      >
        <span :class="['font-medium', day.isToday ? 'text-cyan-300' : '']">{{ day.day }}</span>
        <span v-if="day.count > 0" :class="getCountTextClass(day.count)" class="text-[10px] font-bold">
          {{ day.count }}
        </span>
      </div>
    </div>

    <div v-if="selectedDate" class="mt-4 pt-4 border-t border-slate-700">
      <div class="flex items-center justify-between mb-3">
        <h4 class="text-sm font-bold text-slate-300">{{ selectedDate }} 的实验记录</h4>
        <button @click="selectedDate = null" class="text-xs text-slate-500 hover:text-slate-300">收起</button>
      </div>
      <div v-if="selectedRecords.length > 0" class="space-y-2 max-h-64 overflow-y-auto">
        <div
          v-for="record in selectedRecords"
          :key="record.id"
          class="bg-slate-900 rounded p-2 text-xs flex items-center justify-between"
        >
          <div class="flex-1">
            <div class="flex items-center gap-2">
              <span class="text-cyan-400 font-bold">{{ record.scenarioName }}</span>
              <span class="text-slate-500">{{ formatTime(record.createdAt) }}</span>
            </div>
            <div class="text-slate-400 mt-1">
              估算值: <span class="font-mono text-slate-300">{{ record.estimate.toFixed(6) }}</span>
              <span v-if="record.error !== undefined" class="ml-2">
                误差: <span class="font-mono text-orange-400">{{ record.error.toFixed(6) }}</span>
              </span>
            </div>
          </div>
          <button
            @click.stop="deleteRecord(record.id)"
            class="ml-2 p-1 text-slate-600 hover:text-red-400 transition-colors"
          >
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
            </svg>
          </button>
        </div>
      </div>
      <div v-else class="text-center text-slate-500 text-xs py-4">
        当天暂无实验记录
      </div>
    </div>

    <div v-if="showClearConfirm" class="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
      <div class="bg-slate-800 rounded-lg p-6 border border-slate-700 max-w-sm mx-4">
        <h4 class="text-lg font-bold text-slate-200 mb-2">确认清空</h4>
        <p class="text-sm text-slate-400 mb-4">确定要清空所有历史实验记录吗？此操作不可撤销。</p>
        <div class="flex gap-3 justify-end">
          <button
            @click="showClearConfirm = false"
            class="px-4 py-2 text-sm text-slate-400 hover:text-slate-200 transition-colors"
          >
            取消
          </button>
          <button
            @click="confirmClear"
            class="px-4 py-2 text-sm bg-red-600 hover:bg-red-500 text-white rounded transition-colors"
          >
            确认清空
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useMCStore } from '../store/mc'

const store = useMCStore()
const currentYear = ref(new Date().getFullYear())
const currentMonth = ref(new Date().getMonth())
const selectedDate = ref<string | null>(null)
const showClearConfirm = ref(false)

const weekDays = ['日', '一', '二', '三', '四', '五', '六']

const currentMonthLabel = computed(() => {
  return `${currentYear.value}年${currentMonth.value + 1}月`
})

interface CalendarDay {
  day: number
  dateStr: string
  isCurrentMonth: boolean
  isToday: boolean
  count: number
}

const calendarDays = computed((): CalendarDay[] => {
  const days: CalendarDay[] = []
  const firstDay = new Date(currentYear.value, currentMonth.value, 1)
  const lastDay = new Date(currentYear.value, currentMonth.value + 1, 0)
  const today = new Date()
  const todayStr = formatDateKey(today.getTime())

  const startDayOfWeek = firstDay.getDay()
  const prevMonthLastDay = new Date(currentYear.value, currentMonth.value, 0).getDate()

  for (let i = startDayOfWeek - 1; i >= 0; i--) {
    const day = prevMonthLastDay - i
    const date = new Date(currentYear.value, currentMonth.value - 1, day)
    const dateStr = formatDateKey(date.getTime())
    days.push({
      day,
      dateStr,
      isCurrentMonth: false,
      isToday: dateStr === todayStr,
      count: store.historyByDate.get(dateStr)?.length || 0
    })
  }

  for (let i = 1; i <= lastDay.getDate(); i++) {
    const date = new Date(currentYear.value, currentMonth.value, i)
    const dateStr = formatDateKey(date.getTime())
    days.push({
      day: i,
      dateStr,
      isCurrentMonth: true,
      isToday: dateStr === todayStr,
      count: store.historyByDate.get(dateStr)?.length || 0
    })
  }

  const remaining = 42 - days.length
  for (let i = 1; i <= remaining; i++) {
    const date = new Date(currentYear.value, currentMonth.value + 1, i)
    const dateStr = formatDateKey(date.getTime())
    days.push({
      day: i,
      dateStr,
      isCurrentMonth: false,
      isToday: dateStr === todayStr,
      count: store.historyByDate.get(dateStr)?.length || 0
    })
  }

  return days
})

const selectedRecords = computed(() => {
  if (!selectedDate.value) return []
  return store.getRecordsByDate(selectedDate.value).sort((a, b) => b.createdAt - a.createdAt)
})

function formatDateKey(ts: number): string {
  const d = new Date(ts)
  const y = d.getFullYear()
  const m = String(d.getMonth() + 1).padStart(2, '0')
  const day = String(d.getDate()).padStart(2, '0')
  return `${y}-${m}-${day}`
}

function formatTime(ts: number): string {
  const d = new Date(ts)
  const h = String(d.getHours()).padStart(2, '0')
  const m = String(d.getMinutes()).padStart(2, '0')
  return `${h}:${m}`
}

function prevMonth() {
  if (currentMonth.value === 0) {
    currentMonth.value = 11
    currentYear.value--
  } else {
    currentMonth.value--
  }
}

function nextMonth() {
  if (currentMonth.value === 11) {
    currentMonth.value = 0
    currentYear.value++
  } else {
    currentMonth.value++
  }
}

function selectDate(dateStr: string) {
  selectedDate.value = selectedDate.value === dateStr ? null : dateStr
}

function deleteRecord(id: string) {
  store.removeRecord(id)
}

function confirmClear() {
  store.clearHistory()
  showClearConfirm.value = false
  selectedDate.value = null
}

function getDayBgClass(count: number, isCurrentMonth: boolean): string {
  if (!isCurrentMonth) return 'bg-transparent'
  if (count === 0) return 'bg-slate-900/30'
  if (count <= 3) return 'bg-cyan-900/40 text-cyan-300'
  if (count <= 7) return 'bg-cyan-700/50 text-cyan-200'
  if (count <= 15) return 'bg-cyan-600/60 text-cyan-100'
  return 'bg-cyan-500/70 text-white'
}

function getCountTextClass(count: number): string {
  if (count <= 3) return 'text-cyan-400'
  if (count <= 7) return 'text-cyan-300'
  if (count <= 15) return 'text-cyan-200'
  return 'text-white'
}
</script>
