<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">Attendance Records</h2>

      <div class="flex items-end gap-3">
        <div class="flex flex-col gap-1">
          <span class="text-[10px] uppercase font-semibold text-slate-500 tracking-wider">
            Backfill from
          </span>
          <input
            v-model="refreshStartDate"
            type="date"
            class="bg-slate-950 text-slate-300 text-xs px-3 py-2 rounded-lg border border-white/10 focus:border-white/20 outline-none w-26"
          />
        </div>

        <button
          @click="triggerGlobalRefresh"
          :disabled="isRefreshing"
          class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition disabled:opacity-50"
        >
          <span v-if="isRefreshing">Refreshing...</span>
          <span v-else>Refresh All</span>
        </button>
      </div>
    </div>

    <div
      v-if="errorMessage"
      class="p-4 bg-rose-500/10 border border-rose-500/20 rounded-xl text-rose-300 text-sm"
    >
      {{ errorMessage }}
    </div>

    <div
      v-if="successMessage"
      class="p-4 bg-emerald-500/10 border border-emerald-500/20 rounded-xl text-emerald-300 text-sm"
    >
      {{ successMessage }}
    </div>

    <div class="flex flex-col sm:flex-row gap-3">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Search by user, subject, or section..."
        class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-sm text-white placeholder-slate-600 outline-none focus:border-white/20 transition"
      />

      <input
        v-model="searchDate"
        type="date"
        class="w-36 px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 cursor-pointer"
      />

      <select
        v-model="statusFilter"
        class="px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 cursor-pointer appearance-none"
      >
        <option value="all">All Status</option>
        <option value="present">Present</option>
        <option value="absent">Absent</option>
        <option value="late">Late</option>
        <option value="excused">Excused</option>
      </select>
    </div>

    <div class="border border-white/5 rounded-xl overflow-hidden min-h-[100px]">
      <table class="w-full text-left text-sm table-fixed">
        <thead class="bg-white/5 text-slate-400 uppercase text-[10px] tracking-wider">
          <tr>
            <th class="p-4 font-medium w-1/4">User</th>
            <th class="p-4 font-medium w-1/5">Subject</th>
            <th class="p-4 font-medium w-1/5">Section / Lecturer</th>
            <th class="p-4 font-medium w-1/4">Schedule</th>
            <th class="p-4 font-medium w-1/4">Status</th>
            <th class="p-4 font-medium w-20">Toggle</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-if="isLoading">
            <td colspan="6" class="p-10 text-center text-slate-500">Loading records...</td>
          </tr>
          <template v-else-if="paginatedAttendances.length > 0">
            <tr v-for="item in paginatedAttendances" :key="item.id" class="text-slate-300">
              <td class="p-4 truncate">
                <div class="font-medium">{{ item.user_name }}</div>
              </td>
              <td class="p-4 truncate">{{ item.subject_code }}</td>
              <td class="p-4 truncate">
                <div>{{ item.section_code }}</div>
                <div class="text-[10px] text-slate-500">{{ item.lecturer_name }}</div>
              </td>
              <td class="p-4 text-xs text-slate-500">
                <div>{{ item.day_of_week }}</div>
                <div>{{ item.date }}</div>
                <div class="text-[10px]">
                  {{ formatTime(item.time_slot.split(' - ')[0]) }} -
                  {{ formatTime(item.time_slot.split(' - ')[1]) }}
                </div>
              </td>
              <td class="p-4">
                <div class="flex flex-col gap-1.5">
                  <span
                    class="text-[10px] uppercase font-bold px-2 py-0.5 rounded self-start"
                    :class="statusClass(item.status)"
                  >
                    {{ item.status }}
                  </span>

                  <div v-if="item.remark" class="text-[10px] text-slate-500 italic leading-tight">
                    <span class="block line-clamp-2" :title="item.remark">
                      {{ item.remark }}
                    </span>
                  </div>
                </div>
              </td>
              <td class="p-4">
                <button
                  @click="handleToggleStatus(item)"
                  :class="[
                    'w-8 h-4 rounded-full transition-colors relative',
                    item.status === 'present'
                      ? 'bg-emerald-500'
                      : item.status === 'absent'
                        ? 'bg-rose-500'
                        : item.status === 'late'
                          ? 'bg-amber-500'
                          : 'bg-blue-500',
                  ]"
                >
                  <span
                    class="absolute top-1 left-1 w-2 h-2 rounded-full bg-white transition-all"
                  ></span>
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
    <transition
      enter-active-class="transition ease-out duration-200"
      leave-active-class="transition ease-in duration-150"
    >
      <div
        v-if="toast.show"
        class="fixed bottom-6 right-6 px-6 py-4 rounded-xl border shadow-2xl flex flex-col gap-4 z-50 min-w-[320px] bg-slate-900 border-amber-500/50"
      >
        <span class="text-sm font-medium text-white">{{ toast.message }}</span>

        <input
          v-if="toast.showRemarkInput"
          v-model="remarkInput"
          type="text"
          placeholder="Enter remark here..."
          class="px-3 py-2 bg-slate-800 border border-white/10 rounded text-xs text-white outline-none focus:border-white/20"
        />

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

