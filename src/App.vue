<!-- Html -->
<template>
    <div class="container d-flex justify-content-center align-items-center">
        <div class="card mt-3 col-6">
            <div class="card-body">
                <h1 class="text-center">Task List</h1>
                <div
                    class="d-flex flex-column flex-sm-row justify-content-between mt-3 mt-sm-5"
                >
                    <div class="col-sm-9">
                        <input
                            type="text"
                            class="form-control"
                            placeholder="Add a new task..."
                            v-model="newTask"
                            @keyup.enter="addTask"
                            :disabled="tasks.length > 4"
                        />
                    </div>
                    <div class="mt-3 mt-sm-0">
                        <button
                            type="button"
                            class="btn btn-primary"
                            @click="addTask"
                            v-if="tasks.length <= 4"
                        >
                            Add
                        </button>
                        <p v-else class="message">List completed</p>
                    </div>
                </div>
                <div class="mt-3 mt-sm-5">
                    <div
                        class="card item-card mt-2"
                        v-for="(task, index) in tasks"
                        :key="index"
                    >
                        <div class="card-body">
                            <div class="card-items">
                                <!-- Edit Mode -->
                                <div v-if="task.isEditing">
                                    <input
                                        type="text"
                                        class="form-control"
                                        v-model="task.updatedContent"
                                        placeholder="Edit task content..."
                                    />
                                    <button
                                        type="button"
                                        class="btn btn-success"
                                        @click="updateTask(task.id, index)"
                                    >
                                        Save
                                    </button>
                                    <button
                                        type="button"
                                        class="btn btn-secondary"
                                        @click="cancelEdit(index)"
                                    >
                                        Cancel
                                    </button>
                                </div>

                                <!-- Display Mode -->
                                <div v-else>
                                    <p class="fw-semibold">
                                        {{ task.content }}
                                    </p>
                                    <button
                                        type="button"
                                        class="btn btn-primary"
                                        @click="editTask(index)"
                                    >
                                        Edit
                                    </button>
                                    <button
                                        type="button"
                                        class="btn btn-danger"
                                        @click="deleteTask(task.id, index)"
                                    >
                                        Delete
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<!-- Javascript -->
<script>
import { ref, onMounted, computed } from "vue";

export default {
    name: "HomePage",
    setup() {
        const newTask = ref("");
        const tasks = ref([]);

        // Fetch tasks from the API
        const fetchTasks = async () => {
            try {
                const response = await fetch(
                    "https://2zb7hhngkg.execute-api.eu-north-1.amazonaws.com/default/todolist_get"
                );
                if (response.ok) {
                    tasks.value = await response.json();
                    tasks.value = tasks.value.map((task) => ({
                        ...task,
                        isEditing: false, // Initialize editing state
                        updatedContent: task.content, // Default updated content
                    }));
                } else {
                    console.error(
                        "Failed to fetch tasks:",
                        response.statusText
                    );
                }
            } catch (error) {
                console.error("Error fetching tasks:", error);
            }
        };

        // Add a new task
        const addTask = async () => {
            if (!newTask.value) return;

            try {
                const response = await fetch(
                    "https://2zb7hhngkg.execute-api.eu-north-1.amazonaws.com/default/todolist_post",
                    {
                        method: "POST",
                        headers: { "Content-Type": "application/json" },
                        body: JSON.stringify({ content: newTask.value }),
                    }
                );

                if (response.ok) {
                    const newTaskData = await response.json();
                    tasks.value.unshift({
                        ...newTaskData,
                        isEditing: false,
                        updatedContent: newTaskData.content,
                    });
                    newTask.value = "";
                } else {
                    console.error("Failed to add task:", response.statusText);
                }
            } catch (error) {
                console.error("Error adding task:", error);
            }
        };

        // Edit a task (toggle edit mode)
        const editTask = (index) => {
            tasks.value[index].isEditing = true;
        };

        // Cancel editing a task
        const cancelEdit = (index) => {
            tasks.value[index].isEditing = false;
            tasks.value[index].updatedContent = tasks.value[index].content; // Reset content
        };

        // Update a task using the PUT method
        const updateTask = async (taskId, index) => {
            const updatedContent = tasks.value[index].updatedContent;

            if (!updatedContent) {
                console.error("Updated content cannot be empty");
                return;
            }

            try {
                const response = await fetch(
                    "https://2zb7hhngkg.execute-api.eu-north-1.amazonaws.com/default/todolist_put",
                    {
                        method: "PUT",
                        headers: { "Content-Type": "application/json" },
                        body: JSON.stringify({
                            id: taskId,
                            content: updatedContent,
                        }),
                    }
                );

                if (response.ok) {
                    const updatedTask = await response.json();
                    tasks.value[index].content = updatedTask.content;
                    tasks.value[index].isEditing = false;
                } else {
                    console.error(
                        "Failed to update task:",
                        response.statusText
                    );
                }
            } catch (error) {
                console.error("Error updating task:", error);
            }
        };

        // Delete a single task
        const deleteTask = async (taskId, index) => {
            try {
                const response = await fetch(
                    "https://2zb7hhngkg.execute-api.eu-north-1.amazonaws.com/default/todolist_delete",
                    {
                        method: "DELETE",
                        headers: { "Content-Type": "application/json" },
                        body: JSON.stringify({ id: taskId }),
                    }
                );

                if (response.ok) {
                    tasks.value.splice(index, 1); // Remove the task from the list
                } else {
                    console.error(
                        "Failed to delete task:",
                        response.statusText
                    );
                }
            } catch (error) {
                console.error("Error deleting task:", error);
            }
        };

        onMounted(() => {
            fetchTasks(); // Fetch tasks when the component is mounted
        });

        return {
            newTask,
            tasks,
            addTask,
            editTask, // Ensure this is returned
            cancelEdit, // Ensure this is returned
            updateTask, // Ensure this is returned
            deleteTask, // Ensure this is returned
        };
    },
};
</script>
<!-- Css -->
<style scoped>
.card-body h1 {
    color: #a813ff;
}

.btn-primary {
    background-color: #a813ff;
    border: transparent;
}

.btn-danger {
    background-color: #5f8dba;
    border: transparent;
}

.btn-warning {
    background-color: #952b95;
    color: #fff;
    border: transparent;
}

.message {
    color: green;
    font-weight: bold;
}

.item-card {
    display: flex;
    align-items: center;
    height: 60px;
    background-color: #440c7e;
    color: #fff;
}

.card-items {
    display: flex;
    justify-content: space-evenly;
    flex-direction: row;
    width: 300px;
}
</style>
