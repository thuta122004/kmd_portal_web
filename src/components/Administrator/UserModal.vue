<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-sm animate-fade-in"
  >
    <div class="w-full max-w-md p-8 bg-slate-900 border border-white/5 rounded-2xl shadow-2xl">
      <div class="flex items-center justify-between mb-6">
        <h3 class="text-lg font-semibold text-white">
          {{ isEdit ? 'Edit User' : 'Register User' }}
        </h3>
        <button @click="$emit('close')" class="text-slate-500 hover:text-white transition">
          ✕
        </button>
      </div>

      <div
        v-if="errorMessage"
        class="mb-6 p-3 bg-rose-500/10 border border-rose-500/20 rounded-lg text-rose-300 text-xs"
      >
        {{ errorMessage }}
      </div>

      <form @submit.prevent="handleSubmit" class="space-y-4">
        <div>
          <input
            type="text"
            v-model="form.name"
            placeholder="Full Name"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition"
          />
          <p v-if="errors.name" class="text-rose-500 text-[10px] mt-1">{{ errors.name[0] }}</p>
        </div>

        <div>
          <input
            type="email"
            v-model="form.email"
            placeholder="Email Address"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition"
          />
          <p v-if="errors.email" class="text-rose-500 text-[10px] mt-1">{{ errors.email[0] }}</p>
        </div>

        <div>
          <select
            v-model="form.role_id"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 transition appearance-none"
          >
            <option value="" disabled>Select Role Access</option>
            <option value="1">Administrator</option>
            <option value="2">Student</option>
            <option value="3">Guardian</option>
            <option value="4">Lecturer</option>
            <option value="5">Program Manager</option>
            <option value="6">Principal</option>
          </select>
          <p v-if="errors.role_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.role_id[0] }}
          </p>
        </div>

        <template v-if="!isEdit">
          <div>
            <input
              type="password"
              v-model="form.password"
              placeholder="Password"
              class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition"
            />
            <p v-if="errors.password" class="text-rose-500 text-[10px] mt-1">
              {{ errors.password[0] }}
            </p>
          </div>
          <input
            type="password"
            v-model="form.password_confirmation"
            placeholder="Confirm Password"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition"
          />
        </template>

        <div class="flex gap-3 pt-2">
          <button
            type="button"
            @click="$emit('close')"
            class="flex-1 py-2.5 text-sm font-medium text-slate-400 hover:text-white transition"
          >
            Cancel
          </button>
          <button
            type="submit"
            :disabled="isSaving"
            class="flex-1 py-2.5 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition disabled:opacity-50"
          >
            {{ isSaving ? 'Saving...' : isEdit ? 'Edit' : 'Save' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import api from '@/services/api'

const props = defineProps({
  userToEdit: { type: Object, default: null },
})

const emit = defineEmits(['close', 'refresh'])

const isEdit = computed(() => !!props.userToEdit)
const isSaving = ref(false)
const errorMessage = ref(null)
const errors = ref({})

const form = reactive({
  name: props.userToEdit?.name || '',
  email: props.userToEdit?.email || '',
  role_id: props.userToEdit?.role_id || '',
  password: '',
  password_confirmation: '',
})

const handleSubmit = async () => {
  isSaving.value = true
  errorMessage.value = null
  errors.value = {}

  try {
    if (isEdit.value) {
      await api.put(`/users/${props.userToEdit.id}`, {
        name: form.name,
        email: form.email,
        role_id: form.role_id,
      })
    } else {
      await api.post('/users', form)
    }
    emit('refresh')
    emit('close')
  } catch (error) {
    if (error.response && error.response.status === 422) {
      errors.value = error.response.data.errors || {}
      errorMessage.value = error.response.data.message
    } else {
      errorMessage.value = 'Database transaction runtime mismatch exception.'
    }
  } finally {
    isSaving.value = false
  }
}
</script>
