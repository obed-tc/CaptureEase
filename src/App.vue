<script setup>
import HelloWorld from './components/HelloWorld.vue'
async function button() {
  console.log("Boton")
  const media = await navigator.mediaDevices.getDisplayMedia({
    video: { frameRate: { ideal: 30 } }
  })
  const mediarecorder = new MediaRecorder(media, {
    mimeType: 'video/webm;codecs=vp8,opus'
  })
  mediarecorder.start()

  const [video] = media.getVideoTracks()
  video.addEventListener("ended", () => {
    mediarecorder.stop()
  })

  mediarecorder.addEventListener("dataavailable", (e) => {
    const link = document.createElement("a")
    link.href = URL.createObjectURL(e.data)
    link.download = "captura.mp4"
    link.click()
  })
}
</script>

<template>
  <button @click="button()">Grabar</button>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
