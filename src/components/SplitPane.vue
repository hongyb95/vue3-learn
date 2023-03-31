<script lang="ts" setup>
import { ref, reactive, computed } from 'vue';

const container = ref()

const props = defineProps<{ layout?: string }>()
const isVertical = computed(() => props.layout === 'vertical')

const state = reactive({
  dragging: false,
  split: 50
})

const boundSplit = computed(()=>{
  const { split } = state
  return split < 20 ? 20 : split > 80 ? 80 : split
})

let startPosition = 0
let startSplit = 0

function dragStart(e: MouseEvent) {
  state.dragging = true
  startPosition = isVertical.value ? e.pageY : e.pageX
  startSplit = boundSplit.value
}

function dragMove(e: MouseEvent) {
  if(state.dragging) {
    const position = isVertical.value ? e.pageY : e.pageX
    const containSize = isVertical.value ? container.value.offsetHeight : container.value.offsetWidth
    const dp = position - startPosition
    state.split = startSplit + ~~((dp / containSize) * 100)
  }
}

function dragEnd() {
  state.dragging = false
}
</script>

<template>
  <div class="split-pane"
    :class="{
      dragging: state.dragging,
      vertical: isVertical
    }"
    ref="container"
    @mousemove="dragMove"
    @mouseup="dragEnd"
    @mouseleave="dragEnd"
  >
    <div class="left" :style="{[isVertical ? 'height' : 'width']: boundSplit + '%'}">
      <slot name="left"></slot>
    </div>
    <div class="split-trigger" @mousedown.prevent="dragStart"></div>
    <div class="right">
      <slot name="right"></slot>
    </div>
  </div>
</template>

<style scoped>
.split-pane {
  display: flex;
  height: 100%;
  background: palegreen;
}
.split-pane.dragging {
  cursor: ew-resize;
}
.split-pane.vertical {
  flex-direction: column;
}
.split-pane.vertical.dragging {
  cursor: n-resize;
}
.split-pane.vertical .split-trigger {
  width: 100%;
  height: 10px;
  cursor: n-resize;
}
.left {
  background-color: palevioletred;
}
.right {
  flex: 1;
  background-color: turquoise;
}

.split-trigger {
  height: 100%;
  width: 10px;
  background-color: palegoldenrod;
  cursor: ew-resize;
}
</style>