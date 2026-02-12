<template>
  <q-page class="bg-dark text-white q-pa-md">
    <div class="row justify-between items-center q-mb-xl">
      <div class="col-auto">
        <q-img
          src="/logo.png"
          alt="Logo Millonario"
          style="width: 120px"  
          fit="contain"
        />
      </div>
      <div class="col-auto">
        <div class="row items-center">
          <div class="text-h6 q-mr-md">Pregunta {{ currentQuestion }}/15</div>
          <q-btn 
            v-if="!showStartModal && !showResultModal"
            label="Retirarme" 
            color="warning" 
            class="q-mr-md"
            @click="retireGame"
          />
          <q-btn 
            outline 
            color="yellow-8" 
            icon="volume_up" 
            round 
            class="q-mr-md"
          />
          <q-btn 
            outline 
            color="yellow-8" 
            icon="help_outline" 
            round
          />
        </div>
      </div>
    </div>

    <div v-if="!showStartModal && !showResultModal" class="row q-col-gutter-lg">
      <div class="col-12 col-md-8">
        <div class="row justify-center q-mb-lg">
          <div class="col-auto">
            <div class="timer-circle">
              <div class="text-h4">{{ timer }}</div>
              <div class="text-caption">SEGUNDOS</div>
            </div>
          </div>
        </div>

        <div class="q-mb-xl">
          <div class="text-h5 text-center text-grey-4 q-mb-sm">PREGUNTA #{{ currentQuestion }}</div>
          <div class="text-h3 text-center text-bold">
            {{ currentQuestionData.question }}
          </div>
          <div v-if="currentQuestionData.category" class="text-h6 text-center text-grey-5 q-mt-sm">
            Categoría: {{ currentQuestionData.category }}
          </div>
        </div>

        <div class="q-mb-xl">
          <div class="row q-col-gutter-md">
            <div 
              v-for="(option, index) in currentQuestionData.options" 
              :key="index"
              class="col-12 col-md-6"
              v-show="!usedLifelines.includes('50_50') || option.visible !== false"
            >
              <q-card 
                class="option-card cursor-pointer"
                :class="{
                  'active': activeOption === index,
                  'correct': showCorrect && option.correct,
                  'incorrect': showIncorrect && activeOption === index && !option.correct,
                  'disabled': isAnswering || (usedLifelines.includes('50_50') && option.visible === false)
                }"
                @click="!isAnswering && selectOption(index)"
              >
                <q-card-section class="row items-center">
                  <div class="option-letter q-mr-md">{{ option.letter }}</div>
                  <div class="text-h6">{{ option.text }}</div>
                  <q-space />
                  <q-icon 
                    v-if="showCorrect && option.correct"
                    name="check_circle" 
                    color="green"
                    size="md"
                    class="q-ml-sm"
                  />
                  <q-icon 
                    v-if="showIncorrect && activeOption === index && !option.correct"
                    name="cancel" 
                    color="red"
                    size="md"
                    class="q-ml-sm"
                  />
                </q-card-section>
              </q-card>
            </div>
          </div>
        </div>

        <div class="row justify-center q-gutter-md">
          <q-btn 
            label="50:50" 
            color="blue-8" 
            icon="filter_alt" 
            class="action-btn"
            :disable="usedLifelines.includes('50_50') || isAnswering"
            @click="useLifeline('50_50')"
          />
          <q-btn 
            label="Público" 
            color="green-8" 
            icon="groups" 
            class="action-btn"
            :disable="usedLifelines.includes('audience') || isAnswering"
            @click="useLifeline('audience')"
          />
          <q-btn 
            label="Llamada" 
            color="red-8" 
            icon="call" 
            class="action-btn"
            :disable="usedLifelines.includes('call') || isAnswering"
            @click="useLifeline('call')"
          />
        </div>
      </div>

      <div v-if="showAudienceHelp" class="audience-help-overlay">
        <div class="audience-help-content">
          <div class="text-h4 text-center q-mb-md text-yellow-8">
            <q-icon name="groups" size="xl" class="q-mr-sm" />
            AYUDA DEL PÚBLICO
          </div>
          
          <div class="audience-chart q-mb-lg">
            <div 
              v-for="(option, index) in currentQuestionData.options" 
              :key="index"
              class="chart-bar-container"
            >
              <div class="chart-label">{{ option.letter }}</div>
              <div class="chart-bar-background">
                <div 
                  class="chart-bar-fill"
                  :style="{ width: audiencePercentages[index] + '%' }"
                  :class="'bar-' + option.letter.toLowerCase()"
                >
                  <div class="chart-percentage">{{ audiencePercentages[index] }}%</div>
                </div>
              </div>
              <div class="chart-text">{{ option.text }}</div>
            </div>
          </div>
          
          <div class="audience-crowd">
            <div class="text-h6 text-center q-mb-sm">Reacción del público:</div>
            <div class="crowd-icons">
              <q-icon v-for="n in 20" :key="n" 
                name="person" 
                size="md" 
                :color="getRandomAudienceColor()"
                class="q-mx-xs"
              />
            </div>
          </div>
          
          <div class="text-center q-mt-lg">
            <q-btn 
              label="CONTINUAR" 
              color="green" 
              @click="showAudienceHelp = false"
            />
          </div>
        </div>
      </div>

      <div v-if="showCallHelp" class="call-help-overlay">
        <div class="call-help-content">
          <div class="call-header">
            <div class="text-h4 text-center q-mb-md text-yellow-8">
              <q-icon name="call" size="xl" class="q-mr-sm" />
              LLAMADA DE EMERGENCIA
            </div>
            <div class="call-animation">
              <div class="phone-ring">
                <q-icon name="phone_in_talk" size="4rem" color="green" />
              </div>
            </div>
          </div>
          
          <div class="call-friend-info q-mb-lg">
            <div class="friend-avatar">
              <q-icon name="person" size="3rem" />
            </div>
            <div class="friend-details">
              <div class="text-h6">Tu amigo: {{ getRandomFriendName() }}</div>
              <div class="text-caption">Especialista en: {{ getRandomExpertise() }}</div>
            </div>
          </div>
          
          <div class="call-message q-pa-md">
            <div class="message-bubble">
              <div class="text-h6 q-mb-sm">🗣️ "{{ callMessage }}"</div>
              <div class="text-caption text-grey-6">Nivel de confianza: {{ confidenceLevel }}</div>
            </div>
            
            <div class="confidence-meter q-mt-md">
              <div class="meter-labels row justify-between">
                <span>Bajo</span>
                <span>Medio</span>
                <span>Alto</span>
              </div>
              <div class="meter-bar">
                <div 
                  class="meter-fill"
                  :style="{ width: confidencePercentage + '%' }"
                  :class="confidenceClass"
                ></div>
              </div>
            </div>
          </div>
          
          <div class="text-center q-mt-lg">
            <q-btn 
              label="COLGAR" 
              color="red" 
              @click="showCallHelp = false"
            />
          </div>
        </div>
      </div>

      <div class="col-12 col-md-4">
        <div class="prize-panel q-pa-lg">
          <div class="text-h5 text-center text-yellow-8 q-mb-md">PREMIO</div>
          
          <div class="prize-list">
            <div 
              v-for="(prize, index) in prizes" 
              :key="index"
              class="prize-item row items-center"
              :class="{
                'current': currentQuestion === prize.level,
                'passed': currentQuestion > prize.level,
                'safe-level': prize.safe,
                'next-safe': isNextSafeLevel(prize.level)
              }"
            >
              <div class="col-auto q-mr-md">
                <q-icon 
                  v-if="currentQuestion > prize.level"
                  name="check_circle" 
                  color="green"
                  size="sm"
                />
              </div>
              <div class="col text-right">
                <div class="text-h6">{{ prize.level }}</div>
              </div>
              <div class="col text-left">
                <div class="text-h6">${{ prize.amount.toLocaleString() }}</div>
              </div>
              <div v-if="prize.safe" class="col-auto q-ml-sm">
                <q-icon name="shield" color="yellow-8" size="xs" />
              </div>
            </div>
          </div>
          
          <div class="current-prize q-mt-lg q-pa-md text-center bg-yellow-9 text-black rounded-borders">
            <div class="text-caption">PREMIO ACTUAL</div>
            <div class="text-h5">${{ currentPrizeAmount.toLocaleString() }}</div>
          </div>
        </div>
      </div>
    </div>

    <q-dialog v-model="showStartModal" persistent>
      <q-card class="start-modal">
        <q-card-section class="text-center">
          <q-img
            src="/public/logo.png"
            alt="Logo Millonario"
            style="width: 260px; margin: auto"
            fit="contain"
            class="q-mb-md"
          />

          <div class="text-h6 q-mb-lg">Responde 15 preguntas para ganar $1,000,000</div>
          
          <div class="q-mb-lg">
            <div class="text-h6 q-mb-sm">Niveles seguros:</div>
            <div class="row justify-center q-gutter-sm">
              <div class="safe-level-indicator">$100</div>
              <div class="safe-level-indicator">$1,000</div>
              <div class="safe-level-indicator">$32,000</div>
              <div class="safe-level-indicator">$1,000,000</div>
            </div>
          </div>
          
          <div class="row justify-center q-gutter-md q-mb-lg">
            <div class="lifeline-item">
              <q-icon name="filter_alt" size="lg" color="blue" />
              <div class="text-caption">50:50</div>
            </div>
            <div class="lifeline-item">
              <q-icon name="groups" size="lg" color="green" />
              <div class="text-caption">Público</div>
            </div>
            <div class="lifeline-item">
              <q-icon name="call" size="lg" color="red" />
              <div class="text-caption">Llamada</div>
            </div>
          </div>
          
          <q-btn 
            label="EMPEZAR JUEGO" 
            color="yellow-8" 
            class="start-btn"
            @click="startGame"
          />
        </q-card-section>
      </q-card>
    </q-dialog>

    <q-dialog v-model="showResultModal" persistent>
      <q-card class="result-modal">
        <q-card-section class="text-center">
          <div class="text-h4" :class="resultClass">{{ resultTitle }}</div>
          <div class="text-h6 q-my-md">{{ resultMessage }}</div>
          
          <div v-if="resultPrize > 0" class="prize-result q-my-lg">
            <div class="text-caption text-grey-4">GANASTE</div>
            <div class="text-h3 text-yellow-8">${{ resultPrize.toLocaleString() }}</div>
          </div>
          
          <div class="row justify-center q-gutter-md q-mt-lg">
            <q-btn 
              v-if="gameResult === 'win' || gameResult === 'lose'"
              :label="gameResult === 'win' ? 'Jugar de nuevo' : 'Intentar nuevamente'" 
              :color="gameResult === 'win' ? 'yellow-8' : 'red'"
              @click="restartGame"
            />
            <q-btn 
              v-if="gameResult === 'retire'"
              label="Aceptar premio" 
              color="blue"
              @click="acceptPrize"
            />
            <q-btn 
              v-if="gameResult === 'retire' && currentQuestion < 15"
              label="Continuar jugando" 
              color="yellow-8"
              outline
              @click="continuePlaying"
            />
          </div>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'

