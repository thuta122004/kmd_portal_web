<template>
  <div class="max-w-3xl mx-auto pb-12 space-y-6 animate-fade-in">
    <div class="flex items-center justify-between pb-4 border-b border-white/5">
      <h2 class="text-xl font-semibold text-white">Announcements</h2>
    </div>

    <div
      v-if="errorMessage"
      class="p-4 bg-rose-500/10 border border-rose-500/20 rounded-xl text-rose-300 text-sm"
    >
      {{ errorMessage }}
    </div>

    <div v-if="isLoading" class="space-y-6">
      <div
        v-for="n in 3"
        :key="n"
        class="bg-slate-900 border border-white/5 rounded-2xl p-6 animate-pulse"
      >
        <div class="w-1/4 h-4 bg-slate-800 rounded mb-4"></div>
        <div class="w-3/4 h-6 bg-slate-800 rounded mb-4"></div>
        <div class="space-y-2">
          <div class="w-full h-3 bg-slate-800 rounded"></div>
          <div class="w-full h-3 bg-slate-800 rounded"></div>
          <div class="w-5/6 h-3 bg-slate-800 rounded"></div>
        </div>
      </div>
    </div>

    <div v-else-if="announcements.length > 0" class="space-y-8">
      <article
        v-for="post in announcements"
        :key="post.id"
        class="bg-slate-900 border border-white/5 rounded-2xl overflow-hidden shadow-lg relative group transition-all duration-300 hover:border-white/10"
      >
        <div v-if="post.is_pinned" class="absolute top-0 right-0 w-16 h-16 overflow-hidden z-10">
          <div
            class="absolute top-0 right-0 bg-amber-500/20 border-l border-b border-amber-500/30 w-full h-full transform translate-x-8 -translate-y-8 rotate-45 flex items-end justify-center pb-1"
          >
            <svg class="w-3.5 h-3.5 text-amber-400 mb-1" fill="currentColor" viewBox="0 0 20 20">
              <path d="M5 4a2 2 0 012-2h6a2 2 0 012 2v14l-5-2.5L5 18V4z" />
            </svg>
          </div>
        </div>

        <div v-if="post.banner_url" class="w-full h-48 sm:h-64 bg-slate-950/50 overflow-hidden">
          <img
            :src="post.banner_url"
            alt="Announcement Banner"
            class="w-full h-full object-cover object-center transition-transform duration-700 group-hover:scale-105"
          />
        </div>

        <div class="p-6 sm:p-8">
          <div class="flex flex-wrap items-center gap-3 mb-4">
            <span
              :class="[
                'px-2.5 py-1 rounded-md text-[10px] font-bold tracking-widest uppercase border',
                post.section_id
                  ? 'bg-blue-500/10 text-blue-400 border-blue-500/20'
                  : 'bg-purple-500/10 text-purple-400 border-purple-500/20',
              ]"
            >
              {{ post.section_name }}
            </span>
            <span class="text-xs font-medium text-slate-500 flex items-center gap-1.5">
              <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"
                />
              </svg>
              {{ formatDate(post.created_at) }}
            </span>
          </div>

          <h3 class="text-xl font-semibold text-white mb-3 leading-snug">
            {{ post.title }}
          </h3>

          <p class="text-sm text-slate-300 leading-relaxed whitespace-pre-wrap">
            {{ post.content }}
          </p>
        </div>
      </article>
    </div>

    <div
      v-else
      class="flex flex-col items-center justify-center py-20 px-4 text-center border border-white/5 rounded-2xl bg-slate-900/50"
    >
      <div class="w-16 h-16 bg-slate-800 rounded-full flex items-center justify-center mb-4">
        <svg class="w-8 h-8 text-slate-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="1.5"
            d="M19 20H5a2 2 0 01-2-2V6a2 2 0 012-2h10a2 2 0 012 2v1m2 13a2 2 0 01-2-2V7m2 13a2 2 0 002-2V9.5a2 2 0 00-2-2h-2m-4-3H9M7 16h6M7 8h6v4H7V8z"
          />
        </svg>
      </div>
      <h3 class="text-lg font-medium text-white mb-1">You're all caught up!</h3>
      <p class="text-sm text-slate-500">There are no new announcements.</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import api from '@/services/api'

const announcements = ref([])
const isLoading = ref(true)
const errorMessage = ref(null)

const formatDate = (dateString) => {
  if (!dateString) return ''
  const date = new Date(dateString)
  return date.toLocaleDateString('en-US', {
    month: 'short',
    day: 'numeric',
    year: 'numeric',
    hour: 'numeric',
    minute: '2-digit',
  })
}

const fetchStudentAnnouncements = async () => {
  isLoading.value = true
  errorMessage.value = null
  try {
    const res = await api.get('/student/announcements')

    announcements.value = res.data?.data?.announcements || res.data || []
  } catch (error) {
    console.error(error)
    errorMessage.value = 'Unable to load your announcement feed'
  } finally {
    isLoading.value = false
  }
}

onMounted(fetchStudentAnnouncements)
</script>
