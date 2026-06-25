<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">Attached Students</h2>
      <button
        @click="openModal()"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        Attach Student
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
        placeholder="Search by guardian or student..."
        class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-sm text-white placeholder-slate-600 outline-none focus:border-white/20 transition"
      />
    </div>

    <div class="border border-white/5 rounded-xl overflow-hidden min-h-[100px]">
      <table class="w-full text-left text-sm table-fixed">
        <thead class="bg-white/5 text-slate-400 uppercase text-[10px] tracking-wider">
          <tr>
            <th class="p-4 font-medium w-1/4">Guardian</th>
            <th class="p-4 font-medium w-32">Relationship</th>
            <th class="p-4 font-medium w-1/4">Student</th>
            <th class="p-4 font-medium w-20">Primary Contact</th>
            <th class="p-4 font-medium w-20 text-right">Action</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-if="isLoading">
            <td colspan="5" class="p-10 text-center text-slate-500">
              Loading attached students...
            </td>
          </tr>
          <template v-else-if="totalStudentCount > 0">
            <template v-for="guardian in paginatedData" :key="guardian.id">
              <tr v-for="student in guardian.students" :key="student.id" class="text-slate-300">
                <td class="p-4 truncate">{{ guardian.name }}</td>
                <td class="p-4 text-slate-500 truncate">{{ student.relationship_type }}</td>
                <td class="p-4 truncate">{{ student.name }}</td>
                <td class="p-4">
                  <span
                    :class="[
                      'text-[10px] px-2 py-1 rounded',
                      student.is_primary_contact
                        ? 'bg-indigo-500/20 text-indigo-300'
                        : 'bg-white/5',
                    ]"
                  >
                    {{ student.is_primary_contact ? 'Yes' : 'No' }}
                  </span>
                </td>
                <td class="p-4 text-right">
                  <button
                    @click="confirmDelete(guardian.id, student.id, student.name)"
                    class="text-rose-500 hover:text-rose-400 transition"
                  >
                    Detach
                  </button>
                </td>
              </tr>
            </template>
          </template>
          <tr v-else>
            <td colspan="5" class="p-10 text-center text-slate-500 italic">No records found.</td>
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

    <AttachStudentModal v-if="showModal" @close="showModal = false" @refresh="fetchData" />

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
import AttachStudentModal from './AttachStudentModal.vue'

const data = ref([])
const isLoading = ref(false)
const errorMessage = ref(null)
const showModal = ref(false)
const searchQuery = ref('')
const currentPage = ref(1)
const itemsPerPage = 7
const toast = ref({ show: false, message: '', type: 'success', isConfirm: false, onConfirm: null })

const filteredData = computed(() => {
  return data.value.filter((g) => {
    return (
      g.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      g.students?.some((s) => s.name.toLowerCase().includes(searchQuery.value.toLowerCase()))
    )
  })
})

const totalStudentCount = computed(() => {
  return filteredData.value.reduce((acc, g) => acc + (g.students?.length || 0), 0)
})

const lastPage = computed(() => Math.ceil(filteredData.value.length / itemsPerPage) || 1)
const paginatedData = computed(() =>
  filteredData.value.slice(
    (currentPage.value - 1) * itemsPerPage,
    currentPage.value * itemsPerPage,
  ),
)

watch(searchQuery, () => (currentPage.value = 1))

const fetchData = async () => {
  isLoading.value = true
  errorMessage.value = null
  try {
    const res = await api.get('/guardians')
    data.value = res.data?.data?.guardians || []
  } catch (e) {
    errorMessage.value = 'Failed to load records.'
  } finally {
    isLoading.value = false
  }
}

const openModal = () => (showModal.value = true)
const changePage = (p) => {
  if (p >= 1 && p <= lastPage.value) currentPage.value = p
}

const showToast = (message, type = 'success', isConfirm = false, onConfirm = null) => {
  toast.value = { show: true, message, type, isConfirm, onConfirm }
  if (!isConfirm) setTimeout(() => (toast.value.show = false), 3000)
}

const handleConfirm = () => {
  if (toast.value.onConfirm) toast.value.onConfirm()
  toast.value.show = false
}

const confirmDelete = (guardianId, studentId, studentName) => {
  showToast(`Are you sure you want to detach ${studentName}?`, 'error', true, async () => {
    try {
      await api.delete(`/guardians/${guardianId}/detach-student/${studentId}`)
      showToast('Student successfully detached.', 'success')
      fetchData()
    } catch (e) {
      showToast('Failed to detach student.', 'error')
    }
  })
}

onMounted(fetchData)
</script>
