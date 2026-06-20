<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-sm animate-fade-in"
  >
    <div class="w-full max-w-md p-8 bg-slate-900 border border-white/5 rounded-2xl shadow-2xl">
      <h3 class="text-lg font-semibold text-white mb-6">
        {{ isEdit ? 'Edit Subject' : 'Create Subject' }}
      </h3>

      <form @submit.prevent="handleSubmit" class="space-y-4">
        <div>
          <input
            type="text"
            v-model="form.name"
            placeholder="Subject Name"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
          />
          <p v-if="errors.name" class="text-rose-500 text-[10px] mt-1">{{ errors.name[0] }}</p>
        </div>

        <div>
          <input
            type="text"
            v-model="form.code"
            placeholder="Code"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm uppercase"
          />
          <p v-if="errors.code" class="text-rose-500 text-[10px] mt-1">{{ errors.code[0] }}</p>
        </div>

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
import { ref, reactive, computed } from 'vue'
import api from '@/services/api'

const props = defineProps({ subjectToEdit: Object })
const emit = defineEmits(['close', 'refresh'])

const isEdit = computed(() => !!props.subjectToEdit)
const isSaving = ref(false)
const errors = ref({})

const form = reactive({
  name: props.subjectToEdit?.name || '',
  code: props.subjectToEdit?.code || '',
})

const handleSubmit = async () => {
  isSaving.value = true
  errors.value = {}
  try {
    if (isEdit.value) await api.put(`/subjects/${props.subjectToEdit.id}`, form)
    else await api.post('/subjects', form)
    emit('refresh')
    emit('close')
  } catch (e) {
    errors.value = e.response?.data?.errors || {}
  } finally {
    isSaving.value = false
  }
}
</script>
