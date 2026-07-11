<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-sm animate-fade-in"
  >
    <div class="w-full max-w-md p-8 bg-slate-900 border border-white/5 rounded-2xl shadow-2xl">
      <div class="flex items-center justify-between mb-6">
        <h3 class="text-lg font-semibold text-white">
          {{ isEdit ? 'Edit Document' : 'Upload Document' }}
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
            v-model="form.student_id"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 transition appearance-none"
          >
            <option value="" disabled>Select Student</option>
            <option v-for="stu in students" :key="stu.id" :value="stu.id">
              {{ stu.name }} ({{ stu.email }})
            </option>
          </select>
          <p v-if="errors.student_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.student_id[0] }}
          </p>
        </div>

        <div>
          <input
            type="text"
            v-model="form.title"
            placeholder="Document Title"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none"
          />
          <p v-if="errors.title" class="text-rose-500 text-[10px] mt-1">{{ errors.title[0] }}</p>
        </div>

        <div v-if="!isEdit">
          <input
            type="text"
            v-model="form.document_type"
            placeholder="Document Type"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none"
          />
          <p v-if="errors.document_type" class="text-rose-500 text-[10px] mt-1">
            {{ errors.document_type[0] }}
          </p>
        </div>

        <div>
          <label class="block text-xs text-slate-500 mb-1">Upload File (PDF, JPG, PNG)</label>
          <input
            type="file"
            @change="handleFileChange"
            class="w-full text-sm text-slate-400 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:bg-white/10 file:text-white"
          />
          <p v-if="errors.file" class="text-rose-500 text-[10px] mt-1">{{ errors.file[0] }}</p>
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
            class="flex-1 py-2.5 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
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

const props = defineProps({ docToEdit: Object })
const emit = defineEmits(['close', 'refresh'])

const isEdit = computed(() => !!props.docToEdit)
const isSaving = ref(false)
const errorMessage = ref(null)
const errors = ref({})
const file = ref(null)
const students = ref([])

const form = reactive({
  student_id: props.docToEdit?.student_id || '',
  document_type: props.docToEdit?.document_type || '',
  title: props.docToEdit?.title || '',
})

const fetchStudents = async () => {
  try {
    const res = await api.get('/students')
    students.value = res.data?.data?.students || res.data || []
  } catch (e) {
    console.error(e)
  }
}

onMounted(() => {
  if (!isEdit.value) {
    fetchStudents()
  }
})

const handleFileChange = (e) => {
  file.value = e.target.files[0]
}

const handleSubmit = async () => {
  isSaving.value = true
  errorMessage.value = null
  errors.value = {}

  const formData = new FormData()

  if (isEdit.value) {
    formData.append('_method', 'PUT')
  }

  formData.append('title', form.title)
  if (file.value) formData.append('file', file.value)

  try {
    if (isEdit.value) {
      await api.post(`/academic-documents/${props.docToEdit.id}`, formData, {
        headers: { 'Content-Type': 'multipart/form-data' },
      })
    } else {
      formData.append('student_id', form.student_id)
      formData.append('document_type', form.document_type)
      await api.post('/academic-documents', formData, {
        headers: { 'Content-Type': 'multipart/form-data' },
      })
    }
    emit('refresh')
    emit('close')
  } catch (e) {
    if (e.response?.status === 422) {
      errors.value = e.response.data.errors || {}
      errorMessage.value = e.response.data.message || 'Validation failed.'
    } else {
      errorMessage.value = 'An unexpected error occurred.'
    }
  } finally {
    isSaving.value = false
  }
}
</script>
