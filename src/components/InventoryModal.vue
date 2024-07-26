<script setup>
import { ref, onUpdated } from 'vue';
import ItemPicture from '@components/ItemPicture.vue';
import LazySkeleton from '@components/LazySkeleton.vue';

const emit = defineEmits(['changeItemCount']);
const props = defineProps({
  item: {
    type: Object,
    required: true,
  },
});

const numberCount = ref(null);
const deleteIsActive = ref(false);
const numberDOM = ref();

const transitionDuration = 400;

const showDeleteForm = () => {
  deleteIsActive.value = true;
  setTimeout(() => {
    numberDOM.value.focus();
  }, transitionDuration);
};

const hideDeleteForm = () => {
  deleteIsActive.value = false;
  numberCount.value = null;
};

let prevItem;
onUpdated(() => {
  const isNewItem = prevItem !== props.item;
  if (isNewItem) {
    prevItem = props.item;
    hideDeleteForm();
  }
});

const changeItemCount = (submit) => {
  submit.preventDefault();

  const { count } = props.item.properties;
  emit('changeItemCount', count - numberCount.value);

  hideDeleteForm();
};

</script>

<template>
  <aside
    v-if="item.properties"
    class="modal" :style="`--transition-duration: ${transitionDuration}ms;`"
  >
    <ItemPicture
      margin="1.25rem"
      :color="item.properties.color"
    />
    <span class="modal__separator" />
    <section class="modal__text">
      <LazySkeleton size="large" />
      <LazySkeleton size="small" :rows="10" />
    </section>
    <span class="modal__separator modal__separator_long" />
    <form
      class="modal__count"
      :class="deleteIsActive ? 'active' : ''"
      @submit="changeItemCount($event)"
    >
      <div class="modal__number-wrapper">
        <input
          type="number" name="" id=""
          ref="numberDOM"
          class="modal__number"
          placeholder="Введите количество"
          v-model="numberCount"
          min="1"
          :max="item.properties.count"
          required
        >
      </div>
      <div
        class="modal__button-list"
        :class="deleteIsActive ? 'active' : ''"
      >
        <div class="modal__button-group">
          <button
            type="button"
            class="modal__button modal__button_color_red"
            @click="showDeleteForm"
            :disabled="deleteIsActive"
          >
            Удалить предмет
          </button>
        </div>
        <div class="modal__button-group">
          <button
            type="button"
            class="modal__button modal__button_color_white"
            @click="hideDeleteForm"
            :disabled="!deleteIsActive"
          >
            Отмена
          </button>
          <button
            class="modal__button modal__button_color_red"
            type="submit"
            :disabled="!deleteIsActive"
          >
            Подтвердить
          </button>
        </div>
      </div>
    </form>
  </aside>
  <aside v-else class="modal">
    <div class="modal__image-wrapper">
      <img class="modal__image" src="@images/modal-image.jpg" alt="">
    </div>
    <div class="modal__text">
      <LazySkeleton size="large" />
      <LazySkeleton size="small" :rows="10" />
    </div>
  </aside>
</template>

<style lang="scss" scoped>
@use '@vars/colors';

.modal {
  $padding: 1rem;
  $gap: 1rem;
  $button-height: 2.5em;

  overflow: hidden;
  background-color: colors.$ui;
  padding: #{$padding};
  --padding: #{$padding};

  display: flex;
  flex-direction: column;
  gap: $gap;

  &__separator {
    height: 1px;
    background-color: colors.$border;

    &_long {
      margin-inline: -1rem;
      @media (max-width: 32rem) {
        display: none;
      }
    }
  }

  &__image {
    &-wrapper {
      position: relative;
      &::before {
        content: '';
        position: absolute;
        inset: 0;
        background-color: rgba(#fff, 0.1);
        backdrop-filter: blur(5px);
      }

    }
    width: 100%;
    @media (min-width: 32rem) {
      aspect-ratio: 200 / 240;
    }
    @media (max-width: 32rem) {
      aspect-ratio: 4 / 3;
    }
    object-fit: cover;

    &, &-wrapper::before {
      border-radius: 0.5rem;
      overflow: hidden;
    }
  }

  &__text {
    display: flex;
    flex-direction: column;
    gap: inherit;
    overflow: hidden;

    @media (min-width: 32rem) {
      flex: 1 1 0;
      margin: -1rem;
      padding: 1rem;
    }
  }

  &__count {
    display: flex;
    gap: $gap;

    @media (min-width: 32rem) {
      flex-direction: column;
    }
    @media (max-width: 32rem) {
      flex-direction: column-reverse;
      order: -1;
    }
  }

  &__count &__number {
    opacity: 0;
    visibility: hidden;
  }
  &__count.active &__number {
    opacity: 1;
    visibility: visible;
  }

  &__count &__number-wrapper {
    height: 0;
    margin-bottom: calc(0px - $gap);
  }
  &__count.active &__number-wrapper {
    height: $button-height;
    margin-bottom: 0px;
  }

  &__number {
    width: 100%;
    color: #fff;
    background-color: transparent;
    border: 1px solid colors.$border;
    border-radius: 0.35rem;

    &, &-wrapper {
      transition: all var(--transition-duration);
    }
    &-wrapper {
      z-index: 1;
    }
  }

  &__number,
  &__button {
    font-size: 1rem;
    padding-inline: 0.75rem;
    height: $button-height;
  }

  &__button {
    border-radius: 0.5rem;
    flex: 1 1 100%;

    &-group {
      z-index: 2;
      width: calc(50% - $padding);
      display: flex;
      gap: $gap;
    }

    &-list {
      width: calc((100% + $padding) * 2);

      display: flex;
      gap: $padding;

      margin-left: 0%;
      &.active {
        margin-left: calc(-100% - $padding);
      }
      transition: margin var(--transition-duration);
    }

    &_color {
      &_red {
        color: colors.$white;
        background-color: colors.$pink;
      }
      &_white {
        color: colors.$ui;
        background-color: colors.$white;
      }
    }
  }
}
</style>
