<template>
  <div class="space-y-6 relative">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">
        {{ selectedStudent ? 'Student Details' : 'Attached Students' }}
      </h2>
      <button
        v-if="selectedStudent"
        @click="closeStudentView"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        ← Back to Students
      </button>
    </div>

    <div
      v-if="errorMessage"
      class="p-4 bg-rose-500/10 border border-rose-500/20 rounded-xl text-rose-300 text-sm"
    >
      {{ errorMessage }}
    </div>

    <div v-if="!selectedStudent" class="space-y-6">
      <div v-if="isLoading" class="p-10 text-center text-slate-500">Loading students...</div>

      <div
        v-else-if="currentStudents.length > 0"
        class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4"
      >
        <div
          v-for="stu in currentStudents"
          :key="stu.id"
          @click="viewStudent(stu)"
          class="group bg-slate-950/50 border border-white/5 p-5 rounded-xl cursor-pointer hover:border-blue-500/50 hover:bg-slate-900 transition-all flex flex-col gap-4 shadow-sm"
        >
          <div class="flex justify-between items-start">
            <div>
              <h3 class="text-white font-medium group-hover:text-blue-400 transition text-lg">
                {{ stu.name }}
              </h3>
              <p class="text-xs text-slate-500 mt-1 font-mono">{{ stu.student_reg_number }}</p>
            </div>
            <span
              v-if="stu.is_primary_contact"
              class="px-2 py-1 bg-indigo-500/20 text-indigo-300 rounded text-[10px] font-bold uppercase"
            >
              Primary
            </span>
          </div>
          <div class="flex items-center gap-2">
            <span
              class="text-xs text-slate-400 bg-white/5 px-2.5 py-1 rounded-md border border-white/5"
            >
              {{ stu.relationship_type }}
            </span>
          </div>
        </div>
      </div>

      <div
        v-else
        class="col-span-full p-10 text-center text-slate-500 italic border border-dashed border-white/5 rounded-xl"
      >
        No students attached to this guardian account.
      </div>
    </div>

    <div v-else class="space-y-6">
      <div class="flex border-b border-white/10">
        <button
          @click="activeTab = 'info'"
          :class="[
            'px-5 py-3 text-sm font-medium border-b-2 transition-colors',
            activeTab === 'info'
              ? 'border-blue-500 text-blue-400'
              : 'border-transparent text-slate-400 hover:text-white',
          ]"
        >
          Student Info
        </button>
        <button
          @click="activeTab = 'enrolments'"
          :class="[
            'px-5 py-3 text-sm font-medium border-b-2 transition-colors',
            activeTab === 'enrolments'
              ? 'border-blue-500 text-blue-400'
              : 'border-transparent text-slate-400 hover:text-white',
          ]"
        >
          Enrolment History
        </button>
        <button
          @click="activeTab = 'attendance'"
          :class="[
            'px-5 py-3 text-sm font-medium border-b-2 transition-colors',
            activeTab === 'attendance'
              ? 'border-blue-500 text-blue-400'
              : 'border-transparent text-slate-400 hover:text-white',
          ]"
        >
          Attendance Log & Reports
        </button>
      </div>

      <div v-if="activeTab === 'info'" class="space-y-4">
        <div v-if="studentDetailsLoading" class="p-10 text-center text-slate-500">
          Loading student details...
        </div>
        <div v-else class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div class="bg-slate-900/50 border border-white/5 p-5 rounded-xl space-y-4">
            <h4 class="text-sm font-semibold text-slate-300 border-b border-white/5 pb-2">
              Personal Info
            </h4>
            <div class="grid grid-cols-2 gap-3 text-sm">
              <div class="text-slate-500">Name:</div>
              <div class="text-white font-medium">{{ selectedStudent.name }}</div>

              <div class="text-slate-500">Reg Number:</div>
              <div class="text-white font-mono text-xs">
                {{ selectedStudent.student_reg_number }}
              </div>

              <div class="text-slate-500">Email:</div>
              <div class="text-white truncate" :title="selectedStudent.email">
                {{ selectedStudent.email }}
              </div>

              <div class="text-slate-500">Phone:</div>
              <div class="text-white">{{ selectedStudent.phone || '-' }}</div>

              <div class="text-slate-500">D.O.B:</div>
              <div class="text-white">{{ selectedStudent.date_of_birth || '-' }}</div>

              <div class="text-slate-500">Gender:</div>
              <div class="text-white capitalize">{{ selectedStudent.gender }}</div>

              <div class="text-slate-500">Address:</div>
              <div class="text-white">{{ selectedStudent.address || '-' }}</div>
            </div>
          </div>
        </div>
      </div>

      <div v-if="activeTab === 'enrolments'" class="space-y-4">
        <div class="bg-slate-900/50 border border-white/5 p-4 rounded-xl">
          <h4 class="text-sm font-semibold text-slate-300 mb-4">Enrollment History</h4>
          <div
            v-if="selectedStudent.enrolment_history && selectedStudent.enrolment_history.length > 0"
            class="overflow-x-auto"
          >
            <table class="w-full text-left text-sm text-slate-300">
              <thead class="text-xs text-slate-500 bg-slate-950/50">
                <tr>
                  <th class="px-4 py-3 rounded-tl">Section</th>
                  <th class="px-4 py-3">Enrolled At</th>
                  <th class="px-4 py-3 rounded-tr">Status</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-white/5">
                <tr
                  v-for="(enrolment, index) in selectedStudent.enrolment_history"
                  :key="index"
                  @click="selectSectionFromHistory(enrolment.section)"
                  class="hover:bg-slate-800/20 cursor-pointer"
                >
                  <td class="px-4 py-3 font-medium text-white">
                    {{ enrolment.section }}
                  </td>
                  <td class="px-4 py-3 text-slate-400">{{ enrolment.enrolled_at }}</td>
                  <td class="px-4 py-3">
                    <span
                      class="px-2 py-1 rounded text-[10px] font-bold uppercase"
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
          <div
            v-else
            class="text-sm text-slate-500 italic py-6 text-center border border-dashed border-white/5 rounded-lg"
          >
            No enrollment history available.
          </div>
        </div>
      </div>

      <div v-if="activeTab === 'attendance'" class="space-y-6">
        <div v-if="studentLogsLoading" class="p-10 text-center text-slate-500">
          Loading attendance data...
        </div>

        <template v-else>
          <div v-if="!selectedSectionCode" class="pt-0 p-5 rounded-xl space-y-4">
            <h4 class="text-sm font-semibold text-slate-300">Select Section</h4>
            <div
              v-if="availableSections.length > 0"
              class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4"
            >
              <div
                v-for="section in availableSections"
                :key="section"
                @click="selectedSectionCode = section"
                class="group bg-slate-950/50 p-4 rounded-xl cursor-pointer transition-all flex items-center gap-4 border select-none"
                :class="[
                  selectedSectionCode === section
                    ? 'border-blue-500/50 bg-slate-900/60'
                    : 'border-white/5 hover:border-blue-500/30 hover:bg-slate-900',
                ]"
              >
                <div
                  class="h-9 px-2 rounded-lg flex items-center justify-center font-mono font-bold text-xs tracking-wide transition-colors uppercase border min-w-[38px]"
                  :class="[
                    selectedSectionCode === section
                      ? 'bg-blue-500/20 text-blue-400 border-blue-500/30'
                      : 'bg-white/5 text-slate-400 border-white/5 group-hover:bg-white/10 group-hover:text-slate-200',
                  ]"
                >
                  {{ section ? String(section).substring(0, 2) : '' }}
                </div>

                <div>
                  <h3
                    class="font-medium transition text-sm"
                    :class="
                      selectedSectionCode === section
                        ? 'text-blue-400'
                        : 'text-white group-hover:text-blue-400'
                    "
                  >
                    Section {{ section }}
                  </h3>
                </div>
              </div>
            </div>
            <div
              v-else
              class="text-sm text-slate-500 italic text-center py-6 border border-dashed border-white/5 rounded-lg"
            >
              No enrolled sections found with attendance records.
            </div>
          </div>

          <template v-else>
            <div
              class="flex justify-between items-center bg-slate-900/50 border border-white/5 p-4 rounded-xl"
            >
              <span class="text-sm font-medium text-slate-300"
                >Selected Section:
                <span class="text-blue-400 font-semibold">{{ selectedSectionCode }}</span></span
              >
              <button
                @click="selectedSectionCode = null"
                class="px-3 py-1.5 bg-slate-800 text-slate-300 text-xs font-semibold rounded hover:bg-slate-700 transition"
              >
                ← Back to List
              </button>
            </div>

            <div class="bg-slate-900/50 border border-white/5 p-5 rounded-xl">
              <div class="flex justify-between items-center mb-2">
                <span class="text-sm text-slate-400 font-medium">Attendance Rate</span>
                <span class="text-lg font-bold text-white">{{ overallAttendancePercentage }}%</span>
              </div>
              <div class="w-full bg-slate-950 rounded-full h-3">
                <div
                  class="bg-blue-500 h-3 rounded-full transition-all duration-500"
                  :style="{ width: overallAttendancePercentage + '%' }"
                ></div>
              </div>
              <p class="text-[10px] text-slate-500 mt-1">
                {{
                  studentAttendanceReport
                    ? `${studentAttendanceReport.attended} of ${studentAttendanceReport.total} classes attended`
                    : 'Calculating...'
                }}
              </p>
            </div>

            <div class="bg-slate-900/50 border border-white/5 p-5 rounded-xl space-y-4">
              <h4 class="text-sm font-semibold text-slate-300">Subject-wise Attendance</h4>

              <div v-if="studentSubjectAttendance.length > 0" class="overflow-x-auto">
                <table class="w-full text-left text-sm text-slate-300">
                  <thead class="text-xs text-slate-500 bg-slate-950/50">
                    <tr>
                      <th class="px-4 py-3 rounded-tl">Subject</th>
                      <th class="px-4 py-3 text-center">Attended</th>
                      <th class="px-4 py-3 text-right rounded-tr">Percentage</th>
                    </tr>
                  </thead>
                  <tbody class="divide-y divide-white/5">
                    <tr
                      v-for="(sub, index) in studentSubjectAttendance"
                      :key="index"
                      class="hover:bg-slate-800/20"
                    >
                      <td class="px-4 py-3 text-white font-medium">
                        {{ sub.subject_name }}
                      </td>
                      <td class="px-4 py-3 text-center text-slate-400">
                        {{ sub.attended }} / {{ sub.total }}
                      </td>
                      <td class="px-4 py-3 text-right font-bold text-blue-400">
                        {{ sub.percentage }}%
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <div v-else class="text-sm text-slate-500 italic text-center py-4">
                No subject data available.
              </div>
            </div>

            <div class="bg-slate-900/50 border border-white/5 p-5 rounded-xl space-y-4">
              <h4 class="text-sm font-semibold text-slate-300">Attendance History</h4>

              <div v-if="filteredLogs.length > 0" class="overflow-x-auto">
                <table class="w-full text-left text-sm text-slate-300">
                  <thead class="text-xs text-slate-500 bg-slate-950/50">
                    <tr>
                      <th class="px-4 py-3 rounded-tl">Date</th>
                      <th class="px-4 py-3">Subject</th>
                      <th class="px-4 py-3">Status</th>
                      <th class="px-4 py-3 rounded-tr">Remark</th>
                    </tr>
                  </thead>
                  <tbody class="divide-y divide-white/5">
                    <tr v-for="log in paginatedLogs" :key="log.id" class="hover:bg-slate-800/20">
                      <td class="px-4 py-3 text-slate-400">{{ log.date }}</td>
                      <td class="px-4 py-3">
                        <div class="text-white">{{ log.subject_code }}</div>
                        <div class="text-[10px] text-slate-500">
                          {{ log.section_code }}
                        </div>
                      </td>
                      <td class="px-4 py-3">
                        <div class="flex flex-col items-start gap-1.5">
                          <span
                            class="px-2 py-1 rounded text-[10px] font-bold uppercase inline-block"
                            :class="{
                              'bg-emerald-500/20 text-emerald-400': log.status === 'present',
                              'bg-rose-500/20 text-rose-400': log.status === 'absent',
                              'bg-amber-500/20 text-amber-400': log.status === 'late',
                              'bg-blue-500/20 text-blue-400': log.status === 'excused',
                            }"
                          >
                            {{ log.status }}
                          </span>

                          <button
                            v-if="log.status === 'absent'"
                            @click="requestExcused(log)"
                            class="group flex items-center gap-1 text-[10px] text-blue-400 hover:text-blue-300 font-medium transition-all duration-200"
                          >
                            <svg
                              class="w-3 h-3 transition-transform group-hover:-translate-x-0.5"
                              fill="none"
                              viewBox="0 0 24 24"
                              stroke="currentColor"
                            >
                              <path
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                stroke-width="2"
                                d="M15 19l-7-7 7-7"
                              />
                            </svg>
                            Request Excuse
                          </button>
                        </div>
                      </td>
                      <td
                        class="px-4 py-3 text-xs text-slate-500 max-w-[200px]"
                        :title="log.remark"
                      >
                        <span class="block line-clamp-2 italic leading-tight">
                          {{ log.remark || '-' }}
                        </span>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <div
                v-else
                class="text-sm text-slate-500 italic py-6 text-center border border-dashed border-white/5 rounded-lg"
              >
                No attendance records found for this section.
              </div>
            </div>
            <div v-if="logsLastPage > 1" class="flex justify-between items-center mt-4">
              <span class="text-xs text-slate-500"
                >Page {{ logsCurrentPage }} of {{ logsLastPage }}</span
              >
              <div class="flex gap-2">
                <button
                  @click="logsCurrentPage--"
                  :disabled="logsCurrentPage === 1"
                  class="text-xs text-slate-400 hover:text-white disabled:opacity-30 transition px-2 py-1"
                >
                  Prev
                </button>
                <button
                  @click="logsCurrentPage++"
                  :disabled="logsCurrentPage === logsLastPage"
                  class="text-xs text-slate-400 hover:text-white disabled:opacity-30 transition px-2 py-1"
                >
                  Next
                </button>
              </div>
            </div>
          </template>
        </template>
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

        <div v-if="toast.showRemarkInput" class="w-full">
          <input
            v-model="remarkInput"
            type="text"
            placeholder="Enter reason..."
            class="w-full px-3 py-2 bg-slate-800 border border-white/10 rounded text-xs text-white placeholder-slate-600 outline-none focus:border-blue-500/50 transition"
          />
        </div>

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
import { ref, computed, onMounted, watch } from 'vue'
import api from '@/services/api'

