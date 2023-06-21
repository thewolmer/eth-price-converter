<script setup>
import { useDark, useToggle } from '@vueuse/core'
const isDark = useDark()
const toggleDark = useToggle(isDark)
</script>

<template>
  <div class="flex gap-x-6 items-center justify-center text-center p-8 base-color">
    <router-link to="/">
      <div
        class="flex hover:bg-primary-active/50 px-1 bg-primary-active/30 cursor-pointer transition-all"
      >
        Converter
      </div>
    </router-link>
    <router-link to="/search">
      <div
        class="flex hover:bg-primary-active/50 px-1 bg-primary-active/30 cursor-pointer transition-all"
      >
        Wallet Checker
      </div>
    </router-link>
  </div>
</template>

<script>
export default {
  name: 'NavBar'
}
</script>

<style></style>
