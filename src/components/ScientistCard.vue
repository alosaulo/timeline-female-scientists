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
    compact?: boolean
  }>(),
  {
    showYear: true,
    draggable: false,
    compact: false,
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

function getTextClass(
  prefix: string,
  text: string | undefined,
  thresholds: [number, number, number],
) {
  const length = text?.trim().length ?? 0

  if (length > thresholds[2]) return `${prefix}--xxlong`
  if (length > thresholds[1]) return `${prefix}--xlong`
  if (length > thresholds[0]) return `${prefix}--long`

  return ''
}

const nameClass = computed(() => getTextClass('name', props.card.nome, [18, 26, 34]))
const areaClass = computed(() => getTextClass('area', props.card.area, [16, 24, 32]))
const contributionClass = computed(() =>
  getTextClass('contribuicao', props.card.contribuicao, [52, 72, 92]),
)
</script>

<template>
  <div
    ref="element"
    :class="[
      'card',
      {
        'card--compact': compact,
        'card--dragging': isDragging,
        'card--draggable': draggable,
      },
    ]"
  >
    <header class="card-header">
      <h3 :class="['name', nameClass]">{{ card.nome }}</h3>
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
      <p v-if="card.area" :class="['area', areaClass]">{{ card.area }}</p>
      <p v-if="card.contribuicao" :class="['contribuicao', contributionClass]">
        {{ card.contribuicao }}
      </p>
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
  --card-inner-gap: clamp(3px, calc(var(--card-height) * 0.02), 8px);
  --card-edge-padding: clamp(3px, calc(var(--card-width) * 0.04), 6px);
  --card-bottom-padding: clamp(6px, calc(var(--card-height) * 0.06), 12px);

  flex: 0 0 var(--card-width);
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: stretch;
  width: var(--card-width);
  height: var(--card-height);
  min-height: 0;
  padding: var(--card-edge-padding) var(--card-edge-padding) var(--card-bottom-padding);
  border: 3px solid var(--card-frame);
  border-radius: 4px;
  background: linear-gradient(180deg, #b1c3ff 0%, #8da5ff 100%);
  box-shadow: 0 6px 14px var(--card-shadow);
  gap: var(--card-inner-gap);
  text-align: center;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  overflow: hidden;
}

.card--compact {
  --card-height: var(--timeline-card-height);
  --card-width: var(--timeline-card-width);
}

