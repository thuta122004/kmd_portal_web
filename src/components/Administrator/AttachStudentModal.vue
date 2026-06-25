<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-sm animate-fade-in"
  >
    <div class="w-full max-w-md p-8 bg-slate-900 border border-white/5 rounded-2xl shadow-2xl">
      <h3 class="text-lg font-semibold text-white mb-6">Attach Student</h3>

      <div
        v-if="errorMessage"
        class="mb-6 p-3 bg-rose-500/10 border border-rose-500/20 rounded-lg text-rose-300 text-xs"
      >
        {{ errorMessage }}
      </div>

      <form @submit.prevent="handleSubmit" class="space-y-4">
        <div>
          <select
            v-model="form.guardian_id"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition appearance-none"
          >
            <option value="" disabled class="text-slate-500">Select Guardian</option>
            <option v-for="g in guardians" :key="g.id" :value="g.id">
              {{ g.name }} ({{ g.email }})
            </option>
          </select>
          <p v-if="errors.guardian_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.guardian_id[0] }}
          </p>
        </div>

        <div>
          <select
            v-model="form.student_id"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition appearance-none"
          >
            <option value="" disabled class="text-slate-500">Select Student</option>
            <option v-for="s in students" :key="s.id" :value="s.id">
              {{ s.name }} ({{ s.email }})
            </option>
          </select>
          <p v-if="errors.student_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.student_id[0] }}
          </p>
        </div>

        <div>
          <input
            v-model="form.relationship_type"
            type="text"
            placeholder="Relationship"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition"
          />
          <p v-if="errors.relationship_type" class="text-rose-500 text-[10px] mt-1">
            {{ errors.relationship_type[0] }}
          </p>
        </div>

        <div>
          <label
            class="flex items-center gap-3 w-full px-4 py-3 bg-slate-950/50 border border-white/5 rounded-lg cursor-pointer hover:border-white/10 transition-colors"
          >
            <input
              type="checkbox"
              v-model="form.is_primary_contact"
              class="w-4 h-4 rounded border-white/10 bg-slate-900 text-blue-500 focus:ring-0"
            />
            <span class="text-sm text-slate-400">Set as Primary Contact</span>
          </label>
        </div>

        <div class="flex gap-3 pt-2">
          <button
            type="button"
            @click="$emit('close')"
            class="flex-1 py-2.5 text-slate-400 hover:text-white text-sm"
          >
            Cancel
          </button>
          <button
            type="submit"
            :disabled="isSaving"
            class="flex-1 py-2.5 bg-white text-slate-950 text-sm font-semibold rounded-lg"
          >
            {{ isSaving ? 'Saving...' : 'Save' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import api from '@/services/api'

const emit = defineEmits(['close', 'refresh'])

const isSaving = ref(false)
const errors = ref({})
const errorMessage = ref(null)
const guardians = ref([])
const students = ref([])

const form = reactive({
  guardian_id: '',
  student_id: '',
  relationship_type: '',
  is_primary_contact: false,
})

onMounted(async () => {
  try {
    const [gRes, sRes] = await Promise.all([api.get('/guardians'), api.get('/students')])
    guardians.value = gRes.data?.data?.guardians || []

    const allStudents = sRes.data?.data?.students || []
    students.value = allStudents.filter((s) => s.status === 'active')
  } catch (error) {
    errorMessage.value = 'Failed to load students.'
    console.error(error)
  }
})
const handleSubmit = async () => {
  if (!form.guardian_id) {
    errors.value = { guardian_id: ['Please select a guardian.'] }
    return
  }

  isSaving.value = true
  errors.value = {}

  try {
    await api.post(`/guardians/${form.guardian_id}/attach-student`, {
      student_id: form.student_id,
      relationship_type: form.relationship_type,
      is_primary_contact: form.is_primary_contact,
    })

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
