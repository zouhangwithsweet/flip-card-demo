<script setup lang="ts">
import { ref, onMounted } from 'vue'
import autoAnimate, { getTransitionSizes } from '@formkit/auto-animate'

const dropdown = ref() // we need a DOM node
const show = ref(false)

const slider = ref()
const slide = ref(true)

const top = ref(false)

onMounted(() => {
  autoAnimate(slider.value)
  autoAnimate(dropdown.value, (el, action, oldCoords: any, newCoords: any) => {
    let keyframes: any
    // supply a different set of keyframes for each action
    if (action === 'add') {
      keyframes = [
        { transform: 'rotateY(125deg)', opacity: 1 },
        { transform: 'rotateY(75deg)', opacity: 1, offset: 0.75 },
        { transform: 'rotateY(0deg)', opacity: 1 },
      ]
    }
    // keyframes can have as many "steps" as you prefer
    // and you can use the 'offset' key to tune the timing
    if (action === 'remove') {
      keyframes = [
        { transform: 'rotateY(0deg)', opacity: 1 },
        { transform: 'rotateY(45deg)', opacity: 1, offset: 0.33 },
        { transform: 'rotateY(75deg)', opacity: 0.1, offset: 0.5 },
        { transform: 'rotateY(125deg)', opacity: 0 },
      ]
    }
    if (action === 'remain') {
      // for items that remain, calculate the delta
      // from their old position to their new position
      const deltaX = oldCoords.left - newCoords.left
      const deltaY = oldCoords.top - newCoords.top
      // use the getTransitionSizes() helper function to
      // get the old and new widths of the elements
      const [widthFrom, widthTo, heightFrom, heightTo] = getTransitionSizes(el, oldCoords, newCoords)
      // set up our steps with our positioning keyframes
      const start: any = { transform: `translate(${deltaX}px, ${deltaY}px)` }
      const mid: any = { transform: `translate(${deltaX * -0.15}px, ${deltaY * -0.15}px)`, offset: 0.75 }
      const end: any = { transform: `translate(0, 0)` }
      // if the dimensions changed, animate them too.
      if (widthFrom !== widthTo) {
        start.width = `${widthFrom}px`
        mid.width = `${widthFrom >= widthTo ? widthTo / 1.05 : widthTo * 1.05}px`
        end.width = `${widthTo}px`
      }
      if (heightFrom !== heightTo) {
        start.height = `${heightFrom}px`
        mid.height = `${heightFrom >= heightTo ? heightTo / 1.05 : heightTo * 1.05}px`
        end.height = `${heightTo}px`
      }
      keyframes = [start, mid, end]
    }
    // return our KeyframeEffect() and pass
    // it the chosen keyframes.
    return new KeyframeEffect(el, keyframes, { duration: 500, easing: 'ease-out' })
  })
})
</script>

<template>
  <div
    ref="dropdown"
    class="relative dropdown p-4 transition-all duration-600 ease-out"
    :class="show ? 'bg-blue-100' : 'bg-green-100'"
  >
    <strong class="flex-center border border-solid border-blue-300 rounded-lg mb-2 px-3 py-2" @click="show = !show"
      >Click me to flip!</strong
    >
    <p class="flex-center h-50 m-auto bg-blue-200 rounded-lg" v-if="show">
      {{ show && 'Lorum ipsum...' }}
    </p>
    <p class="flex-center h-50 m-auto bg-green-200 rounded-lg" v-else>
      {{ !show && 'Fazi...' }}
    </p>
  </div>

  <div
    class="p-3 bg-gray-300 h-xl w-full transition duration-400 ease-out"
    :class="[slide ? 'translate-y-50' : 'translate-y-0', top ? '!-translate-y-70' : '']"
  >
    <div ref="slider">
      <strong class="flex-center rounded-lg border border-solid border-blue-300 px-3 py-2" @click="slide = !slide"
        >Click me to slide!</strong
      >

      <input
        v-if="!slide"
        placeholder="please input"
        @focus="top = true"
        @blur="top = false"
        class="mt-100px mb-2 px-3 block h-12 w-full rounded-lg"
      />
      <button v-if="!slide" class="w-full h-12 flex-center bg-white rounded-lg">Submit</button>

      <div v-if="top" class="flex justify-between gap-2 h-25 mt-2">
        <div v-for="i in 2" class="rounded-lg flex-1 bg-emerald-200"></div>
      </div>
    </div>
  </div>
</template>
