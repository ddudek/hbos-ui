<template>
  <div class="now-playing">
    <PageContent
      title="Volume"
      hintLink="/now-playing-minimal"
      hintString="Switch to minimal view"
      class="now-playing__page-content"
    >
      <div class="now-playing__player">

        <!-- Volume control with buttons -->
        <div class="now-playing__volume">
          <VolumeControl size="vertical" />
          <div class="volume-buttons">
            <button class="volume-button volume-button--increase" @click="increaseVolume">+</button>
            <div class="preset-buttons-group">
              <button class="volume-button volume-button--preset" @click="setVolumeToPreset(80)">80</button>
              <button class="volume-button volume-button--preset" @click="setVolumeToPreset(70)">70</button>
              <button class="volume-button volume-button--preset" @click="setVolumeToPreset(60)">60</button>
              <button class="volume-button volume-button--preset" @click="setVolumeToPreset(50)">50</button>
              <button class="volume-button volume-button--preset" @click="setVolumeToPreset(40)">40</button>
              <button class="volume-button volume-button--preset" @click="setVolumeToPreset(30)">30</button>
            </div>
            <button class="volume-button volume-button--decrease" @click="decreaseVolume">-</button>
          </div>
        </div>
      </div>

    </PageContent>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import PageContent from '@/components/PageContent.vue'
import VolumeControl from '@/components/VolumeControl.vue'

import { storeToRefs } from 'pinia'
import { usePlayerStore } from '@/stores/player'

const playerStore = usePlayerStore()
const { currentVolume } = storeToRefs(playerStore)

// Volume control functions
const increaseVolume = () => {
  const newVolume = Math.min(100, currentVolume.value + 1)
  playerStore.setVolume(newVolume)
}

const decreaseVolume = () => {
  const newVolume = Math.max(0, currentVolume.value - 1)
  playerStore.setVolume(newVolume)
}

const setVolumeToPreset = (volume: number) => {
  playerStore.setVolume(volume)
}
</script>

<style lang="scss">
.now-playing {
  display: flex;
  flex-direction: column;
  min-width: 100%;
  height: 100%;

  &__page-content {
    display: flex;
    flex-direction: column;
    flex-grow: 1;

    .content {
      display: flex;
      flex-direction: column;
      flex-grow: 1;
    }
  }

  &__player {
    display: flex;
    flex-grow: 1;
    flex-direction: column;
    align-items: center;
    border-radius: 10px;
    padding: $padding-main-content;
    padding-top: 16px;
    background-color: var(--background-main-content);
    box-shadow: $box-shadow-main-content;
    position: relative;
  }

  &__volume {
    width: 90%; /* Increased from 60% - now 10% wider */
    max-width: 1840px; /* Increased from 400px proportionally */
    display: flex;
    justify-content: center;
    margin-bottom: 16px; /* 40px space at bottom */
    gap: 20px; /* Space between volume control and buttons */

    @include media-down(lg) {
      width: 88%; /* Increased from 80% proportionally */
    }

    @include media-down(md) {
      width: 100%; /* Keep at 100% on mobile */
    }

    // For vertical volume control, make it take full available height
    &:has(.volume-control--vertical) {
      flex: 1;
      height: 100%;
      min-height: 300px;
      align-items: stretch;
    }
  }
}

.volume-buttons {
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 100%;
  min-height: 300px;
  justify-content: flex-start;
  gap: 8px;
}

/* Push the "-" button to the bottom */
.volume-button--decrease {
  margin-top: auto;
}

.preset-buttons-group {
  display: flex;
  flex-direction: column;
  flex: 0.7; /* Take only 60% of the middle section */
  justify-content: space-between; /* Distribute buttons evenly */
  align-items: center;
  padding-top: 64px; /* Small gap from the "+" button */
}

.volume-button {
  width: 36px;
  height: 36px;
  border: 2px solid var(--primary);
  background-color: var(--background-main-content);
  color: var(--primary);
  border-radius: 6px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;

  &:hover {
    background-color: var(--primary);
    color: var(--background-main-content);
    transform: scale(1.05);
  }

  &:active {
    transform: scale(0.95);
  }

  &--increase {
    // Top button
    font-size: 16px;
  }

  &--preset {
    // Preset buttons
    font-size: 12px;
  }

  &--decrease {
    // Bottom button
    font-size: 16px;
  }
}
</style>
