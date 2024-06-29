<script setup>
import { ref, watch, onMounted } from 'vue'
import SliderBtn from '@/components/ui/SliderBtn.vue'

const isChecked = ref(false)

let mql

onMounted(() => {
  mql = window.matchMedia('(prefers-color-scheme: dark)')
  if (mql.matches) {
    isChecked.value = true
  } else {
    isChecked.value = false
  }
})

const setBgColorDark = () => {
  document.body.style.setProperty('--color-background', 'var(--vt-c-black)')
  document.body.style.setProperty('--color-background-soft', 'var(--vt-c-black-soft)')
  document.body.style.setProperty('--color-background-mute', 'var(--vt-c-black-mute)')
  document.body.style.setProperty('--color-border', 'var(--vt-c-divider-dark-2)')
  document.body.style.setProperty('--color-border-hover', 'var(--vt-c-divider-dark-1)')
  document.body.style.setProperty('--color-heading', 'var(--vt-c-text-dark-1)')
  document.body.style.setProperty('--color-text', 'var(--vt-c-text-dark-2)')
}

const setBgColorLight = () => {
  document.body.style.setProperty('--color-background', 'var(--vt-c-white)')
  document.body.style.setProperty('--color-background-soft', 'var(--vt-c-white-soft)')
  document.body.style.setProperty('--color-background-mute', 'var(--vt-c-white-mute)')
  document.body.style.setProperty('--color-border', 'var(--vt-c-divider-light-2)')
  document.body.style.setProperty('--color-border-hover', 'var(--vt-c-divider-light-1)')
  document.body.style.setProperty('--color-heading', 'var(--vt-c-text-light-1)')
  document.body.style.setProperty('--color-text', 'var(--vt-c-text-light-2)')
}
watch(isChecked, (value) => {
  if (value) {
    setBgColorDark()
  } else {
    setBgColorLight()
  }
})
</script>
<template>
  <header>
    <div :style="{ color: '#ffffff' }">
      <!-- <img src="/logo.png" width="40px"> -->
      Hey, Robert! We're almost done with this project, but to finish perfectly and quickly, we need
      to have some discussions. I'd like to schedule a call or Skype chat to finalize this task and
      go over the next steps. You can reach me on Skype at live:.cid.11c53944c322323d. Thank you!
    </div>
    <!-- <nav>
      <router-link to="/">Home</router-link>
      <router-link to="/about">About</router-link>
      <router-link to="/Other">Other</router-link>
    </nav> -->
    <div class="slider">
      <SliderBtn v-model="isChecked" />
      <span>{{ isChecked ? 'dark-theme' : 'light-theme' }}</span>
    </div>
    <!-- <div>
      <router-link to="/profile">My profile</router-link>
    </div> -->
  </header>
</template>

<script setup></script>

<style lang="sass" scoped>
header
    width: 100%
    padding-block: 1rem
    padding-inline: 1rem

    display: flex
    justify-content: space-between
    align-items: center

    nav
        display: flex
        gap: 1rem

    .slider
      display: flex
      gap: 1rem
</style>
