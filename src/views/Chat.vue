<template>
  <!-- Chat interface -->
  <n-card class="chat">
    <n-spin :show="isPersonasLoading || isBackendLoading">
      <template #description>
        <span v-if="isPersonasLoading">Loading personas...</span>
        <span v-else-if="isBackendLoading">Connecting to backend...</span>
      </template>

      <!-- Backend status -->
      <div v-if="backendError" class="backend-error">
        {{ backendError }}
      </div>
      <div v-if="remainingRequests !== null" class="backend-status">
        <n-tag type="info"> Remaining requests: {{ remainingRequests }} / {{ maxRequests }} </n-tag>
      </div>

      <n-button
        ref="selectButtonRef"
        @click="showModal = true"
        class="emphasis-effect"
        style="width: 300px"
        :type="isInitialLoad ? 'info' : 'default'"
        >Select Persona: {{ persona }}</n-button
      >

      <!-- {{ short_persona }} -->
      <n-code :code="short_persona" word-wrap />
      <div class="chat-container">
        <div class="row">
          <div class="message purchase">
            <span v-if="purchase.product_name">
              This agent purchased
              <span class="tag">
                {{ purchase.product_name }}
              </span>
              at
              <span class="tag">
                {{ purchase.price }}
              </span>
              .
            </span>
            <span v-else>This agent did not purchase anything.</span>
            <br />
            <span v-if="purchase.title">
              This agent's action trace leading to the purchase includes:
            </span>
            <span v-else>This agent's action trace includes:</span>
            <n-highlight
              :text="formatted_action_trace"
              :patterns="[
                'clicking',
                'click',
                'adding',
                'add',
                'typing',
                'type',
                'submitting',
                'submit',
                'terminating',
                'terminate',
              ]"
              :highlight-style="{
                padding: '0 6px',
                borderRadius: themeVars.borderRadius,
                display: 'inline-block',
                color: 'rgb(24, 160, 88)',
                border: '1px solid rgba(24, 160, 88, 0.3)',
                background: ' rgba(24, 160, 88, 0.1)',
                transition: `all .3s ${themeVars.cubicBezierEaseInOut}`,
              }"
            ></n-highlight>
          </div>
        </div>
        <div class="row" v-for="message in messages" :key="message.id">
          <div class="space" v-if="message.role == 'user'"></div>
          <div :class="['message', message.role]" v-if="message.role !== 'system'">
            <div v-for="(item, index) in message.content" :key="index">
              <p v-if="item.type === 'text'">
                <span v-html="item.text"></span>
              </p>
              <img
                v-else-if="item.type === 'image'"
                :src="'data:' + item.source.media_type + ';base64,' + item.source.data"
                alt="Uploaded Image"
                class="image"
              />
            </div>
          </div>
          <div class="space" v-if="message.role == 'assistant'"></div>
        </div>
        <!-- Loading indicator -->
        <div class="row">
          <div v-if="isLoading" class="message assistant">
            <p>Assistant is typing...</p>
          </div>

          <div class="space"></div>
        </div>
      </div>
      <n-upload
        directory-dnd
        ref="uploadRef"
        v-model:file-list="fileList"
        multiple
        :max="5"
        list-type="image-card"
        accept="image/*"
        @change="handleImageChange"
      >
      </n-upload>
      <div class="input-container">
        <n-input-group>
          <n-input
            ref="chatInputRef"
            id="chat-input"
            size="large"
            v-model:value="userInput"
            :placeholder="typewriterPaused ? '' : typewriterText"
            :loading="isLoading"
            :disabled="!persona === undefined"
            type="textarea"
            @keydown="
              (e: KeyboardEvent) => {
                if ((e.ctrlKey || e.metaKey) && e.key === 'Enter') {
                  sendMessage()
                }
              }
            "
            @paste="handlePaste"
            @focus="handleInputFocus"
            @blur="handleInputBlur"
            @input="handleInputChange"
          />
          <n-button
            :loading="isLoading"
            type="primary"
            size="large"
            @click="sendMessage"
            style="height: 90px"
            :disabled="userInput.trim() === ''"
            >Send</n-button
          >
        </n-input-group>
      </div>
      This demo uses the gpt-4o model.
      <n-modal v-model:show="showModal">
        <n-card
          style="width: 600px"
          title="Select Persona"
          :bordered="false"
          size="huge"
          role="dialog"
          aria-modal="true"
        >
          <div class="row" style="height: 500px">
            <div
              style="
                flex-basis: 165px;
                flex-grow: 0;
                flex-shrink: 0;
                height: 100%;
                display: flex;
                flex-direction: column;
              "
            >
              <div>
                <n-input v-model:value="persona_search" placeholder="Search" />
              </div>
              <!-- <div style="flex: 1; overflow-y: auto" class="persona-list"> -->
              <!-- <div
                  v-for="p in Object.keys(all_personas).filter(
                    (p) => p.includes(persona_search) || persona_search === ''
                  )"
                  :key="p"
                  style="width: 100%"
                >
                  <n-button @click="persona = p" style="width: 100%">{{ p }}</n-button>
                </div> -->
              <n-virtual-list
                style="flex: 1; overflow-y: auto"
                class="persona-list"
                :item-size="34"
                key-field="value"
                :items="
                  Object.keys(all_personas)
                    .filter((p) => p.includes(persona_search) || persona_search === '')
                    .map((p) => ({
                      value: p,
                    }))
                "
              >
                <template #default="{ item }">
                  <div :key="'persona' + item.value" class="item" style="height: 34px">
                    <n-button
                      :type="persona === item.value ? 'primary' : 'default'"
                      @click="
                        () => {
                          persona = item.value
                          messages = []
                        }
                      "
                      style="width: 100%"
                    >
                      {{ parseInt(item.value) + 1 }}:
                      {{ all_personas[item.value].persona.split('\n')[0].split('Persona: ')[1] }}
                    </n-button>
                  </div>
                </template>
              </n-virtual-list>
            </div>
            <div style="flex: 1; overflow-y: auto; height: 100%">
              <!-- {{ all_personas[persona] }}
                 -->
              <n-code v-if="persona" :code="all_personas[persona].persona" word-wrap />
              <n-empty
                style="height: 100%; display: flex; justify-content: center; align-items: center"
                description="Select a persona to continue"
                v-else
              />
            </div>
          </div>
          <template #footer>
            <div class="row" style="justify-content: center">
              <n-button size="large" @click="showModal = false" type="primary" :disabled="!persona"
                >Confirm</n-button
              >
            </div>
          </template>
        </n-card>
      </n-modal>
    </n-spin>
  </n-card>
