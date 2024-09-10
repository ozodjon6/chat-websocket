<template>
  <div class="flex flex-col h-[600px] border rounded-lg">
    <div class="flex-grow h-full overflow-y-auto p-4" ref="chatContainer">
      <span class="flex items-center justify-center h-full text-gray-500 text-2xl" v-if="!messages?.length">
        Нет сообщения
      </span>
      <div v-for="message in messages" :key="message.id" class="mb-4">
        <div
            class="flex items-center justify-between max-w-max"
            :class="[
          'p-2 px-3 rounded-2xl',
          message.sender_id === currentUserId
            ? 'bg-blue-500 text-white ml-auto'
            : 'bg-gray-200 text-black'
        ]">
          <span>{{ message.message }}</span>
          <span class="text-xs ml-2 pt-0.5">{{ message.timestamp }}</span>
        </div>
      </div>
    </div>
    <div class="p-4 border-t">
      <form @submit.prevent="sendMessage" class="flex">
        <input v-model="newMessage" type="text" placeholder="Введите сообщение..." class="flex-grow mr-2 p-2 border rounded" />
        <button type="submit" class="bg-blue-500 text-white px-4 py-2 rounded">Отправить</button>
      </form>
    </div>
    <div class="p-2 bg-gray-100">
      <div class="flex items-center gap-2">
        <span>Статус соединения:</span>
        <p class="flex items-center gap-2">
          {{ connectionStatus }}
          <i
              class="inline-block w-3 pt-0.5 h-3 rounded-full"
              :class="isError ? 'bg-red-700' : 'bg-green-700'"
          ></i>
        </p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {ref, onMounted, onUnmounted, watch, nextTick} from 'vue';

interface Message {
  id: number;
  message: string;
  sender_id: number;
  timestamp: string;
}

interface Props {
  token: string;
  currentUserId: number;
}

const props = defineProps<Props>();

const messages = ref<Message[]>([]);
const newMessage = ref('');
const socket = ref<WebSocket | null>(null);
const chatContainer = ref<HTMLElement | null>(null);
const connectionStatus = ref('Отключено');

const clearMessages = () => {
  messages.value = [];
  localStorage.removeItem("")
};

// Загрузка сообщений из localStorage
const loadMessagesFromLocalStorage = () => {
  const savedMessages = localStorage.getItem('chatMessages');
  if (savedMessages) {
    messages.value = JSON.parse(savedMessages);
  }
};

// Сохранение сообщений в localStorage
const saveMessagesToLocalStorage = () => {
  localStorage.setItem('chatMessages', JSON.stringify(messages.value));
};

// Синхронизация сообщений между вкладками
window.addEventListener('storage', (event) => {
  if (event.key === 'chatMessages') {
    const updatedMessages = localStorage.getItem('chatMessages');
    if (updatedMessages) {
      messages.value = JSON.parse(updatedMessages);
    }
  }
});

const isError = ref(false)

const connectWebSocket = () => {
  if (socket.value) {
    socket.value.close();
  }

  connectionStatus.value = 'Подключение...';
  socket.value = new WebSocket(`ws://5.182.26.58:4321/ws/web?token=${props.token}`);

  socket.value.onopen = () => {
    console.log('WebSocket соединение установлено');
    connectionStatus.value = 'Подключено';
  };

  socket.value.onmessage = (event) => {
    console.log('Received message:', event.data);
    try {
      const data = JSON.parse(event.data);
      if (data.action === 'new_message') {
        console.log('Adding new message to messages array:', data.payload);
        messages.value.push(data.payload);
        saveMessagesToLocalStorage();  // Сохранение сообщений
        nextTick(() => scrollToBottom());
      } else {
        console.log('Received message with unknown action:', data.action);
      }
    } catch (error) {
      console.error('Error parsing message:', error);
    }
  };

  socket.value.onerror = (error) => {
    console.error('WebSocket ошибка:', error);
    connectionStatus.value = 'Ошибка соединения';
  };

  socket.value.onclose = (event) => {
    console.log('WebSocket соединение закрыто:', event.code, event.reason);
    connectionStatus.value = 'Отключено';
    isError.value = true
    setTimeout(connectWebSocket, 10000); // Попытка переподключения через 10 секунды
  };
};

const getCurrentTimeUTC = () => {
  const now = new Date();
  const utcHours = now.getHours().toString().padStart(2, '0');
  const utcMinutes = now.getMinutes().toString().padStart(2, '0');
  return `${utcHours}:${utcMinutes}`;
};

const sendMessage = () => {
  if (socket.value && socket.value.readyState === WebSocket.OPEN && newMessage.value.trim()) {
    const message = {
      action: 'send_message',
      payload: {
        chat_room_id: 1,
        message: newMessage.value,
        reply_message: null
      }
    };

    const currentTime = getCurrentTimeUTC();
    console.log('Current UTC Time:', getCurrentTimeUTC());

    try {
      socket.value.send(JSON.stringify(message));
      console.log('Message sent:', message);
      // Добавляем отправленное сообщение в локальный массив
      messages.value.push({
        id: Date.now(), // Временный ID
        message: newMessage.value,
        sender_id: props.currentUserId,
        timestamp: currentTime
      });
      newMessage.value = '';
      console.log('Messages after adding:', messages.value);
      scrollToBottom();
      saveMessagesToLocalStorage();
    } catch (error) {
      console.error('Error sending message:', error);
    }
  } else {
    console.log('Cannot send message. Socket status:', socket.value?.readyState);
  }
};

const scrollToBottom = () => {
  nextTick(() => {
    if (chatContainer.value) {
      chatContainer.value.scrollTop = chatContainer.value.scrollHeight;
    }
  });
};

onMounted(() => {
  loadMessagesFromLocalStorage()
  connectWebSocket();
});

onUnmounted(() => {
  if (socket.value) {
    socket.value.close();
  }
});

watch(messages, saveMessagesToLocalStorage, { deep: true });
defineExpose({clearMessages})
</script>
