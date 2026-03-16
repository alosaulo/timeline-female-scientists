<script setup lang="ts">
import { ref } from 'vue'
import { useDraggable } from '@dnd-kit/vue'

const props = withDefaults(
  defineProps<{
    card: { id: string; nome: string; ano: number; area?: string; contribuicao?: string; ilustracao?: string }
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
      <img v-if="card.ilustracao" :src="card.ilustracao" :alt="card.nome" class="ilustracao" />
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

<style scoped>
.ilustracao {
  width: 100%;
  max-height: 120px;
  object-fit: contain;
  display: block;
  margin-bottom: 0.5rem;
}
</style>
