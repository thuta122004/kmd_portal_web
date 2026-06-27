<template>
  <div class="space-y-6 relative">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">
        {{ selectedSection ? selectedSection.name : 'My Enrolled Sections' }}
      </h2>
      <button
        v-if="selectedSection"
        @click="closeSectionView"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        ← Back to Sections
      </button>
    </div>

    <div v-if="!selectedSection" class="flex flex-col sm:flex-row gap-3">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Search my enrolled sections..."
        class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-sm text-white placeholder-slate-600 outline-none focus:border-white/20 transition"
      />
    </div>

    <div v-if="isLoading" class="p-10 text-center text-slate-500">Loading your profile data...</div>

    <div v-else-if="!selectedSection" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3">
      <template v-if="filteredSections.length > 0">
        <div
          v-for="section in filteredSections"
          :key="section.id"
          @click="fetchSectionData(section)"
          class="group bg-slate-950/50 border border-white/5 p-4 rounded-xl cursor-pointer hover:border-white/20 transition flex flex-col justify-between gap-4"
        >
          <div>
            <h3 class="text-white font-medium group-hover:text-blue-400 transition">
              {{ section.name }}
            </h3>
            <p class="text-slate-500 text-xs mt-0.5">{{ section.code }}</p>
          </div>
          <div class="flex justify-between items-center">
            <span
              class="px-2 py-1 rounded text-[10px] font-bold uppercase bg-blue-500/20 text-blue-400"
            >
              {{ formatDate(section.start_date) }} - {{ formatDate(section.end_date) }}
            </span>
          </div>
        </div>
      </template>
      <div v-else class="col-span-full p-10 text-center text-slate-500 italic">
        You are not actively enrolled in any sections.
      </div>
    </div>

    <div v-else class="space-y-6">
      <div class="flex border-b border-white/10">
        <button
          @click="activeTab = 'attendance'"
          :class="[
            'px-5 py-3 text-sm font-medium border-b-2 transition-colors',
            activeTab === 'attendance'
              ? 'border-blue-500 text-blue-400'
              : 'border-transparent text-slate-400 hover:text-white',
          ]"
        >
          My Attendance Logs
        </button>
        <button
          @click="activeTab = 'timetable'"
          :class="[
            'px-5 py-3 text-sm font-medium border-b-2 transition-colors',
            activeTab === 'timetable'
              ? 'border-blue-500 text-blue-400'
              : 'border-transparent text-slate-400 hover:text-white',
          ]"
        >
          Class Timetable
        </button>
      </div>

      <div v-if="activeTab === 'attendance'" class="space-y-4">
        <div class="border border-white/5 rounded-xl overflow-hidden min-h-[100px]">
          <table class="w-full text-left text-sm table-fixed">
            <thead class="bg-white/5 text-slate-400 uppercase text-[10px] tracking-wider">
              <tr>
                <th class="p-4 font-medium w-1/4">Subject Code</th>
                <th class="p-4 font-medium w-1/4">Date & Schedule</th>
                <th class="p-4 font-medium w-1/4">Status</th>
                <th class="p-4 font-medium w-1/6">Remark</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-white/5">
              <template v-if="sectionAttendanceLogs.length > 0">
                <tr v-for="log in sectionAttendanceLogs" :key="log.id" class="text-slate-300">
                  <td class="p-4 font-medium text-white truncate">
                    <div>{{ log.subject_code }}</div>
                    <div class="text-[10px] text-slate-500">{{ log.lecturer_name }}</div>
                  </td>
                  <td class="p-4 text-xs text-slate-400">
                    <div>{{ log.date }}</div>
                    <div class="text-[10px] text-slate-500 mt-0.5">{{ log.day_of_week }}</div>
                  </td>
                  <td class="p-4">
                    <span
                      class="text-[10px] uppercase font-bold px-2 py-0.5 rounded"
                      :class="statusClass(log.status)"
                    >
                      {{ log.status }}
                    </span>
                  </td>
                  <td class="text-[10px] text-slate-500 italic leading-tight" :title="log.remark">
                    <span class="block line-clamp-2">
                      {{ log.remark || 'No extra feedback provided.' }}
                    </span>
                  </td>
                </tr>
              </template>
              <tr v-else>
                <td colspan="4" class="p-10 text-center text-slate-500 italic">
                  No attendance history found for this section.
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <div v-if="activeTab === 'timetable'" class="overflow-x-auto pb-4 custom-scrollbar">
        <div class="flex gap-4 min-w-max">
          <div v-for="day in daysOfWeek" :key="day" class="w-64 flex-shrink-0 flex flex-col gap-3">
            <div
              class="bg-slate-900/80 border border-white/5 py-2.5 px-4 rounded-lg text-center shadow-sm"
            >
              <h4 class="text-white font-semibold text-sm">{{ day }}</h4>
            </div>

            <div
              v-if="groupedTimetable[day].length === 0"
              class="text-center flex items-center justify-center min-h-[80px] text-xs text-slate-600 py-4 border border-dashed border-white/5 rounded-lg"
            >
              No classes
            </div>

            <template v-else>
              <div
                v-for="cls in groupedTimetable[day]"
                :key="cls.id"
                class="bg-slate-950 border-l-4 border-y border-r border-white/5 border-l-blue-500 p-4 rounded-lg hover:border-white/10 transition flex flex-col gap-3 relative shadow-md"
              >
                <div class="flex justify-between items-start gap-2">
                  <h5 class="text-white text-sm font-semibold leading-tight">
                    {{ cls.subject_name }}
                  </h5>
                </div>

                <div
                  class="text-blue-400 text-xs font-mono font-medium flex items-center gap-1.5 bg-blue-500/5 w-fit px-2 py-1 rounded"
                >
                  <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"
                    />
                  </svg>
                  {{ formatTime(cls.start_time) }} - {{ formatTime(cls.end_time) }}
                </div>

                <button
                  @click="handleCheckIn(cls)"
                  :disabled="checkInSubmitting"
                  class="w-full mt-1 py-1.5 bg-blue-600 hover:bg-blue-500 disabled:bg-slate-800 text-white disabled:text-slate-500 text-xs font-medium rounded-md transition flex items-center justify-center gap-1.5 shadow-sm"
                >
                  <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-6 9l2 2 4-4"
                    />
                  </svg>
                  {{ checkInSubmitting ? 'Logging...' : 'Mark My Attendance' }}
                </button>

                <div
                  class="flex flex-col gap-1.5 text-xs text-slate-400 border-t border-white/5 pt-2 mt-1"
                >
                  <div class="flex items-center gap-2">
                    <span class="text-slate-500 font-medium">Lecturer:</span>
                    <span class="truncate text-slate-300">{{ cls.lecturer_name }}</span>
                  </div>
                  <div class="flex items-center gap-2">
                    <span class="text-slate-500 font-medium">Room:</span>
                    <span class="text-slate-300">{{ cls.room_number || '-' }}</span>
                  </div>
                </div>
              </div>
            </template>
          </div>
        </div>
      </div>
    </div>

    <transition
      enter-active-class="transition ease-out duration-200"
      enter-from-class="opacity-0 translate-y-2"
      enter-to-class="opacity-100 translate-y-0"
      leave-active-class="transition ease-in duration-150"
      leave-from-class="opacity-100 translate-y-0"
      leave-to-class="opacity-0 translate-y-2"
    >
      <div
        v-if="toast.show"
        class="fixed bottom-6 right-6 px-6 py-4 rounded-xl border shadow-2xl flex flex-col gap-4 z-50 min-w-[320px]"
        :class="
          toast.type === 'success'
            ? 'bg-slate-900 border-blue-500/50'
            : toast.type === 'warning'
              ? 'bg-slate-900 border-amber-500/50'
              : 'bg-rose-900 border-rose-500'
        "
      >
        <span class="text-sm font-medium text-white">{{ toast.message }}</span>
        <div v-if="toast.isConfirm" class="flex gap-2 justify-end">
          <button
            @click="toast.show = false"
            class="px-3 py-1 text-xs font-semibold text-slate-400 hover:text-white transition"
          >
            Cancel
          </button>
          <button
            @click="handleConfirm"
            class="px-3 py-1 text-xs font-semibold bg-white text-slate-950 rounded hover:bg-slate-200 transition"
          >
            Confirm
          </button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { onMounted, ref, computed, watch } from 'vue'
