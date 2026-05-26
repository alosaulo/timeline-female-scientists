<script setup lang="ts">
import { computed, ref } from 'vue'
import { DragDropProvider, type DragDropEventHandlers, DragOverlay } from '@dnd-kit/vue'
import csvData from '../../public/data/cientistas.csv?raw'
import ScientistCard from './ScientistCard.vue'
import DropZone from './DropZone.vue'

interface Scientist {
  id: string
  nome: string
  ano: number
  area: string
  contribuicao: string
  ilustracao: string
}

const INITIAL_HAND_SIZE = 4
const INITIAL_BOARD_SIZE = 1
const MINIMUM_START_CARDS = INITIAL_HAND_SIZE + INITIAL_BOARD_SIZE

const GAME_DECK_SIZE = 25

const showModal = ref(false)
const modalType = ref<'correct' | 'incorrect'>('correct')
const modalCard = ref<Scientist | null>(null)
const modalTargetIndex = ref(0)

function closeModal() {
  showModal.value = false
}

const allCards: Scientist[] = csvData
  .split(/\r?\n/)
  .slice(1)
  .map((line) => line.trim())
  .filter(Boolean)
  .map((line) => {
    const parts = line.split(';')

    if (parts.length < 6) return null

    const id = parts[0]?.trim() ?? ''
    const nome = parts[1]?.trim() ?? ''
    const ano = parseInt(parts[2] ?? '', 10)
    const area = parts[3]?.trim() ?? ''
    const contribuicao = parts[4]?.trim() ?? ''
    const ilustracao = parts[5]?.trim() ?? ''

    if (!id || !nome || Number.isNaN(ano)) return null

    return {
      id,
      nome,
      ano,
      area,
      contribuicao,
      ilustracao,
    }
  })
  .filter((row): row is Scientist => row !== null)
  .sort(() => Math.random() - 0.5)

const effectiveDeckSize = Math.max(MINIMUM_START_CARDS, Math.min(GAME_DECK_SIZE, allCards.length))
const gameCards = allCards.slice(0, effectiveDeckSize)

const deck = ref<Scientist[]>([...gameCards])
const board = ref<Scientist[]>([deck.value.pop()!])
const hand = ref<Scientist[]>([
  deck.value.pop()!,
  deck.value.pop()!,
  deck.value.pop()!,
  deck.value.pop()!,
])

const boardElement = ref<HTMLElement | null>(null)
const isPanningBoard = ref(false)
const panStartX = ref(0)
const initialScrollLeft = ref(0)
const score = ref(0)
const correctMoves = ref(0)
const wrongMoves = ref(0)

const totalMoves = computed(() => correctMoves.value + wrongMoves.value)
const accuracy = computed(() => {
  if (totalMoves.value === 0) return 0
  return Math.round((correctMoves.value / totalMoves.value) * 100)
})

type DragEndEvent = Parameters<NonNullable<DragDropEventHandlers['onDragEnd']>>[0]

function handleDragEnd(event: DragEndEvent) {
  if (event.canceled) return

  const { operation } = event
  const { source, target } = operation

  if (!target) return

  const cardId = source?.id

  if (!cardId || !target) return

  let targetIndex = target?.data?.index
  if (
    typeof targetIndex !== 'number' &&
    typeof target?.id === 'string' &&
    target.id.startsWith('drop-')
  ) {
    targetIndex = parseInt(target.id.replace('drop-', ''), 10)
  }

  if (typeof targetIndex !== 'number') return

  const handIndex = hand.value.findIndex((c) => c.id === cardId)
  if (handIndex === -1) return

  const playedCard = hand.value[handIndex]
  if (!playedCard) return

  // Verificar se a posição está correta cronologicamente
  const previousCard = targetIndex > 0 ? board.value[targetIndex - 1] : null
  const nextCard = targetIndex < board.value.length ? board.value[targetIndex] : null

  let isCorrect = true
  if (previousCard && playedCard.ano < previousCard.ano) isCorrect = false
  if (nextCard && playedCard.ano > nextCard.ano) isCorrect = false

  if (isCorrect) {
    // Acertou
    score.value += 10
    correctMoves.value += 1
    hand.value.splice(handIndex, 1)
    board.value.splice(targetIndex, 0, playedCard)

    // Mostrar modal de acerto
    modalType.value = 'correct'
    modalCard.value = playedCard
    modalTargetIndex.value = targetIndex
    showModal.value = true

    if (hand.value.length < INITIAL_HAND_SIZE && deck.value.length > 0) {
      hand.value.push(deck.value.pop()!)
    }
  } else {
    // Errou (descarta e compra nova carta)
    score.value = Math.max(0, score.value - 5)
    wrongMoves.value += 1
    modalType.value = 'incorrect'
    modalCard.value = playedCard
    modalTargetIndex.value = targetIndex
    showModal.value = true
    hand.value.splice(handIndex, 1)
    if (deck.value.length > 0) {
      hand.value.push(deck.value.pop()!)
    }
  }
}

