<template>
  <div
    class="swipe-card"
    :style="cardStyle"
    @mousedown="startDrag"
    @touchstart="startDrag"
  >
    <!-- Show placeholder until image loads -->
    <div v-if="!loaded" class="placeholder">Loading...</div>
    <img
      v-show="loaded"
      :src="cat.url"
      alt="Cat"
      class="cat-img"
      @load="loaded = true"
    />
  </div>
</template>



<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const props = defineProps({
  cat: Object
})
const loaded = ref(false)

const emit = defineEmits(['swipe'])

const startX = ref(0)
const currentX = ref(0)
const isDragging = ref(false)
const isAnimating = ref(false)

function startDrag(e) {
  if (isAnimating.value) return
  
  isDragging.value = true
  startX.value = e.type.includes('mouse') ? e.clientX : e.touches[0].clientX
  currentX.value = 0 // Reset position
  
  // Add event listeners
  if (e.type.includes('mouse')) {
    document.addEventListener('mousemove', onDrag, { passive: false })
    document.addEventListener('mouseup', endDrag)
    // Prevent text selection and image dragging
    e.preventDefault()
  } else {
    document.addEventListener('touchmove', onDrag, { passive: false })
    document.addEventListener('touchend', endDrag)
  }
}

function onDrag(e) {
  if (!isDragging.value) return
  
  e.preventDefault() // Prevent scrolling on touch devices
  
  const clientX = e.type.includes('mouse') ? e.clientX : e.touches[0].clientX
  const deltaX = clientX - startX.value
  
  // Add some resistance when dragging beyond certain thresholds
  if (Math.abs(deltaX) > 150) {
    const resistance = 0.3
    const excessDistance = Math.abs(deltaX) - 150
    const sign = deltaX > 0 ? 1 : -1
    currentX.value = sign * (150 + excessDistance * resistance)
  } else {
    currentX.value = deltaX
  }
}

function endDrag() {
  if (!isDragging.value) return
  
  isDragging.value = false
  isAnimating.value = true
  
  const threshold = 80 // Reduced threshold for easier swiping
  
  if (currentX.value > threshold) {
    // Swipe right - like
    animateCardOut(350, true)
  } else if (currentX.value < -threshold) {
    // Swipe left - dislike  
    animateCardOut(-350, false)
  } else {
    // Snap back to center
    animateCardBack()
  }
  
  // Clean up event listeners
  document.removeEventListener('mousemove', onDrag)
  document.removeEventListener('mouseup', endDrag)
  document.removeEventListener('touchmove', onDrag)
  document.removeEventListener('touchend', endDrag)
}

function animateCardOut(targetX, liked) {
  currentX.value = targetX
  setTimeout(() => {
    emit('swipe', { cat: props.cat, liked })
    isAnimating.value = false
  }, 300) 
}

function animateCardBack() {
  currentX.value = 0
  setTimeout(() => {
    isAnimating.value = false
  }, 300) 
}

function animateOut(liked) {
  const targetX = liked ? 350 : -350
  animateCardOut(targetX, liked)
}

// expose method for parent
defineExpose({
  animateOut,
  isAnimating
})

// Keyboard swipe support
function handleKey(e) {
  if (isAnimating.value || isDragging.value) return
  
  if (e.key === 'ArrowRight') {
    isAnimating.value = true
    animateCardOut(350, true)
  }
  if (e.key === 'ArrowLeft') {
    isAnimating.value = true
    animateCardOut(-350, false)
  }
}

onMounted(() => {
  // Only top card should listen to keyboard
  if (props.cat.isTop) {
    document.addEventListener('keydown', handleKey)
  }
})

onBeforeUnmount(() => {
  document.removeEventListener('keydown', handleKey)
  document.removeEventListener('mousemove', onDrag)
  document.removeEventListener('mouseup', endDrag)
  document.removeEventListener('touchmove', onDrag)
  document.removeEventListener('touchend', endDrag)
})

const cardStyle = computed(() => {
  const rotation = currentX.value / 15 // Slightly less rotation for smoother feel
  const opacity = isDragging.value ? Math.max(0.85, 1 - Math.abs(currentX.value) / 600) : 1
  
  return {
    transform: `translateX(${currentX.value}px) rotate(${rotation}deg)`,
    opacity,
    transition: isDragging.value ? 'none' : 'transform 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275), opacity 0.3s ease-out',
    cursor: isDragging.value ? 'grabbing' : 'grab'
  }
})
</script>

<style scoped>
.swipe-card {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border-radius: 12px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
  background: #fff;
  overflow: hidden;
  user-select: none;
}

.cat-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #eee;
  color: #666;
  font-size: 1.2rem;
}
</style>