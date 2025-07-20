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

// URL de ton Space Hugging Face
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

function fileToBase64(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => {
      const base64 = reader.result.split(",")[1]; // retire le préfixe data:video/mp4;base64,
      resolve(base64);
    };
    reader.onerror = error => reject(error);
    reader.readAsDataURL(file);
  });
}

async function upload() {
  if (!selectedFile.value) return;
  loading.value = true;
  error.value = "";
  videoUrl.value = "";

  try {
    const base64Video = await fileToBase64(selectedFile.value);
    const payload = {
      data: [base64Video]  // Gradio attend un tableau des inputs
    };

    const response = await axios.post(API_URL, payload, {
      headers: { "Content-Type": "application/json" },
      responseType: "json",
    });

    // La réponse Gradio contient souvent le résultat dans response.data.data[0] encodé en base64
    const returnedBase64 = response.data.data[0];

    // Convertir base64 en Blob pour afficher et télécharger la vidéo
    const videoBlob = base64ToBlob(returnedBase64, "video/mp4");
    videoUrl.value = URL.createObjectURL(videoBlob);

  } catch (err) {
    console.error(err);
    error.value = "Erreur lors de l’upload ou du traitement.";
  } finally {
    loading.value = false;
  }
}

function base64ToBlob(base64, type = "application/octet-stream") {
  const binary = atob(base64);
  const len = binary.length;
  const buffer = new Uint8Array(len);
  for (let i = 0; i < len; i++) {
    buffer[i] = binary.charCodeAt(i);
  }
  return new Blob([buffer], { type });
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