const showStartModal = ref(true)
const activeOption = ref(null)
const currentQuestion = ref(1)
const usedLifelines = ref([])
const showResultModal = ref(false)
const gameResult = ref(null)
const isAnswering = ref(false)
const timer = ref(30)
const timerInterval = ref(null)
const showCorrect = ref(false)
const showIncorrect = ref(false)

const showAudienceHelp = ref(false)
const showCallHelp = ref(false)
const audiencePercentages = ref([])
const callMessage = ref('')
const confidenceLevel = ref('')
const confidencePercentage = ref(0)
const confidenceClass = ref('')

const questionBank = ref([
  {
    question: "¿Cuál es la capital de Francia?",
    options: [
      { letter: 'A', text: 'Londres', correct: false, visible: true },
      { letter: 'B', text: 'París', correct: true, visible: true },
      { letter: 'C', text: 'Madrid', correct: false, visible: true },
      { letter: 'D', text: 'Roma', correct: false, visible: true }
    ],
    category: "Geografía",
    difficulty: 1
  },
  {
    question: "¿En qué año llegó el hombre a la Luna?",
    options: [
      { letter: 'A', text: '1965', correct: false, visible: true },
      { letter: 'B', text: '1969', correct: true, visible: true },
      { letter: 'C', text: '1972', correct: false, visible: true },
      { letter: 'D', text: '1958', correct: false, visible: true }
    ],
    category: "Historia",
    difficulty: 1
  },
  {
    question: "¿Qué elemento químico tiene el símbolo 'O'?",
    options: [
      { letter: 'A', text: 'Oro', correct: false, visible: true },
      { letter: 'B', text: 'Osmio', correct: false, visible: true },
      { letter: 'C', text: 'Oxígeno', correct: true, visible: true },
      { letter: 'D', text: 'Oganesón', correct: false, visible: true }
    ],
    category: "Ciencia",
    difficulty: 1
  },
  {
    question: "¿Quién escribió 'Cien años de soledad'?",
    options: [
      { letter: 'A', text: 'Gabriel García Márquez', correct: true, visible: true },
      { letter: 'B', text: 'Mario Vargas Llosa', correct: false, visible: true },
      { letter: 'C', text: 'Pablo Neruda', correct: false, visible: true },
      { letter: 'D', text: 'Jorge Luis Borges', correct: false, visible: true }
    ],
    category: "Literatura",
    difficulty: 2
  },
  {
    question: "¿Cuál es el río más largo del mundo?",
    options: [
      { letter: 'A', text: 'Amazonas', correct: true, visible: true },
      { letter: 'B', text: 'Nilo', correct: false, visible: true },
      { letter: 'C', text: 'Yangtsé', correct: false, visible: true },
      { letter: 'D', text: 'Misisipi', correct: false, visible: true }
    ],
    category: "Geografía",
    difficulty: 2
  },
  {
    question: "¿En qué deporte se utiliza el término 'hat-trick'?",
    options: [
      { letter: 'A', text: 'Fútbol', correct: true, visible: true },
      { letter: 'B', text: 'Baloncesto', correct: false, visible: true },
      { letter: 'C', text: 'Tenis', correct: false, visible: true },
      { letter: 'D', text: 'Golf', correct: false, visible: true }
    ],
    category: "Deportes",
    difficulty: 2
  },
  {
    question: "¿Cuál es el planeta más grande del sistema solar?",
    options: [
      { letter: 'A', text: 'Tierra', correct: false, visible: true },
      { letter: 'B', text: 'Marte', correct: false, visible: true },
      { letter: 'C', text: 'Júpiter', correct: true, visible: true },
      { letter: 'D', text: 'Saturno', correct: false, visible: true }
    ],
    category: "Astronomía",
    difficulty: 3
  },
  {
    question: "¿Qué artista pintó 'La noche estrellada'?",
    options: [
      { letter: 'A', text: 'Pablo Picasso', correct: false, visible: true },
      { letter: 'B', text: 'Vincent van Gogh', correct: true, visible: true },
      { letter: 'C', text: 'Claude Monet', correct: false, visible: true },
      { letter: 'D', text: 'Leonardo da Vinci', correct: false, visible: true }
    ],
    category: "Arte",
    difficulty: 3
  },
  {
    question: "¿Cuál es el hueso más largo del cuerpo humano?",
    options: [
      { letter: 'A', text: 'Fémur', correct: true, visible: true },
      { letter: 'B', text: 'Tibia', correct: false, visible: true },
      { letter: 'C', text: 'Húmero', correct: false, visible: true },
      { letter: 'D', text: 'Columna vertebral', correct: false, visible: true }
    ],
    category: "Anatomía",
    difficulty: 3
  },
  {
    question: "¿En qué país se encuentra la Torre Eiffel?",
    options: [
      { letter: 'A', text: 'Italia', correct: false, visible: true },
      { letter: 'B', text: 'Francia', correct: true, visible: true },
      { letter: 'C', text: 'España', correct: false, visible: true },
      { letter: 'D', text: 'Alemania', correct: false, visible: true }
    ],
    category: "Geografía",
    difficulty: 4
  },
  {
    question: "¿Cuál es la velocidad de la luz en el vacío?",
    options: [
      { letter: 'A', text: '300,000 km/s', correct: true, visible: true },
      { letter: 'B', text: '150,000 km/s', correct: false, visible: true },
      { letter: 'C', text: '500,000 km/s', correct: false, visible: true },
      { letter: 'D', text: '1,000,000 km/s', correct: false, visible: true }
    ],
    category: "Física",
    difficulty: 4
  },
  {
    question: "¿Quién fue el primer presidente de Estados Unidos?",
    options: [
      { letter: 'A', text: 'Abraham Lincoln', correct: false, visible: true },
      { letter: 'B', text: 'Thomas Jefferson', correct: false, visible: true },
      { letter: 'C', text: 'George Washington', correct: true, visible: true },
      { letter: 'D', text: 'John Adams', correct: false, visible: true }
    ],
    category: "Historia",
    difficulty: 4
  },
  {
    question: "¿Qué instrumento musical tiene 88 teclas?",
    options: [
      { letter: 'A', text: 'Guitarra', correct: false, visible: true },
      { letter: 'B', text: 'Violín', correct: false, visible: true },
      { letter: 'C', text: 'Piano', correct: true, visible: true },
      { letter: 'D', text: 'Saxofón', correct: false, visible: true }
    ],
    category: "Música",
    difficulty: 5
  },
  {
    question: "¿Cuál es el elemento más abundante en la corteza terrestre?",
    options: [
      { letter: 'A', text: 'Hierro', correct: false, visible: true },
      { letter: 'B', text: 'Oxígeno', correct: true, visible: true },
      { letter: 'C', text: 'Silicio', correct: false, visible: true },
      { letter: 'D', text: 'Aluminio', correct: false, visible: true }
    ],
    category: "Geología",
    difficulty: 5
  },
  {
    question: "¿En qué año se firmó la Declaración de Independencia de Estados Unidos?",
    options: [
      { letter: 'A', text: '1776', correct: true, visible: true },
      { letter: 'B', text: '1789', correct: false, visible: true },
      { letter: 'C', text: '1492', correct: false, visible: true },
      { letter: 'D', text: '1812', correct: false, visible: true }
    ],
    category: "Historia",
    difficulty: 5
  }
])

