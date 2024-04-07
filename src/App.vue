<script setup>
import { ref } from 'vue';
import Footer from './components/Footer.vue';
import Header from './components/Header.vue';

let mediarecorder = null;
let recordingStream = null;

async function startRecording() {
  try {
    const constraints = {
      video: { frameRate: { ideal: 30 }, width: selectedResolution.value.width, height: selectedResolution.value.height }
    };

    if (recordAudio.value) {
      constraints.audio = true;
    }

    recordingStream = await navigator.mediaDevices.getDisplayMedia(constraints);
    mediarecorder = new MediaRecorder(recordingStream, { mimeType: 'video/webm;codecs=vp8,opus' });
    mediarecorder.start();

    const [video] = recordingStream.getVideoTracks();
    video.addEventListener("ended", () => {
      stopRecording();
    });

    mediarecorder.addEventListener("dataavailable", (e) => {
      handleRecordingData(e.data);
    });
  } catch (error) {
    console.error("Error al grabar la pantalla:", error);
  }
}

function stopRecording() {
  if (mediarecorder) {
    mediarecorder.stop();
    mediarecorder = null;
    recordingStream.getTracks().forEach(track => track.stop());
    recordingStream = null;
  }
}

function handleRecordingData(data) {
  const blob = new Blob([data], { type: selectedFormat.value });
  if (saveLocally.value) {
    downloadRecording(blob);
  } else {
    previewRecording(blob);
  }
}

function downloadRecording(blob) {
  const url = URL.createObjectURL(blob);
  const link = document.createElement("a");
  link.href = url;
  link.download = `captura.${selectedFormat.value.split('/')[1]}`;
  link.click();
}
const previewUrl = ref('');

function previewRecording(blob) {
  previewUrl.value = URL.createObjectURL(blob);
}

function saveVideo() {
  if (previewUrl.value) {
    downloadRecording(new Blob([previewUrl.value]));
  }
}

const isRecording = ref(false);
const saveLocally = ref(true);
const selectedFormat = ref('video/webm');
const resolutions = [
  { label: '1080p', width: 1920, height: 1080 },
  { label: '720p', width: 1280, height: 720 },
  { label: '480p', width: 854, height: 480 },
  { label: '2160p (4K)', width: 3840, height: 2160 }
];

const selectedResolution = ref(resolutions[0]);

function toggleRecording() {
  if (!isRecording.value) {
    startRecording();
    isRecording.value = true;
  } else {
    stopRecording();
    isRecording.value = false;
  }
}
function clearPreview() {
  previewUrl.value = '';
}

</script>

<template>
  <Header></Header>

  <div class="container-fluid bg-dark text-white py-5">
    <div class="row justify-content-center">
      <div class="col-md-8">
        <div class="mb-4" v-if="previewUrl">
          <video :src="previewUrl" controls style="max-width: 100%;"></video>
        </div>
        <button v-if="previewUrl" @click="saveVideo" type="button" class="btn btn-outline-primary">
          Guardar Video
        </button>
        <button v-if="previewUrl" @click="clearPreview" type="button" class="btn btn-outline-danger">
          Limpiar Vista Previa
        </button>

        <h1 class="display-4 text-center mb-4 ">Graba tu pantalla</h1>
        <p class="lead text-center mb-5">
          Graba tu pantalla en tiempo real y decide si quieres guardarla localmente o previsualizar el video.
        </p>
        <div class="d-flex justify-content-center mb-4">
          <button @click="toggleRecording" type="button" class="btn btn-outline-success me-3">
            <i :class="isRecording ? 'bi bi-stop-circle' : 'bi bi-record-circle'"></i>
            {{ isRecording ? 'Detener' : 'Grabar' }}
          </button>
          <div class="dropdown me-3">
            <button class="btn btn-outline-info dropdown-toggle" type="button" data-bs-toggle="dropdown"
              aria-expanded="false">
              Formato: {{ selectedFormat.split('/')[1] }}
            </button>
            <ul class="dropdown-menu">
              <li v-for="format in ['video/webm', 'video/mp4']" :key="format">
                <a class="dropdown-item" href="#" @click.prevent="selectedFormat = format">{{ format.split('/')[1]
                  }}</a>
              </li>
            </ul>
          </div>
          <div class="dropdown">
            <button class="btn btn-outline-info dropdown-toggle" type="button" data-bs-toggle="dropdown"
              aria-expanded="false">
              Resolución: {{ selectedResolution.label }}
            </button>
            <ul class="dropdown-menu">
              <li v-for="resolution in resolutions" :key="resolution.label">
                <a class="dropdown-item" href="#" @click.prevent="selectedResolution = resolution">{{ resolution.label
                  }}</a>
              </li>
            </ul>
          </div>
        </div>
        <div class="d-flex justify-content-center mb-4">
          <div class="form-check form-switch">
            <input class="form-check-input" type="checkbox" role="switch" id="saveLocally" v-model="saveLocally">
            <label class="form-check-label" for="saveLocally">Guardar localmente</label>
          </div>
        </div>
        <div class="d-flex justify-content-center mb-4">
          <div class="form-check form-switch">
            <input class="form-check-input" type="checkbox" role="switch" id="recordAudio" v-model="recordAudio">
            <label class="form-check-label" for="recordAudio">Grabar audio</label>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="container my-5">
    <div class="row justify-content-center">
      <div class="col-md-8">
        <h2>Características:</h2>
        <ul>
          <li>Grabación de pantalla en tiempo real</li>
          <li>Opción de guardar localmente o previsualizar en la web</li>
          <li>Selección de formato de video (WebM, MP4)</li>
          <li>Selección de resolución (1080p, 720p, 480p)</li>
          <li>Control de inicio y detención de grabación</li>
          <li>Compatibilidad multiplataforma (Windows, macOS, Linux)</li>
          <li>Interfaz de usuario intuitiva</li>
        </ul>
      </div>
    </div>
  </div>
  <Footer></Footer>

</template>

<style>
.container-fluid {
  min-height: 50vh;
  display: flex;
  align-items: center;
  justify-content: center
}

.container {
  max-width: 800px;
}

::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background-color: #f1f1f1;
}

::-webkit-scrollbar-thumb {
  background-color: #888;
  border-radius: 8px;
  transition: background-color 0.3s ease;
}

::-webkit-scrollbar-thumb:hover {
  background-color: #555;
}
</style>