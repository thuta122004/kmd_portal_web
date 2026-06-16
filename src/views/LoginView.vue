<template>
  <div
    class="relative z-10 w-full max-w-md p-8 md:p-12 bg-slate-900/40 backdrop-blur-xl border border-white/10 rounded-3xl shadow-2xl transition-all duration-500"
  >
    <div class="mb-10 text-center">
      <h2 class="text-white text-2xl font-bold tracking-tight">KMD Portal</h2>
    </div>

    <div
      v-if="feedback"
      :class="[
        'p-4 rounded-xl text-sm mb-6 border font-medium',
        feedback.status === 'error'
          ? 'bg-rose-500/10 border-rose-500/20 text-rose-300'
          : 'bg-emerald-500/10 border-emerald-500/20 text-emerald-300',
      ]"
    >
      {{ feedback.message }}
    </div>

    <form @submit.prevent="handleLogin" novalidate class="space-y-5">
      <div class="flex flex-col">
        <input
          type="email"
          id="email"
          v-model="form.email"
          placeholder="Email address"
          :disabled="isLoading"
          :class="[
            'px-4 py-3.5 bg-slate-950/40 border rounded-xl text-white outline-none transition-all placeholder:text-slate-600',
            errors.email ? 'border-rose-500/50' : 'border-white/10 focus:border-white/20',
          ]"
        />
        <span v-if="errors.email" class="text-rose-500 text-xs mt-2 font-medium">{{
          errors.email[0]
        }}</span>
      </div>

      <div class="flex flex-col">
        <input
          type="password"
          id="password"
          v-model="form.password"
          placeholder="Password"
          :disabled="isLoading"
          :class="[
            'px-4 py-3.5 bg-slate-950/40 border rounded-xl text-white outline-none transition-all placeholder:text-slate-600',
            errors.password ? 'border-rose-500/50' : 'border-white/10 focus:border-white/20',
          ]"
        />
        <span v-if="errors.password" class="text-rose-500 text-xs mt-2 font-medium">{{
          errors.password[0]
        }}</span>
      </div>

      <button
        type="submit"
        :disabled="isLoading"
        class="w-full py-3.5 mt-2 bg-white hover:bg-slate-200 text-slate-950 font-semibold rounded-xl transition-all duration-300 disabled:opacity-50 disabled:cursor-not-allowed"
      >
        <span v-if="isLoading" class="flex items-center justify-center">
          <span
            class="w-5 h-5 border-2 border-slate-900/20 border-t-slate-900 rounded-full animate-spin"
          ></span>
        </span>
        <span v-else>Sign In</span>
      </button>
    </form>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import { useRouter } from 'vue-router'
import api from '@/services/api'

const router = useRouter()

const form = reactive({ email: '', password: '' })
const isLoading = ref(false)
const errors = ref({})
const feedback = ref(null)

const handleLogin = async () => {
  isLoading.value = true
  errors.value = {}
  feedback.value = null

  try {
    const response = await api.post('/login', form)
    if (response.data.status === 'success') {
      const { token, user } = response.data.data

      localStorage.setItem('token', token)
      localStorage.setItem('user', JSON.stringify(user))

      feedback.value = { status: 'success', message: response.data.message }
      form.password = ''

      router.push({ name: 'dashboard' })
    }
  } catch (error) {
    if (error.response) {
      const data = error.response.data
      if (error.response.status === 422 && data.errors) {
        errors.value = data.errors
        feedback.value = { status: 'error', message: data.message }
      } else {
        feedback.value = { status: 'error', message: data.message || 'An error occurred.' }
      }
    } else {
      feedback.value = { status: 'error', message: 'Unable to connect to Laravel backend.' }
    }
  } finally {
    isLoading.value = false
  }
}
</script>
