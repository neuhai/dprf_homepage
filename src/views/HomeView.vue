<script setup lang="ts">
import Author from '@/components/Author.vue'
import Chat from '@/views/Chat.vue'
import { ref, computed } from 'vue'
import { useDialog } from 'naive-ui'
import dprfExampleData from '@/data/dprf_example.json'
import fullExampleData from '../../example_1014.json'

// Import author images
import yuxuanImg from '@/assets/yuxuan.jpg'
import dakuoImg from '@/assets/dakuo.jpg'
import bingshengImg from '@/assets/bingsheng.jpg'
import yuanzheImg from '@/assets/yuanzhe.jpeg'
import boImg from '@/assets/bo.jpg'

const dialog = useDialog()

const citationText = ref<HTMLPreElement | null>(null)
const isCopied = ref(false)
const showMessage = ref(false)

// Persona iteration viewer
const currentPersonaIndex = ref(0)
const personas = dprfExampleData.personas
const totalPersonas = personas.length
const viewType = ref<'persona' | 'response' | 'analysis'>('persona')

const MAX_CONTENT_LENGTH = 1500 // Maximum characters to display

const currentPersona = computed(() => personas[currentPersonaIndex.value])
const previousPersona = computed(() => 
  currentPersonaIndex.value > 0 ? personas[currentPersonaIndex.value - 1].persona : null
)

const goToPersona = (index: number) => {
  if (index >= 0 && index < totalPersonas) {
    currentPersonaIndex.value = index
  }
}

const nextPersona = () => goToPersona(currentPersonaIndex.value + 1)
const prevPersona = () => goToPersona(currentPersonaIndex.value - 1)

// Get content based on view type
const getCurrentContent = computed(() => {
  const iterationData = (fullExampleData as any).dprf_iteration_details?.[currentPersonaIndex.value]
  
  if (viewType.value === 'response') {
    const response = iterationData?.generated_response || ''
    return truncateText(response)
  } else if (viewType.value === 'analysis') {
    const analysis = iterationData?.analysis || ''
    return truncateText(analysis)
  } else {
    return currentPersona.value.persona
  }
})

const truncateText = (text: string): string => {
  if (text.length <= MAX_CONTENT_LENGTH) {
    return text
  }
  return text.substring(0, MAX_CONTENT_LENGTH) + '...'
}

const viewOptions = [
  { label: 'Refined Persona', value: 'persona' },
  { label: 'Generated Response', value: 'response' },
  { label: 'Analysis Result', value: 'analysis' }
]

// Calculate differences (simple word-level diff)
const getHighlightedPersona = computed(() => {
  if (!previousPersona.value) return currentPersona.value.persona
  
  const prev = previousPersona.value.split(/\s+/)
  const curr = currentPersona.value.persona.split(/\s+/)
  
  let result = ''
  let i = 0
  
  while (i < curr.length) {
    if (i >= prev.length || curr[i] !== prev[i]) {
      // Find the end of changed segment
      let j = i
      while (j < curr.length && (j >= prev.length || curr[j] !== prev[j])) {
        j++
      }
      result += '<span class="highlight-change">' + curr.slice(i, j).join(' ') + '</span> '
      i = j
    } else {
      result += curr[i] + ' '
      i++
    }
  }
  
  return result
})

