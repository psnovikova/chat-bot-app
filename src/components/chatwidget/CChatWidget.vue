<script setup lang="ts">
import { ChatBubbleLeftIcon, PaperAirplaneIcon, XMarkIcon } from '@heroicons/vue/24/solid'
import { computed, nextTick, ref, watch } from 'vue'
import { Collapse } from 'vue-collapsed'

interface Suggestion {
  text: string
  userText: string
  answer: string
}

const messages = ref([
  {
    text: 'Привет! Что я могу для вас сделать?',
    isBot: true,
  },
])

const suggestions = ref<Suggestion[]>([
  {
    text: 'Рассказать анекдот',
    userText: 'Расскажи анекдот',
    answer: 'Анекдот: \n Товарищи кремированного мужика решили его скурить и сделали блок Кента',
  },
  {
    text: 'Рассказать о себе',
    userText: 'Расскажи о себе',
    answer: 'Я бот, который умеет рассказывать анекдот про кента, а ещё меня написал человек, которого нужно взять на работу',
  },
  {
    text: 'Отключить будильник',
    userText: 'Отключи будильник',
    answer: 'Ладно',
  },
])

const inputValue = ref('')
const chatContainer = ref<HTMLElement>()

const isChatOpened = ref(false)
const isVisibleChatIcon = ref(true)

const isSuggestionSelected = ref(false)
const isSendingUserMessage = ref(false)
const isSendingBotMessage = ref(false)

const isDidablesSendButton = computed(() => {
  return isSendingUserMessage.value || isSendingBotMessage.value || !inputValue.value
})

function openChat() {
  isChatOpened.value = true
  isVisibleChatIcon.value = false
}

function closeChat() {
  isChatOpened.value = false
  setTimeout(() => {
    isVisibleChatIcon.value = true
  }, 1000)
}

async function sendMessage() {
  if (inputValue.value) {
    if (!isSuggestionSelected.value)
      isSuggestionSelected.value = true

    isSendingUserMessage.value = true
    await new Promise((resolve) => {
      setTimeout(() => {
        isSendingUserMessage.value = false
        resolve(messages.value.push({
          text: inputValue.value,
          isBot: false,
        }))
      }, 1000)
    })
    inputValue.value = ''
    await generateBotMessage()
  }
}

async function generateBotMessage() {
  isSendingBotMessage.value = true
  await new Promise((resolve) => {
    setTimeout(() => {
      isSendingBotMessage.value = false
      resolve(messages.value.push({
        text: 'Я устал, хватит мне писать',
        isBot: true,
      }))
    }, 1000)
  })
}

async function selectSuggestion(suggestion: Suggestion) {
  isSuggestionSelected.value = true
  isSendingUserMessage.value = true
  await new Promise((resolve) => {
    setTimeout(() => {
      isSendingUserMessage.value = false
      resolve(messages.value.push({
        text: suggestion.userText,
        isBot: false,
      }))
    }, 1000)
  })
  isSendingBotMessage.value = true
  await new Promise((resolve) => {
    setTimeout(() => {
      isSendingBotMessage.value = false
      resolve(messages.value.push({
        text: suggestion.answer,
        isBot: true,
      }))
    }, 1000)
  })
}

function scrollToBottom() {
  if (chatContainer.value)
    chatContainer.value.scrollTop = chatContainer.value.scrollHeight
}

watch(messages, async () => {
  await nextTick()
  scrollToBottom()
}, { deep: true })

watch(isSendingBotMessage, async () => {
  await nextTick()
  scrollToBottom()
})

watch(isSendingUserMessage, async () => {
  await nextTick()
  scrollToBottom()
})
</script>