</template>

<script setup lang="tsx">
import { nextTick, ref, onMounted, watch, computed, onUnmounted } from 'vue'
import { useLocalStorage } from '@vueuse/core'
import { api_url } from '@/config'
import type { UploadFileInfo } from 'naive-ui'

import { useDialog, useThemeVars } from 'naive-ui'
const themeVars = useThemeVars()
const dialog = useDialog()
// Define proper types for messages and personas
interface MessageContent {
  type: string
  text?: string
  source?: {
    type: string
    media_type: string
    data: string
  }
}

interface Message {
  id: number
  role: string
  content: MessageContent[]
}

interface Persona {
  persona: string
  memory_trace: string
  action_trace: any[]
  result: {
    product_name?: string
    price?: string
    title?: string
  }
}

// Create a ref to hold the personas data
const all_personas = ref<Record<string, Persona>>({})
const isPersonasLoading = ref(true)

// Backend connection state using VueUse's localStorage
const sessionId = useLocalStorage('chat_session_id', '')
const solution = useLocalStorage('chat_solution', '')
const remainingRequests = ref(0)
const maxRequests = ref(5)
const backendError = ref('')
const isBackendLoading = ref(false)

// Add system prompt for the AI assistant
const systemPrompt = ref(`<IMPORTANT>
You are a participant who just participated in a user study. <Your persona> is given below.  In the study, you were tested to interact with a version of the website design of an online shopping platform like Amazon.com. You used the website for <Your intent>. <Your memory trace> is also given below, which contains your <observation>, your <thought>, your <reasoning/reflection>, and your <actions>.  Now you are interviewed by the website designer to talk about your user experience and feedback on the website design. You will answer based on <your persona> and <Your memory trace>.
</IMPORTANT>

<Your persona>: {persona}

<Your memory trace>: {memory_trace}

<style>: You should talk using a verbal dialog style.   No formal structure no formal language. No written language style. No bullet point. If you have multiple points to make, bring only the top one in a conversation way. Very important is to keep your response succinct and short with maybe less than 3 or 4 sentences.</style>`)

