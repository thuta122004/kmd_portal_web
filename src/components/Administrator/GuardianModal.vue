<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-sm animate-fade-in"
  >
    <div class="w-full max-w-md p-8 bg-slate-900 border border-white/5 rounded-2xl shadow-2xl">
      <h3 class="text-lg font-semibold text-white mb-6">
        {{ isEdit ? 'Edit Guardian' : 'Create Guardian' }}
      </h3>

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
            v-model="form.phone"
            placeholder="Phone Number"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
          />
          <p v-if="errors.phone" class="text-rose-500 text-[10px] mt-1">{{ errors.phone[0] }}</p>
        </div>

        <input
          type="text"
          v-model="form.occupation"
          placeholder="Occupation"
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
            {{ isSaving ? 'Saving...' : isEdit ? 'Edit' : 'Save' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import api from '@/services/api'

const props = defineProps({ guardianToEdit: Object })
const emit = defineEmits(['close', 'refresh'])

const isEdit = computed(() => !!props.guardianToEdit)
const isSaving = ref(false)
const availableUsers = ref([])
const errors = ref({})
const errorMessage = ref(null)

const form = reactive({
  user_id: props.guardianToEdit?.user_id || '',
  phone: props.guardianToEdit?.phone || '',
  occupation: props.guardianToEdit?.occupation || '',
  address: props.guardianToEdit?.address || '',
})

const fetchAvailableUsers = async () => {
  try {
    const [usersRes, guardiansRes] = await Promise.all([api.get('/users'), api.get('/guardians')])
    const allUsers = usersRes.data?.data?.users || usersRes.data || []
    const existingGuardians = guardiansRes.data?.data?.guardians || []
    const usedIds = new Set(existingGuardians.map((g) => Number(g.user_id)))

    availableUsers.value = allUsers.filter(
      (u) => String(u.role_id) === '3' && !usedIds.has(Number(u.id)),
    )
  } catch (error) {
    console.error(error)
  }
}

onMounted(() => {
  if (!isEdit.value) fetchAvailableUsers()
})

const handleSubmit = async () => {
  isSaving.value = true
  try {
    if (isEdit.value) await api.put(`/guardians/${props.guardianToEdit.id}`, form)
    else await api.post('/guardians', form)
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
