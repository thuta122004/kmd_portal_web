<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">
        {{ selectedSection ? selectedSection.name : 'Section Info' }}
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

    <div v-if="!selectedSection" class="flex flex-col sm:flex-row gap-3">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Search sections..."
        class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-sm text-white placeholder-slate-600 outline-none focus:border-white/20 transition"
      />
      <select
        v-model="statusFilter"
        class="px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 cursor-pointer appearance-none"
      >
        <option value="all">All Status</option>
        <option value="active">Active</option>
        <option value="pending">Pending</option>
        <option value="inactive">Inactive</option>
      </select>
    </div>

    <div v-if="isLoading" class="p-10 text-center text-slate-500">Loading data...</div>

    <div v-else-if="!selectedSection" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3">
      <template v-if="paginatedSections.length > 0">
        <div
          v-for="section in paginatedSections"
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
          <span
            class="px-2 py-1 rounded text-[10px] font-bold uppercase w-fit"
            :class="[
              section.status === 'active'
                ? 'bg-blue-500/20 text-blue-400'
                : 'bg-slate-700/50 text-slate-400',
            ]"
          >
            {{ section.status }}
          </span>
        </div>
      </template>
      <div v-else class="col-span-full p-10 text-center text-slate-500 italic">
        No records found.
      </div>
    </div>

    <div v-else class="space-y-6">
      <div class="flex border-b border-white/10">
        <button
          @click="activeTab = 'students'"
          :class="[
            'px-5 py-3 text-sm font-medium border-b-2 transition-colors',
            activeTab === 'students'
              ? 'border-blue-500 text-blue-400'
              : 'border-transparent text-slate-400 hover:text-white',
          ]"
        >
          Students
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

      <div v-if="activeTab === 'students'" class="space-y-4">
        <div v-if="studentDetailsLoading" class="p-10 text-center text-slate-500">
          Loading student details...
        </div>

        <div v-else-if="selectedStudent" class="space-y-6">
          <div class="flex items-center justify-between border-b border-white/5 pb-4">
            <div>
              <h3 class="text-lg font-medium text-white">
                {{ selectedStudent.name }}
              </h3>
              <p class="text-xs text-slate-500 mt-1 font-mono">
                {{ selectedStudent.student_reg_number }}
              </p>
            </div>
            <button
              @click="closeStudentView"
              class="px-3 py-1.5 bg-slate-800 text-slate-300 text-xs font-semibold rounded hover:bg-slate-700 transition"
            >
              ← Back to List
            </button>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div class="bg-slate-900/50 border border-white/5 p-4 rounded-xl space-y-3">
              <h4 class="text-sm font-semibold text-slate-300 border-b border-white/5 pb-2">
                Personal Info
              </h4>
              <div class="grid grid-cols-2 gap-2 text-sm">
                <div class="text-slate-500">Email:</div>
                <div class="text-white truncate" :title="selectedStudent.email">
                  {{ selectedStudent.email }}
                </div>

                <div class="text-slate-500">Phone:</div>
                <div class="text-white">{{ selectedStudent.phone || '-' }}</div>

                <div class="text-slate-500">D.O.B:</div>
                <div class="text-white">{{ selectedStudent.date_of_birth }}</div>

                <div class="text-slate-500">Gender:</div>
                <div class="text-white">{{ selectedStudent.gender }}</div>

                <div class="text-slate-500">Address:</div>
                <div class="text-white">{{ selectedStudent.address || '-' }}</div>
              </div>
            </div>

            <div class="bg-slate-900/50 border border-white/5 p-4 rounded-xl space-y-3">
              <h4 class="text-sm font-semibold text-slate-300 border-b border-white/5 pb-2">
                Guardian Info
              </h4>
              <div
                v-if="selectedStudent.guardians && selectedStudent.guardians.length > 0"
                class="space-y-3"
              >
                <div
                  v-for="guardian in selectedStudent.guardians"
                  :key="guardian.id"
                  class="bg-slate-950/50 p-2.5 rounded border border-white/5"
                >
                  <div class="flex justify-between items-start">
                    <p class="text-sm text-white font-medium">
                      {{ guardian.name }} ({{ guardian.relationship_type }})
                    </p>
                    <span
                      v-if="guardian.is_primary_contact"
                      class="px-1.5 py-0.5 bg-emerald-500/20 text-emerald-400 rounded text-[9px] font-bold uppercase"
                      >Primary</span
                    >
                  </div>
                  <p class="text-xs text-slate-400 mt-1">{{ guardian.phone }}</p>
                </div>
              </div>
              <div v-else class="text-xs text-slate-500 italic py-2">
                No guardian info available.
              </div>
            </div>

            <div
              class="bg-slate-900/50 border border-white/5 p-4 rounded-xl md:col-span-2 space-y-3"
            >
              <h4 class="text-sm font-semibold text-slate-300 border-b border-white/5 pb-2">
                Enrollment History
              </h4>
              <div
                v-if="
                  selectedStudent.enrolment_history && selectedStudent.enrolment_history.length > 0
                "
                class="overflow-x-auto"
              >
                <table class="w-full text-left text-sm text-slate-300">
                  <thead class="text-xs text-slate-500 bg-slate-950/50">
                    <tr>
                      <th class="px-3 py-2 rounded-tl">Section</th>
                      <th class="px-3 py-2">Enrolled At</th>
                      <th class="px-3 py-2 rounded-tr">Status</th>
                    </tr>
                  </thead>
                  <tbody class="divide-y divide-white/5">
                    <tr
                      v-for="(enrolment, index) in selectedStudent.enrolment_history"
                      :key="index"
                      class="hover:bg-slate-800/20"
                    >
                      <td class="px-3 py-2 font-medium text-white">
                        {{ enrolment.section }}
                      </td>
                      <td class="px-3 py-2 text-slate-400">{{ enrolment.enrolled_at }}</td>
                      <td class="px-3 py-2">
                        <span
                          class="px-1.5 py-0.5 rounded text-[10px] font-bold uppercase"
                          :class="[
                            enrolment.status === 'active'
                              ? 'bg-blue-500/20 text-blue-400'
                              : 'bg-slate-700/50 text-slate-400',
                          ]"
                        >
                          {{ enrolment.status }}
                        </span>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <div v-else class="text-xs text-slate-500 italic py-2">
                No enrollment history available.
              </div>
            </div>
          </div>
        </div>

        <div v-else class="space-y-4">
          <div class="flex flex-col sm:flex-row gap-3">
            <input
              v-model="studentSearchQuery"
              type="text"
              placeholder="Search students..."
              class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-sm text-white placeholder-slate-600 outline-none focus:border-white/20 transition"
            />
            <select
              v-model="enrollmentFilter"
              class="px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 cursor-pointer appearance-none"
            >
              <option value="all">All Status</option>
              <option value="active">Active</option>
              <option value="inactive">Inactive</option>
              <option value="suspended">Suspended</option>
            </select>
          </div>

          <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
            <template v-if="paginatedStudents.length > 0">
              <div
                v-for="stu in paginatedStudents"
                :key="stu.student_id"
                @click="viewStudent(stu)"
                class="group bg-slate-950/50 border border-white/5 p-4 rounded-xl flex items-center justify-between cursor-pointer hover:border-white/20 hover:bg-slate-900/80 transition"
              >
                <div>
                  <p class="text-white text-sm font-medium group-hover:text-blue-400 transition">
                    {{ stu.student_name }}
                  </p>
                  <p class="text-xs text-slate-500 mt-0.5">{{ stu.student_email }}</p>
                  <p class="text-[10px] text-slate-600 font-mono">{{ stu.student_reg_number }}</p>
                </div>
                <span
                  class="px-2 py-1 rounded text-[10px] font-bold uppercase w-fit"
                  :class="[
                    stu.enrollment_status === 'active'
                      ? 'bg-blue-500/20 text-blue-400'
                      : 'bg-slate-700/50 text-slate-400',
                  ]"
                >
                  {{ stu.enrollment_status }}
                </span>
              </div>
            </template>
            <div v-else class="col-span-full p-10 text-center text-slate-500 italic">
              No students found.
            </div>
          </div>

          <div
            v-if="studentLastPage > 1"
            class="flex justify-between items-center text-xs text-slate-500 pt-2"
          >
            <span>Page {{ studentCurrentPage }} of {{ studentLastPage }}</span>
            <div class="flex gap-2">
              <button
                @click="studentCurrentPage--"
                :disabled="studentCurrentPage === 1"
                class="hover:text-white transition disabled:opacity-30"
              >
                Prev
              </button>
              <button
                @click="studentCurrentPage++"
                :disabled="studentCurrentPage === studentLastPage"
                class="hover:text-white transition disabled:opacity-30"
              >
                Next
              </button>
            </div>
          </div>
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
                  <span
                    v-if="cls.status === 'inactive'"
                    class="px-1.5 py-0.5 rounded text-[9px] font-bold bg-rose-500/20 text-rose-400 uppercase"
                    >Inactive</span
                  >
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
                    <svg
                      class="w-3.5 h-3.5 text-slate-500"
                      fill="none"
                      viewBox="0 0 24 24"
                      stroke="currentColor"
                    >
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"
                      />
                    </svg>
                    <span class="truncate">{{ cls.lecturer_name }}</span>
                  </div>
                  <div class="flex items-center gap-2">
                    <svg
                      class="w-3.5 h-3.5 text-slate-500"
                      fill="none"
                      viewBox="0 0 24 24"
                      stroke="currentColor"
                    >
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4"
                      />
                    </svg>
                    <span>{{ cls.room_number || '-' }}</span>
                  </div>
                </div>
              </div>
            </template>
          </div>
        </div>
      </div>
    </div>

    <div
      v-if="!selectedSection && lastPage > 1"
      class="flex justify-between items-center text-xs text-slate-500 pt-2"
    >
      <span>Page {{ currentPage }} of {{ lastPage }}</span>
      <div class="flex gap-2">
        <button
          @click="changePage(currentPage - 1)"
          :disabled="currentPage === 1"
          class="hover:text-white transition disabled:opacity-30"
        >
          Prev
        </button>
        <button
          @click="changePage(currentPage + 1)"
          :disabled="currentPage === lastPage"
          class="hover:text-white transition disabled:opacity-30"
        >
          Next
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, computed, watch } from 'vue'
import api from '@/services/api'

