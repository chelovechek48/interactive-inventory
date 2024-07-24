<script setup>

</script>

<template>
  <main class="page">
    <div class="page__container">
      <section class="page__item" style="grid-area: modal" />
      <section class="page__item" style="grid-area: field" />
      <section class="page__item" style="grid-area: alert" />
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

    display: grid;
    gap: 1rem;

    @media (min-width: calc($mobile + 1px)) {
      grid-template-columns: clamp(15rem, 35vw, 17rem) auto;
      grid-template-areas:
        'modal field'
        'alert alert';
    }
    @media (max-width: $mobile) {
      grid-template-columns: 1fr;
      grid-template-areas:
        'alert'
        'field'
        'modal';
    }
  }

  &__item {
    background-color: colors.$ui;
    height: 20rem;
    border-radius: 0.75rem;
  }
}
</style>