const copyCitation = () => {
  if (citationText.value) {
    const text = citationText.value.textContent || ''
    navigator.clipboard
      .writeText(text)
      .then(() => {
        isCopied.value = true
        showMessage.value = true
        setTimeout(() => {
          isCopied.value = false
          showMessage.value = false
        }, 3000)
      })
      .catch((err) => {
        console.error('Failed to copy text: ', err)
      })
  }
}
</script>
<template>
  <main class="container">
    <div class="hero-body">
      <div class="row center">
        <h1>
          DPRF: Dynamic Persona Refinement Framework for High-Fidelity LLM Role-Playing Agents
        </h1>
      </div>
      <div class="author-row">
        <Author
          author="Bingsheng Yao"
          href="https://www.bingshengyao.com/"
          sup="1"
          :image="bingshengImg"
        />
        <Author author="Bo Sun" sup="1" :image="boImg" />
        <Author author="Yuanzhe Dong" sup="2" :image="yuanzheImg" />
        <Author author="Yuxuan Lu" href="https://yuxuan.lu/" sup="1" :image="yuxuanImg" />
        <Author
          author="Dakuo Wang"
          href="https://www.dakuowang.com/"
          sup="1"
          :image="dakuoImg"
          :last="true"
        />
      </div>
      <div class="row center text-lg flex-wrap">
        <div class="affiliation"><sup>1</sup>Northeastern University, USA</div>
        <div class="affiliation"><sup>2</sup>Stanford University, USA</div>
      </div>
      <div class="row center">
        <pre>{lu.yuxuan, d.wang}@northeastern.edu</pre>
      </div>
    </div>
    <div class="row center" style="margin-bottom: 48px">
      <a href="https://arxiv.org/abs/2510.14205" target="_blank">
        <n-button round size="large" type="info">
          <template #icon>
            <n-icon>
              <svg
                xmlns="http://www.w3.org/2000/svg"
                xmlns:xlink="http://www.w3.org/1999/xlink"
                viewBox="0 0 16 16"
              >
                <g fill="none">
                  <path
                    d="M4.5 9.003a.5.5 0 0 0-.5.5v2a.5.5 0 0 0 1 0v-.166h.333a1.167 1.167 0 0 0 0-2.334H4.5zm.833 1.334H5v-.334h.333a.167.167 0 0 1 0 .334zm4.668-.835a.5.5 0 0 1 .5-.499h.998a.5.5 0 0 1 0 1h-.5v.335h.5a.5.5 0 1 1 0 1H11v.164a.5.5 0 0 1-1 .002L10 10.837L10 9.502zm-2.503-.499a.5.5 0 0 0-.5.5v2a.5.5 0 0 0 .5.5H8a1.5 1.5 0 0 0 0-3h-.502zm.5 2v-1H8a.5.5 0 0 1 0 1h-.002zM9 2.002H4.5a1.5 1.5 0 0 0-1.5 1.5v3.582A1.5 1.5 0 0 0 2 8.5v4.003a1.5 1.5 0 0 0 1.5 1.5h9a1.5 1.5 0 0 0 1.5-1.5V8.499a1.5 1.5 0 0 0-1-1.415V6h-2.5A1.5 1.5 0 0 1 9 4.5V2.002zM3.5 7.999h9a.5.5 0 0 1 .5.5v4.003a.5.5 0 0 1-.5.5h-9a.5.5 0 0 1-.5-.5V8.499a.5.5 0 0 1 .5-.5zm9.206-3H10.5a.5.5 0 0 1-.5-.5V2.298L12.706 5z"
                    fill="currentColor"
                  ></path>
                </g>
              </svg>
            </n-icon>
          </template>
          Paper
        </n-button>
      </a>
      <a href="https://github.com/neuhai/uxagent" target="_blank">
        <n-button round size="large" type="info">
          <template #icon>
            <n-icon>
              <svg
                xmlns="http://www.w3.org/2000/svg"
                xmlns:xlink="http://www.w3.org/1999/xlink"
                viewBox="0 0 496 512"
              >
                <path
                  d="M165.9 397.4c0 2-2.3 3.6-5.2 3.6c-3.3.3-5.6-1.3-5.6-3.6c0-2 2.3-3.6 5.2-3.6c3-.3 5.6 1.3 5.6 3.6zm-31.1-4.5c-.7 2 1.3 4.3 4.3 4.9c2.6 1 5.6 0 6.2-2s-1.3-4.3-4.3-5.2c-2.6-.7-5.5.3-6.2 2.3zm44.2-1.7c-2.9.7-4.9 2.6-4.6 4.9c.3 2 2.9 3.3 5.9 2.6c2.9-.7 4.9-2.6 4.6-4.6c-.3-1.9-3-3.2-5.9-2.9zM244.8 8C106.1 8 0 113.3 0 252c0 110.9 69.8 205.8 169.5 239.2c12.8 2.3 17.3-5.6 17.3-12.1c0-6.2-.3-40.4-.3-61.4c0 0-70 15-84.7-29.8c0 0-11.4-29.1-27.8-36.6c0 0-22.9-15.7 1.6-15.4c0 0 24.9 2 38.6 25.8c21.9 38.6 58.6 27.5 72.9 20.9c2.3-16 8.8-27.1 16-33.7c-55.9-6.2-112.3-14.3-112.3-110.5c0-27.5 7.6-41.3 23.6-58.9c-2.6-6.5-11.1-33.3 2.6-67.9c20.9-6.5 69 27 69 27c20-5.6 41.5-8.5 62.8-8.5s42.8 2.9 62.8 8.5c0 0 48.1-33.6 69-27c13.7 34.7 5.2 61.4 2.6 67.9c16 17.7 25.8 31.5 25.8 58.9c0 96.5-58.9 104.2-114.8 110.5c9.2 7.9 17 22.9 17 46.4c0 33.7-.3 75.4-.3 83.6c0 6.5 4.6 14.4 17.3 12.1C428.2 457.8 496 362.9 496 252C496 113.3 383.5 8 244.8 8zM97.2 352.9c-1.3 1-1 3.3.7 5.2c1.6 1.6 3.9 2.3 5.2 1c1.3-1 1-3.3-.7-5.2c-1.6-1.6-3.9-2.3-5.2-1zm-10.8-8.1c-.7 1.3.3 2.9 2.3 3.9c1.6 1 3.6.7 4.3-.7c.7-1.3-.3-2.9-2.3-3.9c-2-.6-3.6-.3-4.3.7zm32.4 35.6c-1.6 1.3-1 4.3 1.3 6.2c2.3 2.3 5.2 2.6 6.5 1c1.3-1.3.7-4.3-1.3-6.2c-2.2-2.3-5.2-2.6-6.5-1zm-11.4-14.7c-1.6 1-1.6 3.6 0 5.9c1.6 2.3 4.3 3.3 5.6 2.3c1.6-1.3 1.6-3.9 0-6.2c-1.4-2.3-4-3.3-5.6-2z"
                  fill="currentColor"
                ></path>
              </svg>
            </n-icon>
          </template>
          Code
        </n-button>
      </a>
      <a href="https://huggingface.co/datasets/NEU-HAI/UXAgent" target="_blank">
        <n-button round size="large" type="info">
          <template #icon>
            <n-icon>
              <svg
                xmlns="http://www.w3.org/2000/svg"
                xmlns:xlink="http://www.w3.org/1999/xlink"
                viewBox="0 0 24 24"
              >
                <g
                  fill="none"
                  stroke="currentColor"
                  stroke-width="2"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                >
                  <ellipse cx="12" cy="6" rx="8" ry="3"></ellipse>
                  <path d="M4 6v6a8 3 0 0 0 16 0V6"></path>
                  <path d="M4 12v6a8 3 0 0 0 16 0v-6"></path>
                </g>
              </svg>
            </n-icon>
          </template>
          Data
        </n-button>
      </a>
    </div>
    <div class="row center flex-wrap">
      <div class="flex-0" style="flex-basis: 300px">
        <img src="@/assets/dprf_paper.png" alt="teaser" style="width: 100%" />
      </div>
      <div class="flex-1 bg-gray-100 p-4 rounded-lg" style="flex-basis: 600px; max-width: 100%">
        <p class="text-lg text-align-left">
          The emerging large language model role-playing agents (LLM RPAs) aim to simulate individual human behaviors, but the persona fidelity is often undermined by manually-created profiles (e.g., cherry-picked information and personality characteristics) without validating the alignment with the target individuals. 
          To address this limitation, our work introduces the Dynamic Persona Refinement Framework (DPRF).
          DPRF aims to optimize the alignment of LLM RPAs' behaviors with those of target individuals by iteratively identifying the cognitive divergence, either through free-form or theory-grounded, structured analysis, between generated behaviors and human ground truth, and refining the persona profile to mitigate these divergences. We evaluate DPRF with five LLMs on four diverse behavior-prediction scenarios: formal debates, social media posts with mental health issues, public interviews, and movie reviews.
          DPRF can consistently improve behavioral alignment considerably over baseline personas and generalizes across models and scenarios. Our work provides a robust methodology for creating high-fidelity persona profiles and enhancing the validity of downstream applications, such as user simulation, social studies, and personalized AI.
        </p>
      </div>
    </div>
    <div class="row center framework-image-container">
      <img src="@/assets/dprf.png" alt="DPRF Framework" style="width: 80%" />
    </div>
    <div class="row center text-lg text-align-left bg-gray-100 p-4 rounded-lg">
      <div>
        <p>
          <span class="font-[1000]">DPRF (Dynamic Persona Refinement Framework)</span> is a generalizable framework that optimizes the alignment of LLM role-playing agents' behaviors with those of target individuals. The framework addresses the limitation of manually-created persona profiles by iteratively identifying the cognitive divergence, either through free-form or theory-grounded structured analysis, between generated behaviors and human ground truth, and refining the persona profile to mitigate these divergences.
        </p>
        <p class="mt-4">
          DPRF has been validated across <strong>five different LLMs</strong> and <strong>four diverse behavior-prediction scenarios</strong>:
        </p>
        <ul class="list-disc ml-10">
          <li><strong>Formal Debates:</strong> Predicting speaker statements on socio-political issues (targeting beliefs, intentions, and knowledge)</li>
          <li><strong>Mental Health Expression:</strong> Generating social media posts reflecting mental health states (targeting emotion)</li>
          <li><strong>Opinionated Reviews:</strong> Creating movie reviews consistent with emotional traits (targeting emotion and knowledge)</li>
          <li><strong>Public Interviews:</strong> Generating interview responses based on conversational context (targeting goals and intentions)</li>
        </ul>
        <p class="mt-4">
          DPRF consistently improves behavioral alignment considerably over baseline personas and generalizes across models and scenarios. Our work provides a robust methodology for creating high-fidelity persona profiles and enhancing the validity of downstream applications, such as user simulation, social studies, and personalized AI.
        </p>
      </div>
    </div>
    <div class="row center">
      <h2>DPRF Framework Overview</h2>
    </div>

    <!-- Three-Agent Architecture -->
    <div class="row center" style="width: 100%">
      <div class="agent-architecture" style="max-width: 1000px; width: 100%">
        <h3 class="text-center mb-4">Three-Agent Architecture</h3>
        <div class="agents-grid">
          <div class="agent-card">
            <div class="agent-icon">🎭</div>
            <h4>Role-Playing Agent<br/>(RPA)</h4>
            <p>
              Takes a persona profile and task context as input, generates behavioral responses by role-playing the given persona.
            </p>
          </div>
          <div class="agent-card">
            <div class="agent-icon">🔍</div>
            <h4>Behavior Analysis Agent<br/>(BAA)</h4>
            <p>
              Compares LLM-predicted behaviors with human ground truth, identifies underlying cognitive divergences through free-form or theory-grounded structured analysis.
            </p>
          </div>
          <div class="agent-card">
            <div class="agent-icon">⚡</div>
            <h4>Persona Refinement Agent<br/>(PRA)</h4>
            <p>
              Takes current persona, divergence analysis, and context to generate revised persona incorporating feedback while preserving effective elements. Terminates when persona converges or max iterations reached.
            </p>
          </div>
        </div>
      </div>
    </div>

    <!-- Case Study -->
    <div class="row center" style="margin-top: 48px">
      <h3>Case Study: Mental Health Expression - Social Media Post</h3>
    </div>
    <div class="row center" style="width: 100%">
      <div class="case-study" style="max-width: 1000px; width: 100%">
        <div class="case-info">
          <span class="badge">Dataset: DepSeverity</span>
          <span class="badge">Depression Level: Moderate</span>
          <span class="badge">Total Iterations: {{ totalPersonas }}</span>
        </div>

        <!-- Persona Viewer -->
        <div class="persona-viewer">
          <div class="persona-header">
            <div style="display: flex; align-items: center; gap: 12px; flex-wrap: wrap;">
              <h4 style="margin: 0;">Iteration {{ currentPersona.iteration }}</h4>
              <n-select
                v-model:value="viewType"
                :options="viewOptions"
                style="width: 200px;"
                size="small"
              />
            </div>
            <div class="persona-navigation">
              <n-button
                :disabled="currentPersonaIndex === 0"
                @click="prevPersona"
                circle
                size="small"
              >
                ‹
              </n-button>
              <span class="page-indicator">{{ currentPersonaIndex + 1 }} / {{ totalPersonas }}</span>
              <n-button
                :disabled="currentPersonaIndex === totalPersonas - 1"
                @click="nextPersona"
                circle
                size="small"
              >
                ›
              </n-button>
            </div>
          </div>

          <div class="persona-content">
            <div v-if="viewType === 'persona' && currentPersonaIndex === 0" class="persona-text">
              {{ getCurrentContent }}
            </div>
            <div v-else-if="viewType === 'persona' && currentPersonaIndex > 0" class="persona-text" v-html="getHighlightedPersona"></div>
            <div v-else class="persona-text" style="white-space: pre-wrap;">{{ getCurrentContent }}</div>
            <div v-if="viewType === 'persona' && currentPersonaIndex > 0" class="legend">
              <span class="legend-item">
                <span class="legend-color changed"></span>
                <span>Modified/Added Text</span>
              </span>
            </div>
          </div>

          <!-- Page Dots -->
          <div class="page-dots">
            <span
              v-for="(p, index) in personas"
              :key="index"
              :class="['dot', { active: index === currentPersonaIndex }]"
              @click="goToPersona(index)"
              :title="`Iteration ${p.iteration}`"
            ></span>
          </div>
        </div>

        <!-- Overall Metrics -->
        <div class="case-description">
          <h4>Task & Results</h4>
          <p>
            <strong>Task:</strong> Given a persona that includes a specified mental health state (moderate depression), predict the content of a social media post that targets the cognitive dimension of emotion.
          </p>
          <p>
            <strong>Result:</strong> Through {{ totalPersonas }} iterations of dynamic persona refinement, DPRF significantly improved the behavioral alignment. The refined persona better captures the nuanced expressions and emotional patterns characteristic of the target individual's mental health state.
          </p>
          
          <div class="metrics-summary">
            <div class="metric-summary-item">
              <span class="metric-label">Embedding Similarity:</span>
              <span class="metric-arrow">0.331 → 0.763 (+130.5%)</span>
            </div>

            <div class="metric-summary-item">
              <span class="metric-label">ROUGE-L F1:</span>
              <span class="metric-arrow">0.152 → 0.252 (+66.2%)</span>
            </div>

            <div class="metric-summary-item">
              <span class="metric-label">BERTScore F1:</span>
              <span class="metric-arrow">0.815 → 0.858 (+5.2%)</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="row center">
      <div class="citation">
        <h3>Citation</h3>
        <pre class="citation-text" ref="citationText">
