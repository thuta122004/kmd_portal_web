<template>
  <div class="relative z-10 w-full min-h-screen flex flex-col animate-fade-in">
    <header
      class="flex items-center justify-between w-full px-8 py-5 border-b border-white/5 bg-slate-950/20 backdrop-blur-md"
    >
      <div class="flex items-center gap-6">
        <h1
          @click="showNotifications = false"
          class="text-lg font-bold text-white tracking-tight cursor-pointer hover:opacity-80 transition-opacity"
        >
          KMD Portal
        </h1>
        <div class="h-4 w-[1px] bg-white/10"></div>
        <div class="text-sm text-slate-400">
          Welcome,
          <span class="text-white font-medium"
            >{{ userData?.name }} ({{ userData?.role_name }})</span
          >
        </div>
      </div>

      <div class="flex items-center gap-4">
        <button
          @click="showNotifications = !showNotifications"
          :class="[
            'relative p-2 rounded-xl border transition-all duration-200 active:scale-95 group',
            showNotifications
              ? 'bg-emerald-500/10 text-emerald-400 border-emerald-500/30'
              : 'text-slate-400 hover:text-slate-200 hover:bg-white/[0.03] border-white/[0.05] hover:border-white/[0.12]',
          ]"
          title="Toggle Notifications"
        >
          <svg
            class="w-4 h-4 transition-transform duration-200 group-hover:rotate-12"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9"
            />
          </svg>

          <span
            v-if="unreadCount > 0"
            class="absolute -top-1 -right-1 px-1.5 py-0.5 text-[9px] font-bold bg-rose-500 text-white rounded-md min-w-[14px] h-3.5 flex items-center justify-center tracking-tight shadow-lg shadow-rose-500/20"
          >
            {{ unreadCount }}
          </span>
        </button>

        <button
          @click="handleLogout"
          :disabled="isLoggingOut"
          class="text-xs font-medium text-rose-400 hover:text-rose-300 transition-colors px-3 py-1.5 rounded-lg hover:bg-rose-950/20"
        >
          {{ isLoggingOut ? 'Signing out...' : 'Sign out' }}
        </button>
      </div>
    </header>

    <main class="flex-1 w-full p-8 overflow-hidden">
      <Notifications
        v-if="showNotifications"
        :user-id="userData?.id"
        @count-updated="updateLocalCount"
        @close="showNotifications = false"
      />
      <component v-else :is="resolvedDashboard" />
    </main>
  </div>
</template>

<script setup>
import { computed, ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import api from '@/services/api'

import AdminDashboard from '@/components/Administrator/AdminDashboard.vue'
import StudentDashboard from '@/components/Student/StudentDashboard.vue'
import GuardianDashboard from '@/components/Guardian/GuardianDashboard.vue'
import LecturerDashboard from '@/components/Lecturer/LecturerDashboard.vue'
import ProgramManagerDashboard from '@/components/ProgramManager/ProgramManagerDashboard.vue'
import PrincipalDashboard from '@/components/Principal/PrincipalDashboard.vue'
import PlaceholderDashboard from '@/components/Shared/PlaceholderDashboard.vue'
import Notifications from './Notifications.vue'

const router = useRouter()
const isLoggingOut = ref(false)
const showNotifications = ref(false)
const unreadCount = ref(0)
const userData = ref(JSON.parse(localStorage.getItem('user')) || null)

const resolvedDashboard = computed(() => {
  const roleId = Number(userData.value?.role_id)
  switch (roleId) {
    case 1:
      return AdminDashboard
    case 2:
      return StudentDashboard
    case 3:
      return GuardianDashboard
    case 4:
      return LecturerDashboard
    case 5:
      return ProgramManagerDashboard
    case 6:
      return PrincipalDashboard
    default:
      return PlaceholderDashboard
  }
})

const fetchInitialUnreadCount = async () => {
  if (!userData.value?.id) return
  try {
    const res = await api.get('/notifications/unread-count', {
      params: { user_id: userData.value.id },
    })
    unreadCount.value = res.data?.data?.unread_count || 0
  } catch (error) {
    console.error('Failed to load unread notification count:', error)
  }
}

const updateLocalCount = (newCount) => {
  unreadCount.value = newCount
}

const handleLogout = async () => {
  isLoggingOut.value = true
  try {
    await api.post('/logout')
  } catch (error) {
    console.warn('Logout processed.')
  } finally {
    localStorage.removeItem('token')
    localStorage.removeItem('user')
    isLoggingOut.value = false
    router.push({ name: 'login' })
  }
}

onMounted(fetchInitialUnreadCount)
</script>

<style scoped>
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
.animate-fade-in {
  animation: fadeIn 0.6s ease-out forwards;
}
</style>
