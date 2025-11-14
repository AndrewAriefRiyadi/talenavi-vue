<script setup>
import { ref, computed } from 'vue'

const editingCell = ref({ rowIndex: null, key: null })

const props = defineProps({
    tasks: Array,
})

const tasks = computed(() => props.tasks || [])

function formatDate(dateString) {
    const date = new Date(dateString)
    if (isNaN(date)) return dateString
    return date.toLocaleDateString('en-GB', {
        day: '2-digit',
        month: 'short',
        year: 'numeric'
    })
}

function startEditing(rowIndex, key) {
    editingCell.value = { rowIndex, key }
}

function stopEditing() {
    editingCell.value = { rowIndex: null, key: null }
}

const typeColors = ref({
    'Feature Enhancements': '#22c55e',
    'Other': '#6b7280',
    'Bug': '#e11d48'
})

const priorityColors = ref({
    'Critical': '#dc2626',
    'High': '#f97316',
    'Medium': '#facc15',
    'Low': '#4ade80',
    'Best Effort': '#94a3b8'
})

const statusColors = ref({
    'Ready to start': '#6b7280',
    'In Progress': '#2563eb',
    'Waiting for review': '#a855f7',
    'Pending Deploy': '#f59e0b',
    'Done': '#22c55e',
    'Stuck': '#ef4444'
})


function getPercentages(key) {
    const total = tasks.value.length
    if (total === 0) return {}

    const counts = {}
    for (const t of tasks.value) {
        const val = t[key] || 'Unknown'
        counts[val] = (counts[val] || 0) + 1
    }

    const result = {}
    for (const [k, v] of Object.entries(counts)) {
        result[k] = ((v / total) * 100).toFixed(1)
    }
    return result
}

</script>