@article{yao2025dprf,
  author    = {Bingsheng Yao and Bo Sun and Yuanzhe Dong and Yuxuan Lu and Dakuo Wang},
  title     = {DPRF: A Generalizable Dynamic Persona Refinement Framework for Optimizing Behavior Alignment Between Personalized LLM Role-Playing Agents and Humans},
  journal   = {arXiv preprint arXiv:2510.14205},
  year      = {2025},
  url       = {https://arxiv.org/abs/2510.14205},
}</pre
        >
        <n-button quaternary circle size="tiny" class="copy-button" @click="copyCitation">
          <template #icon>
            <n-icon>
              <svg
                xmlns="http://www.w3.org/2000/svg"
                xmlns:xlink="http://www.w3.org/1999/xlink"
                viewBox="0 0 24 24"
              >
                <path
                  d="M16 1H4c-1.1 0-2 .9-2 2v14h2V3h12V1zm3 4H8c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h11c1.1 0 2-.9 2-2V7c0-1.1-.9-2-2-2zm0 16H8V7h11v14z"
                  fill="currentColor"
                ></path>
              </svg>
            </n-icon>
          </template>
        </n-button>
      </div>
    </div>
  </main>
</template>

<style scoped lang="scss">
h2 {
  margin-top: 48px;
  // font-size: 32px;
  font-size: 1.5em;
  font-weight: 700;
  color: #333;
  text-align: center;
}
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px;
}
.hero-body {
  margin: 48px 24px;

  .author-row {
    --min-width: 200px;
    --grid-gap: 20px;
    display: grid;
    grid-template-columns: repeat(
      auto-fit,
      minmax(
        min(calc((var(--min-width) - var(--grid-gap)) / 2), calc((100% - var(--grid-gap)) / 2)),
        1fr
      )
    );
    grid-gap: var(--grid-gap);
    max-width: calc(var(--min-width) * 6 + var(--grid-gap) * 5);
    container: grid-wrapper / inline-size;
    @media (max-width: 1024px) {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      column-gap: 12px;
      row-gap: 0px;
    }

    @supports (-webkit-hyphens: none) {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      column-gap: 12px;
      row-gap: 0px;
    }
  }
  h1 {
    // font-size: 48px;
    font-size: 2em;
    font-weight: 700;
    color: #333;
    text-align: center;
  }
  .affiliation {
    font-size: 16px;
    font-weight: 400;
    color: #333;
    text-align: center;
    margin-right: 16px;
  }
}

