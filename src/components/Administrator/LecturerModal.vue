<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-sm animate-fade-in"
  >
    <div class="w-full max-w-md p-8 bg-slate-900 border border-white/5 rounded-2xl shadow-2xl">
      <div class="flex items-center justify-between mb-6">
        <h3 class="text-lg font-semibold text-white">
          {{ isEdit ? 'Edit Lecturer' : 'Create Lecturer' }}
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
        <div v-if="!isEdit">
          <select
            v-model="form.user_id"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 transition appearance-none"
          >
            <option value="" disabled>Select User Account</option>
            <option v-for="user in availableUsers" :key="user.id" :value="user.id">
              {{ user.name }} ({{ user.email }})
            </option>
          </select>
          <p v-if="errors.user_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.user_id[0] }}
          </p>
        </div>

        <div>
          <input
            type="text"
            v-model="form.employee_id"
            placeholder="Employee ID"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition"
          />
          <p v-if="errors.employee_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.employee_id[0] }}
          </p>
        </div>

        <div>
          <input
            type="text"
            v-model="form.department"
            placeholder="Department"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition"
          />
        </div>

        <div>
          <input
            type="text"
            v-model="form.qualification"
            placeholder="Qualification"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition"
          />
        </div>

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
            {{ isSaving ? 'Saving...' : 'Save Changes' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import api from '@/services/api'

const props = defineProps({ lecturerToEdit: Object })
const emit = defineEmits(['close', 'refresh'])

const isEdit = computed(() => !!props.lecturerToEdit)
const isSaving = ref(false)
const errorMessage = ref(null)
const errors = ref({})
const availableUsers = ref([])

const form = reactive({
  user_id: props.lecturerToEdit?.user_id || '',
  employee_id: props.lecturerToEdit?.employee_id || '',
  department: props.lecturerToEdit?.department || '',
  qualification: props.lecturerToEdit?.qualification || '',
})

const fetchAvailableUsers = async () => {
  try {
    const [usersRes, lecturersRes] = await Promise.all([api.get('/users'), api.get('/lecturers')])

    const allUsers = usersRes.data?.data?.users || usersRes.data || []
    const existingLecturers = lecturersRes.data?.data?.lecturers || []

    const existingLecturerIds = new Set(existingLecturers.map((l) => Number(l.user_id)))

    availableUsers.value = allUsers.filter(
      (u) =>
        String(u.role_id) === '4' &&
        u.status === 'active' &&
        !existingLecturerIds.has(Number(u.id)),
    )
  } catch (error) {
    errorMessage.value = 'Could not load user list.'
  }
}

onMounted(() => {
  if (!isEdit.value) fetchAvailableUsers()
})

const handleSubmit = async () => {
  isSaving.value = true
  errorMessage.value = null
  errors.value = {}

  try {
    if (isEdit.value) {
      await api.put(`/lecturers/${props.lecturerToEdit.id}`, form)
    } else {
      await api.post('/lecturers', form)
    }
    emit('refresh')
    emit('close')
  } catch (error) {
    if (error.response?.status === 422) {
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
