<template>
  <v-container>
    <v-row>
      <v-col>
        <div class="recorder_wrapper">
          <div class="recorder">
            <button
              class="record_btn"
              id="button"
              icon="mdi-microphone"
            ></button>
          </div>
        </div>

        <v-btn
          class="btn"
          :loading="recording"
          @click="startRecording"
          :disabled="recording || playing"
        >
          <v-icon v-if="!recording" size="48">mdi-microphone-outline</v-icon>
          <v-icon v-else size="48">mdi-heart-pulse</v-icon>
          {{ recording ? "Recording..." : "" }}
        </v-btn>

        <v-btn
          @click="stopRecording"
          :disabled="!recording || playing"
          class="btn recorder_btn"
          icon
          ><v-icon size="48">mdi-microphone-outline</v-icon>
        </v-btn>
        <v-btn @click="downloadRecording" :disabled="!recordedAudio"
          ><v-icon>mdi-download</v-icon></v-btn
        >
      </v-col>
    </v-row>

    <v-row v-if="recordedAudio">
      <v-col>
        <v-card @click="togglePlayPause">
          <v-card-title>Click to Play / Pause Recording</v-card-title>
        </v-card>
      </v-col>
    </v-row>

    <v-row v-if="recordedAudios.length > 0">
      <v-col v-for="(audio, index) in recordedAudios" :key="index">
        <v-card @click="playRecording(audio)">
          <v-card-title>Recording {{ index + 1 }}</v-card-title>
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
      recordedAudios: [],
      mediaRecorder: null,
      audioPlayer: null,
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
            this.recordedAudios.push(this.recordedAudio);
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
    playRecording(audio) {
      if (this.audioPlayer) {
        this.audioPlayer.pause();
      }

      this.audioPlayer = new Audio(URL.createObjectURL(audio));
      this.audioPlayer.play();
    },
    togglePlayPause() {
      if (this.audioPlayer) {
        if (this.audioPlayer.paused) {
          this.audioPlayer.play();
        } else {
          this.audioPlayer.pause();
        }
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
<style>
.btn {
  border-radius: 50% !important;
  border-color: purple !important;
  color: white !important;
  background: purple !important;
  width: 100px !important;
  height: 100px !important;
}
.recorder_btn {
  animation: pulse 10s ease-in-out 0s infinite reverse forwards;
}

@keyframes pulse {
  0% {
    animation-timing-function: ease-out;
    transform: scale(1);
    transform-origin: center center;
  }

  10% {
    animation-timing-function: ease-in;
    transform: scale(0.91);
  }

  17% {
    animation-timing-function: ease-out;
    transform: scale(0.98);
  }

  33% {
    animation-timing-function: ease-in;
    transform: scale(0.87);
  }

  45% {
    animation-timing-function: ease-out;
    transform: scale(1);
  }
}
</style>

<!--This one just have Preview But not audio in the List-->
<!-- <template>
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
        <v-card @click="playRecording(recordedAudio)">
          <v-card-title>Click to Play Recording</v-card-title>
        </v-card>
      </v-col>
    </v-row>

    <v-row v-if="recordedAudios.length > 0">
      <v-col v-for="(audio, index) in recordedAudios" :key="index">
        <v-card @click="playRecording(audio)">
          <v-card-title>Recording {{ index + 1 }}</v-card-title>
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
      recordedAudios: [],
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
            this.recordedAudios.push(this.recordedAudio);
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
    playRecording(audio) {
      if (this.playingAudio === audio) {
        this.playing = !this.playing;
      } else {
        this.playingAudio = audio;
        this.playing = true;
        const audioElement = new Audio(URL.createObjectURL(audio));
        audioElement.addEventListener("ended", () => {
          this.playing = false;
        });
        audioElement.play();
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
</script> -->

<!--This one just saved it without the Preview-->

<!-- <template>
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
          {{ recording ? "Recording..." : "Start Recording" }}
        </v-btn>
        <v-btn @click="stopRecording" :disabled="!recording || playing"
          >Stop Recording</v-btn
        >
      </v-col>
    </v-row>

    <v-row>
      <v-col v-for="(audio, index) in recordedAudios" :key="index">
        <v-card>
          <v-card-title>Recording {{ index + 1 }}</v-card-title>
          <v-card-actions>
            <v-btn @click="playRecording(audio)">
              <v-icon>{{
                playingAudio === audio ? "mdi-pause" : "mdi-play"
              }}</v-icon>
            </v-btn>
          </v-card-actions>
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
      recordedAudios: [],
      playingAudio: null,
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
            this.recordedAudios.push(this.recordedAudio);
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
    playRecording(audio) {
      if (this.playingAudio === audio) {
        this.playing = !this.playing;
      } else {
        this.playingAudio = audio;
        this.playing = true;
        const audioElement = new Audio(URL.createObjectURL(audio));
        audioElement.addEventListener("ended", () => {
          this.playing = false;
        });
        audioElement.play();
      }
    },
  },
};
</script> -->
