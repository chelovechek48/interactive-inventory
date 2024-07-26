<template>
  <button class="close-button" />
</template>

<style lang="scss" scoped>
@use '@vars/colors';
.close-button {
  $height: 1.5rem;
  min-width: $height;
  min-height: $height;

  margin-left: auto;

  position: relative;
  background-color: transparent;

  &::before, &::after {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    translate: -50% -50%;
    width: 100%;
    height: 2px;

    border-radius: 1rem;
    background-color: #fff;
  }

  &::before {
    rotate: -45deg;
  }
  &::after {
    rotate: 45deg;
  }
}
</style>