import api from '@/services/api'

const sections = ref([])
const activeEnrolments = ref([])
const selectedSection = ref(null)
const isLoading = ref(false)
const checkInSubmitting = ref(false)
const activeTab = ref('attendance')
const searchQuery = ref('')

const currentUserId = ref(null)
const currentStudentName = ref('')

const personalAttendances = ref([])
const timetables = ref([])
const daysOfWeek = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']

const toast = ref({
  show: false,
  message: '',
  type: 'success',
  isConfirm: false,
  onConfirmCallback: null,
})

const showToast = (message, type = 'success', isConfirm = false, callback = null) => {
  toast.value.message = message
  toast.value.type = type
  toast.value.isConfirm = isConfirm
  toast.value.onConfirmCallback = callback
  toast.value.show = true

  if (!isConfirm) {
    setTimeout(() => {
      toast.value.show = false
    }, 5000)
  }
}

const handleConfirm = () => {
  if (toast.value.onConfirmCallback) {
    toast.value.onConfirmCallback()
  }
  toast.value.show = false
}

const formatDate = (dateStr) => {
  if (!dateStr) return 'N/A'

  const date = new Date(dateStr)

  return date.toLocaleDateString('en-GB', {
    day: 'numeric',
    month: 'long',
    year: 'numeric',
  })
}

const statusClass = (status) => ({
  'bg-emerald-500/20 text-emerald-400': status === 'present',
  'bg-rose-500/20 text-rose-400': status === 'absent',
  'bg-amber-500/20 text-amber-400': status === 'late',
  'bg-blue-500/20 text-blue-400': status === 'excused',
})