const prizes = ref([
  { level: 1, amount: 100, safe: true },
  { level: 2, amount: 200, safe: false },
  { level: 3, amount: 300, safe: true },
  { level: 4, amount: 500, safe: false },
  { level: 5, amount: 1000, safe: false },
  { level: 6, amount: 2000, safe: false },
  { level: 7, amount: 4000, safe: true },
  { level: 8, amount: 8000, safe: false },
  { level: 9, amount: 16000, safe: false },
  { level: 10, amount: 32000, safe: false },
  { level: 11, amount: 64000, safe: true },
  { level: 12, amount: 125000, safe: false },
  { level: 13, amount: 250000, safe: false },
  { level: 14, amount: 500000, safe: false },
  { level: 15, amount: 1000000, safe: true }
])

const currentQuestionData = computed(() => {
  return questionBank.value[currentQuestion.value - 1] || {}
})

const currentPrizeAmount = computed(() => {
  const prize = prizes.value.find(p => p.level === currentQuestion.value)
  return prize ? prize.amount : 0
})

const resultTitle = computed(() => {
  switch(gameResult.value) {
    case 'win': return '¡FELICIDADES!'
    case 'lose': return 'RESPUESTA INCORRECTA'
    case 'retire': return 'TE RETIRAS'
    default: return ''
  }
})

