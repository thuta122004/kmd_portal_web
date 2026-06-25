<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-sm animate-fade-in"
  >
    <div class="w-full max-w-md p-8 bg-slate-900 border border-white/5 rounded-2xl shadow-2xl">
      <h3 class="text-lg font-semibold text-white mb-6">
        {{ isEdit ? 'Edit Assignment' : 'Assign Lecturer' }}
      </h3>
      <form @submit.prevent="handleSubmit" class="space-y-4">
        <div>
          <select
            v-model="form.section_id"
            :disabled="isEdit"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition appearance-none disabled:opacity-50"
          >
            <option value="" disabled class="text-slate-500">Select Section</option>
            <option v-for="s in sections" :key="s.id" :value="s.id">{{ s.name }}</option>
          </select>
          <p v-if="errors.section_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.section_id[0] }}
          </p>
        </div>

        <div>
          <select
            v-model="form.subject_id"
            :disabled="isEdit"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition appearance-none disabled:opacity-50"
          >
            <option value="" disabled class="text-slate-500">Select Subject</option>
            <option v-for="s in subjects" :key="s.id" :value="s.id">{{ s.name }}</option>
          </select>
          <p v-if="errors.subject_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.subject_id[0] }}
          </p>
        </div>

        <div>
          <select
            v-model="form.lecturer_id"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition appearance-none"
          >
            <option value="" disabled class="text-slate-500">Select Lecturer</option>
            <option v-for="l in lecturers" :key="l.id" :value="l.id">{{ l.name }}</option>
          </select>
          <p v-if="errors.lecturer_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.lecturer_id[0] }}
          </p>
        </div>

        <div>
          <label
            class="flex items-center gap-3 w-full px-4 py-3 bg-slate-950/50 border border-white/5 rounded-lg cursor-pointer hover:border-white/10 transition-colors"
          >
            <input
              type="checkbox"
              v-model="form.is_primary"
              class="w-4 h-4 rounded border-white/10 bg-slate-900 text-blue-500 focus:ring-0"
            />
            <span class="text-sm text-slate-400">Primary Lecturer</span>
          </label>
          <p v-if="errors.is_primary" class="text-rose-500 text-[10px] mt-1">
            {{ errors.is_primary[0] }}
          </p>
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
            {{ isSaving ? 'Saving...' : isEdit ? 'Edit' : 'Save' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, computed } from 'vue'
import api from '@/services/api'

const props = defineProps({ assignmentToEdit: Object })
const emit = defineEmits(['close', 'refresh'])

const isEdit = computed(() => !!props.assignmentToEdit)
const isSaving = ref(false)
const errors = ref({})
const sections = ref([])
const subjects = ref([])
const lecturers = ref([])

const form = reactive({
  section_id: props.assignmentToEdit?.section_id || '',
  subject_id: props.assignmentToEdit?.subject_id || '',
  lecturer_id: props.assignmentToEdit?.lecturer_id || '',
  is_primary: !!props.assignmentToEdit?.is_primary,
})

const fetchDropdownData = async () => {
  try {
    const [sRes, subRes, lRes] = await Promise.all([
      api.get('/sections?status=active'),
      api.get('/subjects?status=active'),
      api.get('/lecturers?status=active'),
    ])
    sections.value = sRes.data?.data?.sections || sRes.data || []
    subjects.value = subRes.data?.data?.subjects || subRes.data || []
    lecturers.value = lRes.data?.data?.lecturers || lRes.data || []
  } catch (error) {
    console.error(error)
  }
}

const fetchLecturersOnly = async () => {
  try {
    const lRes = await api.get('/lecturers?status=active')
    const activeLecturers = lRes.data?.data?.lecturers || lRes.data || []

    const currentLecturerExists = activeLecturers.some(
      (l) => l.id === props.assignmentToEdit.lecturer_id,
    )
    if (!currentLecturerExists && props.assignmentToEdit) {
      activeLecturers.unshift({
        id: props.assignmentToEdit.lecturer_id,
        name: props.assignmentToEdit.lecturer_name + ' (Inactive)',
      })
    }

    lecturers.value = activeLecturers
  } catch (error) {
    console.error(error)
  }
}

onMounted(() => {
  if (!isEdit.value) {
    fetchDropdownData()
  } else {
    sections.value = [
      {
        id: props.assignmentToEdit.section_id,
        name: props.assignmentToEdit.section_name,
      },
    ]
    subjects.value = [
      {
        id: props.assignmentToEdit.subject_id,
        name: props.assignmentToEdit.subject_name,
      },
    ]
    lecturers.value = [
      {
        id: props.assignmentToEdit.lecturer_id,
        name: props.assignmentToEdit.lecturer_name,
      },
    ]

    fetchLecturersOnly()
  }
})

const handleSubmit = async () => {
  isSaving.value = true
  errors.value = {}
  try {
    if (isEdit.value) {
      await api.put(`/section-assignments/${props.assignmentToEdit.id}`, form)
    } else {
      await api.post('/section-assignments', form)
    }
    emit('refresh')
    emit('close')
  } catch (e) {
    if (e.response?.data?.errors) {
      errors.value = e.response.data.errors
    }
  } finally {
    isSaving.value = false
  }
}
</script>