const attendances = ref([])
const isLoading = ref(false)
const errorMessage = ref(null)
const successMessage = ref(null)
const searchQuery = ref('')
const statusFilter = ref('all')
const currentPage = ref(1)
const itemsPerPage = 5
const remarkInput = ref('')
const isRefreshing = ref(false)
const refreshStartDate = ref('')
const searchDate = ref('')

const toast = ref({
  show: false,
  message: '',
  type: 'success',
  isConfirm: false,
  onConfirmCallback: null,
  showRemarkInput: false,
})

const statusClass = (status) => ({
  'bg-emerald-500/20 text-emerald-400': status === 'present',
  'bg-rose-500/20 text-rose-400': status === 'absent',
  'bg-amber-500/20 text-amber-400': status === 'late',
  'bg-blue-500/20 text-blue-400': status === 'excused',
})

const triggerGlobalRefresh = async () => {
  isRefreshing.value = true
  errorMessage.value = null
  successMessage.value = null

  try {
    const res = await api.post('/attendances/refresh', {
      start_date: refreshStartDate.value,
    })

    if (res.data?.status === 'success' || res.status === 200) {
      const autoFilledCount = res.data?.data?.total_no_shows_auto_filled ?? 0
      successMessage.value = `${res.data.message}. Total absences auto-filled: ${autoFilledCount}`
    }

    await fetchAttendances()
    refreshStartDate.value = ''
  } catch (e) {
    errorMessage.value = e.response?.data?.message || 'Failed to auto-fill attendances.'
  } finally {
    isRefreshing.value = false
  }
}

const filteredAttendances = computed(() => {
  const filtered = attendances.value.filter((a) => {
    const matchesSearch =
      a.user_name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      a.subject_code.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      a.section_code.toLowerCase().includes(searchQuery.value.toLowerCase())

    const matchesStatus = statusFilter.value === 'all' || a.status === statusFilter.value

    const matchesDate = !searchDate.value || a.date === searchDate.value

    return matchesSearch && matchesStatus && matchesDate
  })

  return filtered.sort((a, b) => new Date(b.date) - new Date(a.date))
})

const lastPage = computed(() => Math.ceil(filteredAttendances.value.length / itemsPerPage) || 1)
const paginatedAttendances = computed(() =>
  filteredAttendances.value.slice(
    (currentPage.value - 1) * itemsPerPage,
    currentPage.value * itemsPerPage,
  ),
)

watch([searchQuery, statusFilter, searchDate], () => (currentPage.value = 1))

const fetchAttendances = async () => {
  isLoading.value = true
  try {
    const response = await api.get('/attendances')
    attendances.value = response.data?.data?.attendances || []
  } catch (e) {
    errorMessage.value = 'Failed to load attendances.'
  } finally {
    isLoading.value = false
  }
}

const changePage = (p) => {
  if (p >= 1 && p <= lastPage.value) currentPage.value = p
}

const handleToggleStatus = async (item) => {
  const statusCycle = {
    absent: 'present',
    present: 'late',
    late: 'excused',
    excused: 'absent',
  }
  const nextStatus = statusCycle[item.status] || 'present'

  if (nextStatus === 'excused') {
    toast.value = {
      show: true,
      message: 'Please provide a reason (remark) for marking as EXCUSED',
      type: 'warning',
      isConfirm: true,
      showRemarkInput: true,
      onConfirmCallback: async () => {
        await performToggle(item, remarkInput.value)
        remarkInput.value = ''
        toast.value.show = false
      },
    }
  } else {
    await performToggle(item)
  }
}

const performToggle = async (item, remark = null) => {
  try {
    const res = await api.patch(`/attendances/${item.id}/toggle`, { remark })
    const updated = res.data?.data?.attendance
    if (updated) {
      item.status = updated.status
      item.remark = updated.remark
    }
  } catch (e) {
    errorMessage.value = 'Failed to update status.'
  }
}

const handleConfirm = () => {
  if (toast.value.onConfirmCallback) toast.value.onConfirmCallback()
  toast.value.show = false
}

const formatTime = (timeString) => {
  if (!timeString) return 'N/A'
  const [hour, minute] = timeString.split(':')
  let h = parseInt(hour, 10)
  const ampm = h >= 12 ? 'PM' : 'AM'
  return `${h}:${minute} ${ampm}`
}

onMounted(fetchAttendances)
</script>
