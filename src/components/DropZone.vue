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
    <div class="drop-indicator"></div>
  </div>
</template>

<style scoped>
.drop-zone {
  width: 60px;
  min-height: 200px;
  align-self: stretch;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: all 0.2s ease;
}

.drop-indicator {
  width: 4px;
  height: 80%;
  background-color: rgba(0, 0, 0, 0.1);
  border-radius: 4px;
  transition: background-color 0.2s;
}

.drop-zone.is-active .drop-indicator {
  background-color: #4caf50;
  width: 8px;
  box-shadow: 0 0 8px rgba(76, 175, 80, 0.6);
}
</style>