const resultMessage = computed(() => {
  switch(gameResult.value) {
    case 'win': return '¡Has ganado el premio máximo de $1,000,000!'
    case 'lose': {
      const correctAnswer = currentQuestionData.value.options?.find(opt => opt.correct)
      return correctAnswer ? `La respuesta correcta era: ${correctAnswer.letter}. ${correctAnswer.text}` : 'Respuesta incorrecta'
    }
    case 'retire': return `Te retiras con un premio de $${resultPrize.value.toLocaleString()}`
    default: return ''
  }
})

const resultClass = computed(() => {
  switch(gameResult.value) {
    case 'win': return 'text-yellow-8'
    case 'lose': return 'text-red'
    case 'retire': return 'text-blue'
    default: return ''
  }
})

const resultPrize = computed(() => {
  if (gameResult.value === 'win') return 1000000
  if (gameResult.value === 'lose') {
    const safePrize = prizes.value
      .filter(p => p.safe)
      .sort((a, b) => b.level - a.level)
      .find(p => p.level < currentQuestion.value)
    return safePrize ? safePrize.amount : 0
  }
  if (gameResult.value === 'retire') {
    const currentPrize = prizes.value.find(p => p.level === currentQuestion.value)
    return currentPrize ? currentPrize.amount : 0
  }
  return 0
})

const startGame = () => {
  showStartModal.value = false
  currentQuestion.value = 1
  usedLifelines.value = []
  activeOption.value = null
  isAnswering.value = false
  showCorrect.value = false
  showIncorrect.value = false
  resetTimer()
  startTimer()
  
  questionBank.value.forEach(question => {
    question.options.forEach(option => {
      option.visible = true
    })
  })
}

const startTimer = () => {
  if (timerInterval.value) clearInterval(timerInterval.value)
  
  timerInterval.value = setInterval(() => {
    if (timer.value > 0) {
      timer.value--
    } else {
      clearInterval(timerInterval.value)
      gameResult.value = 'lose'
      showResultModal.value = true
    }
  }, 1000)
}

const resetTimer = () => {
  timer.value = 30
}

const selectOption = (index) => {
  if (isAnswering.value) return
  
  isAnswering.value = true
  activeOption.value = index
  clearInterval(timerInterval.value)
  
  setTimeout(() => {
    const selectedOption = currentQuestionData.value.options[index]
    
    if (selectedOption.correct) {
      showCorrect.value = true
      
      setTimeout(() => {
        if (currentQuestion.value === 15) {
          gameResult.value = 'win'
          showResultModal.value = true
        } else {
          nextQuestion()
        }
      }, 2000)
    } else {
      showIncorrect.value = true
      
      setTimeout(() => {
        gameResult.value = 'lose'
        showResultModal.value = true
      }, 2000)
    }
  }, 1000)
}

const nextQuestion = () => {
  activeOption.value = null
  isAnswering.value = false
  showCorrect.value = false
  showIncorrect.value = false
  
  currentQuestion.value++
  
  resetTimer()
  startTimer()
}

const useLifeline = (type) => {
  if (usedLifelines.value.includes(type) || isAnswering.value) return
  
  usedLifelines.value.push(type)
  
  switch(type) {
    case '50_50':
      applyFiftyFifty()
      break
    case 'audience':
      showAudiencePoll()
      break
    case 'call':
      simulatePhoneCall()
      break
  }
}

const applyFiftyFifty = () => {
  const options = currentQuestionData.value.options
  const incorrectOptions = options.filter(opt => !opt.correct)
  
  const optionsToHide = incorrectOptions
    .sort(() => Math.random() - 0.5)
    .slice(0, 2)
  
  optionsToHide.forEach(opt => {
    const optionIndex = options.findIndex(o => o.text === opt.text)
    if (optionIndex > -1) {
      questionBank.value[currentQuestion.value - 1].options[optionIndex].visible = false
    }
  })
}

const showAudiencePoll = () => {
  const options = currentQuestionData.value.options
  const correctIndex = options.findIndex(opt => opt.correct)
  
  let percentages = [0, 0, 0, 0]
  
  percentages[correctIndex] = Math.floor(Math.random() * 30) + 40 
  
  let remaining = 100 - percentages[correctIndex]
  const otherIndices = [0, 1, 2, 3].filter(i => i !== correctIndex)
  
  for (let i = 0; i < otherIndices.length - 1; i++) {
    const max = Math.min(30, remaining - (otherIndices.length - i - 1) * 5)
    const percent = Math.floor(Math.random() * max) + 5
    percentages[otherIndices[i]] = percent
    remaining -= percent
  }
  
  percentages[otherIndices[otherIndices.length - 1]] = remaining
  
  const total = percentages.reduce((a, b) => a + b, 0)
  if (total !== 100) {
    percentages[correctIndex] += (100 - total)
  }
  
  audiencePercentages.value = percentages
  showAudienceHelp.value = true
}

const getRandomAudienceColor = () => {
  const colors = ['green', 'light-green', 'yellow', 'amber', 'orange', 'deep-orange']
  return colors[Math.floor(Math.random() * colors.length)]
}

const simulatePhoneCall = () => {
  const options = currentQuestionData.value.options
  const correctIndex = options.findIndex(opt => opt.correct)
  const correctOption = options[correctIndex]
  
  const confidenceLevels = [
    { level: 'Muy seguro', percentage: 85, class: 'high-confidence' },
    { level: 'Bastante seguro', percentage: 70, class: 'medium-high-confidence' },
    { level: 'Moderadamente seguro', percentage: 55, class: 'medium-confidence' },
    { level: 'Poco seguro', percentage: 40, class: 'low-confidence' }
  ]
  
  const confidence = confidenceLevels[Math.floor(Math.random() * confidenceLevels.length)]
  
  const messages = [
    `Estoy casi seguro de que es la ${correctOption.letter}. Recuerdo haber estudiado esto en la universidad...`,
    `Hmm, déjame pensar... Sí, creo que la respuesta correcta es la ${correctOption.letter}.`,
    `¡Oh, esa es fácil! Definitivamente es la ${correctOption.letter}.`,
    `No estoy 100% seguro, pero me inclino fuertemente por la ${correctOption.letter}.`,
    `Hace poco leí sobre esto, y si no me equivoco, la respuesta es ${correctOption.letter}.`,
    `Déjame consultar mi memoria... Sí, estoy bastante seguro de que es la ${correctOption.letter}.`,
    `¡Claro que lo sé! Es la ${correctOption.letter}, sin duda alguna.`,
    `Creo que la ${correctOption.letter} es la correcta, pero no me hagas mucha caso...`
  ]
  
  callMessage.value = messages[Math.floor(Math.random() * messages.length)]
  confidenceLevel.value = confidence.level
  confidencePercentage.value = confidence.percentage
  confidenceClass.value = confidence.class
  
  showCallHelp.value = true
}