// Load personas asynchronously
onMounted(async () => {
  try {
    // Dynamic import of the personas JSON file
    const personasModule = await import('@/data/personas.json')
    all_personas.value = personasModule.default
    isPersonasLoading.value = false

    // Initialize backend connection
    await initializeBackendConnection()
  } catch (error) {
    console.error('Failed to load personas:', error)
    isPersonasLoading.value = false
  }
})

// Initialize backend connection
async function initializeBackendConnection() {
  // Check if we have a valid session in localStorage
  if (sessionId.value && solution.value) {
    // We have stored credentials, check if they're still valid
    try {
      const sessionInfo = await getSessionInfo(sessionId.value)
      remainingRequests.value = sessionInfo.remaining_requests
      maxRequests.value = sessionInfo.max_requests

      if (remainingRequests.value <= 0) {
        backendError.value = 'This is only a demo. To explore more, you can run our code.'
      }
    } catch (error) {
      console.error('Stored session invalid:', error)
      // If session is invalid, create a new one
      await createNewSession()
    }
  } else {
    // No stored session, create a new one
    await createNewSession()
  }
}

// Create a new session and solve the challenge
async function createNewSession() {
  isBackendLoading.value = true
  backendError.value = ''

  try {
    // Create session
    const session = await createSession()
    console.log('Session created:', session)

    // Solve challenge
    const challengeSolution = await solveChallenge(session.challenge, session.difficulty)
    console.log('Challenge solved:', challengeSolution)

    // Store session info (automatically persisted by useLocalStorage)
    sessionId.value = session.session_id
    solution.value = challengeSolution

    // Get session info
    const sessionInfo = await getSessionInfo(sessionId.value)
    remainingRequests.value = sessionInfo.remaining_requests
    maxRequests.value = sessionInfo.max_requests
  } catch (error) {
    console.error('Failed to create session:', error)
    backendError.value = 'Failed to connect to backend. This is a demo version.'
  } finally {
    isBackendLoading.value = false
  }
}

// API functions
async function createSession() {
  const response = await fetch(`${api_url}/api/create_session`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
  })

  if (!response.ok) {
    throw new Error(`Error: ${response.status}`)
  }

  return await response.json()
}

async function getSessionInfo(sessionId: string) {
  const response = await fetch(`${api_url}/api/session_info?session_id=${sessionId}`, {
    method: 'GET',
    headers: { 'Content-Type': 'application/json' },
  })

  if (!response.ok) {
    throw new Error(`Error: ${response.status}`)
  }

  return await response.json()
}

async function sendChatMessage(messages: any[]) {
  if (remainingRequests.value <= 0) {
    return { message: 'This is only a demo. To explore more, you can run our code.' }
  }

  const response = await fetch(`${api_url}/api/openai`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      session_id: sessionId.value,
      solution: solution.value,
      messages: messages,
    }),
  })

  if (!response.ok) {
    if (response.status === 429) {
      remainingRequests.value = 0
      return { message: 'This is only a demo. To explore more, you can run our code.' }
    }
    throw new Error(`Error: ${response.status}`)
  }

  // Update remaining requests
  try {
    const sessionInfo = await getSessionInfo(sessionId.value)
    remainingRequests.value = sessionInfo.remaining_requests
  } catch (error) {
    console.error('Failed to update session info:', error)
  }

  return await response.json()
}

