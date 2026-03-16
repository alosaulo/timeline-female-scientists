<script setup lang="ts">
import { ref } from 'vue'
import { DragDropProvider, type DragDropEventHandlers, DragOverlay } from '@dnd-kit/vue'
import Papa from 'papaparse'
import csvData from '../../public/data/cientistas.csv?raw'
import ScientistCard from './ScientistCard.vue'
import DropZone from './DropZone.vue'

interface Scientist {
  id: string
  nome: string
  ano: number
  area: string
  contribuicao: string
}

// Ler as cartas do CSV via papaparse
const parsed = Papa.parse<{
  numero: string
  nome: string
  ano: string
  area: string
  contribuicao: string
}>(csvData, { header: true, skipEmptyLines: true })

// Base de cartas
const allCards: Scientist[] = parsed.data
  .map((row) => ({
    id: row.numero,
    nome: row.nome,
    ano: parseInt(row.ano, 10),
    area: row.area,
    contribuicao: row.contribuicao,
  }))
  .sort(() => Math.random() - 0.5)

const deck = ref<Scientist[]>([...allCards])
const board = ref<Scientist[]>([deck.value.pop()!])
const hand = ref<Scientist[]>([
  deck.value.pop()!,
  deck.value.pop()!,
  deck.value.pop()!,
  deck.value.pop()!,
])

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
    hand.value.splice(handIndex, 1)
    board.value.splice(targetIndex, 0, playedCard)

    // Se a mão não estiver cheia e houver cartas, compra
    if (hand.value.length < 4 && deck.value.length > 0) {
      hand.value.push(deck.value.pop()!)
    }
  } else {
    // Errou (descarta e compra nova carta)
    setTimeout(() => {
      alert(`Incorreto! O ano era ${playedCard.ano}. A carta foi descartada.`)
    }, 10)
    hand.value.splice(handIndex, 1)
    if (deck.value.length > 0) {
      hand.value.push(deck.value.pop()!)
    }
  }
}
</script>

<template>
  <DragDropProvider @dragEnd="handleDragEnd">
    <div class="game-area">
      <!-- Tabuleiro -->
      <section class="timeline-section">
        <h2>Linha do Tempo</h2>
        <div class="board">
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
