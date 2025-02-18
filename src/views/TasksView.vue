<script setup lang="ts">
import { computed, onMounted, ref, toRefs, watch } from 'vue'
import GeneralTable from '@/components/GeneralTable.vue'
import { useTasksStore } from '@/stores/tasks'
import { convertToShortDate } from '@/utils/convertToShortDate'
import ModalWindow from '@/components/AddTaskModal.vue'
import type { Task } from '@/types/Task'
import { useRoute } from 'vue-router'
import { getFromLocalStorage } from '@/utils/getFromLocalStorage'

const route = useRoute()
const projectId = ref(+route.params.projectId)

const tasksStore = useTasksStore()

const { isLoading: isLoadingTasks, error: errorTasks, tasks } = toRefs(tasksStore)

const isModalOpen = ref(false)

const query = ref(getFromLocalStorage<string>('tasksQuery') || '')
const status = ref(getFromLocalStorage<string>('tasksStatus') || '')

const visibleTasks = computed(() => {
  return tasks.value
    .filter((task) => task.projectId === projectId.value)
    .filter((task) => {
      const matchesQuery =
        !query.value || task.performer.toLowerCase().includes(query.value.trim().toLowerCase())
      const matchesStatus = !status.value || task.status === status.value

      return matchesQuery && matchesStatus
    })
})

onMounted(() => {
  tasksStore.fetchTasks()
})

watch(query, (newVal) => {
  localStorage.setItem('tasksQuery', JSON.stringify(newVal))

});

watch(status, (newVal) => {
  localStorage.setItem('tasksStatus', JSON.stringify(newVal))
});

function ToggleModal() {
  isModalOpen.value = !isModalOpen.value
}

function updateStatus(newStatus: string, id: number) {
  console.log(newStatus)
  tasksStore.updateTaskStatus(id, newStatus)
}
</script>

<template>
  <ModalWindow :projectId="projectId" v-if="isModalOpen" @close="ToggleModal"></ModalWindow>

  <header class="projects-container">
    <div class="projects-header">
      <span class="projects-title">Tasks</span>

      <button type="button" class="add-project-btn" @click="ToggleModal">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <line x1="12" y1="5" x2="12" y2="19"></line>
          <line x1="5" y1="12" x2="19" y2="12"></line>
        </svg>
        Add a task
      </button>
    </div>

    <div class="forms-container">
      <div class="form-group">
        <form @submit.prevent="">
          <input
            type="text"
            placeholder="Enter the name of the Performer"
            v-model="query"
            class="project-input"
          />
        </form>
      </div>

      <div class="form-group">
        <form @submit.prevent="">
          <select name="languages" v-model="status" class="project-select">
            <option value="" selected>All statuses</option>
            <option value="To Do">To Do</option>
            <option value="In Progress">In Progress</option>
            <option value="Done">Done</option>
          </select>
        </form>
      </div>
    </div>
  </header>

  <h1 v-if="isLoadingTasks">Loading...</h1>
  <h1 v-if="errorTasks">Error: {{ errorTasks }}</h1>

  <GeneralTable
    v-if="!isLoadingTasks && !errorTasks"
    name="tasks"
    @updateStatus="updateStatus"
    :key="visibleTasks.length"
    :isRowsDraggable="true"
    :customColumns="[
      { name: 'Id', width: 150, sort: true },
      { name: 'Title', width: 150, sort: true },
      { name: 'Performer', width: 150, sort: true },
      { name: 'Status', width: 150, sort: true },
      { name: 'Deadline', width: 150, sort: false },
    ]"
    :customRows="
      visibleTasks.map((task: Task) => [
        task.id.toString(),
        task.name,
        task.performer,
        task.status,
        convertToShortDate(task.deadline),
      ])
    "
  />
</template>
