<template>
  <div class="d-flex justify-center align-center">
    <v-card width="80%" class="mt-5 pa-5">
      <h1 class="text-n1 text-center ma-2">Editar tarea</h1>
      <v-form validate-on="submit" @submit.prevent="submit">
        <v-text-field
          v-model="task.title"
          required
          label="Titulo"
          variant="outlined"
          :rules="titleRules"
        />
        <v-textarea
          v-model="task.description"
          label="Descripción"
          variant="outlined"
        />
        <v-text-field
          label="Vencimiento"
          v-model="task.due_date"
          type="date"
        ></v-text-field>
        <v-text-field
          v-model="task.comments"
          label="Comentarios"
          variant="outlined"
        />
        <v-text-field
          v-model="task.tags"
          required
          label="Tags"
          variant="outlined"
        />
        <div class="d-flex justify-center">
          <v-btn color="primary" type="submit" class="mt-2">Guardar</v-btn>
          <v-btn color="secunday" class="mt-2 ml-5" to="/">Regresar</v-btn>
        </div>
      </v-form>
    </v-card>
  </div>
  <v-overlay :model-value="isActiveOverlay" class="align-center justify-center">
    <v-progress-circular
      color="primary"
      indeterminate
      size="64"
    ></v-progress-circular>
  </v-overlay>
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
  title: "",
  is_completed: 0,
  due_date: null,
  tags: null,
  description: null,
  comments: null,
});

//form rules

const titleRules = ref([
  (value: string) => {
    if (value) return true;
    return "Por favor ingrese un titulo";
  },
]);

//functions

async function submit(event: any) {
  isActiveOverlay.value = true;
  const results = await event;
  if (results.valid) {
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
  }
  isActiveOverlay.value = false;
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
