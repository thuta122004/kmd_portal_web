<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">Lecturer Assignments</h2>
      <button
        @click="openModal(null)"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        Assign Lecturer
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
        placeholder="Search by lecturer or subject..."
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
            <th class="p-4 font-medium w-1/4">Section</th>
            <th class="p-4 font-medium w-1/4">Subject</th>
            <th class="p-4 font-medium w-1/4">Lecturer</th>
            <th class="p-4 font-medium w-32">Primary</th>
            <th class="p-4 font-medium w-20">Status</th>
            <th class="p-4 font-medium w-20 text-right">Action</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-if="isLoading">
            <td colspan="6" class="p-10 text-center text-slate-500">Loading assignments...</td>
          </tr>
          <template v-else-if="paginatedData.length > 0">
            <tr v-for="item in paginatedData" :key="item.id" class="text-slate-300">
              <td class="p-4 truncate">{{ item.section_name }}</td>
              <td class="p-4 truncate">{{ item.subject_name }}</td>
              <td class="p-4 truncate">{{ item.lecturer_name }}</td>
              <td class="p-4">
                <span
                  :class="[
                    'text-[10px] px-2 py-1 rounded',
                    item.is_primary ? 'bg-indigo-500/20 text-indigo-300' : 'bg-white/5',
                  ]"
                >
                  {{ item.is_primary ? 'Yes' : 'No' }}
                </span>
              </td>
              <td class="p-4">
                <button
                  @click="handleToggleStatus(item)"
                  :class="[
                    'w-8 h-4 rounded-full transition-colors relative',
                    item.status === 'active' ? 'bg-blue-500' : 'bg-slate-700',
                  ]"
                >
                  <span
                    :class="[
                      'absolute top-1 w-2 h-2 rounded-full bg-white transition-all',
                      item.status === 'active' ? 'left-5' : 'left-1',
                    ]"
                  ></span>
                </button>
              </td>
              <td class="p-4 text-right">
                <button @click="openModal(item)" class="text-slate-500 hover:text-white transition">
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

    <AssignLecturerModal
      v-if="showModal"
      :assignmentToEdit="selectedAssignment"
      @close="showModal = false"
      @refresh="fetchData"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import api from '@/services/api'
import AssignLecturerModal from './AssignLecturerModal.vue'

const data = ref([])
const isLoading = ref(false)
const errorMessage = ref(null)
const showModal = ref(false)
const selectedAssignment = ref(null)
const searchQuery = ref('')
const statusFilter = ref('all')
const currentPage = ref(1)
const itemsPerPage = 7

const filteredData = computed(() => {
  return data.value.filter(
    (i) =>
      (i.lecturer_name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
        i.subject_name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
        i.section_name.toLowerCase().includes(searchQuery.value.toLowerCase())) &&
      (statusFilter.value === 'all' || i.status === statusFilter.value),
  )
})

const lastPage = computed(() => Math.ceil(filteredData.value.length / itemsPerPage) || 1)
const paginatedData = computed(() =>
  filteredData.value.slice(
    (currentPage.value - 1) * itemsPerPage,
    currentPage.value * itemsPerPage,
  ),
)

watch([searchQuery, statusFilter], () => (currentPage.value = 1))

const fetchData = async () => {
  isLoading.value = true
  try {
    const res = await api.get('/section-assignments')
    data.value = res.data?.data?.assignments || []
  } catch (e) {
    errorMessage.value = 'Failed to load assignments.'
  } finally {
    isLoading.value = false
  }
}

const changePage = (p) => {
  if (p >= 1 && p <= lastPage.value) currentPage.value = p
}

const handleToggleStatus = async (item) => {
  await api.patch(`/section-assignments/${item.id}/toggle`)
  item.status = item.status === 'active' ? 'inactive' : 'active'
}

const openModal = (item) => {
  selectedAssignment.value = item
  showModal.value = true
}

onMounted(fetchData)
</script>