const getRandomFriendName = () => {
  const names = ['Carlos', 'María', 'Pedro', 'Ana', 'Luis', 'Laura', 'Javier', 'Sofía', 'Miguel', 'Elena']
  return names[Math.floor(Math.random() * names.length)]
}

const getRandomExpertise = () => {
  const expertises = [
    'Historia', 'Geografía', 'Ciencia', 'Arte', 'Literatura',
    'Deportes', 'Música', 'Cine', 'Tecnología', 'Cultura General'
  ]
  return expertises[Math.floor(Math.random() * expertises.length)]
}

const retireGame = () => {
  gameResult.value = 'retire'
  showResultModal.value = true
  clearInterval(timerInterval.value)
}

const continuePlaying = () => {
  showResultModal.value = false
  gameResult.value = null
  startTimer()
}

const acceptPrize = () => {
  restartGame()
}

const restartGame = () => {
  showResultModal.value = false
  showStartModal.value = true
  currentQuestion.value = 1
  usedLifelines.value = []
  gameResult.value = null
  clearInterval(timerInterval.value)
  
  questionBank.value.forEach(question => {
    question.options.forEach(option => {
      option.visible = true
    })
  })
}

const isNextSafeLevel = (level) => {
  const nextSafe = prizes.value
    .filter(p => p.safe && p.level > currentQuestion.value)
    .sort((a, b) => a.level - b.level)[0]
  
  return nextSafe && nextSafe.level === level
}

watch(currentQuestion, (newVal) => {
  if (newVal > 0 && newVal <= 15 && !showStartModal.value && !showResultModal.value) {
    resetTimer()
  }
})

import { useQuasar } from 'quasar'
const $q = useQuasar()

onMounted(() => {
  prizes.value.sort((a, b) => b.level - a.level)
})
</script>

