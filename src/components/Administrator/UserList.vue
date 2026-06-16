<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h2 class="text-xl font-semibold text-white">Users</h2>
      <button
        @click="openModal(null)"
        class="px-4 py-2 bg-white text-slate-950 text-sm font-semibold rounded-lg hover:bg-slate-200 transition"
      >
        Add User
      </button>
    </div>

    <div class="flex flex-col sm:flex-row gap-3">
      <div class="relative flex-1">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search users..."
          class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-sm text-white placeholder-slate-600 outline-none focus:border-white/20 focus:bg-slate-950/70 transition"
        />
      </div>

      <div class="relative min-w-[150px]">
        <select
          v-model="statusFilter"
          class="w-full h-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 focus:text-white transition appearance-none cursor-pointer"
        >
          <option value="all">All Status</option>
          <option value="active">Active</option>
          <option value="inactive">Inactive</option>
        </select>
        <div
          class="pointer-events-none absolute inset-y-0 right-3 flex items-center text-slate-500"
        >
          <svg class="h-3 w-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M19 9l-7 7-7-7"
            />
          </svg>
        </div>
      </div>
    </div>

    <div class="border border-white/5 rounded-xl overflow-hidden">
      <table class="w-full text-left text-sm">
        <thead class="bg-white/5 text-slate-400 uppercase text-[10px] tracking-wider">
          <tr>
            <th class="p-4 font-medium">Name</th>
            <th class="p-4 font-medium">Email</th>
            <th class="p-4 font-medium">Role</th>
            <th class="p-4 font-medium">Status</th>
            <th class="p-4 text-right">Action</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-for="user in paginatedUsers" :key="user.id" class="text-slate-300">
            <td class="p-4">{{ user.name }}</td>
            <td class="p-4 text-slate-500">{{ user.email }}</td>
            <td class="p-4">
              <span class="text-xs bg-white/5 px-2 py-1 rounded">{{ user.role_name }}</span>
            </td>
            <td class="p-4">
              <button
                @click="handleToggleStatus(user)"
                :class="[
                  'w-8 h-4 rounded-full transition-colors relative',
                  user.status === 'active' ? 'bg-blue-500' : 'bg-slate-700',
                ]"
              >
                <span
                  :class="[
                    'absolute top-1 w-2 h-2 rounded-full bg-white transition-all',
                    user.status === 'active' ? 'left-5' : 'left-1',
                  ]"
                ></span>
              </button>
            </td>
            <td class="p-4 text-right">
              <button @click="openModal(user)" class="text-slate-500 hover:text-white transition">
                Edit
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="flex justify-between items-center text-xs text-slate-500">
      <span>Page {{ currentPage }} of {{ lastPage }}</span>
      <div class="flex gap-2">
        <button
          @click="changePage(currentPage - 1)"
          :disabled="currentPage === 1"
          class="hover:text-white transition"
        >
          Prev
        </button>
        <button
          @click="changePage(currentPage + 1)"
          :disabled="currentPage === lastPage"
          class="hover:text-white transition"
        >
          Next
        </button>
      </div>
    </div>

    <UserModal
      v-if="showModal"
      :userToEdit="selectedUser"
      @close="showModal = false"
      @refresh="fetchUsers"
    />
  </div>
</template>

<script setup>
import { onMounted, ref, computed, watch } from 'vue'
import api from '@/services/api'
import UserModal from './UserModal.vue'

const users = ref([])
const isLoading = ref(false)
const errorMessage = ref(null)

const showModal = ref(false)
const selectedUser = ref(null)
const togglingId = ref(null)

const searchQuery = ref('')
const statusFilter = ref('all')

const currentPage = ref(1)
const itemsPerPage = 7

const filteredUsers = computed(() => {
  return users.value.filter((user) => {
    const matchesSearch =
      user.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      user.email.toLowerCase().includes(searchQuery.value.toLowerCase())

    const matchesStatus = statusFilter.value === 'all' || user.status === statusFilter.value

    return matchesSearch && matchesStatus
  })
})

const lastPage = computed(() => {
  return Math.ceil(filteredUsers.value.length / itemsPerPage) || 1
})

const paginatedUsers = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  const end = start + itemsPerPage
  return filteredUsers.value.slice(start, end)
})

watch([searchQuery, statusFilter], () => {
  currentPage.value = 1
})

const fetchUsers = async () => {
  isLoading.value = true
  errorMessage.value = null
  try {
    const response = await api.get('/users')
    const payload = response.data?.data || response.data

    users.value = payload.users || payload || []
  } catch (error) {
    errorMessage.value = 'Failed to synchronize user lists from backend cluster.'
    console.error(error)
  } finally {
    isLoading.value = false
  }
}

const changePage = (newPage) => {
  if (newPage >= 1 && newPage <= lastPage.value) {
    currentPage.value = newPage
  }
}

const openModal = (user = null) => {
  selectedUser.value = user
  showModal.value = true
}

const handleToggleStatus = async (user) => {
  togglingId.value = user.id
  try {
    const response = await api.patch(`/users/${user.id}/toggle`)
    user.status =
      response.data?.data?.user?.status ||
      response.data?.user?.status ||
      (user.status === 'active' ? 'inactive' : 'active')
  } catch (error) {
    console.error('Failed to change profile clearance status:', error)
  } finally {
    togglingId.value = null
  }
}

onMounted(() => {
  fetchUsers()
})
</script>
