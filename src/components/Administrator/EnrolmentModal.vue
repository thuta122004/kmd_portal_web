<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-sm animate-fade-in"
  >
    <div class="w-full max-w-md p-8 bg-slate-900 border border-white/5 rounded-2xl shadow-2xl">
      <h3 class="text-lg font-semibold text-white mb-6">
        {{ isEdit ? 'Edit Enrolment' : 'Create Enrolment' }}
      </h3>

      <form @submit.prevent="handleSubmit" class="space-y-4">
        <div>
          <select
            v-model="form.student_id"
            :disabled="isEdit"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 transition appearance-none disabled:opacity-50"
          >
            <option value="" disabled>Select Student</option>
            <option v-for="s in students" :key="s.id" :value="s.id">
              {{ s.name }} ({{ s.student_reg_number }})
            </option>
          </select>
          <p v-if="errors.student_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.student_id[0] }}
          </p>
        </div>

        <div>
          <select
            v-model="form.section_id"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 transition appearance-none"
          >
            <option value="" disabled>Select Section</option>
            <option v-for="sec in sections" :key="sec.id" :value="sec.id">
              {{ sec.name }}
            </option>
          </select>
          <p v-if="errors.section_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.section_id[0] }}
          </p>
        </div>

        <div>
          <textarea
            v-model="form.note"
            placeholder="Note"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition min-h-[80px]"
          ></textarea>
          <p v-if="errors.note" class="text-rose-500 text-[10px] mt-1">
            {{ errors.note[0] }}
          </p>
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
import { ref, reactive, computed, onMounted } from 'vue'
import api from '@/services/api'

const props = defineProps({ enrolmentToEdit: Object })
const emit = defineEmits(['close', 'refresh'])

const isEdit = computed(() => !!props.enrolmentToEdit)
const isSaving = ref(false)
const students = ref([])
const sections = ref([])
const errors = ref({})

const form = reactive({
  student_id: props.enrolmentToEdit?.student_id || '',
  section_id: props.enrolmentToEdit?.section_id || '',
  note: props.enrolmentToEdit?.note || '',
})

const fetchDropdownData = async () => {
  try {
    const [studentsRes, sectionsRes] = await Promise.all([
      api.get('/students'),
      api.get('/sections'),
    ])
    students.value = studentsRes.data?.data?.students || []
    sections.value = sectionsRes.data?.data?.sections || sectionsRes.data || []
  } catch (error) {
    console.error(error)
  }
}

const fetchSectionsOnly = async () => {
  try {
    const sectionsRes = await api.get('/sections')
    const freshSections = sectionsRes.data?.data?.sections || sectionsRes.data || []

    sections.value = freshSections
  } catch (error) {
    console.error(error)
  }
}

onMounted(() => {
  if (!isEdit.value) {
    fetchDropdownData()
  } else {
    students.value = [
      {
        id: props.enrolmentToEdit.student_id,
        name: props.enrolmentToEdit.student_name,
        student_reg_number: props.enrolmentToEdit.student_reg_number,
      },
    ]

    sections.value = [
      {
        id: props.enrolmentToEdit.section_id,
        name: props.enrolmentToEdit.section_name,
        code: '',
      },
    ]
    fetchSectionsOnly()
  }
})

const handleSubmit = async () => {
  isSaving.value = true
  errors.value = {}
  try {
    if (isEdit.value) {
      await api.put(`/enrolments/${props.enrolmentToEdit.id}`, form)
    } else {
      await api.post('/enrolments', form)
    }
    emit('refresh')
    emit('close')
  } catch (e) {
    errors.value = e.response?.data?.errors || {}
  } finally {
    isSaving.value = false
  }
}
</script>
