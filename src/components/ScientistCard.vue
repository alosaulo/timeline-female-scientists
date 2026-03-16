<script setup lang="ts">
import { ref } from 'vue'
import { useDraggable } from '@dnd-kit/vue'

const props = withDefaults(
  defineProps<{
    card: { id: string; nome: string; ano: number; area?: string; contribuicao?: string }
    showYear?: boolean
    draggable?: boolean
  }>(),
  {
    showYear: true,
    draggable: false,
  },
)

const element = ref<HTMLElement | null>(null)

const { isDragging } = useDraggable({
  id: () => props.card.id,
  element,
  data: () => props.card,
  disabled: () => !props.draggable,
})
</script>

<template>
  <div
    ref="element"
    :class="['card', { 'card--dragging': isDragging, 'card--draggable': draggable }]"
  >
    <div class="card-header">
      <h3 class="name">{{ card.nome }}</h3>
      <p v-if="card.area" class="area">{{ card.area }}</p>
    </div>
    <div class="card-body">
      <p v-if="card.contribuicao" class="contribuicao">{{ card.contribuicao }}</p>
    </div>
    <div class="card-footer">
      <p v-if="showYear" class="year">{{ card.ano }}</p>
      <p v-else class="year-hidden">????</p>
    </div>
  </div>
</template>