<template>
  <div>
    <div class="relative flex">
      <Collapse
        :when="isChatOpened"
        class="v-collapse"
      >
        <div class="w-[22rem] h-[32.5rem] flex flex-col rounded-lg overflow-hidden shadow-2xl z-10">
          <header class="w-full h-fit bg-purple-800 text-white px-4 py-2 border-b-2 border-white flex flex-row justify-between items-center">
            <h1 class="text-xl">
              Chat Widget
            </h1>
            <button @click="closeChat">
              <XMarkIcon class="w-8 h-8" />
            </button>
          </header>
          <div
            ref="chatContainer"
            class="w-full h-full bg-gray-50 overflow-auto flex flex-col space-y-3 px-2 py-4"
          >
            <div
              v-for="(message, index) in messages"
              :key="index"
              class="w-full h-fit flex flex-row space-x-2 items-end"
            >
              <div
                v-if="message.isBot"
                class="w-10 h-10 bg-white text-2xl rounded-full rounded-br-none border border-stone-300 flex items-center justify-center shrink-0"
              >
                🤖
              </div>
              <div
                class="max-w-[12.5rem] break-words flex-wrap text-white rounded-lg p-2"
                :class="{ 'bg-purple-100 text-black ml-auto rounded-br-none': !message.isBot,
                          'bg-purple-800 text-white mr-auto rounded-bl-none': message.isBot }"
              >
                {{ message.text }}
              </div>
              <div
                v-if="!message.isBot"
                class="w-10 h-10 bg-white text-2xl rounded-full rounded-bl-none border border-stone-300 flex items-center justify-center shrink-0"
              >
                👤
              </div>
            </div>
            <div
              v-if="!isSuggestionSelected"
              class="w-full h-fit flex flex-col space-y-2"
            >
              <button
                v-for="(suggestion, index) in suggestions"
                :key="index"
                class="bg-emerald-700 hover:bg-emerald-800 text-white rounded-lg p-2 mx-8"
                @click="selectSuggestion(suggestion)"
              >
                {{ suggestion.text }}
              </button>
            </div>
            <div v-if="isSendingBotMessage">
              <div class="w-full h-fit flex flex-row space-x-2 items-end">
                <div class="w-10 h-10 bg-white text-2xl rounded-full rounded-br-none border border-stone-300 flex items-center justify-center shrink-0">
                  🤖
                </div>
                <div class="text-white rounded-lg p-2 bg-purple-800 mr-auto">
                  <span class="dot bg-white" />
                  <span class="dot bg-white" />
                  <span class="dot bg-white" />
                </div>
              </div>
            </div>
            <div
              v-if="isSendingUserMessage"
              class="flex flex-row space-x-2 items-end"
            >
              <div
                class="text-white rounded-lg p-2 bg-purple-100 ml-auto"
              >
                <span class="dot bg-stone-900" />
                <span class="dot bg-stone-900" />
                <span class="dot bg-stone-900 " />
              </div>
              <div class="w-10 h-10 bg-white text-2xl rounded-full rounded-bl-none border border-stone-300 flex items-center justify-center shrink-0">
                👤
              </div>
            </div>
          </div>
          <div class="bg-white flex flex-row">
            <input
              v-model="inputValue"
              placeholder="Type the message..."
              class="w-full h-full outline-none px-2"
              @keyup.enter="sendMessage"
            >
            <button
              class="px-2 py-3 text-gray-700 disabled:text-gray-300 disabled:cursor-default"
              :disabled="isDidablesSendButton"
              @click="sendMessage"
            >
              <PaperAirplaneIcon class="h-6 w-6" />
            </button>
          </div>
        </div>
      </Collapse>
      <button
        v-if="isVisibleChatIcon"
        class="absolute bottom-0 right-0 w-16 h-16 bg-purple-800 hover:bg-purple-700 text-2xl rounded-full flex items-center justify-center shrink-0"
        @click="openChat"
      >
        <ChatBubbleLeftIcon class="text-white stroke-purple-950 h-8 w-8 shrink-0" />
      </button>
    </div>
  </div>
</template>

<style scoped>
.v-collapse { transition: height 1000ms cubic-bezier(0.33, 1, 0.68, 1) }

@keyframes blink {
  0% {
    opacity: 0.3;
  }
  50% {
    opacity: 1;
  }
  100% {
    opacity: 0.3;
  }
}

.dot {
  @apply inline-block w-2 h-2 rounded-full mr-1;
  animation: blink 1.5s infinite;
}
.dot:nth-child(1) {
  animation-delay: 0s;
}
.dot:nth-child(2) {
  animation-delay: 0.5s;
}
.dot:nth-child(3) {
  animation-delay: 1s;
}
</style>
