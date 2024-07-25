<script setup>
import { ref, onMounted, onUpdated } from 'vue';
import ItemPicture from '@components/ItemPicture.vue';

const props = defineProps({
  list: {
    type: Array,
    required: true,
  },
});

const emit = defineEmits(['changeFocus']);

const inventoryListDOM = ref();

const getItemIndex = (target) => {
  const targetId = target.getAttribute('data-id');
  const targetItem = props.list.find((item) => Number(targetId) === Number(item.id));
  const targetIndex = props.list.indexOf(targetItem);
  return targetIndex;
};

const focusHandler = () => {
  inventoryListDOM.value.addEventListener('focusin', (event) => {
    const focusIndex = getItemIndex(event.target.parentNode);
    emit('changeFocus', focusIndex);
  });
};

onMounted(() => {
  focusHandler();
});

onUpdated(() => {
  const inventoryListstWithEmptyCells = props.list.map((inventoryItem, index) => {
    const hasProperties = inventoryItem.properties;
    if (hasProperties) {
      const id = index;
      const { ...properies } = inventoryItem.properties;
      return { id, ...properies };
    }
    return null;
  });

  const inventoryListstt = inventoryListstWithEmptyCells.filter((isItem) => (isItem));
  localStorage.setItem('items', JSON.stringify(inventoryListstt));
});

</script>

<template>
  <ul class="field" ref="inventoryListDOM">
    <li
      v-for="item in props.list" :key="item.id"
      class="field__item"
      :data-id="item.id"
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
