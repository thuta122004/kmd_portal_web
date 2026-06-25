<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">Enrolments</h2>
      <button
        @click="openModal(null)"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        Add Enrolment
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
        placeholder="Search student or section..."
        class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-sm text-white placeholder-slate-600 outline-none focus:border-white/20 transition"
      />
      <select
        v-model="statusFilter"
        class="px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 cursor-pointer appearance-none"
      >
        <option value="all">All Status</option>
        <option value="active">Active</option>
        <option value="inactive">Inactive</option>
        <option value="suspended">Suspended</option>
      </select>
    </div>

    <div class="border border-white/5 rounded-xl overflow-hidden min-h-[100px]">
      <table class="w-full text-left text-sm table-fixed">
        <thead class="bg-white/5 text-slate-400 uppercase text-[10px] tracking-wider">
          <tr>
            <th class="p-4 font-medium w-1/4">Student Name</th>
            <th class="p-4 font-medium w-1/4">Reg Number</th>
            <th class="p-4 font-medium w-1/4">Section Name</th>
            <th class="p-4 font-medium w-1/4">Note</th>
            <th class="p-4 font-medium w-32">Status</th>
            <th class="p-4 font-medium w-20">Toggle</th>
            <th class="p-4 font-medium w-20 text-right">Action</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-if="isLoading">
            <td colspan="7" class="p-10 text-center text-slate-500">Loading enrolments...</td>
          </tr>
          <template v-else-if="paginatedEnrolments.length > 0">
            <tr v-for="item in paginatedEnrolments" :key="item.id" class="text-slate-300">
              <td class="p-4 truncate cursor-pointer hover:text-blue-400" @click="openModal(item)">
                {{ item.student_name }}
              </td>
              <td class="p-4 text-slate-500 truncate">{{ item.student_reg_number }}</td>
              <td class="p-4 text-slate-300 truncate">{{ item.section_name }}</td>
              <td class="p-4 text-slate-500 truncate">{{ item.note || '-' }}</td>
              <td class="p-4">
                <span
                  class="text-[10px] uppercase font-bold px-2 py-0.5 rounded"
                  :class="[
                    item.status === 'active'
                      ? 'bg-blue-500/20 text-blue-400'
                      : item.status === 'inactive'
                        ? 'bg-slate-700/50 text-slate-400'
                        : 'bg-amber-500/20 text-amber-400',
                  ]"
                >
                  {{ item.status }}
                </span>
              </td>
              <td class="p-4">
                <button
                  @click="handleToggleStatus(item)"
                  :class="[
                    'w-8 h-4 rounded-full transition-colors relative',
                    item.status === 'active'
                      ? 'bg-blue-500'
                      : item.status === 'inactive'
                        ? 'bg-slate-700'
                        : 'bg-amber-500',
                  ]"
                >
                  <span
                    :class="[
                      'absolute top-1 w-2 h-2 rounded-full bg-white transition-all',
                      item.status === 'active'
                        ? 'left-5'
                        : item.status === 'inactive'
                          ? 'left-1'
                          : 'left-3',
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
            <td colspan="7" class="p-10 text-center text-slate-500 italic">No records found.</td>
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

    <EnrolmentModal
      v-if="showModal"
      :enrolmentToEdit="selectedEnrolment"
      @close="showModal = false"
      @refresh="fetchEnrolments"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import api from '@/services/api'
import EnrolmentModal from './EnrolmentModal.vue'

const enrolments = ref([])
const isLoading = ref(false)
const errorMessage = ref(null)
const showModal = ref(false)
const selectedEnrolment = ref(null)
const searchQuery = ref('')
const statusFilter = ref('all')
const currentPage = ref(1)
const itemsPerPage = 7

const filteredEnrolments = computed(() => {
  return enrolments.value.filter(
    (e) =>
      (e.student_name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
        e.section_name.toLowerCase().includes(searchQuery.value.toLowerCase())) &&
      (statusFilter.value === 'all' || e.status === statusFilter.value),
  )
})

const lastPage = computed(() => Math.ceil(filteredEnrolments.value.length / itemsPerPage) || 1)
const paginatedEnrolments = computed(() =>
  filteredEnrolments.value.slice(
    (currentPage.value - 1) * itemsPerPage,
    currentPage.value * itemsPerPage,
  ),
)

watch([searchQuery, statusFilter], () => (currentPage.value = 1))

const fetchEnrolments = async () => {
  isLoading.value = true
  try {
    const response = await api.get('/enrolments')
    enrolments.value = response.data?.data?.enrolments || []
  } catch (e) {
    errorMessage.value = 'Failed to load enrolments.'
  } finally {
    isLoading.value = false
  }
}

const changePage = (p) => {
  if (p >= 1 && p <= lastPage.value) currentPage.value = p
}

const openModal = (e) => {
  selectedEnrolment.value = e
  showModal.value = true
}

const handleToggleStatus = async (item) => {
  try {
    const res = await api.patch(`/enrolments/${item.id}/toggle`)
    item.status = res.data?.data?.enrolment?.status || item.status
  } catch (e) {
    console.error(e)
  }
}

onMounted(fetchEnrolments)
</script>
