<script setup>
defineProps({
  tag: {
    type: String,
    required: false,
    default: 'div',
  },
  color: {
    type: String,
    required: true,
  },
  margin: {
    type: String,
    required: true,
  },
});

const hexToRgb = (hex) => {
  const shorthandRegex = /^#?([a-f\d])([a-f\d])([a-f\d])$/i;
  const hexFullSize = hex.replace(shorthandRegex, (m, r, g, b) => r + r + g + g + b + b);
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hexFullSize);
  const color = {
    r: parseInt(result[1], 16),
    g: parseInt(result[2], 16),
    b: parseInt(result[3], 16),
  };
  return `${color.r} ${color.g} ${color.b}`;
};
</script>

<template>
  <component
    :is="tag"
    class="picture"
    :style="`
      --margin: ${margin};
      --color: ${hexToRgb(color)};
    `"
  />
</template>

<style lang="scss" scoped>
@use '@vars/colors';

.picture {
  background-color: transparent;
  font-size: var(--margin);
  aspect-ratio: 1;

  $margin-negative: calc(0px - var(--margin));

  position: relative;
  &::before, &::after {
    content: '';
    position: absolute;
    margin: calc(var(--margin) + var(--padding));
  }
  &::before {
    z-index: 2;
    inset: $margin-negative $margin-negative 0 0;
    background-color: rgb(var(--color) / 0.5);
    backdrop-filter: blur(0.5em);
    filter: brightness(115%);
  }
  &::after {
    z-index: 1;
    inset: 0 0 $margin-negative $margin-negative;
    background-color: rgb(var(--color));
  }

  &:focus {
    box-shadow: 0 0 0 4px colors.$border;
    outline: none;
    z-index: 100;
  }
}
</style>
