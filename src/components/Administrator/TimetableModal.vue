<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-sm animate-fade-in"
  >
    <div class="w-full max-w-md p-8 bg-slate-900 border border-white/5 rounded-2xl shadow-2xl">
      <h3 class="text-lg font-semibold text-white mb-6">
        {{ isEdit ? 'Edit Timetable' : 'Create Timetable' }}
      </h3>

      <div
        v-if="errorMessage"
        class="mb-6 p-3 bg-rose-500/10 border border-rose-500/20 rounded-lg text-rose-300 text-xs"
      >
        {{ errorMessage }}
      </div>

      <form @submit.prevent="handleSubmit" class="space-y-4">
        <div>
          <select
            v-model="form.section_assignments_id"
            :disabled="isEdit"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-300 text-sm outline-none focus:border-white/20 transition appearance-none disabled:opacity-50"
          >
            <option value="" disabled>Select Subject & Section</option>
            <option v-for="a in availableAssignments" :key="a.id" :value="a.id">
              {{ a.subject_code || a.subject_name }} - {{ a.section_code || a.section_name }} ({{
                a.lecturer_name
              }})
            </option>
          </select>
          <p v-if="errors.section_assignments_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.section_assignments_id[0] }}
          </p>
        </div>

        <div>
          <select
            v-model="form.day_of_week"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-300 text-sm outline-none focus:border-white/20 transition appearance-none"
          >
            <option value="" disabled>Select Day of Week</option>
            <option v-for="day in daysOfWeek" :key="day" :value="day">{{ day }}</option>
          </select>
          <p v-if="errors.day_of_week" class="text-rose-500 text-[10px] mt-1">
            {{ errors.day_of_week[0] }}
          </p>
        </div>

        <div class="flex gap-4">
          <div class="flex-1">
            <label class="block text-xs text-slate-500 mb-1">Start Time</label>
            <input
              type="time"
              v-model="form.start_time"
              class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
            />
            <p v-if="errors.start_time" class="text-rose-500 text-[10px] mt-1">
              {{ errors.start_time[0] }}
            </p>
          </div>

          <div class="flex-1">
            <label class="block text-xs text-slate-500 mb-1">End Time</label>
            <input
              type="time"
              v-model="form.end_time"
              class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
            />
            <p v-if="errors.end_time" class="text-rose-500 text-[10px] mt-1">
              {{ errors.end_time[0] }}
            </p>
          </div>
        </div>

        <div>
          <input
            type="text"
            v-model="form.room_number"
            placeholder="Room Number"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
          />
          <p v-if="errors.room_number" class="text-rose-500 text-[10px] mt-1">
            {{ errors.room_number[0] }}
          </p>
        </div>

        <div>
          <input
            type="url"
            v-model="form.link"
            placeholder="Meeting Link"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm"
          />
          <p v-if="errors.link" class="text-rose-500 text-[10px] mt-1">
            {{ errors.link[0] }}
          </p>
        </div>

        <div class="flex gap-3 pt-2">
          <button
            type="button"
            @click="$emit('close')"
            class="flex-1 py-2.5 text-sm text-slate-400 hover:text-white transition"
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
import { ref, reactive, computed, onMounted } from 'vue'
import api from '@/services/api'

const props = defineProps({ timetableToEdit: Object })
const emit = defineEmits(['close', 'refresh'])

const isEdit = computed(() => !!props.timetableToEdit)
const isSaving = ref(false)
const availableAssignments = ref([])
const errors = ref({})
const errorMessage = ref(null)

const daysOfWeek = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']

const formatTimeForInput = (timeString) => {
  if (!timeString) return ''
  return timeString.substring(0, 5)
}

const form = reactive({
  section_assignments_id:
    props.timetableToEdit?.section_assignments_id ||
    props.timetableToEdit?.section_assignment_id ||
    '',
  day_of_week: props.timetableToEdit?.day_of_week || '',
  start_time: formatTimeForInput(props.timetableToEdit?.start_time) || '',
  end_time: formatTimeForInput(props.timetableToEdit?.end_time) || '',
  room_number: props.timetableToEdit?.room_number || '',
  link: props.timetableToEdit?.link || '',
})

const fetchDropdownData = async () => {
  try {
    const response = await api.get('/section-assignments')
    const fetchedAssignments = response.data?.data?.assignments || response.data || []
    availableAssignments.value = fetchedAssignments.filter((a) => a.status !== 'inactive')
  } catch (error) {
    console.error(error)
  }
}

onMounted(() => {
  if (!isEdit.value) {
    fetchDropdownData()
  } else {
    availableAssignments.value = [
      {
        id: form.section_assignments_id,
        subject_code:
          props.timetableToEdit.subject_code ||
          props.timetableToEdit.section_assignment?.subject?.code ||
          props.timetableToEdit.subject_name ||
          'Selected Assignment',
        section_code:
          props.timetableToEdit.section_code ||
          props.timetableToEdit.section_assignment?.section?.name ||
          props.timetableToEdit.section_name ||
          '',
        lecturer_name:
          props.timetableToEdit.lecturer_name ||
          props.timetableToEdit.section_assignment?.lecturer?.user?.name ||
          props.timetableToEdit.lecturer_name ||
          '',
      },
    ]
  }
})

const handleSubmit = async () => {
  isSaving.value = true
  errors.value = {}
  try {
    if (isEdit.value) {
      await api.put(`/timetables/${props.timetableToEdit.id}`, form)
    } else {
      await api.post('/timetables', form)
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
