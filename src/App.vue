<script setup>
import { ref, onBeforeMount } from 'vue';
import InventoryModal from '@components/InventoryModal.vue';
import InventoryField from '@components/InventoryField.vue';
import LazySkeleton from '@components/LazySkeleton.vue';

import inventoryItemsDataDefault from '@json/items';

const inventorySlotsCount = 25;

const inventoryList = ref(Array.from(
  { length: inventorySlotsCount },
  (_, index) => (
    { id: index }
  ),
));

const setInventoryField = () => {
  let inventoryItemsData = JSON.parse(localStorage.getItem('inventory-items'));
  if (!inventoryItemsData) {
    inventoryItemsData = inventoryItemsDataDefault;
    localStorage.setItem('inventory-items', JSON.stringify(inventoryItemsDataDefault));
  }

  inventoryItemsData.forEach((inventoryItem) => {
    const { id, ...properties } = inventoryItem;
    inventoryList.value[id] = {
      id,
      properties,
    };
  });
};

const selectedItem = ref({});

const changeItemCount = (newCount) => {
  if (newCount > 0) {
    selectedItem.value.properties.count = newCount;
    selectedItem.value.dom.focus();
  } else {
    delete selectedItem.value.properties;
  }
};

onBeforeMount(() => {
  setInventoryField();
});
</script>

<template>
  <main class="page">
    <div class="page__container">
      <InventoryModal
        class="page__item"
        style="grid-area: modal"
        :item="selectedItem"
        @changeItemCount="changeItemCount($event)"
      />
      <InventoryField
        class="page__item"
        style="grid-area: field"
        :list="inventoryList"
        :slots-count="inventorySlotsCount"
        @changeFocus="selectedItem = $event"
        @updateInventoryField="inventoryList = $event"
      />
      <aside class="page__item page__alert" style="grid-area: alert">
        <LazySkeleton size="large" />
      </aside>
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
    border-radius: 0.75rem;
    overflow: hidden;
    border: 1px solid colors.$border;
  }

  &__alert {
    padding: 1rem;
  }
}
</style>
