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

// Lê CSV com ';' e mantém a contribuição íntegra mesmo quando ela contém ';'
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
    const ilustracao = parts[parts.length - 1]?.trim() ?? ''
    const contribuicao = parts.slice(4, -1).join(';').trim()

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

    // Se a mão não estiver cheia e houver cartas, compra
    if (hand.value.length < INITIAL_HAND_SIZE && deck.value.length > 0) {
      hand.value.push(deck.value.pop()!)
    }
  } else {
    // Errou (descarta e compra nova carta)
    score.value = Math.max(0, score.value - 5)
    wrongMoves.value += 1
    setTimeout(() => {
      alert(`Incorreto! O ano era ${playedCard.ano}. A carta foi descartada.`)
    }, 10)
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
          <DropZone :index="0" />
          <template v-for="(card, index) in board" :key="card.id">
            <ScientistCard :card="card" :showYear="true" :draggable="false" />
            <DropZone :index="index + 1" />
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
  </DragDropProvider>
</template>