function handleBoardPointerDown(event: PointerEvent) {
  if (event.button !== 0 || !boardElement.value) return

  isPanningBoard.value = true
  panStartX.value = event.clientX
  initialScrollLeft.value = boardElement.value.scrollLeft
  boardElement.value.setPointerCapture(event.pointerId)
}

function handleBoardPointerMove(event: PointerEvent) {
  if (!isPanningBoard.value || !boardElement.value) return

  const dragDelta = event.clientX - panStartX.value
  boardElement.value.scrollLeft = initialScrollLeft.value - dragDelta
}

function endBoardPointerPan(event: PointerEvent) {
  if (!boardElement.value) return

  if (boardElement.value.hasPointerCapture(event.pointerId)) {
    boardElement.value.releasePointerCapture(event.pointerId)
  }

  isPanningBoard.value = false
}

</script>

<style scoped>
/* Modal overlay e animação — sem scoped para o Teleport funcionar bem */
.modal-overlay {
  position: fixed;
  inset: 0;
  z-index: 1000;
  display: flex;
  justify-content: center;
  align-items: center;
  background: rgba(15, 20, 60, 0.55);
  backdrop-filter: blur(4px);
  padding: 16px;
}

.modal-card {
  position: relative;
  width: min(100%, 340px);
  padding: 32px 28px 28px;
  border-radius: 20px;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  box-shadow: 0 16px 48px rgba(19, 35, 112, 0.3);
  overflow: hidden;
}

