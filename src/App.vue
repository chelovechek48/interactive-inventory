<script setup>

</script>

<template>
  <main class="page">
    <div class="page__container">
      123
    </div>
  </main>
</template>

<style lang="scss">
@forward '@styles';
@use '@vars/colors';
@use '@vars/breakpoints' as *;

.page {
  background-color: colors.$background;
  min-height: 100dvh;
  padding: $container-padding;

  display: flex;
  align-items: center;
  justify-content: center;

  &__container {
    width: min(100vw, $container-width);
    background-color: white;
  }
}
</style>
