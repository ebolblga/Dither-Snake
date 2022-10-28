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
      id: "favicon",
    },
  ],
});
</script>

<template>
  <div class="h-screen">
    <slot />
  </div>
</template>

<style>
body {
  background: #616161;
  background-size: 100% 100%;
  font-size: 14px;
  color: #FFFFFF;
  margin: 0;
  font-family: Pixie6x6;
}

@font-face {
  font-family: 'Pixie6x6';
  src: url('/Pixie6x6.ttf');
}
</style>
