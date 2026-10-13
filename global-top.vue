<script setup lang="ts">
import { useNav } from '@slidev/client'

const { currentPage, total } = useNav()
</script>

<template>
  <img
    v-if="currentPage !== 1 && currentPage !== total"
    src="/viti-logo.svg"
    class="absolute top-5 right-6 h-9 opacity-80"
  />
</template>
