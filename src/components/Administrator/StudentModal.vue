<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-sm animate-fade-in"
  >
    <div class="w-full max-w-md p-8 bg-slate-900 border border-white/5 rounded-2xl shadow-2xl">
      <h3 class="text-lg font-semibold text-white mb-6">
        {{ isEdit ? 'Edit Student' : 'Create Student' }}
      </h3>

      <form @submit.prevent="handleSubmit" class="space-y-4">
        <div v-if="!isEdit">
          <select
            v-model="form.user_id"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 transition appearance-none"
          >
            <option value="" disabled>Select User Account</option>
            <option v-for="u in availableUsers" :key="u.id" :value="u.id">
              {{ u.name }} ({{ u.email }})
            </option>
          </select>
          <p v-if="errors.user_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.user_id[0] }}
          </p>
        </div>

        <div>
          <input
            type="text"
            v-model="form.student_reg_number"
            placeholder="Reg Number"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
          />
          <p v-if="errors.student_reg_number" class="text-rose-500 text-[10px] mt-1">
            {{ errors.student_reg_number[0] }}
          </p>
        </div>

        <div>
          <input
            type="date"
            v-model="form.date_of_birth"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
          />
          <p v-if="errors.date_of_birth" class="text-rose-500 text-[10px] mt-1">
            {{ errors.date_of_birth[0] }}
          </p>
        </div>

        <div>
          <select
            v-model="form.gender"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 transition appearance-none"
          >
            <option value="" disabled>Select Gender</option>
            <option value="Male">Male</option>
            <option value="Female">Female</option>
          </select>
          <p v-if="errors.gender" class="text-rose-500 text-[10px] mt-1">
            {{ errors.gender[0] }}
          </p>
        </div>

        <input
          type="text"
          v-model="form.phone"
          placeholder="Phone"
          class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
        />
        <textarea
          v-model="form.address"
          placeholder="Address"
          class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
        ></textarea>

        <div class="flex gap-3 pt-2">
          <button
            type="button"
            @click="$emit('close')"
            class="flex-1 py-2.5 text-sm text-slate-400 hover:text-white"
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
import { ref, reactive, computed, onMounted } from 'vue'
import api from '@/services/api'

const props = defineProps({ studentToEdit: Object })
const emit = defineEmits(['close', 'refresh'])

const isEdit = computed(() => !!props.studentToEdit)
const isSaving = ref(false)
const availableUsers = ref([])
const errors = ref({})

const form = reactive({
  user_id: props.studentToEdit?.user_id || '',
  student_reg_number: props.studentToEdit?.student_reg_number || '',
  date_of_birth: props.studentToEdit?.date_of_birth || '',
  gender: props.studentToEdit?.gender || '',
  phone: props.studentToEdit?.phone || '',
  address: props.studentToEdit?.address || '',
})

const fetchAvailableUsers = async () => {
  try {
    const [usersRes, studentsRes] = await Promise.all([api.get('/users'), api.get('/students')])

    const allUsers = usersRes.data?.data?.users || usersRes.data || []
    const existingStudents = studentsRes.data?.data?.students || []

    const usedIds = new Set(existingStudents.map((s) => Number(s.user_id)))

    availableUsers.value = allUsers.filter(
      (u) => String(u.role_id) === '2' && u.status === 'active' && !usedIds.has(Number(u.id)),
    )
  } catch (error) {
    console.error('Failed to load available users:', error)
  }
}

onMounted(() => {
  if (!isEdit.value) fetchAvailableUsers()
})

const handleSubmit = async () => {
  isSaving.value = true
  try {
    if (isEdit.value) await api.put(`/students/${props.studentToEdit.id}`, form)
    else await api.post('/students', form)
    emit('refresh')
    emit('close')
  } catch (e) {
    errors.value = e.response?.data?.errors || {}
  } finally {
    isSaving.value = false
  }
}
</script>
