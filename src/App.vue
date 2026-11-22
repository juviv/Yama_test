<script setup>
import { onMounted, onBeforeUnmount, computed, ref } from 'vue'
import SlideWheel from './components/SlideWheel.vue'
const currentSlide = ref(0)
const leavingSlide = ref(null)
const direction = ref(1)
const slides = ref([])
const transitioning = computed(() => leavingSlide.value !== null)
let transitionTimer
let lastWheelAt = 0
let wheelDistance = 0
let touchStart = null
function goToSlide(index, step) {
  const next = (index + slides.value.length) % slides.value.length
  if (transitioning.value || next === currentSlide.value) return
  direction.value = step ?? ((next - currentSlide.value + slides.value.length) % slides.value.length === 1 ? 1 : -1)
  leavingSlide.value = currentSlide.value
  slides.value[next].scrollTop = direction.value > 0 ? 0 : slides.value[next].scrollHeight
  currentSlide.value = next
  const reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  transitionTimer = setTimeout(() => {
    leavingSlide.value = null
  }, reduced ? 50 : 800)
}
function canScrollInside(step) {
  const slide = slides.value[currentSlide.value]
  return step > 0 ? slide.scrollTop + slide.clientHeight < slide.scrollHeight - 2 : slide.scrollTop > 2
}
function onWheel(event) {
  if (event.ctrlKey || Math.abs(event.deltaY) <= Math.abs(event.deltaX)) return
  const now = performance.now()
  const fresh = now - lastWheelAt > 180
  lastWheelAt = now
  if (fresh) wheelDistance = 0
  if (transitioning.value) { event.preventDefault(); return }
  const step = Math.sign(event.deltaY)
  if (canScrollInside(step)) return
  event.preventDefault()
  const delta = event.deltaY * (event.deltaMode === 1 ? 16 : event.deltaMode === 2 ? innerHeight : 1)
  if (Math.sign(wheelDistance) !== step) wheelDistance = 0
  if (wheelDistance === Infinity || wheelDistance === -Infinity) return
  wheelDistance += delta
  if (Math.abs(wheelDistance) >= 35) {
    wheelDistance = step * Infinity
    goToSlide(currentSlide.value + step, step)
  }
}
function onTouchStart(event) {
  touchStart = event.touches.length === 1
    ? { y: event.touches[0].clientY, atTop: !canScrollInside(-1), atBottom: !canScrollInside(1) } : null
}
function onTouchEnd(event) {
  if (!touchStart) return
  const gesture = touchStart
  touchStart = null
  const distance = gesture.y - event.changedTouches[0].clientY
  const step = Math.sign(distance)
  if (Math.abs(distance) > 50 && (step > 0 ? gesture.atBottom : gesture.atTop)) goToSlide(currentSlide.value + step, step)
}
function onTouchCancel() { touchStart = null }
function onKey(event) {
  if (event.target.closest('button')) return
  const step = ['ArrowDown', 'ArrowRight', 'PageDown'].includes(event.key) ? 1 : ['ArrowUp', 'ArrowLeft', 'PageUp'].includes(event.key) ? -1 : event.code === 'Space' ? (event.shiftKey ? -1 : 1) : 0
  if (step) { event.preventDefault(); goToSlide(currentSlide.value + step, step) }
}
function slideState(index) {
  return { 'slide-active': currentSlide.value === index, 'slide-entering': transitioning.value && currentSlide.value === index, 'slide-leaving': leavingSlide.value === index }
}
onMounted(() => {
  window.addEventListener('keydown', onKey)
  window.addEventListener('wheel', onWheel, { passive: false })
  window.addEventListener('touchstart', onTouchStart, { passive: true })
  window.addEventListener('touchend', onTouchEnd, { passive: true })
  window.addEventListener('touchcancel', onTouchCancel)
})
onBeforeUnmount(() => {
  clearTimeout(transitionTimer)
  window.removeEventListener('keydown', onKey)
  window.removeEventListener('wheel', onWheel)
  window.removeEventListener('touchstart', onTouchStart)
  window.removeEventListener('touchend', onTouchEnd)
  window.removeEventListener('touchcancel', onTouchCancel)
})
</script>

