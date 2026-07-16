<template>
  <div class="space-y-6 relative">
    <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
      <h2 class="text-xl font-semibold text-white">
        <span v-if="selectedSection">{{ selectedSection.name }} </span>
        <span v-else> Student Timetables </span>
      </h2>
      <button
        v-if="selectedSection"
        @click="closeSectionView"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        ← Back to Sections
      </button>
    </div>

    <div
      v-if="errorMessage"
      class="p-4 bg-rose-500/10 border border-rose-500/20 rounded-xl text-rose-300 text-sm"
    >
      {{ errorMessage }}
    </div>

    <div v-if="isLoading" class="p-10 text-center text-slate-500">Loading student data...</div>

    <div v-else-if="students.length > 0">
      <div
        v-if="!selectedSection && students.length > 1"
        class="flex gap-2 border-b border-white/10 pb-4 mb-6 overflow-x-auto custom-scrollbar"
      >
        <button
          v-for="student in students"
          :key="student.id"
          @click="selectStudent(student)"
          :class="[
            'px-4 py-2 text-sm font-medium rounded-lg border transition-all duration-200 whitespace-nowrap',
            selectedStudent?.id === student.id
              ? 'bg-blue-500/20 border-blue-500/30 text-blue-400 shadow-sm'
              : 'bg-slate-950/40 border-white/5 text-slate-400 hover:text-white hover:border-white/20',
          ]"
        >
          {{ student.name }}
        </button>
      </div>

      <div v-if="!selectedSection" class="space-y-4">
        <h3 class="text-sm font-medium text-slate-400">
          Active Sections for {{ selectedStudent.name }}
        </h3>

        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3">
          <template v-if="filteredSections.length > 0">
            <div
              v-for="section in filteredSections"
              :key="section.id"
              @click="fetchSectionTimetable(section)"
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

          <div
            v-else
            class="col-span-full p-10 bg-slate-900/30 border border-white/5 rounded-xl text-center text-slate-500 italic"
          >
            This student is not currently enrolled in any active sections.
          </div>
        </div>
      </div>

      <div v-else class="space-y-6">
        <div v-if="isLoadingTimetable" class="p-10 text-center text-slate-500">
          Loading timetable...
        </div>

        <div v-else class="overflow-x-auto pb-4 custom-scrollbar">
          <div class="flex gap-4 min-w-max">
            <div
              v-for="day in daysOfWeek"
              :key="day"
              class="w-64 flex-shrink-0 flex flex-col gap-3"
            >
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
    </div>

    <div
      v-else-if="!isLoading"
      class="p-10 bg-slate-900/50 border border-white/5 rounded-xl text-center text-slate-400"
    >
      No students are attached to your guardian profile.
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, computed } from 'vue'
import api from '@/services/api'

const isLoading = ref(false)
const isLoadingTimetable = ref(false)
const errorMessage = ref(null)

const students = ref([])
const selectedStudent = ref(null)
const allSections = ref([])
const activeEnrolments = ref([])

const selectedSection = ref(null)
const timetables = ref([])
const daysOfWeek = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']

const formatDate = (dateStr) => {
  if (!dateStr) return 'N/A'
  const date = new Date(dateStr)
  return date.toLocaleDateString('en-GB', { day: 'numeric', month: 'short', year: 'numeric' })
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

const filteredSections = computed(() => {
  if (!selectedStudent.value) return []
  return allSections.value.filter((s) => {
    return activeEnrolments.value.some(
      (e) =>
        (e.section_name === s.name || e.section_id === s.id) &&
        e.student_id === selectedStudent.value.id,
    )
  })
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

const fetchGuardianProfile = async () => {
  try {
    const userProfile = JSON.parse(localStorage.getItem('user') || '{}')
    const userId = userProfile.id

    if (!userId) {
      errorMessage.value = 'No active login session found.'
      return
    }

    const res = await api.get('/guardians', {
      params: { user_id: userId },
    })

    const responseData = res.data?.data?.guardians || res.data?.data?.guardian || res.data?.data

    if (Array.isArray(responseData) && responseData.length > 0) {
      const filteredRows = responseData.filter(
        (row) => row && String(row.user_id) === String(userId),
      )

      const targetRows = filteredRows.length > 0 ? filteredRows : responseData
      const compiledStudents = []
      const uniquelyIdentifiedIds = new Set()

      targetRows.forEach((guardianRow) => {
        if (Array.isArray(guardianRow.students)) {
          guardianRow.students.forEach((stu) => {
            if (stu && stu.id && !uniquelyIdentifiedIds.has(stu.id)) {
              uniquelyIdentifiedIds.add(stu.id)
              compiledStudents.push(stu)
            }
          })
        } else if (guardianRow.student && guardianRow.student.id) {
          const stu = guardianRow.student
          if (!uniquelyIdentifiedIds.has(stu.id)) {
            uniquelyIdentifiedIds.add(stu.id)
            compiledStudents.push(stu)
          }
        }
      })

      students.value = compiledStudents
    }
  } catch (e) {
    errorMessage.value = 'Failed to load user profile dataset context.'
  }
}

const fetchGlobalData = async () => {
  try {
    const resSections = await api.get('/sections')
    allSections.value = resSections.data?.data?.sections || []

    const resEnrolments = await api.get('/enrolments', {
      params: { status: 'active' },
    })
    activeEnrolments.value = resEnrolments.data?.data?.enrolments || []
  } catch (err) {
    console.error('Failed to load sections/enrolments', err)
  }
}

const selectStudent = (student) => {
  selectedStudent.value = student
  selectedSection.value = null
  timetables.value = []
}

const fetchSectionTimetable = async (section) => {
  isLoadingTimetable.value = true
  selectedSection.value = section

  try {
    const resTimetables = await api.get('/timetables')
    const allTimetables = resTimetables.data?.data?.timetables || []

    timetables.value = allTimetables.filter(
      (t) =>
        (t.section_id === section.id || t.section_name === section.name) && t.status === 'active',
    )
  } catch (err) {
    errorMessage.value = 'Failed to retrieve structured class data maps.'
  } finally {
    isLoadingTimetable.value = false
  }
}

const closeSectionView = () => {
  selectedSection.value = null
  timetables.value = []
}

onMounted(async () => {
  isLoading.value = true
  await fetchGuardianProfile()
  await fetchGlobalData()

  if (students.value.length > 0) {
    selectStudent(students.value[0])
  }
  isLoading.value = false
})
</script>

<style scoped>
.custom-scrollbar::-webkit-scrollbar {
  height: 6px;
  width: 6px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 4px;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 4px;
}
.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.2);
}
</style>
