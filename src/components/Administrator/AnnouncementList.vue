<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">Announcements</h2>
      <button
        @click="openModal(null)"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        Create Announcement
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
        placeholder="Search announcements..."
        class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-sm text-white placeholder-slate-600 outline-none focus:border-white/20 transition"
      />
      <select
        v-model="pinnedFilter"
        class="px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 cursor-pointer appearance-none"
      >
        <option value="all">All Items</option>
        <option value="pinned">Pinned</option>
        <option value="regular">Regular</option>
      </select>
    </div>

    <div class="border border-white/5 rounded-xl overflow-hidden min-h-[100px]">
      <table class="w-full text-left text-sm table-fixed">
        <thead class="bg-white/5 text-slate-400 uppercase text-[10px] tracking-wider">
          <tr>
            <th class="p-4 font-medium w-1/4">Target Audience</th>
            <th class="p-4 font-medium w-1/4">Title</th>
            <th class="p-4 font-medium w-1/3">Content Excerpt</th>
            <th class="p-4 font-medium w-24">Pinned</th>
            <th class="p-4 font-medium w-44 text-right">Action</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-if="isLoading">
            <td colspan="5" class="p-10 text-center text-slate-500">Loading announcements...</td>
          </tr>

          <template v-else-if="paginatedAnnouncements.length > 0">
            <tr
              v-for="announcement in paginatedAnnouncements"
              :key="announcement.id"
              class="text-slate-300"
            >
              <td class="p-4 truncate">
                <span
                  :class="[
                    'px-2 py-0.5 rounded text-[10px] font-semibold tracking-wide uppercase',
                    announcement.section_id
                      ? 'bg-blue-500/10 text-blue-400 border border-blue-500/20'
                      : 'bg-purple-500/10 text-purple-400 border border-purple-500/20',
                  ]"
                >
                  {{ announcement.section_name }}
                </span>
              </td>
              <td class="p-4 truncate font-medium">
                <a
                  v-if="announcement.banner_url"
                  :href="announcement.banner_url"
                  target="_blank"
                  class="text-blue-400 hover:underline inline-flex items-center gap-1.5"
                >
                  <svg
                    class="w-3.5 h-3.5 flex-shrink-0"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"
                    />
                  </svg>
                  {{ announcement.title }}
                </a>
                <span v-else>{{ announcement.title }}</span>
              </td>
              <td class="p-4 text-slate-500 truncate">{{ announcement.content }}</td>
              <td class="p-4">
                <span
                  :class="[
                    'text-xs px-2 py-0.5 rounded-md font-medium',
                    announcement.is_pinned
                      ? 'bg-amber-500/10 text-amber-400'
                      : 'bg-slate-800 text-slate-500',
                  ]"
                >
                  {{ announcement.is_pinned ? 'Yes' : 'No' }}
                </span>
              </td>
              <td class="p-4 text-right flex justify-end gap-2">
                <button
                  @click="togglePinnedStatus(announcement)"
                  :class="[
                    'text-xs px-2 py-1 rounded transition',
                    announcement.is_pinned
                      ? 'text-slate-400 hover:text-white'
                      : 'text-amber-400 hover:text-amber-300',
                  ]"
                >
                  {{ announcement.is_pinned ? 'Unpin' : 'Pin' }}
                </button>
                <button
                  @click="openModal(announcement)"
                  class="text-slate-500 hover:text-white transition"
                >
                  Edit
                </button>
                <button
                  @click="deleteAnnouncement(announcement.id)"
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
                searchQuery || pinnedFilter !== 'all'
                  ? 'No matching announcements found.'
                  : 'No announcements created yet.'
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

    <AnnouncementModal
      v-if="showModal"
      :announcementToEdit="selectedAnnouncement"
      @close="showModal = false"
      @refresh="fetchAnnouncements"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import api from '@/services/api'
import AnnouncementModal from './AnnouncementModal.vue'

const announcements = ref([])
const isLoading = ref(false)
const errorMessage = ref(null)
const showModal = ref(false)
const selectedAnnouncement = ref(null)
const searchQuery = ref('')
const pinnedFilter = ref('all')
const currentPage = ref(1)
const itemsPerPage = 7
const toast = ref({ show: false, message: '', type: 'success', isConfirm: false, onConfirm: null })

const filteredAnnouncements = computed(() => {
  return announcements.value.filter((a) => {
    const matchesSearch =
      a.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      a.content.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      a.section_name.toLowerCase().includes(searchQuery.value.toLowerCase())

    const matchesPinned =
      pinnedFilter.value === 'all' || (pinnedFilter.value === 'pinned' ? a.is_pinned : !a.is_pinned)

    return matchesSearch && matchesPinned
  })
})

const lastPage = computed(() => Math.ceil(filteredAnnouncements.value.length / itemsPerPage) || 1)

const paginatedAnnouncements = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return filteredAnnouncements.value.slice(start, itemsPerPage + start)
})

watch([searchQuery, pinnedFilter], () => {
  currentPage.value = 1
})

const fetchAnnouncements = async () => {
  isLoading.value = true
  try {
    const res = await api.get('/announcements')
    announcements.value = res.data?.data?.announcements || []
  } catch (e) {
    errorMessage.value = 'Failed to load announcements.'
  } finally {
    isLoading.value = false
  }
}

const changePage = (newPage) => {
  if (newPage >= 1 && newPage <= lastPage.value) currentPage.value = newPage
}

const togglePinnedStatus = async (announcement) => {
  try {
    await api.put(`/announcements/${announcement.id}/pinned`, {
      is_pinned: !announcement.is_pinned ? 1 : 0,
    })
    fetchAnnouncements()
  } catch (e) {
    showToast('Failed to change pin priority state.', 'error')
  }
}

const deleteAnnouncement = (id) => {
  showToast(
    'Are you sure you want to permanently clear this announcement entry?',
    'error',
    true,
    async () => {
      try {
        await api.delete(`/announcements/${id}`)
        showToast('Announcement dropped successfully.', 'success')
        fetchAnnouncements()
      } catch (e) {
        showToast('Failed to complete content erasure context.', 'error')
      }
    },
  )
}

const showToast = (message, type = 'success', isConfirm = false, onConfirm = null) => {
  toast.value = { show: true, message, type, isConfirm, onConfirm }
  if (!isConfirm) setTimeout(() => (toast.value.show = false), 3000)
}

const handleConfirm = () => {
  if (toast.value.onConfirm) toast.value.onConfirm()
  toast.value.show = false
}

const openModal = (announcement = null) => {
  selectedAnnouncement.value = announcement
  showModal.value = true
}

onMounted(fetchAnnouncements)
</script>
