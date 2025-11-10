<template>
  <div
    ref="slider"
    class="app-progress-slider"
    :class="{ disabled, 'is-on-header': isOnHeader, 'is-vertical': isVertical }"
    @click="handleClick"
    @mousedown="startDrag"
    @touchstart="startTouch"
  >
    <div class="app-progress-slider__track">
      <div
        class="app-progress-slider__progress"
        :style="isVertical ? { height: displayPercent + '%' } : { width: displayPercent + '%' }"
      />
      <div
        v-if="centerMark !== undefined"
        class="app-progress-slider__center-mark"
        :style="isVertical ? { bottom: centerMarkPercent + '%' } : { left: centerMarkPercent + '%' }"
      />
    </div>
    <div
      v-if="hasThumb && !isOnHeader"
      class="app-progress-slider__thumb"
      :style="isVertical ? { bottom: displayPercent + '%' } : { left: displayPercent + '%' }"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onBeforeUnmount } from 'vue'

interface ProgressSliderProps {
  value: number
  min?: number
  max?: number
  step?: number
  disabled?: boolean
  hasThumb?: boolean
  isDraggable?: boolean
  isOnHeader?: boolean
  isVertical?: boolean
  centerMark?: number // Value at which to show center mark (optional)
}

const {
  value = 0,
  min = 0,
  max = 100,
  step = 1,
  disabled = false,
  hasThumb = true,
  isDraggable = false,
  isOnHeader = false,
  isVertical = false,
  centerMark = undefined,
} = defineProps<ProgressSliderProps>()

const emit = defineEmits<{
  (e: 'click:progress', value: number): void
}>()

const slider = ref<HTMLDivElement | null>(null)
const dragging = ref(false)
const hasDragged = ref(false)
const internalValue = ref(value)

const wasOnDragging = ref(false)

watch(
  () => value,
  (newVal) => {
    if (!dragging.value) internalValue.value = newVal
  },
)

const displayPercent = computed(() => {
  const range = max - min
  if (range === 0) return 0
  return Math.max(0, Math.min(100, ((value - min) / range) * 100))
})

const centerMarkPercent = computed(() => {
  if (centerMark === undefined) return 50
  const range = max - min
  if (range === 0) return 50
  return Math.max(0, Math.min(100, ((centerMark - min) / range) * 100))
})

function getValueFromMouse(event: MouseEvent): number | null {
  if (!slider.value) return null

  const rect = slider.value.getBoundingClientRect()

  let ratio: number
  if (isVertical) {
    const clickY = event.clientY - rect.top
    const height = rect.height
    // For vertical sliders, we want bottom to be 0 and top to be max
    ratio = 1 - (clickY / height)
  } else {
    const clickX = event.clientX - rect.left
    const width = rect.width
    ratio = clickX / width
  }

  let newValue = min + ratio * (max - min)

  if (step > 0) {
    newValue = Math.round((newValue - min) / step) * step + min
  }

  return Math.min(Math.max(newValue, min), max)
}

function handleClick(event: MouseEvent) {
  if (disabled || wasOnDragging.value) {
    wasOnDragging.value = false

    return
  }

  const newValue = getValueFromMouse(event)
  if (newValue !== null) {
    if (isDraggable) {
      internalValue.value = newValue
    }

    emit('click:progress', newValue)
  }
}

function startDrag(event: MouseEvent) {
  if (disabled || !isDraggable) return

  dragging.value = true
  hasDragged.value = false

  document.addEventListener('mousemove', onDrag)
  document.addEventListener('mouseup', stopDrag)
  event.preventDefault()
}

function onDrag(event: MouseEvent) {
  if (!dragging.value || disabled) return

  wasOnDragging.value = true

  const newValue = getValueFromMouse(event)
  if (newValue !== null) {
    internalValue.value = newValue
    hasDragged.value = true
  }
}

function stopDrag() {
  if (!dragging.value) return

  if (hasDragged.value) {
    emit('click:progress', internalValue.value)
  }

  dragging.value = false
  hasDragged.value = false

  document.removeEventListener('mousemove', onDrag)
  document.removeEventListener('mouseup', stopDrag)
}

// For touch devices
function getValueFromTouch(event: TouchEvent, useChangedTouches = false): number | null {
  if (!slider.value) return null

  const touches = useChangedTouches ? event.changedTouches : event.touches
  if (touches.length === 0) return null

  const rect = slider.value.getBoundingClientRect()

  let ratio: number
  if (isVertical) {
    const touchY = touches[0].clientY - rect.top
    const height = rect.height
    // For vertical sliders, we want bottom to be 0 and top to be max
    ratio = 1 - (touchY / height)
  } else {
    const touchX = touches[0].clientX - rect.left
    const width = rect.width
    ratio = touchX / width
  }

  let newValue = min + ratio * (max - min)

  if (step > 0) {
    newValue = Math.round((newValue - min) / step) * step + min
  }

  return Math.min(Math.max(newValue, min), max)
}