// Challenge solving function
async function solveChallenge(challenge: string, difficulty: number) {
  const leadingZeros = '0'.repeat(difficulty)

  while (true) {
    // Generate a random suffix
    const randomSuffix = Math.random().toString(36).substring(2, 15)
    const attemptedSolution = challenge + randomSuffix

    // Calculate SHA-256 hash
    const msgBuffer = new TextEncoder().encode(attemptedSolution)
    const hashBuffer = await crypto.subtle.digest('SHA-256', msgBuffer)
    const hashArray = Array.from(new Uint8Array(hashBuffer))
    const hashHex = hashArray.map((b) => b.toString(16).padStart(2, '0')).join('')

    // Check if hash has required leading zeros
    if (hashHex.startsWith(leadingZeros)) {
      return attemptedSolution
    }
  }
}

const persona = ref<number>(0)
const action_trace = ref<any[]>([])
const purchase = ref<{
  product_name?: string
  price?: string
  title?: string
}>({})
const formatted_action_trace = computed(() => {
  if (!action_trace.value || !Array.isArray(action_trace.value)) return ''
  let result = ''
  let actions = action_trace.value
  // flatten actions
  actions = actions.flat()
  console.log(actions)
  for (const action of actions) {
    result += action.description + '; '
  }
  return result
})

// Update watch to handle async loading
watch(
  [persona, all_personas],
  () => {
    if (all_personas.value[persona.value]) {
      action_trace.value = all_personas.value[persona.value].action_trace
      purchase.value = all_personas.value[persona.value].result
    }
  },
  { immediate: true },
)

const short_persona = computed(() => {
  if (!all_personas.value[persona.value]) return ''
  // from background to financial situation
  let persona_text = all_personas.value[persona.value]['persona']
  persona_text = persona_text.split('Background:')[1].split('Financial Situation:')[0]
  return persona_text.replaceAll('\n\n', '\n').trim()
})
const showModal = ref(false)
const persona_search = ref('')

const uploadRef = ref()
// Reactive variables
const userInput = ref('')
const messages = ref<Message[]>([])
let appendedPrompt = ''

// Loading state
const isLoading = ref(false)

// File list for the upload component
const fileList = ref<UploadFileInfo[]>([])

// Typewriter effect variables
const typewriterText = ref('')
const typewriterActive = ref(true)
const typewriterInterval = ref<number | null>(null)
const placeholderTexts = [
  'Can you tell me more about your shopping experience?',
  'What did you like about the website?',
]
const currentPlaceholderIndex = ref(0)
const isTyping = ref(false)
const isDeleting = ref(false)
const typewriterPaused = ref(false)

// Add these refs for the UI effects
const selectButtonRef = ref(null)
const chatInputRef = ref(null)
const isInitialLoad = ref(true)

// Initialize typewriter effect
onMounted(() => {
  startTypewriterEffect()
})

// Start the typewriter effect
function startTypewriterEffect() {
  if (!typewriterActive.value) return

  clearInterval(typewriterInterval.value)

  typewriterInterval.value = setInterval(() => {
    if (typewriterPaused.value) return

    const currentText = placeholderTexts[currentPlaceholderIndex.value]

    if (isDeleting.value) {
      typewriterText.value = currentText.substring(0, typewriterText.value.length - 1)

      if (typewriterText.value === '') {
        isDeleting.value = false
        currentPlaceholderIndex.value =
          (currentPlaceholderIndex.value + 1) % placeholderTexts.length
        setTimeout(() => {
          isTyping.value = true
        }, 700) // Pause before typing next text
      }
    } else {
      typewriterText.value = currentText.substring(0, typewriterText.value.length + 1)

      if (typewriterText.value === currentText) {
        isTyping.value = false
        setTimeout(() => {
          isDeleting.value = true
        }, 2000) // Wait before starting to delete
      }
    }
  }, 70) // Speed of typing/deleting
}

