<!-- <span class="author-block">
  <a href="https://shuyanzhou.github.io/">Shuyan Zhou</a><sup>1*</sup>,
</span> -->
<script setup lang="ts">
import { defineProps } from 'vue'

defineProps<{
  author: string
  sup?: string
  href?: string
  image?: string
  last?: boolean
}>()
</script>

<template>
  <div class="author-block">
    <img :src="image" :alt="author" class="author-image" />
    <div class="author-info">
      <a
        :href="href"
        :class="{ 'text-blue-500': !!href, 'text-lg': true, 'subpixel-antialiased': true }"
        >{{ author }}</a
      >
      <sup v-if="sup">{{ sup }}</sup>
      <span v-if="!last">,</span>
      <span v-if="last">.</span>
    </div>
  </div>
</template>

<style scoped lang="scss">
.author-block {
  display: flex;
  align-items: center;
  flex-direction: column;
  width: 100%;
  grid-column: span 2;
  .author-image {
    width: 100%;
    max-width: 120px;
    border-radius: 50%;
  }
  .author-info {
    text-align: center;
  }
}
@media (max-width: 1024px) {
  .author-block {
    display: inline-block;
    width: auto;
    img {
      display: none;
    }
  }
}

@supports (-webkit-hyphens: none) {
  .author-block {
    display: inline-block;
    width: auto;
    img {
      display: none;
    }
  }
}
</style>
