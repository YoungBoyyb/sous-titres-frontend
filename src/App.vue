<template>
  <div id="app">
    <h1>Sous-titrage TikTok</h1>

    <input type="file" accept="video/mp4" @change="onFileChange" />
    <button @click="upload" :disabled="!selectedFile || loading">
      {{ loading ? "Traitement en cours..." : "Envoyer" }}
    </button>

    <div v-if="videoUrl" style="margin-top: 20px;">
      <h2>Vidéo sous-titrée</h2>
      <video :src="videoUrl" controls width="320"></video>
      <a :href="videoUrl" download="video_sous_titres.mp4">Télécharger</a>
    </div>

    <div v-if="error" style="color: red; margin-top: 20px;">
      Erreur : {{ error }}
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

const selectedFile = ref(null);
const videoUrl = ref("");
const loading = ref(false);
const error = ref("");

// Remplace par l’URL publique de ton Space Hugging Face
const API_URL = "https://Walshi-sous-titres-tiktok-reels.hf.space/api/predict/";

function onFileChange(event) {
  error.value = "";
  videoUrl.value = "";
  const files = event.target.files;
  if (files.length > 0 && files[0].type === "video/mp4") {
    selectedFile.value = files[0];
  } else {
    error.value = "Merci de sélectionner une vidéo MP4.";
  }
}

async function upload() {
  if (!selectedFile.value) return;
  loading.value = true;
  error.value = "";
  videoUrl.value = "";

  try {
    const formData = new FormData();
    formData.append("file", selectedFile.value);

    const response = await axios.post(API_URL, formData, {
      headers: { "Content-Type": "multipart/form-data" },
      responseType: "blob",
    });

    const blob = new Blob([response.data], { type: "video/mp4" });
    videoUrl.value = URL.createObjectURL(blob);
  } catch (err) {
    error.value = "Erreur lors de l’upload ou du traitement.";
  } finally {
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
