<template>
  <div class="max-w-3xl mx-auto pb-16 space-y-6 animate-fade-in">
    <div class="flex items-center justify-between pb-5 border-b border-white/[0.06]">
      <div class="flex items-center gap-4">
        <button
          @click="emit('close')"
          class="p-2 text-slate-400 hover:text-white bg-white/[0.02] border border-white/[0.05] hover:border-white/[0.1] rounded-xl transition duration-200"
          title="Back to Dashboard"
        >
          <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M10 19l-7-7m0 0l7-7m-7 7h18"
            />
          </svg>
        </button>

        <div>
          <h2 class="text-xl font-semibold text-white tracking-tight">Notifications</h2>
          <p class="text-xs text-slate-400 mt-1">Stay updated with your latest academic alerts</p>
        </div>
      </div>

      <button
        v-if="hasUnreadNotifications"
        @click="markAllNotificationsAsRead"
        :disabled="isUpdating"
        class="text-xs font-medium text-slate-400 hover:text-emerald-400 transition-all duration-200 disabled:opacity-40 flex items-center gap-2 px-3 py-1.5 rounded-lg hover:bg-white/[0.02] border border-transparent hover:border-white/[0.05]"
      >
        <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M5 13l4 4L19 7"
          />
        </svg>
        Mark all as read
      </button>
    </div>

    <div
      v-if="errorMessage"
      class="p-4 bg-rose-500/[0.06] border border-rose-500/20 rounded-xl text-rose-300 text-xs flex items-center gap-3"
    >
      <svg
        class="w-4 h-4 text-rose-400 shrink-0"
        fill="none"
        viewBox="0 0 24 24"
        stroke="currentColor"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
        />
      </svg>
      <span>{{ errorMessage }}</span>
    </div>

    <div v-if="isLoading" class="space-y-4">
      <div
        v-for="n in 3"
        :key="n"
        class="bg-slate-900/40 backdrop-blur-sm border border-white/[0.04] rounded-2xl p-5 space-y-3"
      >
        <div class="flex items-center justify-between">
          <div class="flex items-center gap-3 w-1/2">
            <div class="w-8 h-8 bg-white/[0.04] rounded-xl animate-pulse"></div>
            <div class="h-4 bg-white/[0.04] rounded w-2/3 animate-pulse"></div>
          </div>
          <div class="w-16 h-3 bg-white/[0.04] rounded animate-pulse"></div>
        </div>
        <div class="w-full h-3 bg-white/[0.02] rounded animate-pulse"></div>
        <div class="w-4/5 h-3 bg-white/[0.02] rounded animate-pulse"></div>
      </div>
    </div>

    <div v-else-if="notifications.length > 0" class="space-y-3.5">
      <div
        v-for="item in notifications"
        :key="item.id"
        :class="[
          'group relative border rounded-2xl p-5 transition-all duration-300 flex items-start gap-4 shadow-sm hover:shadow-md',
          item.is_read
            ? 'bg-slate-900/30 border-white/[0.04] opacity-60 hover:opacity-90'
            : 'bg-slate-900/80 border-white/[0.08] hover:border-white/[0.15] hover:-translate-y-0.5',
        ]"
      >
        <div
          :class="[
            'p-2.5 rounded-xl border shrink-0 transition-colors duration-300',
            item.is_read
              ? 'bg-slate-950/40 border-white/[0.04] text-slate-500'
              : 'bg-emerald-500/[0.06] border-emerald-500/20 text-emerald-400 group-hover:bg-emerald-500/[0.1]',
          ]"
        >
          <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9"
            />
          </svg>
        </div>

        <div class="flex-1 min-w-0 space-y-1.5 pr-8">
          <div class="flex items-center gap-2 flex-wrap">
            <span
              v-if="!item.is_read"
              class="w-2 h-2 rounded-full bg-emerald-400 ring-4 ring-emerald-400/20 shrink-0"
              title="Unread"
            ></span>

            <h3
              :class="[
                'text-sm font-semibold truncate tracking-tight max-w-[70%]',
                item.is_read ? 'text-slate-400' : 'text-white',
              ]"
            >
              {{ item.title }}
            </h3>

            <span class="text-[10px] text-slate-500 font-medium ml-auto shrink-0">
              {{ formatDate(item.created_at) }}
            </span>
          </div>

          <p class="text-xs text-slate-400 leading-relaxed whitespace-pre-wrap">
            {{ item.content }}
          </p>
        </div>

        <button
          v-if="!item.is_read"
          @click="markSingleNotificationAsRead(item.id)"
          :disabled="isUpdating"
          title="Mark as read"
          class="opacity-0 group-hover:opacity-100 focus:opacity-100 transition-all duration-200 p-1.5 text-slate-400 hover:text-emerald-400 bg-slate-950/60 hover:bg-slate-950 border border-white/[0.05] rounded-lg shrink-0 absolute right-4 top-4"
        >
          <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M5 13l4 4L19 7"
            />
          </svg>
        </button>
      </div>
    </div>

    <div
      v-else
      class="flex flex-col items-center justify-center py-24 px-4 text-center border border-dashed border-white/[0.06] rounded-2xl bg-slate-900/10"
    >
      <div
        class="w-14 h-14 bg-slate-900/60 border border-white/[0.05] rounded-2xl flex items-center justify-center mb-4 text-slate-500 shadow-inner"
      >
        <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="1.5"
            d="M20 13V6a2 2 0 00-2-2H6a2 2 0 00-2 2v7m16 0a2 2 0 01-2 2H6a2 2 0 01-2-2m16 0v5a2 2 0 01-2 2H6a2 2 0 01-2-2v-5m16 0h-4a2 2 0 00-2 2v1a2 2 0 01-2 2H9a2 2 0 01-2-2v-1a2 2 0 00-2-2H3"
          />
        </svg>
      </div>
      <h3 class="text-sm font-medium text-slate-200">All caught up!</h3>
      <p class="text-xs text-slate-500 mt-1 max-w-xs">
        Your notification tray is completely empty.
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import api from '@/services/api'