const sections = ref([])
const selectedSection = ref(null)
const isLoading = ref(false)
const errorMessage = ref(null)
const activeTab = ref('students')

const searchQuery = ref('')
const statusFilter = ref('all')
const currentPage = ref(1)

const students = ref([])
const studentSearchQuery = ref('')
const enrollmentFilter = ref('all')
const studentCurrentPage = ref(1)

const selectedStudent = ref(null)
const studentDetailsLoading = ref(false)

const timetables = ref([])
const daysOfWeek = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']

const itemsPerPage = 6

const filteredSections = computed(() => {
  return sections.value.filter((s) => {
    const matchSearch =
      s.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      s.code.toLowerCase().includes(searchQuery.value.toLowerCase())
    const matchStatus = statusFilter.value === 'all' || s.status === statusFilter.value
    return matchSearch && matchStatus
  })
})

const lastPage = computed(() => Math.ceil(filteredSections.value.length / itemsPerPage) || 1)
const paginatedSections = computed(() => {
  return filteredSections.value.slice(
    (currentPage.value - 1) * itemsPerPage,
    currentPage.value * itemsPerPage,
  )
})

const filteredStudents = computed(() => {
  return students.value.filter((s) => {
    const matchSearch =
      s.student_name.toLowerCase().includes(studentSearchQuery.value.toLowerCase()) ||
      s.student_reg_number.toLowerCase().includes(studentSearchQuery.value.toLowerCase())
    const matchStatus =
      enrollmentFilter.value === 'all' || s.enrollment_status === enrollmentFilter.value
    return matchSearch && matchStatus
  })
})

