<template>
  <div class="app-container">
    <div class="text-h1 pa-2">Vue ToDo List</div>

    <v-card class="mx-auto" elevation="8" max-width="600" rounded="lg" style="width: 100%">
      <v-card color="grey-lighten-4" flat height="60px">
        <v-toolbar>
          <v-spacer></v-spacer>

          <v-chip icon color="primary" class="mr-2">
            Tasks
            <template v-slot:append>
              <v-badge
                color="primary"
                :content="totalTasks" 
                inline>
              </v-badge>
            </template>
          </v-chip>

          <v-chip v-if="taskDone > 0" icon color="primary" class="mr-2">
            Tasks Done
            <template v-slot:append>
              <v-badge
                color="primary"
                :content="taskDone" 
                inline>
              </v-badge>
            </template>
          </v-chip>

          <v-chip v-if="taskDone > 0" icon color="red" rounded="sm" @click.stop="deleteTaskDone" class="mr-2">
            <v-icon right>mdi-delete</v-icon>
            Tasks Done
          </v-chip>

          <v-chip icon color="red" rounded="sm" @click.stop="deleteAllTasks" class="mr-2">
            <v-icon>mdi-delete</v-icon>
            Tasks
          </v-chip>
        </v-toolbar>
      </v-card>

      <v-card-text>
        <v-list ref="todoList" class="custom-scroll-list">
          <transition-group name="list" tag="v-list">
            <v-list-item
              v-for="(item, i) in items"
              :key="item.id"
              :value="item"
              color="primary"
              rounded="sm"
              class="custom-list-item"
              @mouseenter="visible = i"
              @mouseleave="visible = null"
            >
              <template v-slot:prepend>
                <v-icon @click.stop="doneTheItem(i)" :color="item.status.done ? 'grey' : 'green'" icon="mdi-check-circle"></v-icon>
              </template>
              <v-list-item-title :style="{ textDecoration: item.status.done ? 'line-through' : 'none', color: item.status.done ? 'gray' : 'black' }">
                <b>{{ item.name }}</b>
              </v-list-item-title>
              <template v-if="visible === i" v-slot:append>
                <v-icon color="red" @click.stop="removeItem(i)">mdi-delete</v-icon>
              </template>
            </v-list-item>
          </transition-group>
        </v-list>
      </v-card-text>

      <v-card-actions>
        <v-text-field
          color="primary"
          rounded="xl"
          density="compact"
          placeholder="New task"
          variant="outlined"
          v-model="newTask"
        >
          <template #append-inner>
            <v-icon color="primary" @click.stop="addTask">
              mdi-plus-circle
            </v-icon>
          </template>
        </v-text-field>
      </v-card-actions>
    </v-card>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue';

const STORAGE_KEY = 'tasks';

const todoListRef = ref(null);
const newTask = ref(null);
const visible = ref(null);
const totalTasks = ref(0);
const taskDone = ref(0);
const items = ref([
  { id: 1, name: 'Do something awesome!', status: { done: false } },
  { id: 2, name: 'Buy toilet paper', status: { done: false } },
  { id: 3, name: 'Learn Nuxt', status: { done: false } },
  // ... (remaining items)
]);

const getTasksFromLocalStorage = (() => {
  const storedTasks = localStorage.getItem(STORAGE_KEY);
  return storedTasks ? JSON.parse(storedTasks) : null;
});

const saveTasksToLocalStorage = (() => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(items.value));
  totalTasks.value = items.value.length;
});


const updateTaskDoneCount = (() => {
  taskDone.value = items.value.filter(item => item.status.done).length;
});

const deleteTaskDone = (() => {
  items.value = items.value.filter(item => !item.status.done);
  saveTasksToLocalStorage();
  updateTaskDoneCount();
});

const deleteAllTasks = (() => {
  items.value = [];
  saveTasksToLocalStorage();
  updateTaskDoneCount();
});

onMounted(() => {
  const storedTasks = getTasksFromLocalStorage();
  if (storedTasks) {
    items.value = storedTasks;
    totalTasks.value = storedTasks.length;
  } else {
    totalTasks.value = items.value.length;
  }
  updateTaskDoneCount();
});

onBeforeUnmount(() => {
  saveTasksToLocalStorage();
});

const doneTheItem = (index) => {
  if (index !== -1) {
    items.value[index].status.done = !items.value[index].status.done;
  }
  updateTaskDoneCount();
  saveTasksToLocalStorage();
};

const addTask = () => {
  if (newTask.value) {
    const newItem = {
      id: items.value.length + 1,
      name: newTask.value,
      status: { done: false },
    };
    items.value.push(newItem);
    newTask.value = null;
    if (todoListRef.value && items.value.length > 3) {
      todoListRef.value.style.overflowY = 'auto';
      todoListRef.value.style.maxHeight = '180px';
    }
    saveTasksToLocalStorage();
  }
};

const removeItem = (index) => {
  if (index !== -1) {
    items.value.splice(index, 1);
    saveTasksToLocalStorage();
  }
  updateTaskDoneCount();
};
</script>

<style scoped>
.app-container {
  background: linear-gradient(to bottom, #34b17e, #244455);
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: white;
  margin-bottom: 20px;
}

.custom-list-item {
  border: 1px solid #e0e0e0;
  margin-bottom: 10px;
  padding: 10px;
}

.custom-scroll-list {
  max-height: 180px;
  overflow-y: hidden;
}

.custom-scroll-list:hover {
  overflow-y: auto;
}

.custom-scroll-list::-webkit-scrollbar {
  width: 10px;
}

.custom-scroll-list::-webkit-scrollbar-thumb {
  background-color: #6b6b6b;
  border-radius: 5px;
}

.list-enter-active,
.list-leave-active {
  transition: opacity 0.5s, transform 0.5s;
}

.list-enter,
.list-leave-to {
  opacity: 0;
  transform: translateX(-20px);
}
</style>
