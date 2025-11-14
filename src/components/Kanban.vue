<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
    tasks: Array
})

const emit = defineEmits(["update-status"])

const draggedTask = ref(null)

const priorityColors = {
    'Critical': '#dc2626',
    'High': '#f97316',
    'Medium': '#facc15',
    'Low': '#4ade80',
    'Best Effort': '#94a3b8'
}

const statusColors = {
    'Ready to start': '#6b7280',
    'In Progress': '#2563eb',
    'Waiting for review': '#a855f7',
    'Pending Deploy': '#f59e0b',
    'Done': '#22c55e',
    'Stuck': '#ef4444'
}

const statuses = [
    'Ready to start',
    'In Progress',
    'Waiting for review',
    'Pending Deploy',
    'Done',
    'Stuck'
]

const tasksByStatus = computed(() => {
    const groups = {}
    statuses.forEach(s => groups[s] = [])
    props.tasks.forEach(task => {
        const s = task.status || 'Ready to start'
        groups[s].push(task)
    })
    return groups
})


const onDragStart = (task) => {
    draggedTask.value = task
}


const onDrop = (newStatus) => {
    if (!draggedTask.value) return
    console.log(draggedTask.value)

    emit("update-status", draggedTask.value.title, newStatus)
    draggedTask.value = null
}
</script>

<template>
    <div class="grid grid-cols-6 gap-4 p-4">

        <div v-for="status in statuses" :key="status"
            class="flex flex-col bg-gray-800 rounded-lg h-screen overflow-y-auto border border-gray-700"
            @dragover.prevent @drop="onDrop(status)">
            <p class="text-white font-semibold px-3 py-2 rounded-t" :style="{ backgroundColor: statusColors[status] }">
                {{ status }}
            </p>

            <div class="flex flex-col gap-2 p-2">
                <div v-for="(task, index) in tasksByStatus[status]" :key="index" draggable="true"
                    @dragstart="onDragStart(task)"
                    class="bg-gray-900 border border-gray-700 p-3 rounded hover:bg-gray-700 cursor-pointer">

                    <p class="font-semibold">{{ task.title }}</p>

                    <div :style="{ backgroundColor: priorityColors[task.priority] }" class="badge badge-soft">
                        {{ task.priority }}
                    </div>

                    <p class="text-xs mt-2 text-gray-400">
                        {{ task.date }}
                    </p>

                    <p class="text-xs mt-1 text-blue-400">
                        Est: {{ task['Estimated SP'] }} • Act: {{ task['Actual SP'] }}
                    </p>

                    <p class="text-xs text-gray-300">{{ task.developer }}</p>

                </div>

                <p v-if="tasksByStatus[status].length === 0" class="text-gray-500 text-xs text-center py-4">
                    No tasks
                </p>
            </div>
        </div>

    </div>
</template>
