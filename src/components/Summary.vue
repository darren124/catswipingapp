<template>
  <div class="summary">
    <h2>You liked {{ likedCats.length }} cats 🐾</h2>
    <div class="liked-list">
      <img
        v-for="cat in likedCats"
        :key="cat.id"
        :src="cat.url"
        alt="Liked cat"
        @click="openImage(cat.url)"
      />
    </div>

    <br />
    <h3>Want more cat?</h3>
    <button class="restart-btn" @click="$emit('restart')">Yes Please!</button>

    <div v-if="zoomedImage" class="modal" @click="closeImage">
      <img :src="zoomedImage" alt="Zoomed cat" />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  likedCats: Array
})
defineEmits(['restart'])

const zoomedImage = ref(null)

function openImage(url) {
  zoomedImage.value = url
}

function closeImage() {
  zoomedImage.value = null
}
</script>

<style scoped>
.summary {
  text-align: center;
}
.liked-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
  margin-bottom: 1rem;
}
.liked-list img {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.2s;
}
.liked-list img:hover {
  transform: scale(1.1);
}
.restart-btn {
  margin-top: 0.3rem;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 8px;
  background: #4caf50;
  color: white;
  font-size: 1rem;
  cursor: pointer;
}
.restart-btn:hover {
  background: #45a049;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}
.modal img {
  max-width: 90%;
  max-height: 90%;
  border-radius: 10px;
}
</style>