const filteredSections = computed(() => {
  return sections.value.filter((s) => {
    const isEnrolled = activeEnrolments.value.some(
      (e) => e.section_name === s.name || e.section_id === s.id,
    )

    const matchesSearch =
      s.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      s.code.toLowerCase().includes(searchQuery.value.toLowerCase())

    return isEnrolled && matchesSearch
  })
})

const sectionAttendanceLogs = computed(() => {
  if (!selectedSection.value) return []
  return personalAttendances.value.filter(
    (log) => log.section_code === selectedSection.value.code && log.user_id === currentUserId.value,
  )
})

const groupedTimetable = computed(() => {
  const grouped = {}
  daysOfWeek.forEach((day) => {
    grouped[day] = []
  })

  timetables.value.forEach((t) => {
    const day = t.day_of_week
    const normalizedDay = Object.keys(grouped).find((d) => d.toLowerCase() === day?.toLowerCase())
    if (normalizedDay) {
      grouped[normalizedDay].push(t)
    }
  })

  Object.keys(grouped).forEach((day) => {
    grouped[day].sort((a, b) => (a.start_time || '').localeCompare(b.start_time || ''))
  })

  return grouped
})

const initializeStudentDashboard = async () => {
  isLoading.value = true
  try {
    const userProfile = JSON.parse(localStorage.getItem('user') || '{}')
    currentUserId.value = userProfile.id
    currentStudentName.value = userProfile.name

    const resSections = await api.get('/sections')
    sections.value = resSections.data?.data?.sections || []

    const resEnrolments = await api.get('/enrolments', {
      params: { user_id: currentUserId.value },
    })
    const rawEnrolments = resEnrolments.data?.data?.enrolments || []

    activeEnrolments.value = rawEnrolments.filter(
      (e) =>
        e.status === 'active' &&
        e.student_name.toLowerCase() === currentStudentName.value.toLowerCase(),
    )

    await refreshAttendanceList()
  } catch (err) {
    showToast('Failed to initialize profile catalog assets.', 'error')
  } finally {
    isLoading.value = false
  }
}

const refreshAttendanceList = async () => {
  if (!currentUserId.value) return
  try {
    const resAttendance = await api.get('/attendances', {
      params: { user_id: currentUserId.value },
    })
    personalAttendances.value = resAttendance.data?.data?.attendances || []
  } catch (err) {
    showToast('Failed to sync updated attendance records.', 'warning')
  }
}

const fetchSectionData = async (section) => {
  isLoading.value = true
  activeTab.value = 'attendance'

  try {
    await refreshAttendanceList()

    const resTimetables = await api.get('/timetables')
    const allTimetables = resTimetables.data?.data?.timetables || []

    timetables.value = allTimetables.filter(
      (t) => t.section_id === section.id || t.section_name === section.name,
    )

    selectedSection.value = section
  } catch (err) {
    showToast('Failed to retrieve structured class data maps.', 'error')
  } finally {
    isLoading.value = false
  }
}

const handleCheckIn = (timetableSlot) => {
  showToast(
    `Are you sure you want to log check-in attendance for ${timetableSlot.subject_name}?`,
    'warning',
    true,
    async () => {
      checkInSubmitting.value = true
      try {
        const payload = {
          user_id: currentUserId.value,
          timetable_id: timetableSlot.id,
          remark: 'Self check-in via Student Dashboard.',
        }

        const res = await api.post('/attendances', payload)

        if (res.status === 201) {
          const statusResult = res.data?.data?.attendance?.status

          const hasLink = timetableSlot.link && timetableSlot.link.trim() !== ''

          if (hasLink) {
            showToast(
              `Success! Logged as ${statusResult.toUpperCase()}. Would you like to join the class now?`,
              'success',
              true,
              () => {
                window.open(timetableSlot.link, '_blank')
              },
            )
          } else {
            showToast(
              `Success! Attendance logged dynamically as: ${statusResult.toUpperCase()}.`,
              'success',
            )
          }

          await refreshAttendanceList()
        }
      } catch (err) {
        const backendMessage = err.response?.data?.message
        const errorsDetails = err.response?.data?.errors

        let finalToastText = backendMessage
        if (errorsDetails) {
          const firstKey = Object.keys(errorsDetails)[0]
          if (errorsDetails[firstKey] && errorsDetails[firstKey].length > 0) {
            finalToastText = errorsDetails[firstKey][0]
          }
        }
        showToast(finalToastText, 'error')
      } finally {
        checkInSubmitting.value = false
      }
    },
  )
}

const closeSectionView = () => {
  selectedSection.value = null
  timetables.value = []
}

const formatTime = (timeStr) => {
  if (!timeStr) return 'N/A'
  const parts = timeStr.split(':')
  if (parts.length < 2) return timeStr

  let h = parseInt(parts[0], 10)
  const minute = parts[1]
  const ampm = h >= 12 ? 'PM' : 'AM'
  h = h % 12 || 12
  return `${h}:${minute} ${ampm}`
}

onMounted(initializeStudentDashboard)
</script>
