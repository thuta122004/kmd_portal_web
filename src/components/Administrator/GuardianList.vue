<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">Guardians</h2>
      <button
        @click="openModal(null)"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        Add Guardian
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
        placeholder="Search guardians..."
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
            <th class="p-4 font-medium w-1/4">Address</th>
            <th class="p-4 font-medium w-1/4">Occupation</th>
            <th class="p-4 font-medium w-20">Status</th>
            <th class="p-4 font-medium w-20 text-right">Action</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-if="isLoading">
            <td colspan="6" class="p-10 text-center text-slate-500">Loading guardians...</td>
          </tr>
          <template v-else-if="paginatedGuardians.length > 0">
            <tr v-for="g in paginatedGuardians" :key="g.id" class="text-slate-300">
              <td class="p-4 truncate cursor-pointer hover:text-blue-400" @click="openModal(g)">
                {{ g.name }}
              </td>
              <td class="p-4 text-slate-500 truncate">{{ g.phone }}</td>
              <td class="p-4 text-slate-500 truncate">{{ g.address || '-' }}</td>
              <td class="p-4 text-slate-500 truncate">{{ g.occupation || '-' }}</td>
              <td class="p-4">
                <button
                  @click="handleToggleStatus(g)"
                  :class="[
                    'w-8 h-4 rounded-full transition-colors relative',
                    g.status === 'active' ? 'bg-blue-500' : 'bg-slate-700',
                  ]"
                >
                  <span
                    :class="[
                      'absolute top-1 w-2 h-2 rounded-full bg-white transition-all',
                      g.status === 'active' ? 'left-5' : 'left-1',
                    ]"
                  ></span>
                </button>
              </td>
              <td class="p-4 text-right">
                <button @click="openModal(g)" class="text-slate-500 hover:text-white transition">
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

    <GuardianModal
      v-if="showModal"
      :guardianToEdit="selectedGuardian"
      @close="showModal = false"
      @refresh="fetchGuardians"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import api from '@/services/api'
import GuardianModal from './GuardianModal.vue'

const guardians = ref([])
const isLoading = ref(false)
const errorMessage = ref(null)
const showModal = ref(false)
const selectedGuardian = ref(null)
const searchQuery = ref('')
const statusFilter = ref('all')
const currentPage = ref(1)
const itemsPerPage = 7

const filteredGuardians = computed(() => {
  return guardians.value.filter(
    (g) =>
      g.name.toLowerCase().includes(searchQuery.value.toLowerCase()) &&
      (statusFilter.value === 'all' || g.status === statusFilter.value),
  )
})

const lastPage = computed(() => Math.ceil(filteredGuardians.value.length / itemsPerPage) || 1)
const paginatedGuardians = computed(() =>
  filteredGuardians.value.slice(
    (currentPage.value - 1) * itemsPerPage,
    currentPage.value * itemsPerPage,
  ),
)

watch([searchQuery, statusFilter], () => (currentPage.value = 1))

const fetchGuardians = async () => {
  isLoading.value = true
  try {
    const response = await api.get('/guardians')
    guardians.value = response.data?.data?.guardians || []
  } catch (e) {
    errorMessage.value = 'Failed to load guardians.'
  } finally {
    isLoading.value = false
  }
}

const changePage = (p) => {
  if (p >= 1 && p <= lastPage.value) currentPage.value = p
}
const openModal = (g) => {
  selectedGuardian.value = g
  showModal.value = true
}
const handleToggleStatus = async (g) => {
  await api.patch(`/guardians/${g.id}/toggle`)
  g.status = g.status === 'active' ? 'inactive' : 'active'
}

onMounted(fetchGuardians)
</script>