const props = defineProps({
  userId: {
    type: [Number, String],
    default: () => {
      const userRaw = localStorage.getItem('user')
      if (userRaw) {
        try {
          const parsed = JSON.parse(userRaw)
          return parsed.id || 0
        } catch (e) {
          console.error('Failed to parse user from local storage', e)
        }
      }
      return 0
    },
  },
})

const emit = defineEmits(['count-updated', 'close'])

const notifications = ref([])
const isLoading = ref(true)
const isUpdating = ref(false)
const errorMessage = ref(null)

const hasUnreadNotifications = computed(() => {
  return notifications.value.some((item) => !item.is_read)
})

const formatDate = (dateString) => {
  if (!dateString) return ''
  const date = new Date(dateString)
  return date.toLocaleDateString('en-US', {
    month: 'short',
    day: 'numeric',
    hour: 'numeric',
    minute: '2-digit',
  })
}

const fetchNotifications = async () => {
  isLoading.value = true
  errorMessage.value = null
  try {
    const res = await api.get('/notifications', {
      params: { user_id: props.userId },
    })
    notifications.value = res.data?.data?.notifications || []
    emit('count-updated', res.data?.data?.unread_count || 0)
  } catch (error) {
    console.error(error)
    errorMessage.value = 'Unable to check your notifications inbox.'
  } finally {
    isLoading.value = false
  }
}

const markSingleNotificationAsRead = async (id) => {
  isUpdating.value = true
  try {
    const res = await api.patch('/notifications/read', { id: id })
    const index = notifications.value.findIndex((item) => item.id === id)

    if (index !== -1 && res.data?.status === 'success') {
      notifications.value[index].is_read = true
      emit('count-updated', res.data?.data?.unread_count || 0)
    }
  } catch (error) {
    console.error(error)
  } finally {
    isUpdating.value = false
  }
}

const markAllNotificationsAsRead = async () => {
  isUpdating.value = true
  try {
    const res = await api.patch('/notifications/read', { user_id: props.userId })

    if (res.data?.status === 'success') {
      notifications.value.forEach((item) => {
        item.is_read = true
      })
      emit('count-updated', 0)
    }
  } catch (error) {
    console.error(error)
    errorMessage.value = 'Failed to clear all unread items.'
  } finally {
    isUpdating.value = false
  }
}

onMounted(fetchNotifications)
</script>
