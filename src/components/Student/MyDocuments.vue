<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">My Academic Documents</h2>
    </div>

    <div
      v-if="errorMessage"
      class="p-4 bg-rose-500/10 border border-rose-500/20 rounded-xl text-rose-300 text-sm"
    >
      {{ errorMessage }}
    </div>

    <div class="border border-white/5 rounded-xl overflow-hidden bg-slate-950/30">
      <table class="w-full text-left text-sm table-fixed">
        <thead class="bg-white/5 text-slate-400 uppercase text-[10px] tracking-wider">
          <tr>
            <th class="p-4 font-medium w-1/2">Document Title</th>
            <th class="p-4 font-medium w-1/4">Type</th>
            <th class="p-4 font-medium w-1/4">Date</th>
            <th class="p-4 font-medium w-32 text-right">Actions</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-if="isLoading">
            <td colspan="4" class="p-10 text-center text-slate-500">Loading documents...</td>
          </tr>

          <template v-else-if="documents.length > 0">
            <tr
              v-for="doc in documents"
              :key="doc.id"
              class="text-slate-300 hover:bg-white/5 transition"
            >
              <td class="p-4 font-medium text-white truncate">{{ doc.title }}</td>
              <td class="p-4 text-slate-500">{{ doc.document_type }}</td>
              <td class="p-4 text-slate-500">{{ formatDate(doc.created_at) }}</td>
              <td class="p-4 text-right">
                <div class="flex items-center justify-end gap-3">
                  <a
                    :href="doc.file_url"
                    target="_blank"
                    class="text-blue-400 hover:text-blue-300 transition"
                    title="View"
                  >
                    <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"
                      />
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"
                      />
                    </svg>
                  </a>

                  <!-- Download Button with Loading State -->
                  <button
                    @click="downloadDocument(doc)"
                    :disabled="downloadingIds.has(doc.id)"
                    class="text-slate-400 hover:text-white transition disabled:opacity-50 disabled:cursor-not-allowed"
                    title="Download"
                  >
                    <svg
                      v-if="downloadingIds.has(doc.id)"
                      class="w-5 h-5 animate-spin"
                      viewBox="0 0 24 24"
                    >
                      <circle
                        class="opacity-25"
                        cx="12"
                        cy="12"
                        r="10"
                        stroke="currentColor"
                        stroke-width="4"
                        fill="none"
                      ></circle>
                      <path
                        class="opacity-75"
                        fill="currentColor"
                        d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"
                      ></path>
                    </svg>

                    <svg
                      v-else
                      class="w-5 h-5"
                      fill="none"
                      viewBox="0 0 24 24"
                      stroke="currentColor"
                    >
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"
                      />
                    </svg>
                  </button>
                </div>
              </td>
            </tr>
          </template>

          <tr v-else>
            <td colspan="4" class="p-10 text-center text-slate-500 italic">No documents found.</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import api from '@/services/api'

const documents = ref([])
const isLoading = ref(false)
const errorMessage = ref(null)
const downloadingIds = ref(new Set())

const formatDate = (dateStr) => {
  if (!dateStr) return 'N/A'
  return new Date(dateStr).toLocaleDateString('en-GB', {
    day: 'numeric',
    month: 'short',
    year: 'numeric',
  })
}

const initializeDocuments = async () => {
  isLoading.value = true
  errorMessage.value = null

  try {
    const userProfile = JSON.parse(localStorage.getItem('user') || '{}')
    const currentUserId = userProfile.id

    if (!currentUserId) {
      throw new Error('User session not found.')
    }

    const res = await api.get('/academic-documents', {
      params: { user_id: currentUserId },
    })

    documents.value = res.data?.data?.documents || []
  } catch (err) {
    errorMessage.value = 'Failed to load your documents.'
    console.error(err)
  } finally {
    isLoading.value = false
  }
}

const downloadDocument = async (doc) => {
  downloadingIds.value.add(doc.id)

  try {
    const res = await api.get(`/academic-documents/${doc.id}/download`, {
      responseType: 'blob',
    })

    const contentType = res.headers['content-type']
    const blob = new Blob([res.data], { type: contentType })
    const url = window.URL.createObjectURL(blob)

    let fileName = doc.title
    if (!fileName.includes('.')) {
      const extension = contentType.split('/')[1] || 'bin'
      fileName = `${fileName}.${extension}`
    }

    const a = document.createElement('a')
    a.href = url
    a.download = fileName
    document.body.appendChild(a)
    a.click()
    a.remove()
    window.URL.revokeObjectURL(url)
  } catch (error) {
    console.error('Download failed:', error)
    errorMessage.value = 'Failed to download file. Please check permissions.'
  } finally {
    downloadingIds.value.delete(doc.id)
  }
}

onMounted(initializeDocuments)
</script>
