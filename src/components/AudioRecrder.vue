<script setup lang="ts">
import { onMounted, ref } from 'vue';

const audioDevices = ref<MediaDeviceInfo[]>([]);
const selectedDeviceId = ref('');
const audioUrl = ref('');

let mediaRecorder: MediaRecorder | null = null;
let audioChunks: Blob[] = [];

onMounted(async () => {
  const devices = await navigator.mediaDevices.enumerateDevices();
  audioDevices.value = devices.filter(device => device.kind === 'audioinput');

  if (audioDevices.value.length > 0) {
    selectedDeviceId.value = audioDevices.value[0].deviceId;
  }
});

async function startRecording() {
  const stream = await navigator.mediaDevices.getUserMedia({
    audio: { deviceId: selectedDeviceId.value ? { exact: selectedDeviceId.value } : undefined }
  });

  mediaRecorder = new MediaRecorder(stream);
  audioChunks = [];

  mediaRecorder.ondataavailable = e => {
    audioChunks.push(e.data);
  };

  mediaRecorder.onstop = () => {
    const audioBlob = new Blob(audioChunks);
    audioUrl.value = URL.createObjectURL(audioBlob);
  };

  mediaRecorder.start();
}

function stopRecording() {
  if (mediaRecorder) {
    mediaRecorder.stop();
  }
}
</script>

<template>
  <v-select
    label="Audio Devices"
    v-model="selectedDeviceId"
    :items="audioDevices"
    item-title="label"
    item-value="deviceId"
  ></v-select>
  <v-btn @click="startRecording">Record</v-btn>
  <v-btn @click="stopRecording">Stop Recording</v-btn>
  <audio :src="audioUrl" controls></audio>
</template>