<template>
  <main class="presentation-stage" :style="{ '--slide-direction': direction }">
    <SlideWheel :current-slide="currentSlide" @select="goToSlide" />
    <section :ref="el => slides[0] = el" class="presentation-slide" :class="slideState(0)" :inert="currentSlide !== 0" :aria-hidden="currentSlide !== 0" aria-labelledby="design-title">
      <div class="design-panel slide-panel">
        <h1 id="design-title" class="design-title text-center font-bold text-3xl leading-[1.08]">ФЕН и СТАЙЛЕР<br>в уникальном дизайне</h1>
        <div class="product-gallery">
          <figure class="dryer-card product-card">
            <figcaption class="product-label bg-[#545756] text-white">Фен</figcaption>
            <img src="/images/slide_1_girl_with_hairdryer.png" alt="Укладка волос феном Yamaguchi"  class="w-full shadow-[0_8px_24px_-4px_rgba(65,60,57,0.25)]" fetchpriority="high">
          </figure>
          <figure class="dual-card flex flex-col items-center">
            <figcaption class="mt-6 rounded-full bg-white px-3 py-1 text-3xl font-semibold shadow-[9.95px_8.38px_15.71px_-3.14px_rgba(65,60,57,0.21)]">2 в 1</figcaption>
            <img src="/images/slide_1_long_hair.png" alt="Фен и стайлер для длинных волос" width="674" height="546" class="hidden min-h-0 object-contain object-bottom md:block" fetchpriority="high">
          </figure>
          <figure class="styler-card product-card">
            <figcaption class="product-label bg-white">Стайлер</figcaption>
            <img src="/images/slide_1_girl_with_styler.png" alt="Создание локонов стайлером Yamaguchi" width="245" height="429" class="w-full shadow-[0_8px_24px_-4px_rgba(65,60,57,0.25)]" fetchpriority="high">
          </figure>
        </div>
      </div>
    </section>

    <section :ref="el => slides[1] = el" class="presentation-slide" :class="slideState(1)" :inert="currentSlide !== 1" :aria-hidden="currentSlide !== 1" aria-labelledby="weight-title">
      <div class="mobile-feature"><img src="/svg/feather.svg" alt=""> <span class="text-xl">ЛЁГКОСТЬ<br>И КОМФОРТ<br>В РУКЕ</span></div>
      <div class="weight-panel slide-panel">
        <div class="weight-copy text-center text-white">
          <h1 id="weight-title" class="weight-title font-bold uppercase leading-none">Супер<span class="hidden md:inline">-<br></span>лёгкий</h1>
          <p class="weight-description mx-auto max-w-64 leading-relaxed">Уменьшает усталость рук при сушке волос, делая процедуру удобной и приятной даже после напряжённого рабочего дня.</p>
          <span class="weight-tag inline-block rounded-full bg-white px-6 py-1.5 text-xl font-semibold text-[#4d4d4d]">500 г</span>
        </div>
        <img src="/images/slide_2_girl.png" alt="Девушка держит лёгкий стайлер и чашку кофе" width="669" height="800" class="weight-image hidden md:block" loading="lazy">
        <p class="coffee-badge star-badge font-semibold">Не тяжелее<br>чашки кофе</p>
      </div>
    </section>

    <section :ref="el => slides[2] = el" class="presentation-slide" :class="slideState(2)" :inert="currentSlide !== 2" :aria-hidden="currentSlide !== 2" aria-labelledby="motor-title">
      <div class="mobile-feature"><img src="/svg/wheel.svg" alt="" width="76"><span class="font-semibold">ИННОВАЦИОННЫЙ<br>ЯПОНСКИЙ МОТОР</span></div>
      <div class="motor-panel slide-panel md:p-[clamp(1rem,4vmin,3rem)]">
        <h2 id="motor-title" class="motor-title font-bold uppercase text-center md:text-right">
          <span class="text-[33px] md:text-[29px]">Для ценителей </span>
          <span class="text-[44px] md:text-[29px]">комфорта<br> и качества</span>
          <span class="text-[23px] md:text-[29px]"> в уходе за волосами</span>
        </h2>
        <div class="motor-image" role="img" aria-label="Устройство бесщёточного мотора BLDC"></div>
        <p class="motor-caption font-semibold">Бесщеточный мотор BLDC</p>
        <p class="power-badge star-badge">Салонная<br>мощность</p>
      </div>
    </section>
  </main>
</template>
