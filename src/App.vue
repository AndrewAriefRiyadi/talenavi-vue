<script setup>
import { ref, onMounted } from 'vue';
import TaskTable from './components/TaskTable.vue';


const taskTableRef = ref(null)
const searchQuery = ref('')
const developers = ref([])
const selectedDev = ref('Everyone')


function addNewTask() {
  taskTableRef.value?.addNewTask()
}

function handleDevelopersReady(devs) {
  developers.value = devs
  developers.value.unshift(
    "Everyone"
  )
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

function applySort() {
  const activeSorts = sortOptions.value.filter(o => o.active)
  taskTableRef.value?.applySort(activeSorts)
}
</script>

<template>
  <div class="min-h-screen flex flex-col justify-center bg-linear-to-r from-gray-900 to-gray-800 w-screen text-white">
    <div class=" mx-10">

      <!-- NAV -->
      <div class="w-full flex gap-4 border-b  border-b-white/10 items-center ">
        <button class="px-2 border-b py-2 transition hover:border-b-blue-600  ">
          Main Table
        </button>
        <button class="px-2 py-2 hover:border-b hover:border-b-blue-600 ">
          Kanban
        </button>
        <p class=" px-2 py-2 text-center">+</p>
      </div>

      <!-- Search -->
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

              <!-- Dropdown untuk memilih ASC/DESC jika aktif -->
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

      <!-- TABLE -->
      <TaskTable ref="taskTableRef" :search="searchQuery" @developers-ready="handleDevelopersReady"
        :dev="selectedDev" />

    </div>

  </div>
</template>


<style scoped></style>