// Handle focus event
function handleInputFocus() {
  typewriterText.value = ''
  typewriterPaused.value = true
}

// Handle blur event
function handleInputBlur() {
  if (userInput.value.trim() === '') {
    typewriterPaused.value = false
  }
}

// Handle input event
function handleInputChange() {
  if (userInput.value.trim() !== '') {
    typewriterPaused.value = true
  } else if (!document.activeElement || document.activeElement.id !== 'chat-input') {
    typewriterPaused.value = false
  }
}

// Clean up on component unmount
onUnmounted(() => {
  clearInterval(typewriterInterval.value)
})

// Fix the event parameter type in handleFileUpload
const handleFileUpload = (event: { file: { file: File } }) => {
  console.log(event)
  const file = event.file.file
  if (file) {
    const reader = new FileReader()
    reader.onload = (e) => {
      appendedPrompt = e.target?.result as string
      console.log(appendedPrompt)
    }
    reader.readAsText(file)
  }
}

import { debounce } from 'lodash'

// Process image file
const processImageFile = async (file: File) => {
  try {
    console.log('Processing image file:', file.name, file.type, file.size)

    // Check if file is valid
    if (!file || file.size === 0) {
      console.error('Invalid file object or empty file')
      return
    }

    // Create a unique ID for the file
    const fileId = Date.now().toString()

    // Add to the upload component's file list
    fileList.value.push({
      id: fileId,
      name: file.name || `pasted-image-${fileId}.png`,
      file: file,
      status: 'finished',
      type: file.type || 'image/png',
      percentage: 100,
    })
  } catch (error) {
    console.error('Error processing pasted image:', error)
  }
}

// Update the handleImageChange function to work with the new approach
const handleImageChange = debounce((options: { fileList: UploadFileInfo[] }) => {
  console.log('Image change event:', options.fileList)
  // Update the file list
  fileList.value = options.fileList
}, 10)

// Fix readFileAsDataURL function with proper type
function readFileAsDataURL(file: File): Promise<string> {
  return new Promise((resolve, reject) => {
    if (!file) {
      reject(new Error('No file provided'))
      return
    }

    const reader = new FileReader()

    reader.onload = () => {
      if (reader.result) {
        resolve(reader.result.toString())
      } else {
        reject(new Error('FileReader result is null'))
      }
    }

    reader.onerror = (event) => {
      console.error('FileReader error:', event)
      reject(new Error('Error reading file'))
    }

    try {
      reader.readAsDataURL(file)
    } catch (error) {
      console.error('Error calling readAsDataURL:', error)
      reject(error)
    }
  })
}

// Handle paste event for images
const handlePaste = (event: ClipboardEvent) => {
  console.log('Paste event triggered')

  // Check if clipboardData is available
  if (!event.clipboardData) {
    console.log('No clipboard data available')
    return
  }

  // Log all available formats
  console.log('Available formats:', Array.from(event.clipboardData.types))

  const items = event.clipboardData.items
  console.log('Clipboard items:', items, 'Length:', items?.length)

  // Try to access the clipboard items
  if (items && items.length > 0) {
    for (let i = 0; i < items.length; i++) {
      const item = items[i]
      console.log(`Item ${i}:`, item.kind, item.type)

      // Check if the pasted content is an image
      if (item.kind === 'file' && item.type.indexOf('image') !== -1) {
        // Prevent the default paste behavior for images
        event.preventDefault()

        // Get the image as a file
        const file = item.getAsFile()
        console.log('Image file:', file)

        if (!file) {
          console.log('Failed to get file from clipboard item')
          continue
        }

        // Process the image
        processImageFile(file)
      }
    }
  } else {
    // Try alternative method for some browsers
    const files = event.clipboardData.files
    console.log('Clipboard files:', files, 'Length:', files?.length)

    if (files && files.length > 0) {
      for (let i = 0; i < files.length; i++) {
        const file = files[i]
        console.log(`File ${i}:`, file.type, file.name)

        if (file.type.indexOf('image') !== -1) {
          event.preventDefault()
          processImageFile(file)
        }
      }
    }
  }
}

