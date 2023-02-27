<template>
  <div class="d-flex justify-center">
    <v-card width="80%" class="mt-10 pa-5">
      <h1 class="text-n1 text-center mb-5">Agregar tarea</h1>
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
          <v-btn color="primary" type="submit" class="mt-2">Agregar</v-btn>
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
  <!-- Message -->
  <v-snackbar v-model="showSnackbar" timeout="2000" :color="message.type">
    {{ message.text }}
    <template v-slot:actions>
      <v-btn variant="text" @click="showSnackbar = false"> Cerrar </v-btn>
    </template>
  </v-snackbar>
</template>
<script setup lang="ts">
import { ref, reactive } from "vue";
import axios from "axios";

//variables

const token =
  "e864a0c9eda63181d7d65bc73e61e3dc6b74ef9b82f7049f1fc7d9fc8f29706025bd271d1ee1822b15d654a84e1a0997b973a46f923cc9977b3fcbb064179ecd";
const isActiveOverlay = ref(false);
const router = useRouter();
const task = reactive({
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

//form rules

const titleRules = ref([
  (value: string) => {
    if (value) return true;
    return "Por favor ingrese un titulo";
  },
]);

//functions

function showMessage(text: string, type: string) {
  message.text = text;
  message.type = type;
  showSnackbar.value = true;
}
async function submit(event: any) {
  isActiveOverlay.value = true;
  try {
    const results = await event;
    if (results.valid) {
      const filterTask = Object.fromEntries(
        Object.entries(task).filter(([_, v]) => v != null)
      );
      await axios.post(
        `https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks`,
        null,
        {
          headers: {
            Authorization: `Bearer ${token}`,
            "Content-Type": "application/x-www-form-urlencoded",
          },
          params: { token: token, ...filterTask },
        }
      );
      showMessage("Tarea creada exitosamente", "success");
      router.push("/");
    }
  } catch {
    showMessage("Ha habido un problema al crear la tarea", "error");
  }
  isActiveOverlay.value = false;
}
</script>
