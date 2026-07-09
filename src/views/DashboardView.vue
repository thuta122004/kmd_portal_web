<template>
  <div class="relative z-10 w-full min-h-screen flex flex-col animate-fade-in">
    <header
      class="flex items-center justify-between w-full px-8 py-5 border-b border-white/5 bg-slate-950/20 backdrop-blur-md"
    >
      <div class="flex items-center gap-6">
        <h1 class="text-lg font-bold text-white tracking-tight">KMD Portal</h1>
        <div class="h-4 w-[1px] bg-white/10"></div>
        <div class="text-sm text-slate-400">
          Welcome,
          <span class="text-white font-medium"
            >{{ userData?.name }} ({{ userData?.role_name }})</span
          >
        </div>
      </div>

      <button
        @click="handleLogout"
        :disabled="isLoggingOut"
        class="text-xs font-medium text-rose-400 hover:text-rose-300 transition-colors px-3 py-1.5 rounded-lg hover:bg-rose-950/20"
      >
        {{ isLoggingOut ? 'Signing out...' : 'Sign out' }}
      </button>
    </header>

    <main class="flex-1 w-full p-8 overflow-hidden">
      <component :is="resolvedDashboard" />
    </main>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue'
import { useRouter } from 'vue-router'
import api from '@/services/api'

import AdminDashboard from '@/components/Administrator/AdminDashboard.vue'
import StudentDashboard from '@/components/Student/StudentDashboard.vue'
import GuardianDashboard from '@/components/Guardian/GuardianDashboard.vue'
import LecturerDashboard from '@/components/Lecturer/LecturerDashboard.vue'
import ProgramManagerDashboard from '@/components/ProgramManager/ProgramManagerDashboard.vue'
import PrincipalDashboard from '@/components/Principal/PrincipalDashboard.vue'
import PlaceholderDashboard from '@/components/Shared/PlaceholderDashboard.vue'

const router = useRouter()
const isLoggingOut = ref(false)
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
