<script setup>
defineProps({
  size: {
    type: String,
    required: true,
  },
  rows: {
    type: Number,
    required: false,
    default: 1,
  },
});

</script>

<template>
  <ul class="skeleton__list">
    <li
      v-for="row in rows" :key="row"
      class="skeleton"
      :class="`skeleton_size_${size}`"
    />
  </ul>
</template>

<style lang="scss" scoped>
@use '@vars/colors';

.skeleton {
  &__list {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  &_size {
    &_large {
      height: 2rem;
    }
    &_small {
      height: 0.75rem;
    }
  }

  border-radius: 0.5rem;
  background-image: linear-gradient(
    to right,
    darken(colors.$border, 5) 0%,
    darken(colors.$border, 0) 50%,
    darken(colors.$border, 5) 100%,
  );

  background-size: 200%;
  background-position-x: 0%;
  animation: move-skeleton 1.5s infinite linear;

  @keyframes move-skeleton {
    0% {
      background-position-x: 0%;
    }
    100% {
      background-position-x: 200%;
    }
  }
}
</style>
