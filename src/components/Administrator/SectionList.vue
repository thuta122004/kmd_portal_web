<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">Sections</h2>
      <button
        @click="openModal(null)"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        Add Section
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
        placeholder="Search sections..."
        class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-sm text-white placeholder-slate-600 outline-none focus:border-white/20 transition"
      />
      <select
        v-model="statusFilter"
        class="px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 cursor-pointer appearance-none"
      >
        <option value="all">All Status</option>
        <option value="pending">Pending</option>
        <option value="active">Active</option>
        <option value="inactive">Inactive</option>
      </select>
    </div>

    <div class="border border-white/5 rounded-xl overflow-hidden min-h-[100px]">
      <table class="w-full text-left text-sm table-fixed">
        <thead class="bg-white/5 text-slate-400 uppercase text-[10px] tracking-wider">
          <tr>
            <th class="p-4 font-medium w-1/4">Name</th>
            <th class="p-4 font-medium w-1/4">Start Date</th>
            <th class="p-4 font-medium w-1/4">End Date</th>
            <th class="p-4 font-medium w-32">Code</th>
            <th class="p-4 font-medium w-20">Status</th>
            <th class="p-4 font-medium w-20 text-right">Action</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-if="isLoading">
            <td colspan="6" class="p-10 text-center text-slate-500">Loading sections...</td>
          </tr>
          <template v-else-if="paginatedSections.length > 0">
            <tr v-for="sec in paginatedSections" :key="sec.id" class="text-slate-300">
              <td class="p-4 truncate cursor-pointer hover:text-blue-400" @click="openModal(sec)">
                {{ sec.name }}
              </td>
              <td class="p-4 text-slate-500">{{ sec.start_date }}</td>
              <td class="p-4 text-slate-500">{{ sec.end_date || '-' }}</td>
              <td class="p-4">{{ sec.code }}</td>
              <td class="p-4">
                <button
                  @click="handleToggleStatus(sec)"
                  :class="[
                    'px-2 py-1 rounded text-[10px] font-bold uppercase',
                    sec.status === 'active'
                      ? 'bg-blue-500/20 text-blue-400'
                      : 'bg-slate-700/50 text-slate-400',
                  ]"
                >
                  {{ sec.status }}
                </button>
              </td>
              <td class="p-4 text-right">
                <button @click="openModal(sec)" class="text-slate-500 hover:text-white transition">
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

    <SectionModal
      v-if="showModal"
      :sectionToEdit="selectedSection"
      @close="showModal = false"
      @refresh="fetchSections"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import api from '@/services/api'
import SectionModal from './SectionModal.vue'

const sections = ref([])
const isLoading = ref(false)
const errorMessage = ref(null)
const showModal = ref(false)
const selectedSection = ref(null)
const searchQuery = ref('')
const statusFilter = ref('all')
const currentPage = ref(1)
const itemsPerPage = 7

const filteredSections = computed(() => {
  return sections.value.filter(
    (s) =>
      (s.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
        s.code.toLowerCase().includes(searchQuery.value.toLowerCase())) &&
      (statusFilter.value === 'all' || s.status === statusFilter.value),
  )
})

const lastPage = computed(() => Math.ceil(filteredSections.value.length / itemsPerPage) || 1)
const paginatedSections = computed(() =>
  filteredSections.value.slice(
    (currentPage.value - 1) * itemsPerPage,
    currentPage.value * itemsPerPage,
  ),
)

watch([searchQuery, statusFilter], () => (currentPage.value = 1))

const fetchSections = async () => {
  isLoading.value = true
  try {
    const response = await api.get('/sections')
    sections.value = response.data?.data?.sections || []
  } catch (e) {
    errorMessage.value = 'Failed to load sections.'
  } finally {
    isLoading.value = false
  }
}

const changePage = (p) => {
  if (p >= 1 && p <= lastPage.value) currentPage.value = p
}

const openModal = (s) => {
  selectedSection.value = s
  showModal.value = true
}

const handleToggleStatus = async (s) => {
  await api.patch(`/sections/${s.id}/toggle`)
  fetchSections()
}

onMounted(fetchSections)
</script>
