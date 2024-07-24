<script setup>
import ItemPicture from '@components/ItemPicture.vue';

const props = defineProps({
  list: {
    type: Array,
    required: true,
  },
});

</script>

<template>
  <ul class="field">
    <li
      v-for="item in props.list" :key="item.id"
      class="field__item"
    >
      <ItemPicture
        v-if="item.properties"
        tag="button"
        margin="0.375rem"
        :color="item.properties.color"
      />
      <span v-if="item.properties" class="field__count">
        {{ item.properties.count }}
      </span>
    </li>
  </ul>
</template>

<style lang="scss" scoped>
@use '@vars/colors';

.field {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(clamp(4rem, 11.5vw, 6rem), 1fr));
  gap: 1px;

  &__item {
    position: relative;
    display: flex;
    aspect-ratio: 1;
    box-shadow: 0 0 0 1px colors.$border;

    --padding: 20%;
  }

  &__count {
    color: #7d7d7d;

    width: 1.25rem;
    height: 1.25rem;
    position: absolute;
    bottom: 0;
    right: 0;

    border-top-left-radius: 0.75rem;
    box-shadow: inherit;

    display: flex;
    align-items: center;
    justify-content: center;
  }
}
</style>
