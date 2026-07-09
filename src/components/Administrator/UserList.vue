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

    <div
      v-if="errorMessage"
      class="p-4 bg-rose-500/10 border border-rose-500/20 rounded-xl text-rose-300 text-sm"
    >
      {{ errorMessage }}
    </div>

    <div class="flex flex-col sm:flex-row gap-3">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Search users..."
        class="w-full px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-sm text-white placeholder-slate-600 outline-none focus:border-white/20 transition"
      />
      <select
        v-model="statusFilter"
        class="px-4 py-2.5 bg-slate-950/50 border border-white/5 rounded-lg text-slate-400 text-sm outline-none focus:border-white/20 cursor-pointer appearance-none"
      >
        <option value="all">All Status</option>
        <option value="active">Active</option>
        <option value="inactive">Inactive</option>
        <option value="suspended">Suspended</option>
      </select>
    </div>

    <div class="border border-white/5 rounded-xl overflow-hidden min-h-[100px]">
      <table class="w-full text-left text-sm table-fixed">
        <thead class="bg-white/5 text-slate-400 uppercase text-[10px] tracking-wider">
          <tr>
            <th class="p-4 font-medium w-1/4">Name</th>
            <th class="p-4 font-medium w-1/4">Email</th>
            <th class="p-4 font-medium w-1/5">Role</th>
            <th class="p-4 font-medium w-32">Status</th>
            <th class="p-4 font-medium w-20">Toggle</th>
            <th class="p-4 font-medium w-20 text-right">Action</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-white/5">
          <tr v-if="isLoading">
            <td colspan="6" class="p-10 text-center text-slate-500">Loading users...</td>
          </tr>
          <template v-else-if="paginatedUsers.length > 0">
            <tr v-for="user in paginatedUsers" :key="user.id" class="text-slate-300">
              <td class="p-4 truncate cursor-pointer hover:text-blue-400" @click="openModal(user)">
                {{ user.name }}
              </td>
              <td class="p-4 text-slate-500 truncate">{{ user.email }}</td>
              <td class="p-4">
                <span class="text-xs bg-white/5 px-2 py-1 rounded">{{ user.role_name }}</span>
              </td>
              <td class="p-4">
                <span
                  class="text-[10px] uppercase font-bold px-2 py-0.5 rounded"
                  :class="[
                    user.status === 'active'
                      ? 'bg-blue-500/20 text-blue-400'
                      : user.status === 'inactive'
                        ? 'bg-slate-700/50 text-slate-400'
                        : 'bg-amber-500/20 text-amber-400',
                  ]"
                >
                  {{ user.status }}
                </span>
              </td>
              <td class="p-4">
                <button
                  @click="handleToggleStatus(user)"
                  :class="[
                    'w-8 h-4 rounded-full transition-colors relative',
                    user.status === 'active'
                      ? 'bg-blue-500'
                      : user.status === 'inactive'
                        ? 'bg-slate-700'
                        : 'bg-amber-500',
                  ]"
                >
                  <span
                    :class="[
                      'absolute top-1 w-2 h-2 rounded-full bg-white transition-all',
                      user.status === 'active'
                        ? 'left-5'
                        : user.status === 'inactive'
                          ? 'left-1'
                          : 'left-3',
                    ]"
                  ></span>
                </button>
              </td>
              <td class="p-4 text-right flex justify-end gap-3">
                <button
                  @click="handleResetPassword(user)"
                  class="text-amber-500 hover:text-amber-400 transition"
                >
                  Reset
                </button>
                <button @click="openModal(user)" class="text-slate-500 hover:text-white transition">
                  Edit
                </button>
              </td>
            </tr>
          </template>
          <tr v-else>
            <td colspan="6" class="p-10 text-center text-slate-500 italic">No records found.</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="lastPage > 1" class="flex justify-between items-center text-xs text-slate-500">
      <span>Page {{ currentPage }} of {{ lastPage }}</span>
      <div class="flex gap-2">
        <button
          @click="changePage(currentPage - 1)"
          :disabled="currentPage === 1"
          class="hover:text-white transition disabled:opacity-30"
        >
          Prev
        </button>
        <button
          @click="changePage(currentPage + 1)"
          :disabled="currentPage === lastPage"
          class="hover:text-white transition disabled:opacity-30"
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

    <transition
      enter-active-class="transition ease-out duration-200"
      enter-from-class="opacity-0 translate-y-2"
      enter-to-class="opacity-100 translate-y-0"
      leave-active-class="transition ease-in duration-150"
      leave-from-class="opacity-100 translate-y-0"
      leave-to-class="opacity-0 translate-y-2"
    >
      <div
        v-if="toast.show"
        class="fixed bottom-6 right-6 px-6 py-4 rounded-xl border shadow-2xl flex flex-col gap-4 z-50 min-w-[320px]"
        :class="
          toast.type === 'success'
            ? 'bg-slate-900 border-blue-500/50'
            : toast.type === 'warning'
              ? 'bg-slate-900 border-amber-500/50'
              : 'bg-rose-900 border-rose-500'
        "
      >
        <span class="text-sm font-medium text-white">{{ toast.message }}</span>

        <div v-if="toast.isConfirm" class="flex gap-2 justify-end">
          <button
            @click="toast.show = false"
            class="px-3 py-1 text-xs font-semibold text-slate-400 hover:text-white transition"
          >
            Cancel
          </button>
          <button
            @click="handleConfirm"
            class="px-3 py-1 text-xs font-semibold bg-white text-slate-950 rounded hover:bg-slate-200 transition"
          >
            Confirm
          </button>
        </div>
      </div>
    </transition>
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

const lastPage = computed(() => Math.ceil(filteredUsers.value.length / itemsPerPage) || 1)
const paginatedUsers = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return filteredUsers.value.slice(start, itemsPerPage + start)
})

watch([searchQuery, statusFilter], () => {
  currentPage.value = 1
})

const fetchUsers = async () => {
  isLoading.value = true
  try {
    const response = await api.get('/users')
    users.value = response.data?.data?.users || response.data || []
  } catch (error) {
    errorMessage.value = 'Failed to load users.'
  } finally {
    isLoading.value = false
  }
}

const changePage = (newPage) => {
  if (newPage >= 1 && newPage <= lastPage.value) currentPage.value = newPage
}
const openModal = (user = null) => {
  selectedUser.value = user
  showModal.value = true
}

const handleToggleStatus = async (user) => {
  try {
    const response = await api.patch(`/users/${user.id}/toggle`)
    user.status = response.data?.data?.user?.status || user.status
  } catch (error) {
    console.error(error)
  }
}

const toast = ref({ show: false, message: '', type: 'success', isConfirm: false, onConfirm: null })

const showToast = (message, type = 'success', isConfirm = false, onConfirm = null) => {
  toast.value = { show: true, message, type, isConfirm, onConfirm }
  if (!isConfirm) {
    setTimeout(() => {
      toast.value.show = false
    }, 3000)
  }
}

const handleConfirm = () => {
  if (toast.value.onConfirm) {
    toast.value.onConfirm()
  }
  toast.value.show = false
}

const handleResetPassword = (user) => {
  showToast(`Reset password for ${user.name} to Kmd123!@#?`, 'warning', true, async () => {
    try {
      await api.patch(`/users/${user.id}/reset-password`)
      showToast('Password successfully reset.', 'success')
    } catch (error) {
      showToast('Failed to reset password.', 'error')
    }
  })
}

onMounted(fetchUsers)
</script>