// Send message - updated to use fileList directly
const sendMessage = async () => {
  if (userInput.value.trim() !== '' || fileList.value.length > 0) {
    // Construct the message content
    const messageContent = []

    // Process images from fileList
    if (fileList.value.length > 0) {
      for (const fileInfo of fileList.value) {
        if (fileInfo.file) {
          try {
            // Read the file as base64
            const base64Data = await readFileAsDataURL(fileInfo.file)
            const base64Content = base64Data.split(',')[1]

            if (base64Content) {
              messageContent.push({
                type: 'image',
                source: {
                  type: 'base64',
                  media_type: fileInfo.type || 'image/png',
                  data: base64Content,
                },
              })
            }
          } catch (error) {
            console.error('Error processing file for message:', error)
          }
        }
      }

      // Clear the file list after processing
      fileList.value = []
    }

    if (userInput.value.trim() !== '') {
      messageContent.push({
        type: 'text',
        text: userInput.value.trim(),
      })
    }

    // Add user message to chat
    messages.value.push({
      id: Date.now(),
      role: 'user',
      content: messageContent,
    })

    // Clear input and set loading
    const userMessageText = userInput.value
    userInput.value = ''
    isLoading.value = true

    try {
      // Prepare messages for API
      const apiMessages = [
        {
          role: 'system',
          content: [
            {
              type: 'text',
              text: systemPrompt.value
                .replace('{persona}', all_personas.value[persona.value]?.persona || '')
                .replace('{memory_trace}', all_personas.value[persona.value]?.memory_trace || ''),
            },
          ],
          id: 'system_1',
        },
        ...messages.value.map((msg) => ({
          role: msg.role,
          content: msg.content,
          id: msg.id.toString(),
        })),
      ]

      // Check if we have remaining requests
      if (remainingRequests.value <= 0) {
        // Add demo response if no requests left
        const msg =
          "To explore more, check out our <a href='https://github.com/neuhai/uxagent' style='color: #007bff; text-decoration: underline;' target='_blank'>GitHub repository</a>. If you are interested in, please contact us at <a href='mailto:lu.yuxuan@northeastern.edu' style='color: #007bff; text-decoration: underline;' target='_blank'>lu.yuxuan@northeastern.edu</a>!"
        messages.value.push({
          id: Date.now() + 1,
          role: 'assistant',
          content: [
            {
              type: 'text',
              text: msg,
            },
          ],
        })
        dialog.info({
          title: 'Thanks for your interest!',
          content: () => <span v-html={msg} />,
        })
      } else {
        // Send to backend
        const response = await sendChatMessage(apiMessages)

        // Add the assistant's reply to the chat
        messages.value.push({
          id: Date.now() + 1,
          role: 'assistant',
          content: [
            {
              type: 'text',
              text: response.message,
            },
          ],
        })
      }
    } catch (error) {
      console.error('Error:', error)
      // Add error message
      messages.value.push({
        id: Date.now() + 1,
        role: 'assistant',
        content: [
          {
            type: 'text',
            text: 'Sorry, there was an error connecting to the backend. This is a demo version.',
          },
        ],
      })
    } finally {
      isLoading.value = false

      // Scroll to bottom
      nextTick(() => {
        document.querySelector('.chat-container')?.scrollTo({
          top: document.querySelector('.chat-container')?.scrollHeight,
          behavior: 'smooth',
        })
      })
    }
  }
}

