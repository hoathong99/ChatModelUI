<script setup lang="ts">
import { ref } from 'vue';

const userInput = ref('');
const responseText = ref('');
const loading = ref(false);

const sendMessage = async () => {
  if (!userInput.value) return;
  loading.value = true;
  responseText.value = '';

  try {
    const res = await fetch('http://localhost:11434/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        model: 'deepseek-r1:1.5b', // Correct model name
        prompt: userInput.value,
        stream: false, // Change to true if you want streaming responses
      }),
    });

    if (!res.ok) {
      throw new Error(`HTTP error! Status: ${res.status}`);
    }

    const data = await res.json();
    responseText.value = data.response;
  } catch (error) {
    responseText.value = `Error: ${error.message}`;
    console.error(error);
  } finally {
    loading.value = false;
  }
};

</script>

<template>
  <div class="chat-container">
    <h1>Deepseek Chatbot</h1>
    <textarea v-model="userInput" placeholder="Type your message..." />
    <button @click="sendMessage" :disabled="loading">Send</button>
    <p v-if="loading">Loading...</p>
    <div v-if="responseText" class="response">{{ responseText }}</div>
  </div>
</template>

<style scoped>
.chat-container {
  max-width: 500px;
  margin: auto;
  text-align: center;
}
textarea {
  width: 100%;
  height: 100px;
  margin-bottom: 10px;
}
button {
  padding: 8px 16px;
  cursor: pointer;
}
.response {
  margin-top: 15px;
  padding: 10px;
  background: #f3f3f3;
  border-radius: 5px;
}
</style>
