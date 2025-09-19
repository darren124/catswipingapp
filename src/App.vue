<template>
  <div class="app">
    <h1>Find Your Favourite Kitty</h1>

    <div v-if="loading">
      <p>Loading cats...</p>
    </div>

    <div v-else>
      <div v-if="currentIndex < cats.length" class="card-stack-container">
        <div class="card-stack">
          <SwipeCard
            v-for="(cat, index) in stackedCats"
            :key="cat.id"
            :cat="cat"
            ref="swipeCards"
            @swipe="handleSwipe"
            :style="{ zIndex: cats.length - (currentIndex + index) }"
          />
        </div>

        <div class="labels">
          <span class="dislike-label" @click="triggerSwipe(false)">&larr; Nope</span>
          <span class="like-label" @click="triggerSwipe(true)">Love &rarr;</span>
        </div>
      </div>

      <Summary
        v-else
        :likedCats="likedCats"
        @restart="restartCats"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import SwipeCard from './components/SwipeCard.vue'
import Summary from './components/Summary.vue'

const cats = ref([])
const currentIndex = ref(0)
const likedCats = ref([])
const swipeCards = ref([]) 
const loading = ref(true)


const stackedCats = computed(() => {
  return cats.value.slice(currentIndex.value, currentIndex.value + 3)
})

function handleSwipe({ cat, liked }) {
  if (liked) likedCats.value.push(cat)
  currentIndex.value++
}

function triggerSwipe(liked) {
  const card = swipeCards.value[0] 
  if (card && card.animateOut && !card.isAnimating) {
    card.animateOut(liked) 
  }
}

function handleKey(e) {
  if (e.key === 'ArrowRight') triggerSwipe(true)
  if (e.key === 'ArrowLeft') triggerSwipe(false)
}

onMounted(() => {
  fetchCats(12)
  document.addEventListener('keydown', handleKey)
})

onBeforeUnmount(() => {
  document.removeEventListener('keydown', handleKey)
})

async function fetchCats(count = 12) {
  loading.value = true
  const promises = Array.from({ length: count }, () =>
    fetch('https://cataas.com/cat?json=true').then(res => res.json())
  )

  const results = await Promise.all(promises)

  cats.value = results.map(item => ({
    id: item.id,
    url: `https://cataas.com/cat/${item.id}?width=500&height=500`
  }))

  loading.value = false
}

function restartCats() {
  likedCats.value = []
  currentIndex.value = 0
  fetchCats(12)
}
</script>

<style>
.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 1rem;
  font-family: sans-serif;
}

.card-stack-container {
  position: relative;
  width: 320px;
  height: 500px; 
  display: flex;
  flex-direction: column;
  align-items: center;
}

.card-stack {
  position: relative;
  width: 100%;
  height: 400px;
}

.action-buttons {
  display: flex;
  justify-content: space-around;
  width: 100%;
  margin-top: 12px;
}

.nope-btn, .love-btn {
  padding: 10px 18px;
  font-size: 1.2rem;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  transition: 0.2s;
}

.nope-btn {
  background: #ff6b6b;
  color: white;
}
.nope-btn:hover {
  background: #ff4d4d;
}

.love-btn {
  background: #2ed573;
  color: white;
}
.love-btn:hover {
  background: #28c76f;
}

.labels {
  width: 100%;
  display: flex;
  justify-content: space-between;
  margin-top: 12px;
  font-size: 1rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.dislike-label {
  color: #ff4757;
  background: rgba(255, 71, 87, 0.1);
  padding: 6px 10px;
  border-radius: 20px;
  border: 2px solid rgba(255, 71, 87, 0.2);
}

.like-label {
  color: #2ed573;
  background: rgba(46, 213, 115, 0.1);
  padding: 6px 10px;
  border-radius: 20px;
  border: 2px solid rgba(46, 213, 115, 0.2);
}

.dislike-label,
.like-label {
  cursor: pointer;
  transition: all 0.2s ease;
}

.dislike-label:hover {
  background: rgba(255, 71, 87, 0.2);
  border-color: rgba(255, 71, 87, 0.5);
  transform: scale(1.05);
  box-shadow: 0 4px 8px rgba(255, 71, 87, 0.2);
}

.like-label:hover {
  background: rgba(46, 213, 115, 0.2);
  border-color: rgba(46, 213, 115, 0.5);
  transform: scale(1.05);
  box-shadow: 0 4px 8px rgba(46, 213, 115, 0.2);
}

</style>