// Update the showModal watcher to handle button type changes
watch(showModal, (newValue) => {
  if (newValue === true) {
    // Modal is opening, remove emphasis effect and change button type
    if (selectButtonRef.value && isInitialLoad.value) {
      // Remove primary type after first click
      nextTick(() => {
        const button = selectButtonRef.value.$el
        button.classList.remove('emphasis-effect')
      })
      isInitialLoad.value = false
    }
  } else if (newValue === false && persona.value) {
    // Modal is closing and persona is selected, scroll to input
    nextTick(() => {
      if (chatInputRef.value) {
        chatInputRef.value.focus()

        // Scroll to the input
        const inputElement = document.getElementById('chat-input')
        if (inputElement) {
          inputElement.scrollIntoView({ behavior: 'smooth', block: 'end' })
        }
      }
    })
  }
})
</script>

<style scoped lang="scss">
.chat {
  max-width: 800px;
  min-height: 1000px;
  :deep(.n-card__content) {
    display: flex;
    flex: 1;
    flex-direction: column;
  }
  :deep(.n-spin-container) {
    display: flex;
    flex: 1;
    flex-direction: column;
  }
  :deep(.n-spin-content) {
    display: flex;
    flex: 1;
    flex-direction: column;
  }
}
/* No styles here, styles are in main.css and base.css */
.row {
  display: flex;
  flex-direction: row;
}
.space {
  flex-grow: 1;
  flex-shrink: 1;
}
.message {
  flex-shrink: 0;
  max-width: 100%;
}
.message:has(.n-input) {
  width: 100%;
}
.persona-list {
  overflow-y: auto;
  .n-button {
    width: 100%;
    border-radius: 0;
  }
}

/* Chat styles */
.chat-container {
  flex: 1;
  border: 1px solid var(--color-border);
  padding: 15px;
  overflow-y: auto;
  min-height: 0;
  margin: 20px 0;
  background-color: #f9f9f9;
  border-radius: 8px;
}

.message {
  margin-bottom: 15px;
  padding: 10px;
  border-radius: 5px;
  color: var(--color-text); // Ensure text color is set

  p {
    margin: 0;
  }

  &.system {
    background-color: #e9ecef;
    color: #333333;
  }

  &.user {
    background-color: rgb(225, 255, 204);
  }

  &.purchase {
    background-color: #eaeaea; /* New color */
    text-align: left;
    width: 100%;
  }
  &.action {
    background-color: #e8e1ff; /* New color */
    width: 100%;
  }

  &.assistant {
    background-color: rgb(207, 247, 255);
    text-align: left;
  }

  .image {
    max-width: 400px;
    max-height: 400px;
    height: auto;
    border-radius: 5px;
    margin-top: 5px;
  }
}

.input-container {
  display: flex;
  gap: 10px;
  margin-top: 10px;
  align-items: center;

  input[type='file'] {
    margin-top: 10px;
  }
}
.tag {
  background-color: rgba(32, 128, 240, 0.1); /* Softer blue background */
  color: rgb(32, 128, 240);
  border: 1px solid rgba(32, 128, 240, 0.3);
  padding: 2px 4px; /* Add padding for better spacing */
  border-radius: 3px; /* More rounded corners for a modern look */
  // font-weight: bold; /* Bold text for emphasis */
}

/* Add these styles */
.backend-error {
  background-color: #fff2f0;
  border: 1px solid #ffccc7;
  color: #ff4d4f;
  padding: 8px 12px;
  border-radius: 4px;
  margin-bottom: 16px;
}

.backend-status {
  margin-bottom: 16px;
}
.input-container {
  /* Add a style for the placeholder */
  :deep(.n-input__placeholder) {
    color: #000000 !important;
    opacity: 1 !important;
  }
}

/* Add emphasis effect animation */
@keyframes emphasis-pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

.emphasis-effect {
  animation: emphasis-pulse 1.5s infinite ease-in-out;
  transition: all 0.3s;
}
</style>