const studentLastPage = computed(() => Math.ceil(filteredStudents.value.length / itemsPerPage) || 1)
const paginatedStudents = computed(() => {
  return filteredStudents.value.slice(
    (studentCurrentPage.value - 1) * itemsPerPage,
    studentCurrentPage.value * itemsPerPage,
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
    grouped[day].sort((a, b) => {
      const timeA = a.start_time || ''
      const timeB = b.start_time || ''
      return timeA.localeCompare(timeB)
    })
  })

  return grouped
})

watch(activeTab, () => {
  selectedStudent.value = null
})
watch([searchQuery, statusFilter], () => {
  currentPage.value = 1
})
watch([studentSearchQuery, enrollmentFilter], () => {
  studentCurrentPage.value = 1
})

const fetchSections = async () => {
  isLoading.value = true
  try {
    const res = await api.get('/sections')
    sections.value = res.data?.data?.sections || []
  } catch (err) {
    errorMessage.value = 'Failed to load sections.'
  } finally {
    isLoading.value = false
  }
}

const fetchSectionData = async (section) => {
  isLoading.value = true
  errorMessage.value = null
  studentCurrentPage.value = 1
  activeTab.value = 'students'
  selectedStudent.value = null

  try {
    const resStudents = await api.get(`/sections/${section.id}/students`)
    students.value = resStudents.data?.data?.section?.students || []

    const resTimetables = await api.get('/timetables')
    const allTimetables = resTimetables.data?.data?.timetables || []

    timetables.value = allTimetables.filter(
      (t) => t.section_id === section.id || t.section_name === section.name,
    )

    selectedSection.value = section
  } catch (err) {
    errorMessage.value = 'Failed to load section data.'
  } finally {
    isLoading.value = false
  }
}

const viewStudent = async (stu) => {
  const id = stu.student_id || stu.id

  studentDetailsLoading.value = true
  try {
    const res = await api.get(`/students/${id}`)
    selectedStudent.value = res.data?.data?.student || res.data?.data || stu
  } catch (err) {
    errorMessage.value = 'Failed to load student details.'
  } finally {
    studentDetailsLoading.value = false
  }
}

const closeStudentView = () => {
  selectedStudent.value = null
}

const closeSectionView = () => {
  selectedSection.value = null
  selectedStudent.value = null
  students.value = []
  timetables.value = []
}

const changePage = (p) => {
  if (p >= 1 && p <= lastPage.value) currentPage.value = p
}

const formatTime = (timeStr) => {
  if (!timeStr) return '-'
  return timeStr.substring(0, 5)
}

onMounted(fetchSections)
</script>

<style scoped>
.custom-scrollbar::-webkit-scrollbar {
  height: 6px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.02);
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
