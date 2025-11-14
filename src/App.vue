<script setup>
import { ref, onMounted, computed } from 'vue'
import TaskTable from './components/TaskTable.vue'
import Kanban from './components/Kanban.vue'

const tasks = ref([])
const searchQuery = ref('')
const developers = ref([])
const selectedDev = ref('Everyone')

const activeComponent = ref('TaskTable')


const sortRules = ref([])

function setActiveComponent(name) {
  activeComponent.value = name
}

async function fetchTasks() {
  const res = await fetch('https://mocki.io/v1/f7861fc0-9071-4034-afed-777f3b590c3c')
  const data = await res.json()

  tasks.value = data.data

  developers.value = [
    ...new Set(
      data.data.flatMap(t => t.developer.split(',')).map(n => n.trim())
    )
  ]
}


const filteredTasks = computed(() => {
  let result = tasks.value.filter(task => {
    const matchSearch =
      !searchQuery.value ||
      task.title.toLowerCase().includes(searchQuery.value.toLowerCase())

    const matchDev =
      selectedDev.value === 'Everyone' ||
      task.developer
        .split(',')
        .map(d => d.trim().toLowerCase())
        .includes(selectedDev.value.toLowerCase())

    return matchSearch && matchDev
  })

  
  if (sortRules.value.length > 0) {
    result = [...result].sort((a, b) => {
      for (const rule of sortRules.value) {
        const aVal = (a[rule.key] || '').toString().toLowerCase()
        const bVal = (b[rule.key] || '').toString().toLowerCase()

        if (aVal < bVal) return rule.order === 'asc' ? -1 : 1
        if (aVal > bVal) return rule.order === 'asc' ? 1 : -1
      }
      return 0
    })
  }

  return result
})

function applySort() {
  sortRules.value = sortOptions.value
    .filter(opt => opt.active)
    .map(opt => ({ key: opt.key, order: opt.order }))
}

function addNewTask() {
  tasks.value.unshift({
    title: 'New Task',
    developer: '',
    status: 'Ready to start',
    priority: 'Low',
    type: 'Other',
    date: new Date().toISOString().split('T')[0],
    'Estimated SP': 0,
    'Actual SP': 0
  })
}

function filterByDeveloper(dev) {
  selectedDev.value = dev
}

const sortOptions = ref([
  { key: 'title', label: 'Task', active: false, order: 'asc' },
  { key: 'status', label: 'Status', active: false, order: 'asc' },
  { key: 'priority', label: 'Priority', active: false, order: 'asc' },
  { key: 'type', label: 'Type', active: false, order: 'asc' },
  { key: 'developer', label: 'Developer', active: false, order: 'asc' },
])

const updateTaskStatus = (title, newStatus) => {
    const task = tasks.value.find(t => t.title === title)
    if (task) task.status = newStatus
}


onMounted(fetchTasks)
</script>

<template>
  <div class="min-h-screen flex flex-col pt-16  bg-linear-to-r from-gray-900 to-gray-800 w-screen text-white">
    <div class=" mx-10">

      <div class="w-full flex gap-4 border-b  border-b-white/10 items-center ">
        <button :class="activeComponent === 'TaskTable' ? 'border-b' : 'border-0'"
          @click="setActiveComponent('TaskTable')" class="px-2  py-2 hover:border-b hover:border-b-blue-600  ">
          Main Table
        </button>
        <button :class="activeComponent === 'Kanban' ? 'border-b' : 'border-0'" @click="setActiveComponent('Kanban')"
          class="px-2 py-2 hover:border-b hover:border-b-blue-600 ">
          Kanban
        </button>
        <p class=" px-2 py-2 text-center">+</p>
      </div>

      <div class=" flex gap-2 mt-6 items-center justify-items-center">
        <button @click="addNewTask" class=" bg-blue-600 px-2 py-1 rounded cursor-pointer hover:bg-blue-700">
          New Task
        </button>
        <div class=" flex hover:bg-gray-800 px-2 py-1 rounded  w-fit gap-1 focus:bg-gray-800 ">
          <label for="search">🔍</label>
          <input v-model="searchQuery" id="search" name="search" type="text" placeholder="Search"
            class=" max-w-16 px-2 focus:outline-0"></input>
        </div>
        <details class="dropdown hover:bg-gray-800 px-2 py-1 rounded cursor-pointer">
          <summary>{{ selectedDev }}</summary>
          <ul class="menu dropdown-content bg-base-100 rounded-box z-1 w-52 p-2 shadow-sm">
            <li v-for="dev in developers" :key="dev">
              <a @click="filterByDeveloper(dev)">{{ dev }}</a>
            </li>
          </ul>
        </details>
        <details class="dropdown  hover:bg-gray-800 px-3 py-2 rounded-lg cursor-pointer">
          <summary class="flex items-center gap-2 font-medium select-none">
            🧭 Sort
          </summary>

          <ul
            class="menu dropdown-content bg-gray-900 text-white rounded-lg w-56 mt-2 p-2 shadow-lg border border-gray-700">
            <li v-for="col in sortOptions" :key="col.key"
              class="flex justify-between items-center py-1 px-2 rounded hover:bg-gray-800">
              <label class="flex items-center justify-between w-full cursor-pointer">
                <span>{{ col.label }}</span>
                <input type="checkbox" v-model="col.active" class="checkbox checkbox-sm checkbox-primary" />
              </label>

              <select v-if="col.active" v-model="col.order"
                class="select select-xs bg-gray-700 text-white border-none focus:outline-none">
                <option value="asc">ASC</option>
                <option value="desc">DESC</option>
              </select>
            </li>

            <li class="mt-2">
              <button @click="applySort" class="btn btn-sm btn-primary w-full">
                Apply Sort
              </button>
            </li>
          </ul>
        </details>

        <div class=" flex py-1 px-2 rounded  w-fit gap-1">

        </div>
      </div>

      
      <TaskTable v-if="activeComponent === 'TaskTable'" :tasks="filteredTasks" />
      <Kanban v-else :tasks="filteredTasks" @update-status="updateTaskStatus" />
    </div>

  </div>
</template>


<style scoped></style>
