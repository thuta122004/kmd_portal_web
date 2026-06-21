<template>
  <div class="flex gap-8 w-full h-[calc(100vh-100px)]">
    <aside class="w-64 shrink-0 h-full overflow-y-auto pr-2 custom-scrollbar">
      <div class="pb-10 space-y-6">
        <div>
          <button
            @click="activeSection = activeSection === 'admin' ? null : 'admin'"
            class="w-full text-xs font-semibold text-slate-500 uppercase tracking-widest px-4 mb-3 flex items-center justify-between hover:text-white transition-colors"
          >
            Administration
            <span>{{ activeSection === 'admin' ? '−' : '+' }}</span>
          </button>
          <nav v-if="activeSection === 'admin'" class="flex flex-col space-y-1">
            <button
              v-for="tab in ['users', 'lecturers', 'students', 'guardians', 'subjects', 'sections']"
              :key="tab"
              @click="currentTab = tab"
              :class="[
                'w-full px-4 py-2.5 text-sm font-medium rounded-lg transition-colors flex items-center justify-between',
                currentTab === tab
                  ? 'bg-white/5 text-white'
                  : 'text-slate-400 hover:text-white hover:bg-white/5',
              ]"
            >
              {{ tab.charAt(0).toUpperCase() + tab.slice(1) }}
            </button>
          </nav>
        </div>

        <div>
          <button
            @click="activeSection = activeSection === 'info' ? null : 'info'"
            class="w-full text-xs font-semibold text-slate-500 uppercase tracking-widest px-4 mb-3 flex items-center justify-between hover:text-white transition-colors"
          >
            Information
            <span>{{ activeSection === 'info' ? '−' : '+' }}</span>
          </button>
          <nav v-if="activeSection === 'info'">
            <button
              @click="currentTab = 'section-info'"
              class="w-full px-4 py-2.5 text-sm font-medium rounded-lg transition-colors flex items-center justify-between text-slate-400 hover:text-white hover:bg-white/5"
            >
              Section Info
            </button>
          </nav>
        </div>

        <div>
          <button
            @click="activeSection = activeSection === 'op' ? null : 'op'"
            class="w-full text-xs font-semibold text-slate-500 uppercase tracking-widest px-4 mb-3 flex items-center justify-between hover:text-white transition-colors"
          >
            Operation
            <span>{{ activeSection === 'op' ? '−' : '+' }}</span>
          </button>
          <nav v-if="activeSection === 'op'" class="flex flex-col space-y-1">
            <button
              @click="currentTab = 'assign-lecturer'"
              class="w-full px-4 py-2.5 text-sm font-medium rounded-lg transition-colors flex items-center justify-between text-slate-400 hover:text-white hover:bg-white/5"
            >
              Assign Lecturer
            </button>
            <button
              @click="currentTab = 'timetable'"
              class="w-full px-4 py-2.5 text-sm font-medium rounded-lg transition-colors flex items-center justify-between text-slate-400 hover:text-white hover:bg-white/5"
            >
              Timetable
            </button>
            <button
              @click="currentTab = 'attach-student'"
              class="w-full px-4 py-2.5 text-sm font-medium rounded-lg transition-colors flex items-center justify-between text-slate-400 hover:text-white hover:bg-white/5"
            >
              Attach Student
            </button>
          </nav>
        </div>
      </div>
    </aside>

    <section class="flex-1 h-full overflow-y-auto">
      <UserList v-if="currentTab === 'users'" />
      <LecturerList v-if="currentTab === 'lecturers'" />
      <StudentList v-if="currentTab === 'students'" />
      <GuardianList v-if="currentTab === 'guardians'" />
      <SubjectList v-if="currentTab === 'subjects'" />
      <SectionList v-if="currentTab === 'sections'" />
      <SectionInfo v-if="currentTab === 'section-info'" />
      <AssignLecturerList v-if="currentTab === 'assign-lecturer'" />
      <AttachStudentList v-if="currentTab === 'attach-student'" />
      <TimetableList v-if="currentTab === 'timetable'" />
    </section>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import UserList from './UserList.vue'
import LecturerList from './LecturerList.vue'
import StudentList from './StudentList.vue'
import GuardianList from './GuardianList.vue'
import SubjectList from './SubjectList.vue'
import SectionList from './SectionList.vue'
import SectionInfo from './SectionInfo.vue'
import AssignLecturerList from './AssignLecturerList.vue'
import AttachStudentList from './AttachStudentList.vue'
import TimetableList from './TimetableList.vue'

const currentTab = ref('users')
const activeSection = ref('admin')
</script>

<style>
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
}
</style>
