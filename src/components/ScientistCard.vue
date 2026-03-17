<script setup lang="ts">
import { computed, ref } from 'vue'
import { useDraggable } from '@dnd-kit/vue'

type ScientistCardData = {
  id: string
  nome: string
  ano: number
  area?: string
  contribuicao?: string
  ilustracao?: string
}

const props = withDefaults(
  defineProps<{
    card: ScientistCardData
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

const displayYear = computed(() => (props.showYear ? String(props.card.ano) : '????'))
const hasDescription = computed(() => Boolean(props.card.area || props.card.contribuicao))
const illustrationFallback = computed(() =>
  props.card.nome
    .split(' ')
    .filter(Boolean)
    .slice(0, 2)
    .map((name) => name[0]?.toUpperCase() ?? '')
    .join(''),
)
</script>

<template>
  <div
    ref="element"
    :class="['card', { 'card--dragging': isDragging, 'card--draggable': draggable }]"
  >
    <header class="card-header">
      <h3 class="name">{{ card.nome }}</h3>
    </header>

    <div class="card-illustration">
      <div class="card-illustration__frame">
        <div class="card-illustration__backdrop" aria-hidden="true"></div>
        <div class="card-ornament card-ornament--top" aria-hidden="true"></div>
        <div class="card-ornament card-ornament--bottom" aria-hidden="true"></div>

        <img v-if="card.ilustracao" :src="card.ilustracao" :alt="card.nome" class="ilustracao" />
        <div v-else class="ilustracao ilustracao--fallback">
          {{ illustrationFallback }}
        </div>
      </div>
    </div>

    <p class="card-section-title">Area de atuacao</p>

    <div class="card-body">
      <p v-if="card.area" class="area">{{ card.area }}</p>
      <p v-if="card.contribuicao" class="contribuicao">{{ card.contribuicao }}</p>
      <p v-if="!hasDescription" class="contribuicao contribuicao--empty">
        Informacoes em atualizacao.
      </p>
    </div>

    <footer class="card-footer">
      <span class="year-ribbon">{{ displayYear }}</span>
    </footer>
  </div>
</template>

<style scoped>
.card {
  --card-blue-dark: #1d28cf;
  --card-frame: #7a92f0;
  --card-surface: #fdfdff;
  --card-ribbon: #e35599;
  --card-ribbon-shadow: #b73d78;
  --card-text: #1f2240;
  --card-shadow: rgba(19, 35, 112, 0.18);
  --card-inner-gap: clamp(6px, 0.9vw, 8px);
  --card-edge-padding: clamp(4px, 0.8vw, 6px);
  --card-bottom-padding: clamp(10px, 1.6vw, 12px);

  flex: 0 0 var(--card-width);
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: stretch;
  width: var(--card-width);
  height: auto;
  min-height: var(--card-height);
  padding: var(--card-edge-padding) var(--card-edge-padding) var(--card-bottom-padding);
  border: 3px solid var(--card-frame);
  border-radius: 4px;
  background: linear-gradient(180deg, #b1c3ff 0%, #8da5ff 100%);
  box-shadow: 0 10px 20px var(--card-shadow);
  gap: var(--card-inner-gap);
  text-align: center;
  transition: transform 0.2s;
  overflow: visible;
}

.card--draggable {
  cursor: grab;
  background: linear-gradient(180deg, #b1c3ff 0%, #8da5ff 100%);
}

.card-header,
.card-section-title {
  margin: 0;
  padding: clamp(3px, 0.7vw, 4px) clamp(6px, 1vw, 8px);
  border: 1px solid rgba(255, 255, 255, 0.35);
  background: linear-gradient(180deg, #2d34ec 0%, var(--card-blue-dark) 100%);
  color: #fff;
  text-align: center;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  text-shadow: 0 1px 0 rgba(6, 10, 65, 0.55);
}

.card-header {
  min-height: 26px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.name {
  margin: 0;
  font-family: 'Arial Narrow', 'Franklin Gothic Heavy', sans-serif;
  font-size: clamp(0.62rem, 1vw, 0.75rem);
  line-height: 1.05;
  display: -webkit-box;
  line-clamp: 2;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.card-illustration {
  display: flex;
}

.card-illustration__frame {
  position: relative;
  width: 100%;
  min-height: clamp(110px, calc(var(--card-height) * 0.37), 146px);
  border: 2px solid var(--card-frame);
  background: var(--card-surface);
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: clamp(8px, 1.2vw, 12px) clamp(8px, 1vw, 10px);
  isolation: isolate;
}

.card-illustration__backdrop {
  position: absolute;
  top: clamp(10px, 1.4vw, 14px);
  left: 50%;
  width: clamp(90px, calc(var(--card-width) * 0.62), 122px);
  height: clamp(90px, calc(var(--card-width) * 0.62), 122px);
  border-radius: 999px;
  background: #6c87ea;
  transform: translateX(-50%);
  z-index: -2;
}

.card-ornament {
  position: absolute;
  width: clamp(42px, calc(var(--card-width) * 0.31), 62px);
  height: clamp(42px, calc(var(--card-width) * 0.31), 62px);
  opacity: 0.88;
  background:
    radial-gradient(circle at center, #ffffff 0 42%, transparent 43%) 0 0 / 18px 18px,
    radial-gradient(circle at center, #ffffff 0 42%, transparent 43%) 9px 9px / 18px 18px,
    radial-gradient(circle at center, rgba(124, 147, 241, 0.8) 0 28%, transparent 29%) 0 0 /
      18px 18px,
    radial-gradient(circle at center, rgba(124, 147, 241, 0.8) 0 28%, transparent 29%) 9px 9px /
      18px 18px;
  background-repeat: repeat;
  z-index: -1;
}

.card-ornament--top {
  top: -12px;
  right: -12px;
  transform: rotate(180deg);
}

.card-ornament--bottom {
  bottom: -12px;
  left: -12px;
}

.ilustracao {
  position: relative;
  width: clamp(88px, calc(var(--card-width) * 0.64), 126px);
  height: clamp(88px, calc(var(--card-width) * 0.64), 126px);
  object-fit: contain;
  display: block;
  filter: drop-shadow(0 6px 8px rgba(27, 42, 114, 0.12));
  z-index: 1;
}

.ilustracao--fallback {
  display: flex;
  border-radius: 999px;
  background: #ffffff;
  color: var(--card-blue-dark);
  font-family: 'Arial Black', 'Impact', sans-serif;
  font-size: clamp(1rem, 2.4vw, 1.4rem);
  align-items: center;
  justify-content: center;
}

.card-section-title {
  font-family: 'Arial Narrow', 'Franklin Gothic Medium', sans-serif;
  font-size: clamp(0.58rem, 0.85vw, 0.7rem);
  line-height: 1;
}

.card-body {
  flex: 1 1 auto;
  min-height: clamp(84px, calc(var(--card-height) * 0.28), 102px);
  margin: 0;
  padding: clamp(8px, 1.2vw, 12px) clamp(8px, 1.2vw, 12px) clamp(10px, 1.4vw, 14px);
  border: 2px solid rgba(122, 146, 240, 0.28);
  background: rgba(255, 255, 255, 0.98);
  color: var(--card-text);
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: stretch;
  gap: clamp(6px, 0.8vw, 8px);
}

.area {
  margin: 0;
  color: #2434b6;
  font-size: clamp(0.68rem, 0.95vw, 0.78rem);
  font-weight: 800;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  line-height: 1.15;
  display: -webkit-box;
  line-clamp: 2;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.contribuicao {
  margin: 0;
  color: #1f2240;
  font-size: clamp(0.66rem, 0.9vw, 0.76rem);
  line-height: 1.22;
  text-align: center;
  display: -webkit-box;
  line-clamp: 5;
  -webkit-line-clamp: 5;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.contribuicao--empty {
  margin: auto 0;
  color: rgba(31, 34, 64, 0.72);
}

.card-footer {
  flex: 0 0 auto;
  display: flex;
  justify-content: center;
  margin-top: auto;
  padding-top: 0;
}

.year-ribbon {
  position: relative;
  min-width: clamp(74px, calc(var(--card-width) * 0.46), 92px);
  padding: clamp(3px, 0.6vw, 4px) clamp(10px, 1vw, 14px) clamp(4px, 0.7vw, 5px);
  background: linear-gradient(180deg, #f46ea9 0%, var(--card-ribbon) 100%);
  color: #fff;
  font-family: 'Arial Black', 'Impact', sans-serif;
  font-size: clamp(0.78rem, 1vw, 0.9rem);
  line-height: 1;
  letter-spacing: 0.04em;
  text-align: center;
  clip-path: polygon(10px 0, calc(100% - 10px) 0, 100% 50%, calc(100% - 10px) 100%, 10px 100%, 0 50%);
  box-shadow: 0 4px 0 var(--card-ribbon-shadow);
}

.year-ribbon::before,
.year-ribbon::after {
  content: '';
  position: absolute;
  bottom: -7px;
  border-top: 8px solid var(--card-ribbon-shadow);
  border-bottom: 0;
}

.year-ribbon::before {
  left: 8px;
  border-right: 8px solid transparent;
}

.year-ribbon::after {
  right: 8px;
  border-left: 8px solid transparent;
}
</style>
