<template>
  <div>
    <h1 class="text-n1 text-center my-10">Lista de tareas</h1>
    <div v-if="tasks.length > 0">
      <div class="d-flex justify-center" v-for="task in tasks" :key="task.id">
        <v-card
          class="task"
          :class="taskClasses(task.is_completed)"
          width="80%"
          max-width="700"
          height="150"
        >
          <template v-slot:title>
            <v-toolbar color="rgba(0, 0, 0, 0)">
              <div>
                <span
                  class="text-h6 text--primary"
                  :class="titleTaskClasses(task.is_completed)"
                >
                  {{ task.title }}
                </span>
                <br />
                <span class="text-caption text-medium-emphasis">
                  {{ task.due_date }}
                </span>
              </div>
              <!-- Drop list -->
              <template v-slot:append>
                <v-menu>
                  <template v-slot:activator="{ props }">
                    <v-btn icon="mdi-dots-vertical" v-bind="props"></v-btn>
                  </template>
                  <v-list>
                    <v-list-item
                      v-for="(item, i) in listItems"
                      :key="i"
                      :value="item"
                      @click.once="onSelectAction(item.action, task.id)"
                    >
                      <template v-slot:prepend>
                        <v-icon :icon="item.icon"></v-icon>
                      </template>
                      <v-list-item-title v-text="item.text"></v-list-item-title>
                    </v-list-item>
                  </v-list>
                </v-menu>
              </template>
            </v-toolbar>
          </template>
          <v-card-actions>
            <v-checkbox
              class="check-box"
              :model-value="task.is_completed == 1"
              :label="checkBoxTaskLabel(task.is_completed)"
              :color="checkBoxTaskColor(task.is_completed)"
              @change="onChangeTaskStatus(task.id, task.is_completed)"
            />
            <v-btn class="mb-5" prepend-icon="mdi-eye" :to="`/task/${task.id}`"
              >Ver detalles</v-btn
            >
          </v-card-actions>
        </v-card>
      </div>
    </div>
    <div v-else>
      <p class="text-center mt-9">Aun no hay tareas</p>
    </div>
    <v-btn
      class="float-left add-btn"
      icon="mdi-plus"
      color="primary"
      to="/task/add"
    />

    <v-overlay
      :model-value="isActiveOverlay"
      class="align-center justify-center"
    >
      <v-progress-circular
        color="primary"
        indeterminate
        size="64"
      ></v-progress-circular>
    </v-overlay>
    <!-- Message -->
    <v-snackbar v-model="showSnackbar" timeout="2000" :color="message.type">
      {{ message.text }}
      <template v-slot:actions>
        <v-btn variant="text" @click="showSnackbar = false"> Cerrar </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>
<script setup lang="ts">
import { ref, reactive } from "vue";
import axios from "axios";

interface Task {
  id: string;
  title: string;
  is_completed: 1 | 0;
  due_date: string;
  tags: string;
  description: string;
  comments: string;
}

//variables

const token =
  "e864a0c9eda63181d7d65bc73e61e3dc6b74ef9b82f7049f1fc7d9fc8f29706025bd271d1ee1822b15d654a84e1a0997b973a46f923cc9977b3fcbb064179ecd";
const tasks = ref([] as Task[]);
const isActiveOverlay = ref(false);
const listItems = reactive([
  { text: "Editar", icon: "mdi-pencil", action: "edit" },
  { text: "Eliminar", icon: "mdi-delete", action: "delete" },
]);
const router = useRouter();
const showSnackbar = ref(false);
const message = reactive({
  type: "",
  text: "",
});

//functions

function showMessage(text: string, type: string) {
  message.text = text;
  message.type = type;
  showSnackbar.value = true;
}
function taskClasses(is_completed: number) {
  return is_completed ? "task-completed " : "task-pending";
}
function titleTaskClasses(is_completed: number) {
  return is_completed ? " text-decoration-line-through" : "";
}
function checkBoxTaskColor(is_completed: number) {
  return is_completed === 1 ? " success" : "info";
}
function checkBoxTaskLabel(is_completed: number) {
  return is_completed ? "Tarea completada" : "Tarea pendiente";
}
async function getTasks() {
  isActiveOverlay.value = true;
  try {
    const res = await axios.get(
      "https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks",
      {
        headers: { Authorization: `Bearer ${token}` },
        params: { token: token },
      }
    );
    tasks.value = res.data;
  } catch {
    showMessage("Ha habido un problema al obtener las tareas", "error");
  }

  isActiveOverlay.value = false;
}
async function onDelteTask(id: string) {
  isActiveOverlay.value = true;
  try {
    await axios.delete(
      `https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks/${id}`,
      {
        headers: {
          Authorization: `Bearer ${token}`,
        },
        params: { token: token },
      }
    );
  } catch {
    showMessage("Ha habido un problema al eliminar la tarea", "error");
  }
  isActiveOverlay.value = false;
  getTasks();
}
async function onUpdateTask(task: Task) {
  try {
    await axios.put(
      `https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks/${task.id}`,
      null,
      {
        headers: {
          Authorization: `Bearer ${token}`,
          "Content-Type": "application/x-www-form-urlencoded",
        },
        params: { token: token, ...task },
      }
    );
  } catch {
    showMessage("Ha habido un problema al actualizar la tarea", "error");
  }
}
function onChangeTaskStatus(id: string, is_completed: 0 | 1) {
  const taskIndex = tasks.value.findIndex((task) => task.id === id);
  tasks.value[taskIndex].is_completed = is_completed ? 0 : 1;
  onUpdateTask(tasks.value[taskIndex]);
}
async function onSelectAction(action: string, id: string) {
  if (action === "delete") await onDelteTask(id);
  if (action === "edit") router.push(`task/edit/${id}`);
  if (action === "show") router.push(`task/${id}`);
}
onMounted(() => {
  getTasks();
});
</script>
<style scoped>
.task {
  margin: 20px;
}
.task-pending {
  background-color: #d9d9d9;
}
.task-completed {
  background-color: #6fce81;
}

.add-btn {
  position: fixed;
  bottom: 40px;
  right: 40px;
}
</style>
