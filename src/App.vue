<template>
  <div class="container mx-auto py-4">
    <div class="flex flex-col">
      <h1 class="text-2xl font-bold mb-4">Приватный чат</h1>
      <div class="mb-4 flex items-center justify-between">
        <div>
          <label class="mr-2">Выберите пользователя:</label>
          <select v-model="currentUser" class="border p-2 rounded">
            <option value="user1">Пользователь 1</option>
            <option value="user2">Пользователь 2</option>
          </select>
        </div>
        <button @click="handleClearMessages" class="bg-red-500 text-white px-4 py-2 rounded mt-4">
          Очистить чат
        </button>
      </div>
    </div>
    <ChatComponent
        ref="chatComponent"
        :token="currentToken"
        :current-user-id="currentUserId"
    />
  </div>
</template>

<script setup lang="ts">
import ChatComponent from "@/components/ChatComponent.vue";
import {computed, ref} from "vue";

const chatComponent = ref<InstanceType<typeof ChatComponent> | null>(null);

const currentUser = ref('user1');
const tokens = {
  user1: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjozMywicm9sZSI6ImN1c3RvbWVyIiwiaWF0IjoxNzI1OTQ2MTY2LCJleHAiOjE3MjYwNzU3NjYsImp0aSI6IjBlOWU4YjkzOGQxNjQ4MzI4ODZmYjAyNmM0ODliZTUwIiwidG9rZW5fdHlwZSI6ImFjY2VzcyJ9.WPPrAOfMnFXdkP7pL9XMcKa8pH9P06IUc4bLDQGqW6A',
  user2: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjozNCwicm9sZSI6ImN1c3RvbWVyIiwiaWF0IjoxNzI1OTQ2MTg3LCJleHAiOjE3MjYwNzU3ODcsImp0aSI6ImEzMGUwZjlmOTYzYjQ5MDA4ZGQ1MjUwMjE3MWM4ZTZjIiwidG9rZW5fdHlwZSI6ImFjY2VzcyJ9.eyYJV4Ek5pL5Fw3A9TpgIi9NGitUCsymwkenAoImW-0'
};

const currentToken = computed(() => tokens[currentUser.value as keyof typeof tokens]);
const currentUserId = computed(() => currentUser.value === 'user1' ? 33 : 34);

const handleClearMessages = () => {
  chatComponent.value?.clearMessages()
}
</script>

<style scoped>
</style>
