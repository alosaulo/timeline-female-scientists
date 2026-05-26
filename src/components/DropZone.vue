<script setup lang="ts">
import { ref } from 'vue'
import { useDroppable } from '@dnd-kit/vue'

const props = defineProps<{
  index: number
  compact?: boolean
}>()

const element = ref<HTMLElement | null>(null)

const { isDropTarget } = useDroppable({
  id: () => `drop-${props.index}`,
  element,
  data: () => ({ index: props.index }),
})
</script>

<template>
  <div
    ref="element"
    :class="['drop-zone', { 'drop-zone--compact': compact, 'is-active': isDropTarget }]"
  >
    <div class="ghost-card" aria-hidden="true">
      <span class="ghost-plus">+</span>
    </div>
  </div>
</template>

<style scoped>
.drop-zone {
  flex: 0 0 var(--drop-zone-width);
  width: var(--drop-zone-width);
  height: var(--card-height);
  min-height: var(--card-height);
  display: flex;
  justify-content: stretch;
  align-items: stretch;
  transition: transform 0.2s ease;
}

.drop-zone--compact {
  --card-height: var(--timeline-card-height);
  --card-width: var(--timeline-card-width);
  --drop-zone-width: var(--timeline-drop-zone-width);
}

.ghost-card {
  width: 100%;
  height: 100%;
  border: 2px dashed rgba(92, 118, 222, 0.45);
  border-radius: 4px;
  background: linear-gradient(180deg, rgba(214, 223, 255, 0.72) 0%, rgba(188, 201, 255, 0.78) 100%);
  opacity: 0.8;
  display: flex;
  justify-content: center;
  align-items: center;
  pointer-events: none;
  transition:
    border-color 0.2s ease,
    background-color 0.2s ease,
    opacity 0.2s ease,
    transform 0.2s ease;
}

.ghost-plus {
  font-size: clamp(0.9rem, calc(var(--card-width) * 0.22), 2rem);
  line-height: 1;
  font-weight: 700;
  color: rgba(44, 58, 156, 0.55);
}

.drop-zone.is-active {
  width: var(--card-width);
  flex: 0 0 var(--card-width);
}

.drop-zone.is-active .ghost-card {
  opacity: 1;
  border-color: #4669ec;
  background: linear-gradient(180deg, rgba(162, 183, 255, 0.8) 0%, rgba(128, 154, 247, 0.85) 100%);
  transform: scale(1.01);
}

.drop-zone.is-active .ghost-plus {
  color: #ffffff;
}
</style>
