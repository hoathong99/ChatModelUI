<template>
    <div class="chat-container">
        <header class="chat-header">Chatbot</header>
        <main class="chat-body">
            <div v-for="(msg, index) in messages" :key="index" :class="msg.sender">
                <p>{{ msg.text }}</p>
            </div>
        </main>
        
        <div ref="promptBox" class="chat-input" :class="{ 'sticky-input': inputSent }">
            <div class="welcome_intro" v-if="!inputSent">Hello, What can I help you with?</div>
            <textarea class="text-input" v-model="userInput" ref="textArea" @keydown.enter.prevent="handleEnterKey"
                @input="adjustInputHeight" placeholder="Just ask..." :disabled="isLoading"></textarea>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick } from "vue";

// Define the message structure
interface Message {
    text: string;
    sender: "user" | "bot";
}

// Reactive state variables
const messages = ref<Message[]>([]);
const userInput = ref("");
const textArea = ref<HTMLTextAreaElement | null>(null);
const isLoading = ref(false);
const inputSent = ref(false);

// Adjust input field height dynamically
const adjustInputHeight = () => {
    if (textArea.value) {
        textArea.value.style.height = "auto";
        textArea.value.style.height = Math.min(textArea.value.scrollHeight, 200) + "px";
    }
};

// Handle sending a message
const sendMessage = async () => {
  if (!userInput.value.trim() || isLoading.value) return;

  const userMessage: Message = { text: userInput.value, sender: "user" };
  messages.value.push(userMessage);
  userInput.value = "";
  inputSent.value = true;
  adjustInputHeight();
  isLoading.value = true;

  // Placeholder bot message
  const botMessage: Message = { text: "Generating response...", sender: "bot" };
  messages.value.push(botMessage);

  try {
    const response = await fetch("http://localhost:11434/api/generate", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        model: "deepseek-r1:1.5b",
        prompt: userMessage.text,
        stream: false, // Disable streaming
      }),
    });

    const json = await response.json();

    if (json.response) {
      // Remove the first 15 characters before setting the response
      botMessage.text = json.response.substring(17);
    } else {
      botMessage.text = "No response received.";
    }
  } catch (error) {
    botMessage.text = "Error connecting to AI.";
    console.error("API Error:", error);
  }

  isLoading.value = false;
};


// Handle enter key behavior
const handleEnterKey = (event: KeyboardEvent) => {
    if (event.shiftKey) {
        userInput.value += "\n"; // Allow new line with Shift+Enter
    } else {
        event.preventDefault();
        sendMessage();
    }
};

// Adjust height when mounted
onMounted(() => {
    nextTick(() => {
        if (textArea.value) {
            adjustInputHeight();
        }
    });
});
</script>

<style scoped>
/* Chat container */
.chat-container {
    display: flex;
    flex-direction: column;
    height: 100vh;
    width: 100vh;
}

/* Header */
.chat-header {
    position: sticky;
    top: 0;
    color: #5d5d5d;
    padding: 10px;
    font-weight: bold;
    display: flex;
    background: white;
    z-index: 10;
    border-bottom: 1px solid #ddd;
}

/* Chat body */
.chat-body {
    flex-grow: 1;
    overflow-y: auto;
    padding: 10px;
    display: flex;
    flex-direction: column;
}

/* User message */
.user {
    align-self: flex-end;
    background: #f3f3f3;
    color: #232323;
    padding: 10px;
    border-radius: 10px;
    max-width: 60%;
    margin-bottom: 10px;
}

/* Bot message */
.bot {
    align-self: flex-start;
    background: #d8d7d7;
    padding: 10px;
    border-radius: 10px;
    max-width: 60%;
    margin-bottom: 10px;
}

/* Input area */
.chat-input {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 80%;
    max-width: 600px;
    margin-bottom: 50px;
    resize: vertical;
    max-height: 20vh;
    height: 10vh;
}

/* Sticky input after first message */
.sticky-input {
    position: sticky;
    bottom: 0;
    transform: none;
    width: 100%;
    padding: 10px;
    background: white;
    border-top: 1px solid #ddd;
}

/* Text input field */
.text-input {
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    resize: vertical;
    max-height: 10vh;
    height: 10vh;
}

.welcome_intro{
    text-align: center;
    font-weight: bold;
}

/* Disabled input style */
.text-input:disabled {
    background-color: #f0f0f0;
    cursor: not-allowed;
}
</style>