<script setup lang="ts">
import { ref, onUnmounted } from 'vue'

const transcript = ref('')
const isListening = ref(false)
const error = ref('')
const history = ref<string[]>([])

const SpeechRecognition =
  (window as any).SpeechRecognition || (window as any).webkitSpeechRecognition

let recognition: InstanceType<typeof SpeechRecognition> | null = null

function startListening() {
  if (!SpeechRecognition) {
    error.value = 'Your browser does not support the Web Speech API. Try Chrome or Edge.'
    return
  }

  error.value = ''
  transcript.value = ''
  recognition = new SpeechRecognition()
  recognition.lang = 'en-US'
  recognition.interimResults = true
  recognition.continuous = false

  recognition.onresult = (event: any) => {
    const result = event.results[event.results.length - 1]
    transcript.value = result[0].transcript
  }

  recognition.onerror = (event: any) => {
    error.value = `Speech error: ${event.error}`
    isListening.value = false
  }

  recognition.onend = () => {
    isListening.value = false
    if (transcript.value.trim()) {
      history.value.unshift(transcript.value.trim())
    }
  }

  recognition.start()
  isListening.value = true
}

function stopListening() {
  recognition?.stop()
  isListening.value = false
}

function clearHistory() {
  history.value = []
  transcript.value = ''
}

onUnmounted(() => {
  recognition?.abort()
})
</script>

<template>
  <div class="voice-caller">
    <h2>Voice Code Caller</h2>

    <div class="controls">
      <button v-if="!isListening" class="btn btn-start" @click="startListening">
        Speak
      </button>
      <button v-else class="btn btn-stop" @click="stopListening">Stop</button>
    </div>

    <div v-if="isListening" class="status">Listening...</div>

    <div v-if="error" class="error">{{ error }}</div>

    <div v-if="transcript" class="transcript">
      <strong>Heard:</strong> {{ transcript }}
    </div>

    <div v-if="history.length" class="history">
      <div class="history-header">
        <strong>History</strong>
        <button class="btn-clear" @click="clearHistory">Clear</button>
      </div>
      <ul>
        <li v-for="(item, i) in history" :key="i">{{ item }}</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.voice-caller {
  padding: 1.5rem;
  border: 1px solid var(--color-border);
  border-radius: 8px;
  background: var(--color-background-soft);
  max-width: 480px;
  margin: 0 auto;
}

h2 {
  margin-bottom: 1rem;
  font-size: 1.2rem;
  font-weight: 600;
  color: var(--color-heading);
}

.controls {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.btn {
  padding: 0.5rem 1.25rem;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.95rem;
  font-weight: 500;
  transition: opacity 0.2s;
}

.btn:hover {
  opacity: 0.85;
}

.btn-start {
  background: hsla(160, 100%, 37%, 1);
  color: #fff;
}

.btn-stop {
  background: #e53e3e;
  color: #fff;
}

.btn-clear {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 0.8rem;
  color: var(--color-text);
  opacity: 0.6;
  padding: 0;
}

.btn-clear:hover {
  opacity: 1;
}

.status {
  font-size: 0.9rem;
  color: hsla(160, 100%, 37%, 1);
  margin-bottom: 0.75rem;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}

.error {
  color: #e53e3e;
  font-size: 0.9rem;
  margin-bottom: 0.75rem;
}

.transcript {
  padding: 0.75rem;
  background: var(--color-background-mute);
  border-radius: 6px;
  font-size: 0.95rem;
  margin-bottom: 1rem;
}

.history {
  margin-top: 1rem;
}

.history-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.5rem;
}

.history ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.history li {
  padding: 0.5rem 0.75rem;
  background: var(--color-background-mute);
  border-radius: 4px;
  font-size: 0.9rem;
}
</style>
