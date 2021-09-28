<template>
  <q-page class="column">
    <div class="text_block column">
      <textarea class="textarea" placeholder="Enter you script here" />
    </div>
    <div class="player_block">
      <div class="player_container">
        <video
          ref="player"
          class="player"
          width="100%"
          height="auto"
          autoplay
          muted
        ></video>
      </div>
      <div class="control_buttons_block">
        <div class="buttons_panel">
          <q-btn
            v-if="recorderState === RecorderStates.STOP"
            class="q-ma-sm"
            outline
            round
            color="red"
            icon="fiber_manual_record"
            @click="onStartClick"
          />
          <q-btn
            v-if="recorderState === RecorderStates.RECORD"
            class="q-ma-sm"
            outline
            round
            color="red"
            icon="stop"
            @click="onStopClick"
          />
          <!-- <q-btn
            v-if="recorderState === RecorderStates.RECORD"
            class="q-ma-sm"
            outline
            round
            color="red"
            icon="pause"
          /> -->
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref, onMounted } from "vue";
import { saveAs } from "file-saver";

const RecorderStates = Object.freeze({
  STOP: Symbol("stop"),
  RECORD: Symbol("record"),
  PAUSE: Symbol("pause"),
});

// https://stackoverflow.com/questions/41739837/all-mime-types-supported-by-mediarecorder-in-firefox-and-chrome
function getAllSupportedMimeTypes(...mediaTypes) {
  // iOS Fix
  if (!MediaRecorder.isTypeSupported) {
    return [
      {
        mimeType: "video/mp4",
        mediaType: "video",
        ext: "mp4",
        codec: "h.265",
      },
    ];
  }

  if (!mediaTypes.length) mediaTypes.push(...["video", "audio"]);
  const FILE_EXTENSIONS = ["mp4", "webm", "ogg", "x-matroska"];
  const CODECS = [
    "h265",
    "h.265",
    "vp9",
    "vp9.0",
    "vp8",
    "vp8.0",
    "avc1",
    "av1",
    "h264",
    "h.264",
    "opus",
  ];

  const mimeTypesFormats = (mediaType, ext, codec) => [
    `${mediaType}/${ext};codecs:${codec}`,
    `${mediaType}/${ext};codecs=${codec}`,
    `${mediaType}/${ext};codecs:${codec.toUpperCase()}`,
    `${mediaType}/${ext};codecs=${codec.toUpperCase()}`,
    `${mediaType}/${ext}`,
  ];

  return [
    ...new Set(
      FILE_EXTENSIONS.flatMap((ext) =>
        CODECS.flatMap((codec) =>
          mediaTypes.flatMap((mediaType) =>
            mimeTypesFormats(mediaType, ext, codec).map((mimeType) => ({
              mimeType,
              mediaType,
              ext,
              codec,
            }))
          )
        )
      )
    ),
  ].filter((variation) => MediaRecorder.isTypeSupported(variation.mimeType));
}
const supportedMimeTypes = getAllSupportedMimeTypes("video");
const defaultMimeType =
  supportedMimeTypes.length > 0
    ? supportedMimeTypes[0]
    : {
        mimeType: "video/webm",
        mediaType: "video",
        ext: "webm",
        codec: "vp8",
      };

export default defineComponent({
  setup() {
    const player = ref(null);
    const recorderState = ref(RecorderStates.STOP);

    // let mediaStream;
    let recordedChunks = [];
    let mediaRecorder;

    onMounted(() => {
      console.log("onMounted", player.value);
      navigator.mediaDevices
        .getUserMedia({ audio: true, video: { facingMode: "user" } })
        .then((stream) => {
          player.value.srcObject = stream;

          mediaRecorder = new MediaRecorder(stream, {
            mimeType: defaultMimeType.mimeType,
          });

          mediaRecorder.ondataavailable = (e) => {
            if (e.data.size > 0) {
              recordedChunks.push(e.data);
            }
          };

          mediaRecorder.onstop = (e) => {
            download();
          };
        });
    });

    const onStartClick = () => {
      console.log("onStartClick");
      recordedChunks = [];
      mediaRecorder.start();
      recorderState.value = RecorderStates.RECORD;
    };
    const onStopClick = () => {
      console.log("onStopClick");

      mediaRecorder.stop();
      recorderState.value = RecorderStates.STOP;
    };
    const download = () => {
      console.log("onDownloadClick");

      const blob = new Blob(recordedChunks, {
        type: `${defaultMimeType.mediaType}/${defaultMimeType.ext}`,
      });
      const fileName = `vb_${Date.now()}.${defaultMimeType.ext}`;
      saveAs(blob, fileName);
    };

    return {
      player,
      onStartClick,
      onStopClick,
      // onDownloadClick,
      recorderState,
      RecorderStates,
    };
  },
});
</script>
<style lang="scss" scoped>
.text_block {
  flex: 2 1 0;
  padding: 10px;
  position: relative;
}
.textarea {
  display: block;
  outline: none;
  resize: none;
  border: 1px solid #888;
  border-radius: 3px;
  overflow: auto;
  padding: 10px;
  flex-grow: 1;
}
.player_block {
  flex: 1 1 0;
  position: relative;
}
.control_buttons_block {
  position: absolute;
  margin-left: auto;
  margin-right: auto;
  left: 0;
  right: 0;
  text-align: center;
  bottom: 20px;
  display: flex;
  justify-content: center;
}
.buttons_panel {
  background-color: white;
  border-radius: 30px;
  opacity: 0.5;
}

.player_container {
  position: absolute;
  height: 100%;
  width: 100%;
  overflow: hidden;
  padding: 10px;
}
.player {
  min-width: 100%;
  min-height: 100%;
}
</style>
