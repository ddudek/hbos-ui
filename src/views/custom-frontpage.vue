<template>
  <div class="now-playing">
    <PageContent
      title="Volume"
      hintLink="/now-playing-minimal"
      hintString="Switch to minimal view"
      class="now-playing__page-content"
    >
      <div class="now-playing__player">

        <!-- Volume control -->
        <div class="now-playing__volume">
          <VolumeControl size="wide" />
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
</style>