.card--draggable {
  position: relative;
  z-index: 1;
  cursor: grab;
  background: linear-gradient(180deg, #b1c3ff 0%, #8da5ff 100%);
}

@media (hover: hover) and (pointer: fine) {
  .card--draggable:not(.card--dragging):hover {
    transform: translateY(-4px) scale(1.03);
    box-shadow: 0 12px 24px rgba(19, 35, 112, 0.28);
    z-index: 4;
  }

  .card--compact:not(.card--dragging):hover {
    transform: translateY(-4px) scale(1.03);
    box-shadow: 0 12px 24px rgba(19, 35, 112, 0.28);
    z-index: 4;
  }
}

.card-header,
.card-section-title {
  margin: 0;
  padding: clamp(2px, calc(var(--card-height) * 0.012), 4px)
    clamp(4px, calc(var(--card-width) * 0.065), 8px);
  border: 1px solid rgba(255, 255, 255, 0.35);
  background: linear-gradient(180deg, #2d34ec 0%, var(--card-blue-dark) 100%);
  color: #fff;
  text-align: center;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  text-shadow: 0 1px 0 rgba(6, 10, 65, 0.55);
}

.card-header {
  min-height: clamp(24px, calc(var(--card-height) * 0.14), 44px);
  display: flex;
  align-items: center;
  justify-content: center;
}

.name {
  --name-font-size: clamp(0.62rem, calc(var(--card-width) * 0.1), 1rem);

  margin: 0;
  font-family: 'Arial Narrow', 'Franklin Gothic Heavy', sans-serif;
  font-size: var(--name-font-size);
  line-height: 1.02;
  display: -webkit-box;
  line-clamp: 2;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-wrap: balance;
}

.name--long {
  --name-font-size: clamp(0.56rem, calc(var(--card-width) * 0.092), 0.92rem);
}

.name--xlong {
  --name-font-size: clamp(0.5rem, calc(var(--card-width) * 0.084), 0.84rem);
}

.name--xxlong {
  --name-font-size: clamp(0.46rem, calc(var(--card-width) * 0.078), 0.78rem);
}

.card-illustration {
  display: flex;
}

.card-illustration__frame {
  position: relative;
  width: 100%;
  height: calc(var(--card-height) * 0.28);
  min-height: 0;
  border: 2px solid var(--card-frame);
  background: var(--card-surface);
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: clamp(4px, calc(var(--card-width) * 0.05), 10px);
  isolation: isolate;
}

.card--compact .card-illustration__frame {
  height: calc(var(--card-height) * 0.34);
}

.card-illustration__backdrop {
  position: absolute;
  top: clamp(6px, calc(var(--card-height) * 0.05), 14px);
  left: 50%;
  width: clamp(38px, calc(var(--card-width) * 0.62), 122px);
  height: clamp(38px, calc(var(--card-width) * 0.62), 122px);
  border-radius: 999px;
  background: #6c87ea;
  transform: translateX(-50%);
  z-index: -2;
}

.card-ornament {
  position: absolute;
  width: clamp(20px, calc(var(--card-width) * 0.31), 62px);
  height: clamp(20px, calc(var(--card-width) * 0.31), 62px);
  opacity: 0.88;
  background:
    radial-gradient(circle at center, #ffffff 0 42%, transparent 43%) 0 0 / 18px 18px,
    radial-gradient(circle at center, #ffffff 0 42%, transparent 43%) 9px 9px / 18px 18px,
    radial-gradient(circle at center, rgba(124, 147, 241, 0.8) 0 28%, transparent 29%) 0 0 / 18px
      18px,
    radial-gradient(circle at center, rgba(124, 147, 241, 0.8) 0 28%, transparent 29%) 9px 9px /
      18px 18px;
  background-repeat: repeat;
  z-index: -1;
}

.card-ornament--top {
  top: -8px;
  right: -8px;
  transform: rotate(180deg);
}

.card-ornament--bottom {
  bottom: -8px;
  left: -8px;
}

.ilustracao {
  position: relative;
  width: clamp(34px, calc(var(--card-height) * 0.24), 126px);
  height: clamp(34px, calc(var(--card-height) * 0.24), 126px);
  object-fit: contain;
  display: block;
  filter: drop-shadow(0 4px 6px rgba(27, 42, 114, 0.12));
  z-index: 1;
}

.ilustracao--fallback {
  display: flex;
  border-radius: 999px;
  background: #ffffff;
  color: var(--card-blue-dark);
  font-family: 'Arial Black', 'Impact', sans-serif;
  font-size: clamp(0.72rem, calc(var(--card-width) * 0.16), 1.4rem);
  align-items: center;
  justify-content: center;
}

.card-section-title {
  min-height: clamp(16px, calc(var(--card-height) * 0.06), 24px);
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Arial Narrow', 'Franklin Gothic Medium', sans-serif;
  font-size: clamp(0.44rem, calc(var(--card-width) * 0.072), 0.74rem);
  line-height: 1;
}

.card-body {
  flex: 1 1 auto;
  min-height: 0;
  margin: 0;
  padding: clamp(4px, calc(var(--card-width) * 0.05), 12px)
    clamp(4px, calc(var(--card-width) * 0.05), 12px)
    clamp(6px, calc(var(--card-height) * 0.05), 14px);
  border: 2px solid rgba(122, 146, 240, 0.28);
  background: rgba(255, 255, 255, 0.98);
  color: var(--card-text);
  display: grid;
  grid-template-rows: auto minmax(0, 1fr);
  align-content: start;
  align-items: stretch;
  gap: clamp(3px, calc(var(--card-height) * 0.014), 8px);
}

.area {
  --area-font-size: clamp(0.58rem, calc(var(--card-width) * 0.082), 0.9rem);

  margin: 0;
  color: #2434b6;
  font-size: var(--area-font-size);
  font-weight: 800;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  line-height: 1.15;
  min-height: 1.15em;
  display: -webkit-box;
  line-clamp: 1;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-wrap: balance;
}

.area--long {
  --area-font-size: clamp(0.54rem, calc(var(--card-width) * 0.076), 0.82rem);
}

.area--xlong {
  --area-font-size: clamp(0.48rem, calc(var(--card-width) * 0.07), 0.76rem);
}

.area--xxlong {
  --area-font-size: clamp(0.42rem, calc(var(--card-width) * 0.066), 0.7rem);
}

.contribuicao {
  --contribuicao-font-size: clamp(0.9rem, calc(var(--card-width) * 0.076), 1rem);

  margin: 0;
  color: #1f2240;
  font-size: var(--contribuicao-font-size);
  line-height: 1.16;
  text-align: center;
  display: -webkit-box;
  line-clamp: 6;
  -webkit-line-clamp: 6;
  -webkit-box-orient: vertical;
  overflow: hidden;
  overflow-wrap: anywhere;
  text-wrap: pretty;
  hyphens: auto;
}

.contribuicao--long {
  --contribuicao-font-size: clamp(0.68rem, calc(var(--card-width) * 0.068), 0.86rem);
}

.contribuicao--xlong {
  --contribuicao-font-size: clamp(0.58rem, calc(var(--card-width) * 0.062), 0.78rem);
}

.contribuicao--xxlong {
  --contribuicao-font-size: clamp(0.5rem, calc(var(--card-width) * 0.058), 0.68rem);
  line-clamp: 7;
  -webkit-line-clamp: 7;
}

.card--compact .name {
  --name-font-size: clamp(0.44rem, calc(var(--card-width) * 0.084), 0.74rem);
}

.card--compact .name--long {
  --name-font-size: clamp(0.4rem, calc(var(--card-width) * 0.078), 0.68rem);
}

.card--compact .name--xlong,
.card--compact .name--xxlong {
  --name-font-size: clamp(0.36rem, calc(var(--card-width) * 0.072), 0.62rem);
}

.card--compact .card-section-title {
  font-size: clamp(0.34rem, calc(var(--card-width) * 0.065), 0.6rem);
}

.card--compact .area {
  --area-font-size: clamp(0.38rem, calc(var(--card-width) * 0.068), 0.68rem);
}

.card--compact .area--long,
.card--compact .area--xlong,
.card--compact .area--xxlong {
  --area-font-size: clamp(0.34rem, calc(var(--card-width) * 0.062), 0.6rem);
}

.card--compact .contribuicao {
  --contribuicao-font-size: clamp(0.42rem, calc(var(--card-width) * 0.064), 0.62rem);
  line-height: 1.16;
  line-clamp: 2;
  -webkit-line-clamp: 2;
}

.card--compact .contribuicao--long,
.card--compact .contribuicao--xlong,
.card--compact .contribuicao--xxlong {
  --contribuicao-font-size: clamp(0.34rem, calc(var(--card-width) * 0.056), 0.54rem);
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
  min-width: clamp(42px, calc(var(--card-width) * 0.62), 92px);
  padding: clamp(2px, calc(var(--card-height) * 0.012), 4px)
    clamp(8px, calc(var(--card-width) * 0.08), 14px)
    clamp(3px, calc(var(--card-height) * 0.015), 5px);
  background: linear-gradient(180deg, #f46ea9 0%, var(--card-ribbon) 100%);
  color: #fff;
  font-family: 'Arial Black', 'Impact', sans-serif;
  font-size: clamp(0.54rem, calc(var(--card-width) * 0.092), 0.92rem);
  line-height: 1;
  letter-spacing: 0.04em;
  text-align: center;
  clip-path: polygon(8px 0, calc(100% - 8px) 0, 100% 50%, calc(100% - 8px) 100%, 8px 100%, 0 50%);
  box-shadow: 0 3px 0 var(--card-ribbon-shadow);
}

.card--compact .year-ribbon {
  font-size: clamp(0.4rem, calc(var(--card-width) * 0.08), 0.74rem);
}

</style>
