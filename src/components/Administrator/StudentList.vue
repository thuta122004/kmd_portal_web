<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">Students</h2>
      <button
        @click="openModal(null)"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        Add Student
      </button>
    </div>

    <div
      v-if="errorMessage"
      class="p-4 bg-rose-500/10 border border-rose-500/20 rounded-xl text-rose-300 text-sm"
    >
      {{ errorMessage }}
    </div>

    <div class="flex flex-col sm:flex-row gap-3">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Search students..."
        class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-sm text-white placeholder-slate-600 outline-none focus:border-white/20 transition"
      />
      <select
        v-model="statusFilter"
        class="px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 cursor-pointer appearance-none"
      >
        <option value="all">All Status</option>
        <option value="active">Active</option>
        <option value="inactive">Inactive</option>
      </select>
    </div>

    <div class="border border-white/5 rounded-xl overflow-hidden min-h-[100px]">
      <table class="w-full text-left text-sm table-fixed">
        <thead class="bg-white/5 text-slate-400 uppercase text-[10px] tracking-wider">
          <tr>
            <th class="p-4 font-medium w-1/4">Name</th>
            <th class="p-4 font-medium w-1/4">Phone</th>
            <th class="p-4 font-medium w-32">Age</th>
            <th class="p-4 font-medium w-1/4">Address</th>
            <th class="p-4 font-medium w-1/4">Reg Number</th>
            <th class="p-4 font-medium w-32">Gender</th>
            <th class="p-4 font-medium w-20">Status</th>
            <th class="p-4 font-medium w-20 text-right">Action</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-if="isLoading">
            <td colspan="8" class="p-10 text-center text-slate-500">Loading students...</td>
          </tr>
          <template v-else-if="paginatedStudents.length > 0">
            <tr v-for="stu in paginatedStudents" :key="stu.id" class="text-slate-300">
              <td class="p-4 truncate cursor-pointer hover:text-blue-400" @click="openModal(stu)">
                {{ stu.name }}
              </td>
              <td class="p-4 text-slate-500 truncate">{{ stu.phone }}</td>
              <td class="p-4 text-slate-500 truncate">{{ calculateAge(stu.date_of_birth) }}</td>
              <td class="p-4 text-slate-500 truncate">{{ stu.address }}</td>
              <td class="p-4">{{ stu.student_reg_number }}</td>
              <td class="p-4">
                <span class="text-xs bg-white/5 px-2 py-1 rounded">{{ stu.gender }}</span>
              </td>
              <td class="p-4">
                <button
                  @click="handleToggleStatus(stu)"
                  :class="[
                    'w-8 h-4 rounded-full transition-colors relative',
                    stu.status === 'active' ? 'bg-blue-500' : 'bg-slate-700',
                  ]"
                >
                  <span
                    :class="[
                      'absolute top-1 w-2 h-2 rounded-full bg-white transition-all',
                      stu.status === 'active' ? 'left-5' : 'left-1',
                    ]"
                  ></span>
                </button>
              </td>
              <td class="p-4 text-right">
                <button @click="openModal(stu)" class="text-slate-500 hover:text-white transition">
                  Edit
                </button>
              </td>
            </tr>
          </template>
          <tr v-else>
            <td colspan="8" class="p-10 text-center text-slate-500 italic">No records found.</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="lastPage > 1" class="flex justify-between items-center text-xs text-slate-500">
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

    <StudentModal
      v-if="showModal"
      :studentToEdit="selectedStudent"
      @close="showModal = false"
      @refresh="fetchStudents"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import api from '@/services/api'
import StudentModal from './StudentModal.vue'

const students = ref([])
const isLoading = ref(false)
const errorMessage = ref(null)
const showModal = ref(false)
const selectedStudent = ref(null)
const searchQuery = ref('')
const statusFilter = ref('all')
const currentPage = ref(1)
const itemsPerPage = 7

const filteredStudents = computed(() => {
  return students.value.filter(
    (s) =>
      s.name.toLowerCase().includes(searchQuery.value.toLowerCase()) &&
      (statusFilter.value === 'all' || s.status === statusFilter.value),
  )
})

const lastPage = computed(() => Math.ceil(filteredStudents.value.length / itemsPerPage) || 1)
const paginatedStudents = computed(() =>
  filteredStudents.value.slice(
    (currentPage.value - 1) * itemsPerPage,
    currentPage.value * itemsPerPage,
  ),
)

watch([searchQuery, statusFilter], () => (currentPage.value = 1))

const fetchStudents = async () => {
  isLoading.value = true
  try {
    const response = await api.get('/students')
    students.value = response.data?.data?.students || []
  } catch (e) {
    errorMessage.value = 'Failed to load students.'
  } finally {
    isLoading.value = false
  }
}

const changePage = (p) => {
  if (p >= 1 && p <= lastPage.value) currentPage.value = p
}
const openModal = (s) => {
  selectedStudent.value = s
  showModal.value = true
}
const handleToggleStatus = async (s) => {
  await api.patch(`/students/${s.id}/toggle`)
  s.status = s.status === 'active' ? 'inactive' : 'active'
}

const calculateAge = (dob) => {
  if (!dob) return '-'
  const birthDate = new Date(dob)
  const today = new Date()
  let age = today.getFullYear() - birthDate.getFullYear()
  const m = today.getMonth() - birthDate.getMonth()

  if (m < 0 || (m === 0 && today.getDate() < birthDate.getDate())) {
    age--
  }
  return age
}

onMounted(fetchStudents)
</script>
