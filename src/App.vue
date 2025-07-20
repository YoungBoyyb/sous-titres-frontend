<template>
  <div id="app">
    <h1>Sous-titrage TikTok</h1>

    <input type="file" accept="video/mp4" @change="onFileChange" />
    <button @click="startProcess" :disabled="!selectedFile || loading">
      {{ loading ? "Traitement en cours..." : "Envoyer" }}
    </button>

    <div v-if="progressMessage" style="margin-top: 20px;">{{ progressMessage }}</div>

    <div v-if="videoUrl" style="margin-top: 20px;">
      <h2>Vidéo sous-titrée</h2>
      <video :src="videoUrl" controls width="320"></video>
      <a :href="videoUrl" download="video_sous_titres.mp4">Télécharger</a>
    </div>

    <div v-if="error" style="color: red; margin-top: 20px;">Erreur : {{ error }}</div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

const selectedFile = ref(null);
const loading = ref(false);
const progressMessage = ref("");
const videoUrl = ref("");
const error = ref("");
let taskId = null;
let pollInterval = null;

function onFileChange(event) {
  selectedFile.value = event.target.files[0];
  videoUrl.value = "";
  progressMessage.value = "";
  error.value = "";
}

async function startProcess() {
  if (!selectedFile.value) return;
  loading.value = true;
  progressMessage.value = "Envoi de la vidéo...";
  error.value = "";
  videoUrl.value = "";

  const formData = new FormData();
  formData.append("file", selectedFile.value);

  try {
    const response = await axios.post("https://ton-backend/start", formData, {
      headers: { "Content-Type": "multipart/form-data" },
    });
    taskId = response.data.task_id;
    progressMessage.value = "Traitement démarré, attente du résultat...";

    pollInterval = setInterval(checkStatus, 3000);
  } catch (err) {
    error.value = "Erreur lors de l'envoi.";
    loading.value = false;
  }
}

async function checkStatus() {
  try {
    const response = await axios.get(`https://ton-backend/status/${taskId}`);
    const status = response.data.status;

    if (status === "done") {
      clearInterval(pollInterval);
      progressMessage.value = "Traitement terminé.";
      videoUrl.value = response.data.video_url;
      loading.value = false;
    } else if (status === "error") {
      clearInterval(pollInterval);
      error.value = response.data.error || "Erreur pendant le traitement.";
      progressMessage.value = "";
      loading.value = false;
    } else {
      progressMessage.value = "Traitement en cours...";
    }
  } catch {
    clearInterval(pollInterval);
    error.value = "Erreur lors du suivi.";
    progressMessage.value = "";
    loading.value = false;
  }
}
</script>

<style>
#app {
  max-width: 400px;
  margin: 40px auto;
  font-family: Arial, sans-serif;
  text-align: center;
}
button {
  margin-top: 10px;
  padding: 8px 16px;
}
</style>
