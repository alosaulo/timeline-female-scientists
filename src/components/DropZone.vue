<script setup lang="ts">
import { ref } from 'vue'
import { useDroppable } from '@dnd-kit/vue'

const props = defineProps<{
  index: number
}>()

const element = ref<HTMLElement | null>(null)

const { isDropTarget } = useDroppable({
  id: () => `drop-${props.index}`,
  element,
  data: () => ({ index: props.index }),
})
</script>

<template>
  <div ref="element" :class="['drop-zone', { 'is-active': isDropTarget }]">
    <div class="ghost-card" aria-hidden="true">
      <span class="ghost-plus">+</span>
    </div>
  </div>
</template>

<style scoped>
.drop-zone {
  flex: 0 0 200px;
  width: 200px;
  height: 280px;
  display: flex;
  justify-content: stretch;
  align-items: stretch;
  transition: transform 0.2s ease;
}

.ghost-card {
  width: 100%;
  height: 100%;
  border: 2px dashed rgba(91, 122, 142, 0.3);
  border-radius: 12px;
  background: rgba(239, 244, 248, 0.7);
  opacity: 0.8;
  display: flex;
  justify-content: center;
  align-items: center;
  pointer-events: none;
  transition: border-color 0.2s ease, background-color 0.2s ease, opacity 0.2s ease,
    transform 0.2s ease;
}

.ghost-plus {
  font-size: 2rem;
  line-height: 1;
  font-weight: 400;
  color: rgba(91, 122, 142, 0.6);
}

.drop-zone.is-active {
  transform: translateY(-2px);
}

.drop-zone.is-active .ghost-card {
  opacity: 1;
  border-color: #4caf50;
  background: rgba(76, 175, 80, 0.16);
  transform: scale(1.01);
}

.drop-zone.is-active .ghost-plus {
  color: #2e7d32;
}
</style>
