<template>
  <header class="fixed w-full h-12 bg-white flex justify-between items-center p-4 z-10">
    <img src="@/assets/logo.svg" class="w-28" alt="Algecom's logo">
    <a href="https://app.algecom.com" class="font-family-dela text-xs bg-neutral-800 hover:bg-neutral-700 duration-300 text-white text-center py-1 px-6 rounded-md cursor-pointer">
      Go to Console
    </a>
  </header>
  <div class="fixed inset-0 flex justify-center items-center bg-neutral-100 bg-cover bg-center bg-[url(https://picsum.photos/2560/1440)]">
    <div :class="conversation.length ? 'md:w-5/12 py-8 px-2' : 'md:w-5/12 py-16 px-4'" class="bg-white text-center flex flex-col justify-center items-center gap-12 rounded-xl border-2 border-neutral-200 mx-4">
      <h1 class="font-family-dela text-5xl my-4 overflow-hidden cursor-default">Mr7ba bik M3ana</h1>
      <p class="w-11/12 md:w-9/12 text-sm font-semibold text-neutral-500 text-center cursor-default">
        Algecom isn’t just here to take orders — it chats with your customers,
        answers their questions, and sells like a pro, all in Algerian slang!
        Focus on growing, we’ll handle the hustle.
      </p>
      <div class="space-y-2 duration-500" :class="conversation.length ? 'w-10/12 md:w-8/12' : 'w-8/12 md:w-6/12'">
        <h6 class="text-start font-extrabold text-neutral-500 cursor-default">Give it a try</h6>
        <form v-if="!conversation.length" class="relative" @submit.prevent>
          <input v-model="message" maxlength="100" placeholder="Slm wchrak ?" class="w-full bg-neutral-100 border-2 border-neutral-300 hover:border-neutral-400 duration-300 text-sm px-2 py-1.5 rounded-lg placeholder:text-xs placeholder:text-neutral-400 placeholder:pb-0.5">
          <button @click="send(message)" type="submit" title="Send message" class="absolute inset-y-0 right-0 flex items-center px-1 cursor-pointer text-neutral-400 hover:text-neutral-600 duration-300">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
              <path fill="currentColor" d="M5.133 18.02q-.406.163-.77-.066T4 17.288v-3.942L9.846 12L4 10.654V6.712q0-.438.364-.666t.77-.067l12.512 5.269q.49.225.49.756q0 .53-.49.748z"/>
            </svg>
          </button>
        </form>
        <div v-else class="relative w-full bg-neutral-100 border-2 border-neutral-300 px-2 pt-2 pb-1 rounded-lg">
          <div class="h-32 mb-2">
            <div ref="chatContainer" class="flex flex-col-reverse flex-1 gap-4 overflow-y-auto scroll-smooth duration-300 scrollbar-hide">
              <div v-if="sending" class="w-fit text-xs text-start text-neutral-400">Thinking...</div>
              <div v-for="(message, index) in [ ...conversation ].reverse()" :key="index" class="w-fit max-w-[60%] text-sm text-start" :class="message.type == 'model' ? 'mr-auto text-neutral-700' : 'ml-auto bg-neutral-200 px-3 py-1 rounded-md'">
                <template v-if="message.text">{{ message.text }}</template>
                <span v-else class="text-xs text-neutral-400">no response</span>
              </div>
            </div>
          </div>
          <form class="relative" @submit.prevent>
            <input v-model="message" maxlength="100" placeholder="Your message" class="w-full bg-neutral-100 border-t-2 border-neutral-200 text-sm px-2 pb-1.5 pt-3 placeholder:text-xs placeholder:text-neutral-400 placeholder:pb-0.5">
            <button @click="send(message)" type="submit" title="Send message" class="absolute inset-y-0 right-0 flex items-center px-1 pt-2 cursor-pointer text-neutral-400 hover:text-neutral-600 duration-300">
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                <path fill="currentColor" d="M5.133 18.02q-.406.163-.77-.066T4 17.288v-3.942L9.846 12L4 10.654V6.712q0-.438.364-.666t.77-.067l12.512 5.269q.49.225.49.756q0 .53-.49.748z"/>
              </svg>
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const message = ref("");
const sending = ref(false);
const conversation = ref([]);
const chatContainer = ref(null);

const send = async msg => {
  try {
    if (msg?.trim() === "") return;

    sending.value = true;
    conversation.value.push({ type: "user", text: msg, timestamp: new Date() });
    setTimeout(() => chatContainer.value.scrollIntoView(0), 100);
    message.value = "";

    const response = await fetch("https://api.algecom.com/v1/chat/test", {
      method: "POST",
      credentials: "include",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ 
        conversation: conversation.value,
        message: { text: msg } 
      }),
    });

    if (!response.ok) {
      console.error("Error:", response.statusText);
      return;
    }

    const data = await response.json();

    conversation.value.push({ type: "model", text: data.data.text, timestamp: new Date() });
    setTimeout(() => chatContainer.value.scrollIntoView(0), 100); 
  } catch (error) {
    console.log(error);
    alert(error.message || "Something went wrong");  
    conversation.value.pop();
    message.value = msg;
  } finally {
    sending.value = false;
  }
};
</script>

<style>
.fill-available {
  height: -webkit-fill-available;
  width: -webkit-fill-available;
}
</style>