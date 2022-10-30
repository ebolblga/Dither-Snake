<script setup lang="ts">
const {
  public: { base },
} = useRuntimeConfig();
useHead({
  link: [
    {
      rel: "icon",
      href: `${base}favicon.ico`,
      type: "image/x-icon",
      id: "favicon"
    }
  ],
  htmlAttrs: {
      lang: 'en'
    },
  meta: [
    { name: 'description', content: 'Author: ebolblga, Snake game with shading and dithering.' }
  ],
});
</script>

<template>
  <div class="h-screen">
    <link rel="preload" href="/Dither-Snake/Pixie6x6.ttf" as="font" type="font/woff2" />
    <slot />
  </div>
</template>

<style>
body {
  background: #000000;
  background-size: 100% 100%;
  font-size: 14px;
  color: #FFFFFF;
  margin: 0;
  font-family: Pixie6x6;
  cursor: url("/cursor.cur"), auto;
}

@font-face {
  font-family: 'Pixie6x6';
  src: url('/Pixie6x6.ttf');
}
</style>
