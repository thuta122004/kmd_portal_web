<template>
  <div class="flex gap-8 w-full h-[calc(100vh-100px)]">
    <aside class="w-64 shrink-0 h-full overflow-y-auto pr-2 custom-scrollbar">
      <div class="pb-10 space-y-6">
        <div>
          <button
            @click="activeSection = activeSection === 'attendances' ? null : 'attendances'"
            class="w-full text-xs font-semibold text-slate-500 uppercase tracking-widest px-4 mb-3 flex items-center justify-between hover:text-white transition-colors"
          >
            Attendance
            <span>{{ activeSection === 'attendances' ? '−' : '+' }}</span>
          </button>
          <nav v-if="activeSection === 'attendances'">
            <button
              @click="currentTab = 'sections'"
              class="w-full px-4 py-2.5 text-sm font-medium rounded-lg transition-colors flex items-center justify-between text-slate-400 hover:text-white hover:bg-white/5"
            >
              Section Info
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
              @click="currentTab = 'documents'"
              class="w-full px-4 py-2.5 text-sm font-medium rounded-lg transition-colors flex items-center justify-between text-slate-400 hover:text-white hover:bg-white/5"
            >
              My Documents
            </button>
          </nav>
        </div>
      </div>
    </aside>

    <section class="flex-1 h-full overflow-y-auto">
      <StudentSectionInfo v-if="currentTab === 'sections'" />
      <MyDocuments v-if="currentTab === 'documents'" />
    </section>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import StudentSectionInfo from './StudentSectionInfo.vue'
import MyDocuments from './MyDocuments.vue'

const currentTab = ref('sections')
const activeSection = ref('attendances')
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
