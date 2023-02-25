<template>
  <h1 class="text-n1 text-center">Lista de tareas</h1>
  <div class="d-flex justify-center" v-for="task in tasks" :key="task.id">
    <v-card
      class="task"
      :title="task.title"
      :subtitle="task.due_date"
      :text="task.description"
    >
    </v-card>
  </div>
  <v-btn class="float-left add-btn" icon="mdi-plus" color="primary"></v-btn>
</template>
<script setup lang="ts">
import { ref } from "vue";
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
const tasks = ref([] as Task[]);
const token =
  "e864a0c9eda63181d7d65bc73e61e3dc6b74ef9b82f7049f1fc7d9fc8f29706025bd271d1ee1822b15d654a84e1a0997b973a46f923cc9977b3fcbb064179ecd";
//functions
async function getTasks() {
  const res = await axios.get(
    "https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks",
    {
      headers: { Authorization: `Bearer ${token}` },
      params: { token: token },
    }
  );
  tasks.value = res.data;
}
onMounted(() => {
  getTasks();
});
</script>
<style scoped>
.task {
  margin: 20px;
  height: 150px;
  width: 80%;
  max-width: 700px;
}
.add-btn {
  position: fixed;
  bottom: 40px;
  right: 40px;
}
</style>
