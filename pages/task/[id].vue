<template>
  <div class="d-flex justify-center">
    <v-card width="80%" class="mt-10 pa-5">
      <h1 class="text-n1 text-center mb-5">{{ task.title }}</h1>
      <v-divider></v-divider>
      <p v-if="task.comments" class="mt-5 font-weight-light">
        <span>Vencimiento: </span>
        <span>{{ task.due_date }}</span>
      </p>
      <v-divider v-if="task.due_date"></v-divider>
      <p class="mt-5 text-medium-emphasis">
        <span>Descripción: </span>
        <span v-if="task.description">{{ task.description }}</span>
        <span v-else>Sin descripción</span>
      </p>
      <p v-if="task.comments" class="mt-5 text-medium-emphasis">
        Comentarios: {{ task.comments }}
      </p>
      <p class="mt-6 font-italic text-disabled text-end text-subtitle-2">
        {{ task.tags }}
      </p>
      <v-divider></v-divider>
      <v-checkbox
        class="check-box"
        :model-value="task.is_completed == 1"
        :label="checkBoxTaskLabel()"
        :color="checkBoxTaskColor()"
        @change="onChangeTaskStatus()"
      />
      <div class="d-flex justify-center">
        <v-btn color="secunday" class="mt-2 ml-5" :to="`/task/edit/${task.id}`"
          >Editar</v-btn
        >
        <v-btn color="secunday" class="mt-2 ml-5" to="/">Regresar</v-btn>
      </div>
    </v-card>
    <v-snackbar v-model="showSnackbar" timeout="2000" :color="message.type">
      {{ message.text }}
      <template v-slot:actions>
        <v-btn variant="text" @click="showSnackbar = false"> Cerrar </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>
<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";

//variables
const token =
  "e864a0c9eda63181d7d65bc73e61e3dc6b74ef9b82f7049f1fc7d9fc8f29706025bd271d1ee1822b15d654a84e1a0997b973a46f923cc9977b3fcbb064179ecd";
const isActiveOverlay = ref(false);
const route = useRoute();
const router = useRouter();
const task = ref({
  id: "",
  title: "",
  is_completed: 0,
  due_date: null,
  tags: null,
  description: null,
  comments: null,
});
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
function checkBoxTaskColor() {
  return task.value.is_completed === 1 ? "success" : "info";
}
function checkBoxTaskLabel() {
  return task.value.is_completed ? "Tarea completada" : "Tarea pendiente";
}
async function onChangeTaskStatus() {
  try {
    task.value.is_completed = task.value.is_completed ? 0 : 1;
    await axios.put(
      `https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks/${route.params.id}`,
      null,
      {
        headers: {
          Authorization: `Bearer ${token}`,
          "Content-Type": "application/x-www-form-urlencoded",
        },
        params: { token: token, ...task.value },
      }
    );
  } catch {
    showMessage("Ha habido un error al actualizar la tarea", "error");
  }
}
async function getTask() {
  isActiveOverlay.value = true;
  const res = await axios.get(
    `https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks/${route.params.id}`,
    {
      headers: { Authorization: `Bearer ${token}` },
      params: { token: token },
    }
  );
  if (res.status === 202) router.push("/");
  else task.value = res.data[0];
  isActiveOverlay.value = false;
}
onMounted(() => {
  getTask();
});
</script>
<style lang="scss" scoped>
.task-pending {
  color: #d9d9d9;
}
.task-completed {
  color: #6fce81;
}
</style>