.citation {
  margin: 32px 0;
  width: 80%;
  max-width: 800px;
  position: relative;

  h3 {
    font-size: 24px;
    font-weight: 600;
    color: #333;
    margin-bottom: 16px;
    text-align: center;
  }

  .citation-text {
    background-color: #f5f5f5;
    padding: 16px;
    border-radius: 8px;
    font-family: monospace;
    font-size: 14px;
    overflow-x: auto;
    white-space: pre-wrap;
    text-align: left;
  }

  .copy-button {
    font-size: 24px;
    position: absolute;
    top: calc(1.6 * 24px + 16px + 10px);
    right: 10px;
    cursor: pointer;
    transition: background-color 0.2s;

    &:hover {
      background-color: #e0e0e0;
    }

    &.copied {
      background-color: #4caf50;
      color: white;
    }
  }
}

// Framework image container - mobile optimization
.framework-image-container {
  @media (max-width: 768px) {
    width: 100vw;
    margin-left: calc(-50vw + 50%);
    margin-right: calc(-50vw + 50%);
    
    img {
      width: 100% !important;
    }
  }
}

// Agent Architecture Styles
.agent-architecture {
  width: 100%;
  padding: 32px 24px;

  h3 {
    font-size: 28px;
    font-weight: 700;
    color: #333;
    margin-bottom: 32px;
  }

  .agents-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 24px;
    margin-top: 24px;
  }

  .agent-card {
    background: white;
    border: 2px solid #e5e7eb;
    border-radius: 12px;
    padding: 24px;
    text-align: center;
    transition: all 0.3s ease;

    &:hover {
      border-color: #3b82f6;
      box-shadow: 0 4px 12px rgba(59, 130, 246, 0.1);
      transform: translateY(-4px);
    }

    .agent-icon {
      font-size: 48px;
      margin-bottom: 16px;
    }

    h4 {
      font-size: 18px;
      font-weight: 600;
      color: #1f2937;
      margin-bottom: 12px;
    }

    p {
      font-size: 14px;
      color: #6b7280;
      line-height: 1.6;
    }
  }
}

