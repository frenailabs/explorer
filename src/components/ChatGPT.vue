<template>
  <div class="flex justify-center items-center pt-20">
    <div class="flex flex-col justify-between bg-base-100 border border-gray-500 p-8 w-full max-w-md lg:max-w-lg gap-8 h-[60vh] aspect-[4/3] rounded-lg shadow-lg">
      <div v-if="!messages.length" class="flex flex-col items-center gap-4">
        <img src="/fren-ai-logo.svg" class="w-28 h-28" />
        <span class="text-lg text-normal text-center">frenly blockchain oracle and scripting expert</span>
      </div>
      <div v-else class="flex flex-col items-center gap-4 w-full overflow-y-auto h-full bg-base-100 p-4 rounded-lg">
        <div v-for="(message, index) in messages" :key="index" :class="{'text-blue-500': message.role === 'user', 'text-white': message.role === 'assistant'}" class="w-full">
          {{ message.content }}
        </div>
      </div>
      <div class="flex flex-col gap-4 w-full items-center">
         <div class="relative w-full max-w-lg">
          <input v-model="userInput" placeholder="Ask me anything about engineering..." class="text-normal bg-base-100 text-slate-50 p-4 border border-gray-400 rounded-full w-full focus:outline-none focus:border-blue-500" @keydown.enter="sendMessage" />
          <button @click="sendMessage" class="absolute right-2 top-1/2 transform -translate-y-1/2 w-10 h-10 bg-gray-400 rounded-full flex items-center justify-center">
            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="none" viewBox="0 0 32 32" class="icon-2xl">
              <path fill="#2a334c" fill-rule="evenodd" d="M15.192 8.906a1.143 1.143 0 0 1 1.616 0l5.143 5.143a1.143 1.143 0 0 1-1.616 1.616l-3.192-3.192v9.813a1.143 1.143 0 0 1-2.286 0v-9.813l-3.192 3.192a1.143 1.143 0 1 1-1.616-1.616z" clip-rule="evenodd"></path>
          </svg>
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