let touchStartX = 0
let touchStartY = 0
let touchMoved = false

function startTouch(event: TouchEvent) {
  if (disabled) return

  touchMoved = false
  touchStartX = event.touches[0].clientX
  touchStartY = event.touches[0].clientY

  if (!isDraggable) return

  dragging.value = true
  hasDragged.value = false

  document.addEventListener('touchmove', onTouchMove)
  document.addEventListener('touchend', stopTouch)
}

function onTouchMove(event: TouchEvent) {
  if (event.touches.length === 0) return

  const delta = isVertical
    ? Math.abs(event.touches[0].clientY - touchStartY)
    : Math.abs(event.touches[0].clientX - touchStartX)
  if (delta > 5) touchMoved = true // threshold to detect dragging

  if (!dragging.value || disabled) return

  const newValue = getValueFromTouch(event)
  if (newValue !== null) {
    internalValue.value = newValue
    hasDragged.value = true
  }
}

function stopTouch(event: TouchEvent) {
  if (!touchMoved) {
    const newValue = getValueFromTouch(event, true) // use changedTouches
    if (newValue !== null) {
      if (isDraggable) {
        internalValue.value = newValue
      }

      emit('click:progress', newValue)
    }
  }

  if (dragging.value && hasDragged.value) {
    emit('click:progress', internalValue.value)
  }

  dragging.value = false
  hasDragged.value = false

  document.removeEventListener('touchmove', onTouchMove)
  document.removeEventListener('touchend', stopTouch)
}

onBeforeUnmount(() => {
  document.removeEventListener('mousemove', onDrag)
  document.removeEventListener('mouseup', stopDrag)
  document.removeEventListener('touchmove', onTouchMove)
  document.removeEventListener('touchend', stopTouch)
})
</script>

<style scoped lang="scss">
.app-progress-slider {
  $root: &;

  position: relative;
  display: flex;
  align-items: center;
  width: 100%;
  height: 12px;
  cursor: pointer;
  user-select: none;

  &.disabled {
    opacity: 0.5;
    pointer-events: none;

    #{$root}__thumb {
      background-color: $progress-slider-bg--dark;
      pointer-events: all;
      cursor: not-allowed;
    }
  }

  // Vertical orientation
  &.is-vertical {
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 12px;
    height: 100%;
    min-height: 100px;

    #{$root}__track {
      width: 8px;
      height: 100%;

      @include media-down(md) {
        width: 6px;
      }
    }

    #{$root}__progress {
      width: 100%;
      // Height is controlled by the style binding in template
      // Position from bottom for vertical sliders
      position: absolute;
      bottom: 0;
      left: 0;
    }

    #{$root}__center-mark {
      left: 50%;
      bottom: auto; // Reset bottom positioning
      transform: translate(-50%, 50%);
      width: 120%;
      height: 2px;

      @include media-down(md) {
        width: 140%;
      }
    }

    #{$root}__thumb {
      left: 50%;
      top: auto;
      transform: translate(-50%, 50%);
      // Bottom positioning is handled by the style binding in template
    }
  }

  &__track {
    position: absolute;
    height: 8px;
    width: 100%;
    border-radius: 4px;
    background: var(--progress-slider-bg);

    @include media-down(md) {
      height: 6px;
      border-radius: 3px;
    }
  }

  &__progress {
    height: 100%;
    background-color: var(--primary);
    border-radius: 4px;
    transition: width 0.1s linear;

    @include media-down(md) {
      border-radius: 3px;
    }
  }

  &__center-mark {
    position: absolute;
    top: 50%;
    transform: translate(-50%, -50%);
    width: 2px;
    height: 120%;
    background-color: var(--color-border, #d1d5db);
    z-index: 1;
    pointer-events: none;

    @include media-down(md) {
      height: 140%;
    }
  }

  &__thumb {
    position: absolute;
    top: 50%;
    transform: translate(-50%, -50%);
    width: 16px;
    height: 16px;
    background-color: var(--primary);
    border-radius: 50%;
    pointer-events: none;
    transition: left 0.1s linear;
    z-index: 2;
  }

  &.is-on-header {
    margin-top: 8px;

    @include media-down(md) {
      margin-top: 6px;
    }

    .app-progress-slider__track {
      height: 2px;
      border-radius: 1px;
      background: var(--progress-slider-on-header-bg);
    }

    .app-progress-slider__progress {
      border-radius: 1px;
      background-color: var(--progress-slider-on-header-progess);
    }
  }
}
</style>