<template>
    <div class="mt-6 overflow-x-auto bg-neutral-primary-soft shadow-xs rounded-base border border-default">
        <table class="w-full text-sm text-left text-body">
            <thead class="text-sm bg-neutral-secondary-soft border-b border-default">
                <tr>
                    <th class="px-3 py-2 font-medium text-center">
                        <input type="checkbox" class="w-4 h-4 border border-gray-500 rounded" />
                    </th>
                    <th class="px-3 py-2 font-medium">Task</th>
                    <th class="px-3 py-2 font-medium">Developer</th>
                    <th class="px-3 py-2 font-medium">Status</th>
                    <th class="px-3 py-2 font-medium">Priority</th>
                    <th class="px-3 py-2 font-medium">Type</th>
                    <th class="px-3 py-2 font-medium">Date</th>
                    <th class="px-3 py-2 font-medium text-center">Estimated SP</th>
                    <th class="px-3 py-2 font-medium text-center">Actual SP</th>
                </tr>
            </thead>

            <tbody>
                <tr v-for="(task, index) in tasks" :key="index" class="hover:bg-gray-900 transition">
                    <td class="px-3 py-2 border text-center">
                        <input type="checkbox" class="w-4 h-4 border rounded" />
                    </td>

                    <td class="px-3 py-2 border cursor-pointer" @click="startEditing(index, 'title')">
                        <template v-if="editingCell.rowIndex === index && editingCell.key === 'title'">
                            <input v-model="task.title" class="bg-gray-700 text-white w-full px-1" @blur="stopEditing"
                                @keyup.enter="stopEditing" autofocus />
                        </template>
                        <template v-else>{{ task.title }}</template>
                    </td>

                    <td class="px-3 py-2 border cursor-pointer" @click="startEditing(index, 'developer')">
                        <template v-if="editingCell.rowIndex === index && editingCell.key === 'developer'">
                            <input v-model="task.developer" class="bg-gray-700 text-white w-full px-1"
                                @blur="stopEditing" @keyup.enter="stopEditing" />
                        </template>
                        <template v-else>{{ task.developer }}</template>
                    </td>

                    <td :style="{ backgroundColor: statusColors[task.status] }" class="px-3 py-2 border cursor-pointer"
                        @click="startEditing(index, 'status')">
                        <template v-if="editingCell.rowIndex === index && editingCell.key === 'status'">
                            <select v-model="task.status" class="bg-gray-700 text-white w-full" @blur="stopEditing">
                                <option>Ready to start</option>
                                <option>In Progress</option>
                                <option>Waiting for review</option>
                                <option>Pending Deploy</option>
                                <option>Done</option>
                                <option>Stuck</option>
                            </select>
                        </template>
                        <template v-else>{{ task.status }}</template>
                    </td>

                    <td :style="{ backgroundColor: priorityColors[task.priority] }"
                        class="px-3 py-2 border cursor-pointer" @click="startEditing(index, 'priority')">
                        <template v-if="editingCell.rowIndex === index && editingCell.key === 'priority'">
                            <select v-model="task.priority" class="bg-gray-700 text-white w-full" @blur="stopEditing">
                                <option>Critical</option>
                                <option>High</option>
                                <option>Medium</option>
                                <option>Low</option>
                                <option>Best Effort</option>
                            </select>
                        </template>
                        <template v-else>{{ task.priority }}</template>
                    </td>

                    <td :style="{ backgroundColor: typeColors[task.type] }" class="px-3 py-2 border cursor-pointer"
                        @click="startEditing(index, 'type')">
                        <template v-if="editingCell.rowIndex === index && editingCell.key === 'type'">
                            <select :style="{ backgroundColor: typeColors[task.type] }" v-model="task.type"
                                class="bg-gray-700 text-white w-full" @blur="stopEditing">
                                <option>Feature Enhancements</option>
                                <option>Other</option>
                                <option>Bug</option>
                            </select>
                        </template>
                        <template v-else>{{ task.type }}</template>
                    </td>

                    <td class="px-3 py-2 border cursor-pointer" @click="startEditing(index, 'date')">
                        <template v-if="editingCell.rowIndex === index && editingCell.key === 'date'">
                            <input type="date" v-model="task.date" class="bg-gray-700 text-white w-full px-1"
                                @blur="stopEditing" />
                        </template>
                        <template v-else>{{ formatDate(task.date) }}</template>
                    </td>

                    <td class="px-3 py-2 border text-center cursor-pointer"
                        @click="startEditing(index, 'Estimated SP')">
                        <template v-if="editingCell.rowIndex === index && editingCell.key === 'Estimated SP'">
                            <input type="number" v-model.number="task['Estimated SP']"
                                class="bg-gray-700 text-white w-full text-center" @blur="stopEditing" />
                        </template>
                        <template v-else>{{ task['Estimated SP'] }} SP</template>
                    </td>

                    <td class="px-3 py-2 border text-center cursor-pointer" @click="startEditing(index, 'Actual SP')">
                        <template v-if="editingCell.rowIndex === index && editingCell.key === 'Actual SP'">
                            <input type="number" v-model.number="task['Actual SP']"
                                class="bg-gray-700 text-white w-full text-center" @blur="stopEditing" />
                        </template>
                        <template v-else>{{ task['Actual SP'] }} SP</template>
                    </td>
                </tr>

                <tr v-if="tasks.length === 0">
                    <td colspan="9" class="text-center py-3 text-gray-400">No tasks found</td>
                </tr>
                <tr v-if="tasks.length != 0">
                    <td class="px-3 py-2 border cursor-pointer"></td>
                    <td class="px-3 py-2 border cursor-pointer"></td>
                    <td class="px-3 py-2 border cursor-pointer"></td>


                    <td class="px-3 py-2 border">
                        <div class="flex gap-1">
                            <div v-for="(percent, key) in getPercentages('status')" :key="key"
                                class="rounded text-xs text-center text-black font-semibold"
                                :style="{ backgroundColor: statusColors[key] || '#ccc', width: percent + '%' }"
                                :title="`${key}: ${percent}%`">
                                {{ percent }}%
                            </div>
                        </div>
                    </td>


                    <td class="px-3 py-2 border">
                        <div class="flex gap-1">
                            <div v-for="(percent, key) in getPercentages('priority')" :key="key"
                                class="rounded text-xs text-center text-black font-semibold"
                                :style="{ backgroundColor: priorityColors[key] || '#ccc', width: percent + '%' }"
                                :title="`${key}: ${percent}%`">
                                {{ percent }}%
                            </div>
                        </div>
                    </td>


                    <td class="px-3 py-2 border">
                        <div class="flex gap-1">
                            <div v-for="(percent, key) in getPercentages('type')" :key="key"
                                class="rounded text-xs text-center text-black font-semibold"
                                :style="{ backgroundColor: typeColors[key] || '#ccc', width: percent + '%' }"
                                :title="`${key}: ${percent}%`">
                                {{ percent }}%
                            </div>
                        </div>
                    </td>


                    <td class="px-3 py-2 border cursor-pointer"></td>
                </tr>
            </tbody>
        </table>
    </div>
</template>
