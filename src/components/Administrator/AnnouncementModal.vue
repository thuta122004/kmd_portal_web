<template>
  <div
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-sm animate-fade-in"
  >
    <div class="w-full max-w-md p-8 bg-slate-900 border border-white/5 rounded-2xl shadow-2xl">
      <div class="flex items-center justify-between mb-6">
        <h3 class="text-lg font-semibold text-white">
          {{ isEdit ? 'Edit Announcement' : 'Create Announcement' }}
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
        <div>
          <select
            v-if="!isEdit"
            v-model="form.section_id"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-300 text-sm outline-none focus:border-white/20 transition appearance-none"
          >
            <option value="">Global (All Sections)</option>
            <option v-for="sec in sections" :key="sec.id" :value="sec.id">
              {{ sec.code }}
            </option>
          </select>

          <div
            v-else
            class="w-full px-4 py-2.5 bg-slate-950/30 border border-white/5 rounded-lg text-sm select-none flex justify-between items-center"
          >
            <span class="text-slate-500">Target Audience:</span>
            <span
              class="text-blue-400 font-medium bg-blue-500/10 border border-blue-500/20 px-2.5 py-0.5 rounded text-xs uppercase tracking-wide"
            >
              {{
                sections.find((sec) => sec.id === form.section_id)?.code ||
                (form.section_id ? 'Loading...' : 'Global (All Sections)')
              }}
            </span>
          </div>

          <p v-if="errors.section_id" class="text-rose-500 text-[10px] mt-1">
            {{ errors.section_id[0] }}
          </p>
        </div>

        <div>
          <input
            type="text"
            v-model="form.title"
            placeholder="Announcement Title"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition"
          />
          <p v-if="errors.title" class="text-rose-500 text-[10px] mt-1">{{ errors.title[0] }}</p>
        </div>

        <div>
          <textarea
            v-model="form.content"
            placeholder="Write your announcement details here..."
            rows="4"
            class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-white text-sm outline-none focus:border-white/20 transition resize-none"
          ></textarea>
          <p v-if="errors.content" class="text-rose-500 text-[10px] mt-1">
            {{ errors.content[0] }}
          </p>
        </div>

        <div>
          <label class="block text-xs text-slate-500 mb-1"
            >Optional Banner Image (JPEG, JPG, PNG)</label
          >
          <input
            type="file"
            accept="image/*"
            @change="handleFileChange"
            class="w-full text-sm text-slate-400 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:bg-white/10 file:text-white hover:file:bg-white/20 file:transition cursor-pointer"
          />
          <p v-if="errors.banner" class="text-rose-500 text-[10px] mt-1">{{ errors.banner[0] }}</p>
        </div>

        <div v-if="!isEdit" class="flex items-center gap-2 pt-1">
          <input
            id="is_pinned"
            type="checkbox"
            v-model="form.is_pinned"
            class="w-4 h-4 rounded bg-slate-950/50 border border-white/5 accent-blue-500 cursor-pointer"
          />
          <label
            for="is_pinned"
            class="text-xs text-slate-400 select-none cursor-pointer hover:text-white transition"
          >
            Pin this announcement at the top of feeds
          </label>
        </div>

        <div class="flex gap-3 pt-4">
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

const props = defineProps({ announcementToEdit: Object })
const emit = defineEmits(['close', 'refresh'])

const isEdit = computed(() => !!props.announcementToEdit)
const isSaving = ref(false)
const errorMessage = ref(null)
const errors = ref({})
const bannerFile = ref(null)
const sections = ref([])

const form = reactive({
  section_id: props.announcementToEdit?.section_id || '',
  title: props.announcementToEdit?.title || '',
  content: props.announcementToEdit?.content || '',
  is_pinned: props.announcementToEdit?.is_pinned || false,
})

const fetchSections = async () => {
  try {
    const res = await api.get('/sections')
    sections.value = res.data?.data?.sections || res.data || []
  } catch (e) {
    console.error('Failed to locate target school structured section array data patterns.', e)
  }
}

onMounted(fetchSections)

const handleFileChange = (e) => {
  bannerFile.value = e.target.files[0]
}

const handleSubmit = async () => {
  isSaving.value = true
  errorMessage.value = null
  errors.value = {}

  const formData = new FormData()

  if (isEdit.value) {
    formData.append('_method', 'PUT')
  }

  if (form.section_id) {
    formData.append('section_id', form.section_id)
  }
  formData.append('title', form.title)
  formData.append('content', form.content)
  formData.append('is_pinned', form.is_pinned ? '1' : '0')

  if (bannerFile.value) {
    formData.append('banner', bannerFile.value)
  }

  try {
    if (isEdit.value) {
      await api.post(`/announcements/${props.announcementToEdit.id}`, formData, {
        headers: { 'Content-Type': 'multipart/form-data' },
      })
    } else {
      await api.post('/announcements', formData, {
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
