<script setup>
import { computed } from 'vue'
const props = defineProps({ currentSlide: { type: Number, required: true } })
defineEmits(['select'])
const items = [
  { image: 'hairdryer', label: 'Фен-стайлер 2 в 1' },
  { image: 'feather', label: 'Суперлёгкий' },
  { image: 'wheel', label: 'Инновационный японский мотор' },
]
const rotation = computed(() => -props.currentSlide * 120)
</script>

<template>
  <nav class="slide-wheel" aria-label="Слайды презентации">
    <div aria-hidden="true" class="wheel-ring" :style="{ transform: `rotate(${rotation}deg)` }">
      <div v-for="(item, index) in items" :key="item.image" class="wheel-spoke" :style="{ transform: `rotate(${index * 120}deg)` }">
        <div class="wheel-button" :class="{ 'wheel-button-active': currentSlide === index }" :style="{ transform: `translate(50%, -50%) rotate(${-rotation - index * 120}deg)` }">
          <img :src="`/svg/${item.image}.svg`" alt="" class="wheel-icon">
        </div>
      </div>
    </div>
    <Transition name="wheel-label" mode="out-in"><p :key="currentSlide" class="wheel-caption">{{ items[currentSlide].label }}</p></Transition>
    <button v-for="offset in [-1, 1]" :key="offset" type="button" class="wheel-neighbor" :class="offset < 0 ? 'wheel-neighbor-top' : 'wheel-neighbor-bottom'" :aria-label="items[(currentSlide + offset + 3) % 3].label" @click="$emit('select', (currentSlide + offset + 3) % 3)">
      <img :src="`/svg/${items[(currentSlide + offset + 3) % 3].image}.svg`" alt="">
    </button>
  </nav>
</template>
