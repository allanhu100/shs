<template>
  <div class="info-tooltip-wrapper">
    <div
      class="info-icon"
      tabindex="0"
      role="button"
      aria-label="More information"
      @mouseenter="showTooltip"
      @mouseleave="hideTooltip"
      @focus="showTooltip"
      @blur="hideTooltip"
      @click="toggleTooltip"
    >
      <slot name="trigger">
        <font-awesome-icon :icon="icon" />
      </slot>
    </div>
    <teleport to="body">
      <div v-if="isVisible" ref="tooltip" class="info-tooltip" :style="tooltipStyle">
        <slot />
      </div>
    </teleport>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick, useTemplateRef } from 'vue'
import { faInfoCircle } from '@fortawesome/free-solid-svg-icons'

const { icon = faInfoCircle } = defineProps<{
  icon?: object
}>()

const isVisible = ref(false)
const tooltipStyle = ref<Record<string, string>>({})
const tooltip = useTemplateRef<HTMLDivElement>('tooltip')

function showTooltip(event) {
  isVisible.value = true
  const target = event.currentTarget
  nextTick(() => {
    const rect = target.getBoundingClientRect()
    const tooltipEl = tooltip.value

    if (tooltipEl) {
      const tooltipWidth = tooltipEl.offsetWidth
      const tooltipHeight = tooltipEl.offsetHeight

      let left = rect.left + rect.width / 2 - tooltipWidth / 2
      let top = rect.top - tooltipHeight - 8

      // Keep within viewport
      if (left < 10) left = 10
      if (left + tooltipWidth > window.innerWidth - 10) {
        left = window.innerWidth - tooltipWidth - 10
      }
      if (top < 10) top = rect.bottom + 8 // flip to bottom if no room above

      tooltipStyle.value = {
        left: `${left}px`,
        top: `${top}px`,
        background: var(--accent),
        borderColor: var(--accent),
      }
    }
  })
}

function hideTooltip() {
  isVisible.value = false
}

function toggleTooltip(event) {
  if (isVisible.value) {
    hideTooltip()
  } else {
    showTooltip(event)
  }
}
</script>

<style lang="sass" scoped>
.info-tooltip-wrapper
  display: inline-flex
  align-items: center

.info-icon
  color: var(--accent)
  opacity: 0.8
  cursor: help
  transition: opacity 0.2s
  display: flex
  align-items: center

  &:hover
    opacity: 1

.info-tooltip
  position: fixed
  max-width: 200px
  width: max-content
  padding: 10px 12px
  background: var(--accent)
  color: white
  border: 1px solid var(--accent)
  border-radius: 8px
  font-size: 12px
  line-height: 1.5
  z-index: 100000
  pointer-events: none
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4)
</style>
