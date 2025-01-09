<!-- Html -->
<template>
    <div class="container">
        <div class="task-card">
            <h1 class="title">Task List</h1>
            <div class="input-section">
                <input
                    type="text"
                    class="task-input"
                    placeholder="Add a new task..."
                    v-model="newTask"
                    @keyup.enter="addTask"
                    :disabled="tasks.length > 4"
                />
                <button
                    class="btn add-btn"
                    @click="addTask"
                    v-if="tasks.length <= 4"
                >
                    Add
                </button>
                <p v-else class="error-message">Maximum tasks reached</p>
            </div>
            <div class="task-list">
                <div
                    class="task-item"
                    v-for="(task, index) in tasks"
                    :key="task.id"
                >
                    <div class="task-content">
                        <div v-if="task.isEditing">
                            <input
                                type="text"
                                class="task-edit-input"
                                v-model="task.updatedContent"
                                placeholder="Edit task..."
                            />
                            <button
                                class="btn save-btn"
                                @click="updateTask(task.id, index)"
                            >
                                Save
                            </button>
                            <button
                                class="btn cancel-btn"
                                @click="cancelEdit(index)"
                            >
                                Cancel
                            </button>
                        </div>
                        <div v-else>
                            <p class="task-text">{{ task.content }}</p>
                            <button
                                class="btn edit-btn"
                                @click="editTask(index)"
                            >
                                Edit
                            </button>
                            <button
                                class="btn delete-btn"
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
/* General Styles */
body {
    font-family: "Poppins", sans-serif;
    background: linear-gradient(135deg, #6a11cb, #2575fc);
    color: #333;
    margin: 0;
    padding: 0;
}

.container {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    padding: 20px;
}

/* Task Card */
.task-card {
    background: #fff;
    border-radius: 15px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
    width: 100%;
    max-width: 500px;
    padding: 20px 30px;
}

.title {
    font-size: 1.8rem;
    font-weight: bold;
    color: #6a11cb;
    text-align: center;
    margin-bottom: 20px;
}

/* Input Section */
.input-section {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 20px;
}

.task-input {
    flex: 1;
    padding: 10px;
    border: 2px solid #6a11cb;
    border-radius: 10px;
    font-size: 1rem;
    outline: none;
    transition: 0.3s;
}

.task-input:focus {
    border-color: #2575fc;
    box-shadow: 0 0 8px rgba(0, 0, 0, 0.1);
}

.error-message {
    color: red;
    font-size: 0.9rem;
    font-weight: bold;
}

/* Task List */
.task-list {
    display: flex;
    flex-direction: column;
    gap: 15px;
}

.task-item {
    background: #f9f9f9;
    border-radius: 10px;
    padding: 10px 15px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.task-content {
    display: flex;
    align-items: center;
    gap: 10px;
}

.task-text {
    font-size: 1rem;
    color: #333;
    flex: 1;
}

.task-edit-input {
    flex: 1;
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 5px;
    font-size: 1rem;
}

/* Buttons */
.btn {
    border: none;
    padding: 8px 12px;
    font-size: 0.9rem;
    font-weight: bold;
    color: #fff;
    border-radius: 5px;
    cursor: pointer;
    transition: 0.3s;
}

.add-btn {
    background: #6a11cb;
}

.add-btn:hover {
    background: #2575fc;
}

.edit-btn {
    background: #f0ad4e;
}

.edit-btn:hover {
    background: #ec971f;
}

.save-btn {
    background: #5cb85c;
}

.save-btn:hover {
    background: #4cae4c;
}

.cancel-btn {
    background: #d9534f;
}

.cancel-btn:hover {
    background: #c9302c;
}

.delete-btn {
    background: #d9534f;
}

.delete-btn:hover {
    background: #c9302c;
}
</style>