const selectedGuardian = ref(null)
const isLoading = ref(false)
const errorMessage = ref(null)

const selectedStudent = ref(null)
const studentDetailsLoading = ref(false)
const activeTab = ref('info')

const studentLogs = ref([])
const studentLogsLoading = ref(false)
const logsCurrentPage = ref(1)
const logsPerPage = 5
const selectedSectionCode = ref(null)

const remarkInput = ref('')
const toast = ref({
  show: false,
  message: '',
  type: 'success',
  isConfirm: false,
  showRemarkInput: false,
  onConfirmCallback: null,
})

const currentStudents = computed(() => {
  return selectedGuardian.value?.students || []
})

const showToast = (
  message,
  type = 'success',
  isConfirm = false,
  callback = null,
  showInput = false,
) => {
  toast.value = {
    show: true,
    message,
    type,
    isConfirm,
    showRemarkInput: showInput,
    onConfirmCallback: callback,
  }

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

const refreshStudentLogsOnly = async () => {
  const studentUserId =
    selectedStudent.value?.user_id ||
    selectedStudent.value?.student?.user_id ||
    selectedStudent.value?.user?.id ||
    selectedStudent.value?.id

  if (!studentUserId) {
    return
  }

  try {
    const logRes = await api.get('/attendances', {
      params: { user_id: studentUserId },
    })

    studentLogs.value = (logRes.data?.data?.attendances || []).sort(
      (a, b) => new Date(b.date) - new Date(a.date),
    )
  } catch (err) {
    showToast('Failed to sync updated attendance records.', 'warning')
  }
}

const requestExcused = (log) => {
  remarkInput.value = ''

  toast.value = {
    show: true,
    message: 'Provide a reason for being excused',
    type: 'warning',
    isConfirm: true,
    showRemarkInput: true,
    onConfirmCallback: async () => {
      try {
        const res = await api.put(`/attendances/${log.id}`, {
          status: 'excused',
          remark: remarkInput.value,
        })
        if (res.data?.status === 'success' || res.status === 200) {
          showToast('Request submitted successfully!', 'success')
          await refreshStudentLogsOnly()
        }
      } catch (err) {
        showToast('Failed to update status.', 'error')
      }
    },
  }
}

const fetchGuardianProfile = async () => {
  isLoading.value = true
  errorMessage.value = null

  try {
    const userProfile = JSON.parse(localStorage.getItem('user') || '{}')
    const userId = userProfile.id

    if (!userId) {
      errorMessage.value = 'No active login session found.'
      isLoading.value = false
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
      const baseGuardian = { ...targetRows[0] }
      const compiledStudents = []
      const uniquelyIdentifiedIds = new Set()

      targetRows.forEach((guardianRow) => {
        if (Array.isArray(guardianRow.students)) {
          guardianRow.students.forEach((stu) => {
            if (stu && stu.id && !uniquelyIdentifiedIds.has(stu.id)) {
              uniquelyIdentifiedIds.add(stu.id)
              compiledStudents.push({
                ...stu,
                relationship_type: stu.relationship_type || guardianRow.relationship_type,
                is_primary_contact:
                  stu.is_primary_contact !== undefined
                    ? stu.is_primary_contact
                    : guardianRow.is_primary_contact,
              })
            }
          })
        } else if (guardianRow.student && guardianRow.student.id) {
          const stu = guardianRow.student
          if (!uniquelyIdentifiedIds.has(stu.id)) {
            uniquelyIdentifiedIds.add(stu.id)
            compiledStudents.push({
              ...stu,
              relationship_type: stu.relationship_type || guardianRow.relationship_type,
              is_primary_contact:
                stu.is_primary_contact !== undefined
                  ? stu.is_primary_contact
                  : guardianRow.is_primary_contact,
            })
          }
        }
      })

      baseGuardian.students = compiledStudents
      selectedGuardian.value = baseGuardian
    } else if (responseData && !Array.isArray(responseData)) {
      selectedGuardian.value = responseData
    } else {
      selectedGuardian.value = null
    }
  } catch (e) {
    errorMessage.value = 'Failed to load user profile dataset context.'
  } finally {
    isLoading.value = false
  }
}

const viewStudent = async (stu) => {
  selectedStudent.value = stu
  studentDetailsLoading.value = true
  studentLogsLoading.value = true
  activeTab.value = 'info'
  logsCurrentPage.value = 1
  studentLogs.value = []
  selectedSectionCode.value = null

  try {
    const res = await api.get(`/students/${stu.id}`)
    selectedStudent.value = res.data?.data?.student || res.data?.data || stu

    await refreshStudentLogsOnly()
  } catch (e) {
    errorMessage.value = 'Failed to load complete student details.'
  } finally {
    studentDetailsLoading.value = false
    studentLogsLoading.value = false
  }
}

const closeStudentView = () => {
  selectedStudent.value = null
  studentLogs.value = []
  selectedSectionCode.value = null
}

const overallAttendancePercentage = computed(() => {
  const logs = selectedSectionCode.value
    ? studentLogs.value.filter((log) => log.section_code === selectedSectionCode.value)
    : studentLogs.value
  if (logs.length === 0) return 0
  const attendedCount = logs.filter((log) => ['present', 'excused'].includes(log.status)).length
  return Math.round((attendedCount / logs.length) * 100)
})

const studentSubjectAttendance = computed(() => {
  const logs = selectedSectionCode.value
    ? studentLogs.value.filter((log) => log.section_code === selectedSectionCode.value)
    : studentLogs.value
  if (logs.length === 0) return []

  const subjectsMap = {}
  logs.forEach((log) => {
    const subName = log.subject_code
    if (!subjectsMap[subName]) {
      subjectsMap[subName] = { attended: 0, total: 0 }
    }

    subjectsMap[subName].total++
    if (['present', 'excused'].includes(log.status)) {
      subjectsMap[subName].attended++
    }
  })

  return Object.keys(subjectsMap)
    .map((sub) => ({
      subject_name: sub,
      attended: subjectsMap[sub].attended,
      total: subjectsMap[sub].total,
      percentage: Math.round((subjectsMap[sub].attended / subjectsMap[sub].total) * 100),
    }))
    .sort((a, b) => b.percentage - a.percentage)
})

const studentAttendanceReport = computed(() => {
  if (studentLogsLoading.value) return null

  const logs = selectedSectionCode.value
    ? studentLogs.value.filter((log) => log.section_code === selectedSectionCode.value)
    : studentLogs.value

  const total = logs.length
  const attended = logs.filter((log) => ['present', 'excused'].includes(log.status)).length

  return {
    attended,
    total,
  }
})

const filteredLogs = computed(() => {
  if (!selectedSectionCode.value) return []
  return studentLogs.value.filter((log) => log.section_code === selectedSectionCode.value)
})

const paginatedLogs = computed(() => {
  const start = (logsCurrentPage.value - 1) * logsPerPage
  return filteredLogs.value.slice(start, start + logsPerPage)
})

const logsLastPage = computed(() => Math.ceil(filteredLogs.value.length / logsPerPage) || 1)

const availableSections = computed(() => {
  const sections = new Set()
  studentLogs.value.forEach((log) => {
    if (log.section_code) {
      sections.add(log.section_code)
    }
  })
  return Array.from(sections)
})

const selectSectionFromHistory = (sectionTerm) => {
  const found = studentLogs.value.find(
    (l) => l.section_code === sectionTerm || l.section_name === sectionTerm,
  )
  if (found) {
    selectedSectionCode.value = found.section_code
  } else {
    selectedSectionCode.value = sectionTerm
  }
  activeTab.value = 'attendance'
}

watch(activeTab, () => {
  if (activeTab.value === 'attendance') {
    logsCurrentPage.value = 1
  }
})

watch(selectedSectionCode, () => {
  logsCurrentPage.value = 1
})

onMounted(fetchGuardianProfile)
</script>
