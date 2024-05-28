<template>
  <div class="flex justify-center items-center pt-20">
    <div class="flex flex-col justify-between bg-gray-900 p-8 w-full max-w-md lg:max-w-lg gap-8 h-[60vh] aspect-[4/3] rounded-lg shadow-lg">
      <div v-if="!messages.length" class="flex flex-col items-center gap-4">
        <img src="/fren-ai-logo.svg" class="w-28 h-28" />
        <span class="text-sm text-normal">By Julius Hellen</span>
        <span class="text-lg text-normal text-center">frenly blockchain oracle and scripting expert</span>
      </div>
      <div v-else class="flex flex-col items-center gap-4 w-full overflow-y-auto h-full bg-gray-800 p-4 rounded-lg">
        <div v-for="(message, index) in messages" :key="index" :class="{'text-blue-500': message.role === 'user', 'text-white': message.role === 'assistant'}" class="w-full">
          {{ message.content }}
        </div>
      </div>
      <div class="flex flex-col gap-4 w-full items-center">
         <div class="relative w-full max-w-lg">
          <input v-model="userInput" placeholder="Ask me anything about engineering..." class="text-normal p-4 border border-gray-400 rounded-full w-full focus:outline-none focus:border-blue-500" @keydown.enter="sendMessage" />
          <button @click="sendMessage" class="absolute right-2 top-1/2 transform -translate-y-1/2 w-10 h-10 bg-gray-400 rounded-full flex items-center justify-center text-gray-900">
            <svg class="w-6 h-6" xmlns="http://www.w3.org/2000/svg" shape-rendering="geometricPrecision" text-rendering="geometricPrecision" image-rendering="optimizeQuality" fill-rule="evenodd" clip-rule="evenodd" viewBox="0 0 463.96 512"><path fill-rule="nonzero" d="M332.67 512V268.5h92.3c15.48-.68 26.47-5.77 32.82-15.42 17.21-25.8-5.25-52.31-22.6-69.25L261.61 14.33c-17.29-19.11-41.93-19.11-59.22 0L24.42 188.72C8.03 204.78-9.67 229.27 6.21 253.08c6.35 9.65 17.34 14.74 32.81 15.42h92.31V512h201.34z"/></svg>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import axios from 'axios';

interface Message {
  role: 'user' | 'assistant';
  content: string;
}

const userInput = ref<string>('');
const messages = ref<Message[]>([]);

const sendMessage = async () => {
  if (userInput.value.trim() === '') return; // Prevent sending empty messages
  messages.value.push({ role: 'user', content: userInput.value });
  let responseContent = 'Thinking...';
  messages.value.push({ role: 'assistant', content: responseContent });
  const assistantIndex = messages.value.length - 1;

  try {
    const body = { model: 'gpt-4', messages: [{ role: 'user', content: userInput.value }] };
    const res = await axios.post(
      'https://api.openai.com/v1/chat/completions',
      body,
      {
        headers: {
          'Authorization': `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
          'Content-Type': 'application/json',
        },
      }
    );
    responseContent = res.data.choices[0].message.content;
  } catch (error) {
    responseContent = 'Error: Unable to get a response.';
    console.error(error);
  } finally {
    messages.value[assistantIndex].content = responseContent;
    userInput.value = ''; // Reset the input field
  }
};
</script>

<style scoped>
/* Add some custom styles to make the UI more appealing */
input::placeholder {
  color: #9ca3af;
}
button:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(66, 153, 225, 0.6);
}
</style>