<template>
  <v-container>
    <v-row>
      <v-col>
        <v-btn
          :loading="recording"
          @click="startRecording"
          :disabled="recording || playing"
        >
          <v-icon v-if="!recording">mdi-microphone</v-icon>
          <v-icon v-else>mdi-heart-pulse</v-icon>
          {{ recording ? "Recording..." : "" }}
        </v-btn>
        <v-btn @click="stopRecording" :disabled="!recording || playing"
          >Stop Recording</v-btn
        >
        <v-btn @click="downloadRecording" :disabled="!recordedAudio"
          ><v-icon>mdi-download</v-icon></v-btn
        >
      </v-col>
    </v-row>

    <v-row v-if="recordedAudio">
      <v-col>
        <v-card @click="playRecording">
          <v-card-title>Click to Play Recording</v-card-title>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      recording: false,
      playing: false,
      recordedAudio: null,
      mediaRecorder: null,
    };
  },
  methods: {
    startRecording() {
      navigator.mediaDevices
        .getUserMedia({ audio: true })
        .then((stream) => {
          this.mediaRecorder = new MediaRecorder(stream);
          let chunks = [];

          this.mediaRecorder.ondataavailable = (e) => {
            chunks.push(e.data);
          };

          this.mediaRecorder.onstop = () => {
            this.recordedAudio = new Blob(chunks, { type: "audio/wav" });
          };

          this.mediaRecorder.start();
          this.recording = true;
        })
        .catch((err) => {
          console.error("Error accessing microphone:", err);
        });
    },
    stopRecording() {
      if (this.mediaRecorder) {
        this.mediaRecorder.stop();
        this.recording = false;
      }
    },
    playRecording() {
      if (this.recordedAudio) {
        const audio = new Audio(URL.createObjectURL(this.recordedAudio));
        audio.play();
      }
    },
    downloadRecording() {
      if (this.recordedAudio) {
        const url = URL.createObjectURL(this.recordedAudio);
        const a = document.createElement("a");
        document.body.appendChild(a);
        a.style = "display: none";
        a.href = url;
        a.download = "recorded_audio.wav";
        a.click();
        window.URL.revokeObjectURL(url);
      }
    },
  },
};
</script>
