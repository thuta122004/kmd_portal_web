<template>
  <div class="flex gap-8 w-full">
    <aside class="w-64 shrink-0 space-y-8">
      <div>
        <h2 class="text-xs font-semibold text-slate-500 uppercase tracking-widest px-4 mb-3">
          Administration
        </h2>
        <nav class="flex flex-col space-y-1">
          <button
            @click="currentTab = 'users'"
            :class="[
              'w-full px-4 py-2.5 text-sm font-medium rounded-lg transition-colors flex items-center justify-between',
              currentTab === 'users'
                ? 'bg-white/5 text-white'
                : 'text-slate-400 hover:text-white hover:bg-white/5',
            ]"
          >
            User Accounts
          </button>
          <button
            @click="currentTab = 'lecturers'"
            :class="[
              'w-full px-4 py-2.5 text-sm font-medium rounded-lg transition-colors flex items-center justify-between',
              currentTab === 'lecturers'
                ? 'bg-white/5 text-white'
                : 'text-slate-400 hover:text-white hover:bg-white/5',
            ]"
          >
            Lecturers
          </button>
          <button
            @click="currentTab = 'students'"
            :class="[
              'w-full px-4 py-2.5 text-sm font-medium rounded-lg transition-colors flex items-center justify-between',
              currentTab === 'students'
                ? 'bg-white/5 text-white'
                : 'text-slate-400 hover:text-white hover:bg-white/5',
            ]"
          >
            Students
          </button>
        </nav>
      </div>
    </aside>

    <section class="flex-1">
      <div v-if="currentTab === 'users'">
        <UserList />
      </div>
      <div v-if="currentTab === 'lecturers'">
        <LecturerList />
      </div>
      <div v-if="currentTab === 'students'">
        <StudentList />
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import UserList from './UserList.vue'
import LecturerList from './LecturerList.vue'
import StudentList from './StudentList.vue'

const currentTab = ref('users')
</script>