<style lang="scss" scoped>
.bg-dark {
  background: radial-gradient(ellipse at top, #1a0b2e 0%, #0a0118 50%, #000000 100%);
  min-height: 100vh;
  position: relative;
  overflow-x: hidden;
  
  &::before {
    content: '';
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-image: 
      radial-gradient(2px 2px at 20% 30%, white, transparent),
      radial-gradient(2px 2px at 60% 70%, white, transparent),
      radial-gradient(1px 1px at 50% 50%, white, transparent),
      radial-gradient(1px 1px at 80% 10%, white, transparent),
      radial-gradient(2px 2px at 90% 60%, white, transparent),
      radial-gradient(1px 1px at 33% 80%, white, transparent),
      radial-gradient(1px 1px at 70% 40%, white, transparent);
    background-size: 200% 200%;
    background-position: 0% 0%;
    animation: twinkle 8s ease-in-out infinite;
    pointer-events: none;
    opacity: 0.6;
  }
  
  &::after {
    content: '';
    position: fixed;
    top: 50%;
    left: 50%;
    width: 200%;
    height: 200%;
    background: radial-gradient(ellipse at center, rgba(255, 215, 0, 0.05) 0%, transparent 70%);
    transform: translate(-50%, -50%);
    animation: pulse-glow 4s ease-in-out infinite;
    pointer-events: none;
  }
}

@keyframes twinkle {
  0%, 100% { 
    opacity: 0.6;
    background-position: 0% 0%;
  }
  50% { 
    opacity: 1;
    background-position: 100% 100%;
  }
}

@keyframes pulse-glow {
  0%, 100% { opacity: 0.3; transform: translate(-50%, -50%) scale(1); }
  50% { opacity: 0.6; transform: translate(-50%, -50%) scale(1.1); }
}

.timer-circle {
  width: 120px;
  height: 120px;
  border: 4px solid gold;
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: rgba(0, 0, 0, 0.5);
  margin: 0 auto;
  position: relative;
  box-shadow: 0 0 30px rgba(255, 215, 0, 0.4),
              inset 0 0 20px rgba(255, 215, 0, 0.1);
  animation: float 3s ease-in-out infinite;
  
  &::before {
    content: '';
    position: absolute;
    inset: -4px;
    border-radius: 50%;
    padding: 4px;
    background: linear-gradient(45deg, #FFD700, #FFA500, #FFD700);
    -webkit-mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
    -webkit-mask-composite: xor;
    mask-composite: exclude;
    animation: rotate-border 3s linear infinite;
  }
  
  .text-h4 {
    font-size: 2.5rem;
    font-weight: bold;
    color: #FFD700;
    text-shadow: 0 0 20px rgba(255, 215, 0, 0.8);
    animation: pulse-number 1s ease-in-out infinite;
  }
  
  .text-caption {
    color: gold;
    font-weight: bold;
    margin-top: -8px;
    letter-spacing: 2px;
  }
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

@keyframes rotate-border {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

@keyframes pulse-number {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

.option-card {
  position: relative;
  background: linear-gradient(135deg, rgba(26, 35, 126, 0.6), rgba(13, 71, 161, 0.4));
  border: 3px solid gold;
  border-radius: 15px;
  min-height: 80px;
  transition: all 0.4s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  margin-bottom: 12px;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  
  &::before {
    content: '';
    position: absolute;
    top: -50%;
    left: -50%;
    width: 200%;
    height: 200%;
    background: radial-gradient(circle, rgba(255, 215, 0, 0.2) 0%, transparent 70%);
    opacity: 0;
    transition: opacity 0.4s;
    pointer-events: none;
  }
  
  &:hover:not(.disabled) {
    background: linear-gradient(135deg, rgba(255, 215, 0, 0.2), rgba(255, 140, 0, 0.2));
    transform: translateY(-5px) scale(1.02);
    box-shadow: 0 8px 30px rgba(255, 215, 0, 0.5);
    cursor: pointer;
    
    &::before {
      opacity: 1;
    }
    
    .option-letter {
      transform: scale(1.1) rotate(360deg);
      box-shadow: 0 0 20px rgba(255, 215, 0, 0.8);
    }
  }
  
  &.active {
    background: linear-gradient(135deg, rgba(255, 215, 0, 0.3), rgba(255, 165, 0, 0.2));
    border-color: #FFD700;
    transform: scale(1.02);
    box-shadow: 0 6px 25px rgba(255, 215, 0, 0.6);
  }
  
  &.correct {
    background: linear-gradient(135deg, #2ecc71, #27ae60);
    border-color: #4caf50;
    animation: correct-pulse 0.6s ease-in-out;
    box-shadow: 0 8px 30px rgba(46, 204, 113, 0.6);
  }
  
  &.incorrect {
    background: linear-gradient(135deg, #e74c3c, #c0392b);
    border-color: #f44336;
    animation: incorrect-shake 0.5s ease-in-out;
    box-shadow: 0 8px 30px rgba(231, 76, 60, 0.6);
  }
  
  &.disabled {
    opacity: 0.3;
    cursor: not-allowed !important;
    pointer-events: none !important;
    filter: grayscale(1);
    
    &:hover {
      transform: none !important;
    }
  }
}

@keyframes correct-pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

@keyframes incorrect-shake {
  0%, 100% { transform: translateX(0); }
  20%, 60% { transform: translateX(-10px); }
  40%, 80% { transform: translateX(10px); }
}

.option-letter {
  width: 45px;
  height: 45px;
  background: linear-gradient(135deg, #FFD700, #FFA500);
  color: #000;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: bold;
  font-size: 1.3rem;
  transition: all 0.4s ease;
  box-shadow: 0 4px 15px rgba(255, 215, 0, 0.4);
}

.action-btn {
  position: relative;
  min-width: 140px;
  height: 50px;
  font-size: 1.1rem;
  font-weight: bold;
  border-radius: 25px;
  padding: 0 20px;
  overflow: hidden;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
  
  &::before {
    content: '';
    position: absolute;
    top: -50%;
    left: -50%;
    width: 200%;
    height: 200%;
    background: linear-gradient(
      45deg,
      transparent 30%,
      rgba(255, 255, 255, 0.3) 50%,
      transparent 70%
    );
    transform: translateX(-100%);
    transition: transform 0.6s;
  }
  
  &:hover:not(:disabled) {
    transform: translateY(-3px) scale(1.05);
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.4);
    
    &::before {
      transform: translateX(100%);
    }
  }
  
  &:disabled {
    opacity: 0.4;
    cursor: not-allowed;
    filter: grayscale(1);
  }
}

.prize-panel {
  background: linear-gradient(135deg, rgba(26, 35, 126, 0.7), rgba(13, 71, 161, 0.5));
  border-radius: 20px;
  border: 3px solid gold;
  padding: 10px;
  height: auto;
  max-height: 80vh;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5),
              0 0 30px rgba(255, 215, 0, 0.3);
  animation: panel-glow 3s ease-in-out infinite;
  position: relative;
  
  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: radial-gradient(ellipse at top, rgba(255, 215, 0, 0.1), transparent 70%);
    pointer-events: none;
    border-radius: 20px;
  }
}

@keyframes panel-glow {
  0%, 100% {
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5),
                0 0 20px rgba(255, 215, 0, 0.2);
  }
  50% {
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5),
                0 0 40px rgba(255, 215, 0, 0.5);
  }
}

.prize-list {
  max-height: 60vh;
  overflow-y: auto;
  padding-right: 5px;
  position: relative;
  
  &::-webkit-scrollbar {
    width: 6px;
  }
  
  &::-webkit-scrollbar-track {
    background: rgba(255, 255, 255, 0.1);
    border-radius: 10px;
  }
  
  &::-webkit-scrollbar-thumb {
    background: linear-gradient(180deg, #FFD700, #FFA500);
    border-radius: 10px;
  }
}

.prize-item {
  padding: 8px 10px;
  margin: 6px 0;
  border-radius: 8px;
  font-size: 0.9rem;
  min-height: 32px;
  line-height: 1.2;
  transition: all 0.3s ease;
  background: rgba(255, 255, 255, 0.03);
  border: 2px solid transparent;
  
  .text-h6 {
    font-size: 0.95rem;
    margin: 0;
    font-weight: 600;
    color:white !important;
  }
  
  &.current {
    background: linear-gradient(135deg, #FFD700, #FFA500);
    color: black;
    font-weight: bold;
    transform: scale(1.05);
    border-color: #FFD700;
    box-shadow: 0 5px 20px rgba(255, 215, 0, 0.6);
    animation: current-pulse 1.5s ease-in-out infinite;
    
    .text-h6 {
      color: white !important;
    }
  }
  
  &.passed {
    color: #4caf50;
    text-decoration: line-through;
    opacity: 0.6;
  }
  
  &.safe-level {
    background: rgba(255, 215, 0, 0.15);
    border-left: 4px solid gold;
  }
  
  &.next-safe {
    border: 2px dashed gold;
    animation: next-safe-glow 2s ease-in-out infinite;
  }
}

@keyframes current-pulse {
  0%, 100% {
    box-shadow: 0 5px 20px rgba(255, 215, 0, 0.4);
  }
  50% {
    box-shadow: 0 5px 30px rgba(255, 215, 0, 0.8);
  }
}

@keyframes next-safe-glow {
  0%, 100% { border-color: rgba(255, 215, 0, 0.4); }
  50% { border-color: rgba(255, 215, 0, 1); }
}

.current-prize {
  background: linear-gradient(135deg, #FFD700, #FFA500);
  border-radius: 15px;
  padding: 15px;
  margin-top: 15px;
  box-shadow: 0 8px 25px rgba(255, 215, 0, 0.5);
  position: relative;
  overflow: hidden;
  
  &::before {
    content: '';
    position: absolute;
    top: -50%;
    left: -50%;
    width: 200%;
    height: 200%;
    background: radial-gradient(circle, rgba(255, 255, 255, 0.3) 0%, transparent 70%);
    animation: shine-rotate 3s linear infinite;
  }
  
  .text-caption {
    color: #000;
    font-weight: bold;
    letter-spacing: 2px;
    position: relative;
    z-index: 1;
  }
  
  .text-h5 {
    color: #000;
    font-weight: bold;
    font-size: 1.8rem;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
    position: relative;
    z-index: 1;
  }
}

@keyframes shine-rotate {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.start-modal, .result-modal {
  background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
  border: 4px solid gold;
  border-radius: 25px;
  max-width: 600px;
  width: 90%;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.7),
              0 0 40px rgba(255, 215, 0, 0.3);
  position: relative;
  overflow: hidden;
  
  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: radial-gradient(circle at 50% 0%, rgba(255, 215, 0, 0.1), transparent 70%);
    animation: modal-glow 3s ease-in-out infinite;
  }
}

@keyframes modal-glow {
  0%, 100% { opacity: 0.5; }
  50% { opacity: 1; }
}

.start-btn {
  position: relative;
  background: linear-gradient(135deg, #FFD700, #FFA500);
  color: black;
  font-size: 1.5rem;
  font-weight: bold;
  padding: 15px 50px;
  border-radius: 30px;
  margin-top: 20px;
  overflow: hidden;
  box-shadow: 0 8px 25px rgba(255, 215, 0, 0.5);
  transition: all 0.3s ease;
  
  &::before {
    content: '';
    position: absolute;
    top: -50%;
    left: -50%;
    width: 200%;
    height: 200%;
    background: linear-gradient(
      45deg,
      transparent 30%,
      rgba(255, 255, 255, 0.4) 50%,
      transparent 70%
    );
    transform: translateX(-100%);
    transition: transform 0.6s;
  }
  
  &:hover {
    transform: translateY(-3px) scale(1.05);
    box-shadow: 0 12px 35px rgba(255, 215, 0, 0.7);
    
    &::before {
      transform: translateX(100%);
    }
  }
}

.lifeline-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 15px;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 15px;
  min-width: 100px;
  border: 2px solid rgba(255, 215, 0, 0.3);
  transition: all 0.3s ease;
  
  &:hover {
    transform: translateY(-5px);
    border-color: rgba(255, 215, 0, 0.6);
    box-shadow: 0 8px 20px rgba(255, 215, 0, 0.3);
  }
  
  .text-caption {
    color: gold;
    margin-top: 5px;
    font-weight: bold;
  }
}

.safe-level-indicator {
  padding: 10px 22px;
  background: rgba(255, 215, 0, 0.2);
  border: 2px solid gold;
  border-radius: 25px;
  color: gold;
  font-weight: bold;
  transition: all 0.3s ease;
  
  &:hover {
    transform: scale(1.1);
    box-shadow: 0 5px 20px rgba(255, 215, 0, 0.5);
  }
}

.prize-result {
  padding: 30px;
  background: rgba(0, 0, 0, 0.6);
  border-radius: 20px;
  border: 3px solid gold;
  margin: 25px 0;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5),
              inset 0 0 30px rgba(255, 215, 0, 0.1);
  position: relative;
  overflow: hidden;
  
  &::before {
    content: '';
    position: absolute;
    top: -50%;
    left: -50%;
    width: 200%;
    height: 200%;
    background: radial-gradient(circle, rgba(255, 215, 0, 0.2) 0%, transparent 70%);
    animation: prize-glow 2s ease-in-out infinite;
  }
  
  .text-caption {
    color: gold;
    position: relative;
    z-index: 1;
  }
  
  .text-h3 {
    color: gold;
    font-weight: bold;
    text-shadow: 0 0 30px rgba(255, 215, 0, 0.8);
    position: relative;
    z-index: 1;
    animation: prize-pulse 1.5s ease-in-out infinite;
  }
}

@keyframes prize-glow {
  0%, 100% { opacity: 0.3; transform: rotate(0deg); }
  50% { opacity: 0.6; transform: rotate(180deg); }
}

@keyframes prize-pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

.audience-help-overlay,
.call-help-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.95);
  z-index: 1000;
  display: flex;
  justify-content: center;
  align-items: center;
  backdrop-filter: blur(10px);
  animation: overlay-appear 0.3s ease-out;
}

@keyframes overlay-appear {
  from { opacity: 0; }
  to { opacity: 1; }
}

.audience-help-content,
.call-help-content {
  background: linear-gradient(135deg, #0b1c3d, #102a5c);
  border: 4px solid gold;
  border-radius: 25px;
  padding: 35px;
  max-width: 800px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.7),
              0 0 50px rgba(255, 215, 0, 0.4);
  animation: modal-slide-in 0.5s cubic-bezier(0.68, -0.55, 0.265, 1.55);
}

@keyframes modal-slide-in {
  from {
    opacity: 0;
    transform: translateY(-50px) scale(0.9);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.audience-chart {
  .chart-bar-container {
    display: flex;
    align-items: center;
    margin: 25px 0;
    padding: 15px;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 15px;
    transition: all 0.3s ease;
    
    &:hover {
      background: rgba(255, 255, 255, 0.08);
      transform: translateX(5px);
    }
    
    .chart-label {
      width: 45px;
      height: 45px;
      background: linear-gradient(135deg, #FFD700, #FFA500);
      color: black;
      border-radius: 50%;
      display: flex;
      justify-content: center;
      align-items: center;
      font-weight: bold;
      font-size: 1.3rem;
      margin-right: 20px;
      box-shadow: 0 4px 15px rgba(255, 215, 0, 0.4);
    }
    
    .chart-bar-background {
      flex: 1;
      height: 45px;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 25px;
      overflow: hidden;
      position: relative;
      
      .chart-bar-fill {
        height: 100%;
        border-radius: 25px;
        transition: width 1.5s cubic-bezier(0.68, -0.55, 0.265, 1.55);
        position: relative;
        overflow: hidden;
        
        &::before {
          content: '';
          position: absolute;
          top: 0;
          left: -100%;
          width: 100%;
          height: 100%;
          background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.4), transparent);
          animation: bar-shine 2s ease-in-out infinite;
        }
        
        &.bar-a { background: linear-gradient(90deg, #FF5252, #FF8A80); }
        &.bar-b { background: linear-gradient(90deg, #4CAF50, #81C784); }
        &.bar-c { background: linear-gradient(90deg, #2196F3, #64B5F6); }
        &.bar-d { background: linear-gradient(90deg, #FF9800, #FFB74D); }
        
        .chart-percentage {
          position: absolute;
          right: 20px;
          top: 50%;
          transform: translateY(-50%);
          color: white;
          font-weight: bold;
          font-size: 1.2rem;
          text-shadow: 2px 2px 4px rgba(0,0,0,0.7);
          z-index: 1;
        }
      }
    }
    
    .chart-text {
      width: 220px;
      margin-left: 20px;
      color: white;
      font-size: 1.1rem;
      font-weight: 500;
    }
  }
}

@keyframes bar-shine {
  to { left: 100%; }
}

.audience-crowd {
  margin-top: 30px;
  padding: 25px;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 20px;
  
  .crowd-icons {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 12px;
    
    .q-icon {
      animation: crowd-bounce 2s ease-in-out infinite;
      transition: transform 0.3s ease;
      
      &:hover {
        transform: scale(1.3) translateY(-10px);
      }
    }
  }
}

@keyframes crowd-bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}

.call-animation {
  text-align: center;
  margin: 25px 0;
  
  .phone-ring {
    display: inline-block;
    animation: phone-ring 1.5s ease-in-out infinite;
  }
}

@keyframes phone-ring {
  0%, 100% { transform: rotate(0deg); }
  10%, 30% { transform: rotate(-15deg); }
  20%, 40% { transform: rotate(15deg); }
}

.call-friend-info {
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.1);
  padding: 25px;
  border-radius: 20px;
  margin-bottom: 25px;
  border: 2px solid rgba(255, 215, 0, 0.3);
  
  .friend-avatar {
    width: 85px;
    height: 85px;
    background: linear-gradient(135deg, #667eea, #764ba2);
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-right: 20px;
    box-shadow: 0 8px 20px rgba(102, 126, 234, 0.4);
    position: relative;
    
    &::before {
      content: '';
      position: absolute;
      inset: -5px;
      border-radius: 50%;
      border: 3px solid #FFD700;
      animation: avatar-pulse 2s ease-in-out infinite;
    }
    
    .q-icon {
      color: white;
    }
  }
  
  .friend-details {
    color: white;
    
    .text-h6 {
      margin: 0 0 5px 0;
      color: gold;
      font-size: 1.3rem;
    }
  }
}

@keyframes avatar-pulse {
  0%, 100% { transform: scale(1); opacity: 0.7; }
  50% { transform: scale(1.1); opacity: 1; }
}

.call-message {
  background: rgba(255, 255, 255, 0.08);
  border-radius: 20px;
  
  .message-bubble {
    background: linear-gradient(135deg, #2c3e50, #34495e);
    padding: 25px;
    border-radius: 20px;
    position: relative;
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
    
    &::before {
      content: '';
      position: absolute;
      top: -12px;
      left: 35px;
      width: 0;
      height: 0;
      border-left: 12px solid transparent;
      border-right: 12px solid transparent;
      border-bottom: 12px solid #2c3e50;
    }
  }
}

.confidence-meter {
  .meter-labels {
    color: white;
    font-size: 0.95rem;
    margin-bottom: 8px;
    font-weight: 500;
  }
  
  .meter-bar {
    height: 25px;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 15px;
    overflow: hidden;
    box-shadow: inset 0 2px 8px rgba(0, 0, 0, 0.3);
    
    .meter-fill {
      height: 100%;
      border-radius: 15px;
      transition: width 1.5s cubic-bezier(0.68, -0.55, 0.265, 1.55);
      position: relative;
      overflow: hidden;
      
      &::before {
        content: '';
        position: absolute;
        top: 0;
        left: -100%;
        width: 100%;
        height: 100%;
        background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
        animation: confidence-shine 2s ease-in-out infinite;
      }
      
      &.high-confidence {
        background: linear-gradient(90deg, #00C853, #64DD17);
      }
      
      &.medium-high-confidence {
        background: linear-gradient(90deg, #FFD600, #FFEA00);
      }
      
      &.medium-confidence {
        background: linear-gradient(90deg, #FF9100, #FFAB00);
      }
      
      &.low-confidence {
        background: linear-gradient(90deg, #FF3D00, #FF6E40);
      }
    }
  }
}

@keyframes confidence-shine {
  to { left: 100%; }
}

.text-yellow-8 {
  color: gold !important;
  text-shadow: 0 0 10px rgba(255, 215, 0, 0.5);
}

.text-h3,
.text-h4,
.text-h5 {
  text-shadow: 2px 2px 8px rgba(0, 0, 0, 0.5);
}

.q-btn[color="warning"] {
  background: linear-gradient(135deg, #ff4444, #cc0000);
  color: white;
  font-weight: bold;
  border: 2px solid #ff6666;
  transition: all 0.3s ease;
  
  &:hover {
    background: linear-gradient(135deg, #ff6666, #ff0000);
    transform: scale(1.05);
    box-shadow: 0 6px 20px rgba(255, 68, 68, 0.5);
  }
}

.q-btn[outline] {
  border: 2px solid gold;
  color: gold;
  transition: all 0.3s ease;
  
  &:hover {
    background: rgba(255, 215, 0, 0.15);
    transform: scale(1.05);
    box-shadow: 0 4px 15px rgba(255, 215, 0, 0.3);
  }
}

@media (max-width: 768px) {
  .timer-circle {
    width: 100px;
    height: 100px;
    
    .text-h4 {
      font-size: 2rem;
    }
  }
  
  .option-card {
    min-height: 70px;
    
    .text-h6 {
      font-size: 1rem;
      color: white !important;
    }
  }
  
  .option-letter {
    width: 38px;
    height: 38px;
    font-size: 1.1rem;
  }
  
  .action-btn {
    min-width: 120px;
    height: 48px;
    font-size: 1rem;
    padding: 0 15px;
  }
  
  .prize-item {
    padding: 6px 8px;
    font-size: 0.85rem;
  }
  
  .start-modal, .result-modal {
    margin: 10px;
    
    .text-h4 {
      font-size: 1.8rem;
    }
    
    .text-h6 {
      font-size: 1.1rem;
      color: white !important;
    }
  }
  
  .start-btn {
    font-size: 1.3rem;
    padding: 12px 35px;
  }
  
  .lifeline-item {
    min-width: 85px;
    padding: 12px;
  }
  
  .current-prize {
    .text-h5 {
      font-size: 1.5rem;
    }
  }
  
  .audience-help-content,
  .call-help-content {
    padding: 20px;
    width: 95%;
  }
  
  .audience-chart .chart-bar-container {
    flex-direction: column;
    align-items: flex-start;
    
    .chart-text {
      width: 100%;
      margin-left: 0;
      margin-top: 10px;
      text-align: center;
    }
  }
  
  .call-friend-info {
    flex-direction: column;
    text-align: center;
    
    .friend-avatar {
      margin-right: 0;
      margin-bottom: 15px;
    }
  }
}

@media (max-width: 480px) {
  .timer-circle {
    width: 85px;
    height: 85px;
    
    .text-h4 {
      font-size: 1.6rem;
    }
  }
  
  .option-card {
    min-height: 65px;
    
    .text-h6 {
      font-size: 0.95rem;
      color: white !important;
    }
  }
  
  .action-btn {
    min-width: 100px;
    height: 45px;
    font-size: 0.95rem;
  }
  
  .prize-item {
    font-size: 0.8rem;
  }
}

.text-h6{
  color: white !important;
}
</style>