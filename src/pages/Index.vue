<template>
  <q-page>
    <div>
      <q-btn label="Start" @click="onStartClick" />
      <q-btn label="Stop" @click="onStopClick" />
      <q-btn label="Download" @click="onDownloadClick" />
    </div>
    <video ref="player" autoplay muted></video>
  </q-page>
</template>

<script>
import { defineComponent, ref, onMounted } from "vue";

export default defineComponent({
  setup() {
    const player = ref(null);

    let mediaStream;
    const recordedChunks = [];
    let mediaRecorder;

    onMounted(() => {
      console.log("onMounted", player.value);
      navigator.mediaDevices
        .getUserMedia({ audio: true, video: true })
        .then((stream) => {
          player.value.srcObject = mediaStream = stream;
        });
    });

    const onStartClick = () => {
      console.log("onStartClick");

      mediaRecorder = new MediaRecorder(mediaStream, {
        mimeType: "video/webm; codecs=vp9",
      });
      mediaRecorder.ondataavailable = (e) => {
        if (e.data.size > 0) {
          recordedChunks.push(e.data);
        }
      };
      mediaRecorder.start();
    };
    const onStopClick = () => {
      console.log("onStopClick");

      mediaRecorder.stop();
    };
    const onDownloadClick = () => {
      console.log("onDownloadClick");

      const blob = new Blob(recordedChunks, {
        type: "video/webm",
      });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      document.body.appendChild(a);
      a.style = "display: none";
      a.href = url;
      a.download = "cam.webm";
      a.click();
      window.URL.revokeObjectURL(url);
    };

    return { player, onStartClick, onStopClick, onDownloadClick };
  },
});
</script>