// Case Study Styles
.case-study {
  width: 100%;
  background: white;
  border-radius: 16px;
  padding: 32px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);

  // Mobile optimization
  @media (max-width: 768px) {
    padding: 16px;
    border-radius: 12px;
  }

  h3 {
    font-size: 24px;
    font-weight: 700;
    color: #333;
    margin-bottom: 24px;
  }

  .case-info {
    display: flex;
    gap: 12px;
    margin-bottom: 32px;
    flex-wrap: wrap;

    @media (max-width: 768px) {
      margin-bottom: 16px;
    }

    .badge {
      background: #eff6ff;
      color: #1e40af;
      padding: 6px 16px;
      border-radius: 20px;
      font-size: 14px;
      font-weight: 500;
    }
  }

  .persona-viewer {
    background: #f9fafb;
    border-radius: 12px;
    padding: 24px;
    margin-bottom: 32px;
    border: 2px solid #e5e7eb;

    @media (max-width: 768px) {
      margin-left: -16px;
      margin-right: -16px;
      margin-bottom: 16px;
      padding: 16px;
      border-radius: 8px;
      border-left: none;
      border-right: none;
    }

    .persona-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
      flex-wrap: wrap;
      gap: 16px;

      h4 {
        font-size: 18px;
        font-weight: 600;
        color: #1f2937;
        margin: 0;
      }

      .persona-navigation {
        display: flex;
        align-items: center;
        gap: 12px;

        .page-indicator {
          font-size: 14px;
          color: #6b7280;
          font-weight: 500;
          min-width: 60px;
          text-align: center;
        }
      }
    }

    .persona-content {
      background: white;
      border-radius: 8px;
      padding: 20px;
      min-height: 150px;
      margin-bottom: 20px;

      @media (max-width: 768px) {
        padding: 16px;
        margin-bottom: 16px;
        min-height: 120px;
      }

      .persona-text {
        font-size: 15px;
        line-height: 1.8;
        color: #374151;
        white-space: pre-wrap;

        :deep(.highlight-change) {
          background: #fef3c7;
          color: #92400e;
          padding: 2px 4px;
          border-radius: 3px;
          font-weight: 500;
        }
      }

      .legend {
        margin-top: 16px;
        padding-top: 12px;
        border-top: 1px solid #e5e7eb;

        .legend-item {
          display: inline-flex;
          align-items: center;
          gap: 6px;
          font-size: 13px;
          color: #6b7280;

          .legend-color {
            width: 20px;
            height: 14px;
            border-radius: 3px;

            &.changed {
              background: #fef3c7;
              border: 1px solid #fbbf24;
            }
          }
        }
      }
    }

    .page-dots {
      display: flex;
      justify-content: center;
      gap: 8px;
      flex-wrap: wrap;

      .dot {
        width: 10px;
        height: 10px;
        border-radius: 50%;
        background: #d1d5db;
        cursor: pointer;
        transition: all 0.2s;

        &:hover {
          background: #9ca3af;
          transform: scale(1.2);
        }

        &.active {
          background: #3b82f6;
          width: 12px;
          height: 12px;
        }
      }
    }
  }

  .case-description {
    background: #f8fafc;
    border-left: 4px solid #3b82f6;
    padding: 20px;
    border-radius: 8px;

    @media (max-width: 768px) {
      margin-left: -16px;
      margin-right: -16px;
      padding: 16px;
      border-radius: 12px;
      border-left-width: 4px;
      border-right: none;
    }

    h4 {
      font-size: 18px;
      font-weight: 600;
      color: #1e293b;
      margin-bottom: 12px;

      @media (max-width: 768px) {
        font-size: 16px;
        margin-bottom: 10px;
      }
    }

    p {
      margin-bottom: 12px;
      font-size: 15px;
      line-height: 1.7;
      color: #475569;

      @media (max-width: 768px) {
        font-size: 14px;
        line-height: 1.6;
        margin-bottom: 10px;
      }

      strong {
        color: #1e293b;
        font-weight: 600;
      }
    }

    .metrics-summary {
      margin-top: 24px;
      display: flex;
      flex-direction: column;
      gap: 16px;

      @media (max-width: 768px) {
        margin-top: 16px;
        gap: 12px;
      }

      .metric-summary-item {
        display: flex;
        align-items: center;
        gap: 12px;
        flex-wrap: wrap;

        @media (max-width: 768px) {
          gap: 8px;
        }

        .metric-label {
          font-size: 15px;
          font-weight: 600;
          color: #374151;
          min-width: 160px;

          @media (max-width: 768px) {
            font-size: 14px;
            min-width: auto;
            flex-basis: 100%;
          }
        }

        .metric-arrow {
          font-size: 15px;
          font-weight: 500;
          color: #475569;
          font-family: monospace;

          @media (max-width: 768px) {
            font-size: 13px;
          }
          
          strong {
            color: #10b981;
            font-weight: 700;
          }
        }
      }
    }
  }
}
</style>
