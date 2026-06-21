<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">
        {{ selectedSection ? selectedSection.name : 'Section Info' }}
      </h2>
      <button
        v-if="selectedSection"
        @click="selectedSection = null"
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

    <div v-else class="flex flex-col sm:flex-row gap-3">
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

    <div v-if="isLoading" class="p-10 text-center text-slate-500">Loading section info...</div>

    <div v-else-if="!selectedSection" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3">
      <template v-if="paginatedSections.length > 0">
        <div
          v-for="section in paginatedSections"
          :key="section.id"
          @click="fetchSectionStudents(section)"
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

    <div v-else class="space-y-4">
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
        <template v-if="paginatedStudents.length > 0">
          <div
            v-for="stu in paginatedStudents"
            :key="stu.student_id"
            class="bg-slate-950/50 border border-white/5 p-4 rounded-xl flex items-center justify-between"
          >
            <div>
              <p class="text-white text-sm font-medium">{{ stu.student_name }}</p>
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
const students = ref([])
const selectedSection = ref(null)
const isLoading = ref(false)
const errorMessage = ref(null)

const searchQuery = ref('')
const statusFilter = ref('all')
const currentPage = ref(1)

const studentSearchQuery = ref('')
const enrollmentFilter = ref('all')
const studentCurrentPage = ref(1)

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

const fetchSectionStudents = async (section) => {
  isLoading.value = true
  studentCurrentPage.value = 1
  try {
    const res = await api.get(`/sections/${section.id}/students`)
    students.value = res.data?.data?.section?.students || []
    selectedSection.value = section
  } catch (err) {
    errorMessage.value = 'Failed to load students.'
  } finally {
    isLoading.value = false
  }
}

const changePage = (p) => {
  if (p >= 1 && p <= lastPage.value) currentPage.value = p
}

onMounted(fetchSections)
</script>