.modal-card--correct {
  background: linear-gradient(180deg, #e8f5e9 0%, #c8e6c9 100%);
  border: 3px solid #4caf50;
  color: #1b5e20;
}

.modal-card--incorrect {
  background: linear-gradient(180deg, #fce4ec 0%, #f8bbd0 100%);
  border: 3px solid #e35599;
  color: #880e4f;
}

.modal-ornament {
  position: absolute;
  top: -40px;
  right: -40px;
  width: 120px;
  height: 120px;
  border-radius: 50%;
  opacity: 0.12;
  background: currentColor;
  pointer-events: none;
}

.modal-icon {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.8rem;
  font-weight: 700;
  color: #fff;
  line-height: 1;
}

.modal-card--correct .modal-icon {
  background: linear-gradient(135deg, #66bb6a, #43a047);
}

.modal-card--incorrect .modal-icon {
  background: linear-gradient(135deg, #e35599, #c2185b);
}

.modal-title {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.modal-scientist-name {
  margin: 0;
  font-size: 1.1rem;
  font-weight: 600;
  opacity: 0.85;
}

.modal-year-badge {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 16px;
  border-radius: 30px;
  background: rgba(255, 255, 255, 0.7);
  backdrop-filter: blur(4px);
  border: 1px solid rgba(255, 255, 255, 0.9);
}

.modal-year-label {
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  opacity: 0.7;
  font-weight: 600;
}

.modal-year-value {
  font-size: 1.3rem;
  font-weight: 800;
}

.modal-extra {
  margin: 0;
  font-size: 0.85rem;
  opacity: 0.75;
}

.modal-btn {
  margin-top: 4px;
  padding: 10px 32px;
  border: none;
  border-radius: 30px;
  font-size: 0.95rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  cursor: pointer;
  transition: transform 0.15s, box-shadow 0.15s;
}

.modal-card--correct .modal-btn {
  background: linear-gradient(180deg, #43a047, #388e3c);
  color: #fff;
  box-shadow: 0 4px 12px rgba(67, 160, 71, 0.35);
}

.modal-card--incorrect .modal-btn {
  background: linear-gradient(180deg, #e35599, #c2185b);
  color: #fff;
  box-shadow: 0 4px 12px rgba(227, 85, 153, 0.35);
}

.modal-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.2);
}

.modal-btn:active {
  transform: translateY(0);
}

/* Transição de entrada/saída */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.25s ease;
}

.modal-enter-active .modal-card,
.modal-leave-active .modal-card {
  transition: transform 0.25s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .modal-card {
  transform: scale(0.85) translateY(20px);
}

.modal-leave-to .modal-card {
  transform: scale(0.9) translateY(-10px);
}
</style>

<template>
  <DragDropProvider @dragEnd="handleDragEnd">
    <div class="game-area">
      <section class="score-section" aria-live="polite">
        <h2>Pontuacao</h2>
        <div class="score-board">
          <p class="score-main">Score: {{ score }}</p>
          <p>Acertos: {{ correctMoves }}</p>
          <p>Erros: {{ wrongMoves }}</p>
          <p>Precisao: {{ accuracy }}%</p>
          <p>Cartas no deck: {{ deck.length }}</p>
        </div>
      </section>

      <!-- Tabuleiro -->
      <section class="timeline-section">
        <h2>Linha do Tempo</h2>
        <div
          ref="boardElement"
          :class="['board', { 'board--panning': isPanningBoard }]"
          @pointerdown="handleBoardPointerDown"
          @pointermove="handleBoardPointerMove"
          @pointerup="endBoardPointerPan"
          @pointercancel="endBoardPointerPan"
          @pointerleave="endBoardPointerPan"
        >
          <DropZone :index="0" :compact="true" />
          <template v-for="(card, index) in board" :key="card.id">
            <ScientistCard :card="card" :showYear="true" :draggable="false" :compact="true" />
            <DropZone :index="index + 1" :compact="true" />
          </template>
        </div>
      </section>

      <!-- Mão do jogador -->
      <section class="hand-section">
        <h2>Sua Mão</h2>
        <div class="hand">
          <ScientistCard
            v-for="card in hand"
            :key="card.id"
            :card="card"
            :showYear="false"
            :draggable="true"
          />
        </div>
        <p v-if="hand.length === 0" class="win-message">
          Parabéns! Você colocou todas as cartas na linha do tempo!
        </p>
      </section>

      <DragOverlay>
        <template #default="{ source }">
          <ScientistCard
            v-if="source?.data"
            :card="source.data"
            :showYear="false"
            :draggable="false"
          />
        </template>
      </DragOverlay>
    </div>

    <Teleport to="body">
      <Transition name="modal">
        <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
          <div :class="['modal-card', `modal-card--${modalType}`]">
            <div class="modal-ornament" aria-hidden="true"></div>

            <div class="modal-icon">
              {{ modalType === 'correct' ? '✓' : '✗' }}
            </div>

            <h3 class="modal-title">
              {{ modalType === 'correct' ? 'Correto!' : 'Incorreto!' }}
            </h3>

            <p class="modal-scientist-name">{{ modalCard?.nome }}</p>

            <div class="modal-year-badge">
              <span class="modal-year-label">Ano</span>
              <span class="modal-year-value">{{ modalCard?.ano }}</span>
            </div>

            <p class="modal-extra" v-if="modalCard?.area">
              Área: {{ modalCard.area }}
            </p>

            <button class="modal-btn" @click="closeModal">
              {{ modalType === 'correct' ? 'Continuar' : 'Descartar' }}
            </button>
          </div>
        </div>
      </Transition>
    </Teleport>

  </DragDropProvider>
</template>
