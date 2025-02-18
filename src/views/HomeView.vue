<template>

  <h1>Speech Recognition Demo</h1>
  <p>using webRTC & webkit speech recognition.</p>

  <div class="input-container" :class="{micActive: isMicActive, isTalking: isTalking }">
    <input v-model="transcript" type="text" placeholder="Say words and they appear here!" />
    <button @click="toggleRecording">
      {{ isRecording ? 'Stop Recording' : 'Start Recording' }}
    </button>
  </div>

</template>

<script setup>
import { ref } from 'vue'

// variables
const transcript = ref('');
const isRecording = ref(false);
let recognition = null;
let isMicActive = false;
let isTalking = false;
let silenceTimeout = null;
let silenceAmount = 3000; // 3 seconds

// check if browser supports webkitSpeechRecognition
if ('webkitSpeechRecognition' in window) {
  // eslint-disable-next-line no-undef
  recognition = new webkitSpeechRecognition();
  recognition.continuous = true;
  recognition.interimResults = true;
  recognition.lang = 'en-US';

  // when speech is recognized
  recognition.onresult = (event) => {
    isTalking = true;

    let fullTranscript = '';
    for (let i = 0; i < event.results.length; i++) {
      fullTranscript += event.results[i][0].transcript + ' ';
    }
    transcript.value = fullTranscript.trim();
    resetSilenceTimer();
  }
  // when speech ends start timer to stop recording
  recognition.onspeechend = () => {
    startSilenceTimer();
    // stopRecording();
  }
}

// toggle recording
const toggleRecording = () => {
  if (isRecording.value) {
    stopRecording();
  } else {
    startRecording();
  }
}

// start recording
const startRecording = () => {
  if (recognition) {
    transcript.value = '';
    isRecording.value = true;
    isMicActive = true;
    recognition.start();
    resetSilenceTimer();

  }
}

// stop recording
const stopRecording = () => {
  if (recognition) {
    isRecording.value = false;
    recognition.stop();
    isMicActive = false;
    clearTimeout(silenceTimeout);
  }
}

// silence timer
const startSilenceTimer = () => {
  clearTimeout(silenceTimeout);
  silenceTimeout = setTimeout(() => {
    stopRecording();
    isTalking = false;
  }, silenceAmount);
};

// reset silence timer
const resetSilenceTimer = () => {
  clearTimeout(silenceTimeout);
  startSilenceTimer();
};
</script>

<style lang="scss" scoped>

// variables
$borderWidth: 3px;
$black: #222222;
$green: #21FA90;
$red: #F02D3A;
$purple: #9191E9;

// main title
h1 {
  text-align: center;
  padding: 40px 0 0 0;
  color: rgb(203, 203, 203);
}

// sub title
p {
  text-align: center;
  padding:  0 0 20px 0;
}

// input container
.input-container {
  display: block;
  position: relative;
  width: 800px;
  margin: auto;

  // when mic is active add a gradient border
  .micActive {
    &:after {
      content: '';
      position: absolute;
      top: calc(-1 * $borderWidth);
      left: calc(-1 * $borderWidth);
      height: calc(100% + $borderWidth * 2);
      width: calc(100% + $borderWidth * 2);
      background: linear-gradient(
        60deg,
        #c633f7,
        #f37055,
        #ef4e7b,
        #a166ab,
        #5073b8,
        #1098ad,
        #07b39b,
        #6fba82
      );
      border-radius: 10px;
      z-index: -1;
      background-size: 400% 400%;
      filter:blur(10px);
      -webkit-backdrop-filter: blur(5px); /*fixes blur for Safari*/
    }
  }

  // when user is talking add a gradient animation
  .isTalking {
    &:after {
      animation: animatedgradient 3s ease alternate infinite;
    }
  }

  // input to take in the transcript
  input {
    position: relative;
    display: inline-block;
    padding: 20px;
    width: calc(100% - 133px);
    border-radius: 10px;
    border-top-right-radius: 0;
    border-bottom-right-radius: 0;
    border: none;

    &:focus {
      outline: none;
    }
  }

  // recording button
  button {
    display: inline-block;
    padding: 20px;
    background-color: #373737fc;
    color: white;
    border: none;
    cursor: pointer;
    border-radius: 10px;
    border-top-left-radius: 0;
    border-bottom-left-radius: 0;
    transition: 0.5s all ease-in-out;

    &:hover {
      background-color: $green;
      transition: 0.5s all ease-in-out;
    }
  }
}

// gradient animation
@keyframes animatedgradient {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
</style>
