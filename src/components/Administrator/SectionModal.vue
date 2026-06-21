<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-sm animate-fade-in"
  >
    <div class="w-full max-w-md p-8 bg-slate-900 border border-white/5 rounded-2xl shadow-2xl">
      <h3 class="text-lg font-semibold text-white mb-6">
        {{ isEdit ? 'Edit Section' : 'Create Section' }}
      </h3>

      <form @submit.prevent="handleSubmit" class="space-y-4">
        <div>
          <input
            v-model="form.name"
            placeholder="Section Name"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
          />
          <p v-if="errors.name" class="text-rose-500 text-[10px] mt-1">{{ errors.name[0] }}</p>
        </div>

        <div>
          <input
            v-model="form.code"
            placeholder="Code"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm uppercase"
          />
          <p v-if="errors.code" class="text-rose-500 text-[10px] mt-1">{{ errors.code[0] }}</p>
        </div>

        <div class="flex gap-4">
          <div class="flex-1">
            <label class="text-[10px] text-slate-500 uppercase">Start Date</label>
            <input
              type="date"
              v-model="form.start_date"
              class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
            />
            <p v-if="errors.start_date" class="text-rose-500 text-[10px] mt-1">
              {{ errors.start_date[0] }}
            </p>
          </div>
          <div class="flex-1">
            <label class="text-[10px] text-slate-500 uppercase">End Date</label>
            <input
              type="date"
              v-model="form.end_date"
              class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
            />
          </div>
        </div>

        <div class="flex gap-3 pt-4">
          <button
            type="button"
            @click="$emit('close')"
            class="flex-1 py-2.5 text-sm text-slate-400"
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

const props = defineProps({ sectionToEdit: Object })
const emit = defineEmits(['close', 'refresh'])

const isEdit = computed(() => !!props.sectionToEdit)
const isSaving = ref(false)
const errors = ref({})

const form = reactive({
  name: props.sectionToEdit?.name || '',
  code: props.sectionToEdit?.code || '',
  start_date: props.sectionToEdit?.start_date || '',
  end_date: props.sectionToEdit?.end_date || '',
})

const handleSubmit = async () => {
  isSaving.value = true
  try {
    if (isEdit.value) await api.put(`/sections/${props.sectionToEdit.id}`, form)
    else await api.post('/sections', form)
    emit('refresh')
    emit('close')
  } catch (e) {
    errors.value = e.response?.data?.errors || {}
  } finally {
    isSaving.value = false
  }
}
</script>
