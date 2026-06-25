<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">Lecturers</h2>
      <button
        @click="openModal(null)"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        Add Lecturer
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
        placeholder="Search lecturers..."
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
            <th class="p-4 font-medium w-1/4">Department</th>
            <th class="p-4 font-medium w-1/4">Qualification</th>
            <th class="p-4 font-medium w-32">Employee ID</th>
            <th class="p-4 font-medium w-20">Status</th>
            <th class="p-4 font-medium w-20 text-right">Action</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-if="isLoading">
            <td colspan="6" class="p-10 text-center text-slate-500">Loading lecturers...</td>
          </tr>
          <template v-else-if="paginatedLecturers.length > 0">
            <tr v-for="lec in paginatedLecturers" :key="lec.id" class="text-slate-300">
              <td class="p-4 truncate cursor-pointer hover:text-blue-400" @click="openModal(lec)">
                {{ lec.name }}
              </td>
              <td class="p-4 text-slate-500 truncate">{{ lec.department || '-' }}</td>
              <td class="p-4 text-slate-500 truncate">{{ lec.qualification || '-' }}</td>
              <td class="p-4">{{ lec.employee_id || '-' }}</td>
              <td class="p-4">
                <span
                  :class="[
                    'text-xs px-2 py-0.5 rounded-md font-medium capitalize',
                    lec.status === 'active'
                      ? 'bg-blue-500/10 text-blue-400'
                      : 'bg-slate-500/10 text-slate-400',
                  ]"
                >
                  {{ lec.status }}
                </span>
              </td>
              <td class="p-4 text-right flex justify-end gap-3">
                <button @click="openModal(lec)" class="text-slate-500 hover:text-white transition">
                  Edit
                </button>
              </td>
            </tr>
          </template>
          <tr v-else>
            <td colspan="6" class="p-10 text-center text-slate-500 italic">No records found.</td>
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

    <LecturerModal
      v-if="showModal"
      :lecturerToEdit="selectedLecturer"
      @close="showModal = false"
      @refresh="fetchLecturers"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import api from '@/services/api'
import LecturerModal from './LecturerModal.vue'

const lecturers = ref([])
const isLoading = ref(false)
const errorMessage = ref(null)
const showModal = ref(false)
const selectedLecturer = ref(null)
const searchQuery = ref('')
const statusFilter = ref('all')
const currentPage = ref(1)
const itemsPerPage = 7

const filteredLecturers = computed(() => {
  return lecturers.value.filter((l) => {
    const matchesSearch = l.name.toLowerCase().includes(searchQuery.value.toLowerCase())
    const matchesStatus = statusFilter.value === 'all' || l.status === statusFilter.value
    return matchesSearch && matchesStatus
  })
})

const lastPage = computed(() => Math.ceil(filteredLecturers.value.length / itemsPerPage) || 1)

const paginatedLecturers = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return filteredLecturers.value.slice(start, itemsPerPage + start)
})

watch([searchQuery, statusFilter], () => {
  currentPage.value = 1
})

const fetchLecturers = async () => {
  isLoading.value = true
  errorMessage.value = null
  try {
    const response = await api.get('/lecturers')
    lecturers.value = response.data?.data?.lecturers || []
  } catch (error) {
    errorMessage.value = 'Failed to load lecturers.'
  } finally {
    isLoading.value = false
  }
}

const changePage = (newPage) => {
  if (newPage >= 1 && newPage <= lastPage.value) currentPage.value = newPage
}

const openModal = (lec = null) => {
  selectedLecturer.value = lec
  showModal.value = true
}

onMounted(fetchLecturers)
</script>
