<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">Academic Documents</h2>
      <button
        @click="openModal(null)"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        Upload Document
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
        placeholder="Search documents..."
        class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-sm text-white placeholder-slate-600 outline-none focus:border-white/20 transition"
      />
      <select
        v-model="statusFilter"
        class="px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 cursor-pointer appearance-none"
      >
        <option value="all">All Status</option>
        <option value="verified">Verified</option>
        <option value="pending">Pending</option>
      </select>
    </div>

    <div class="border border-white/5 rounded-xl overflow-hidden min-h-[100px]">
      <table class="w-full text-left text-sm table-fixed">
        <thead class="bg-white/5 text-slate-400 uppercase text-[10px] tracking-wider">
          <tr>
            <th class="p-4 font-medium w-1/4">Student</th>
            <th class="p-4 font-medium w-1/4">Type</th>
            <th class="p-4 font-medium w-1/4">Title</th>
            <th class="p-4 font-medium w-20">Verified</th>
            <th class="p-4 font-medium w-40 text-right">Action</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-if="isLoading">
            <td colspan="5" class="p-10 text-center text-slate-500">Loading documents...</td>
          </tr>
          <template v-else-if="paginatedDocs.length > 0">
            <tr v-for="doc in paginatedDocs" :key="doc.id" class="text-slate-300">
              <td class="p-4 truncate">{{ doc.student_name }}</td>
              <td class="p-4 text-slate-500 truncate">{{ doc.document_type }}</td>
              <td class="p-4 truncate">
                <a :href="doc.file_url" target="_blank" class="text-blue-400 hover:underline">
                  {{ doc.title }}
                </a>
              </td>
              <td class="p-4">
                <span
                  :class="[
                    'text-xs px-2 py-0.5 rounded-md font-medium capitalize',
                    doc.is_verified
                      ? 'bg-emerald-500/10 text-emerald-400'
                      : 'bg-amber-500/10 text-amber-400',
                  ]"
                >
                  {{ doc.is_verified ? 'Yes' : 'No' }}
                </span>
              </td>
              <td class="p-4 text-right flex justify-end gap-2">
                <button
                  @click="toggleVerification(doc)"
                  :class="[
                    'text-xs px-2 py-1 rounded transition',
                    doc.is_verified
                      ? 'text-amber-400 hover:text-amber-300'
                      : 'text-emerald-400 hover:text-emerald-300',
                  ]"
                >
                  {{ doc.is_verified ? 'Unverify' : 'Verify' }}
                </button>
                <button @click="openModal(doc)" class="text-slate-500 hover:text-white transition">
                  Edit
                </button>
                <button
                  @click="deleteDocument(doc.id)"
                  class="text-rose-500 hover:text-white transition"
                >
                  Delete
                </button>
              </td>
            </tr>
          </template>
          <tr v-else>
            <td colspan="5" class="p-10 text-center text-slate-500 italic">
              {{
                searchQuery || statusFilter !== 'all'
                  ? 'No matching documents found.'
                  : 'No documents available yet.'
              }}
            </td>
          </tr>
        </tbody>
      </table>
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

    <AcademicDocumentModal
      v-if="showModal"
      :docToEdit="selectedDoc"
      @close="showModal = false"
      @refresh="fetchDocuments"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import api from '@/services/api'
import AcademicDocumentModal from './AcademicDocumentModal.vue'

const documents = ref([])
const isLoading = ref(false)
const errorMessage = ref(null)
const showModal = ref(false)
const selectedDoc = ref(null)
const searchQuery = ref('')
const statusFilter = ref('all')
const currentPage = ref(1)
const itemsPerPage = 7
const toast = ref({ show: false, message: '', type: 'success', isConfirm: false, onConfirm: null })

const filteredDocs = computed(() => {
  return documents.value.filter((d) => {
    const matchesSearch =
      d.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      d.student_name.toLowerCase().includes(searchQuery.value.toLowerCase())
    const matchesStatus =
      statusFilter.value === 'all' ||
      (statusFilter.value === 'verified' ? d.is_verified : !d.is_verified)
    return matchesSearch && matchesStatus
  })
})
const lastPage = computed(() => Math.ceil(filteredDocs.value.length / itemsPerPage) || 1)
const paginatedDocs = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return filteredDocs.value.slice(start, itemsPerPage + start)
})

watch([searchQuery, statusFilter], () => {
  currentPage.value = 1
})

const fetchDocuments = async () => {
  isLoading.value = true
  try {
    const res = await api.get('/academic-documents')
    documents.value = res.data?.data?.documents || []
  } catch (e) {
    errorMessage.value = 'Failed to load.'
  } finally {
    isLoading.value = false
  }
}

const changePage = (newPage) => {
  if (newPage >= 1 && newPage <= lastPage.value) currentPage.value = newPage
}

const toggleVerification = async (doc) => {
  try {
    await api.put(`/academic-documents/${doc.id}/verify`, { is_verified: !doc.is_verified ? 1 : 0 })
    fetchDocuments()
  } catch (e) {
    showToast('Failed to update status.', 'error')
  }
}

const deleteDocument = (id) => {
  showToast('Are you sure you want to delete this document?', 'error', true, async () => {
    try {
      await api.delete(`/academic-documents/${id}`)
      showToast('Document deleted successfully', 'success')
      fetchDocuments()
    } catch (e) {
      showToast('Failed to delete document.', 'error')
    }
  })
}

const showToast = (message, type = 'success', isConfirm = false, onConfirm = null) => {
  toast.value = { show: true, message, type, isConfirm, onConfirm }
  if (!isConfirm) setTimeout(() => (toast.value.show = false), 3000)
}

const handleConfirm = () => {
  if (toast.value.onConfirm) toast.value.onConfirm()
  toast.value.show = false
}

const openModal = (doc = null) => {
  selectedDoc.value = doc
  showModal.value = true
}

onMounted(fetchDocuments)
</script>
