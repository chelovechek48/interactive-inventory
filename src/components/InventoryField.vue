<script setup>
import { ref, onMounted, onUpdated } from 'vue';
import ItemPicture from '@components/ItemPicture.vue';

const emit = defineEmits(['changeFocus', 'updateInventoryField']);
const props = defineProps({
  list: {
    type: Array,
    required: true,
  },
  slotsCount: {
    type: Number,
    required: true,
  },
});

const inventoryListDOM = ref();
const numberItemsInRow = ref(undefined);

const getItemIndex = (target) => {
  const targetId = target.getAttribute('data-id');
  const targetItem = props.list.find((item) => Number(targetId) === Number(item.id));
  const targetIndex = props.list.indexOf(targetItem);
  return targetIndex;
};

const moveItem = (indexFrom, indexTo, makeFocus = false) => {
  const isValidMove = props.slotsCount > indexTo && indexTo >= 0;
  if (isValidMove) {
    const inventoryField = Array.from(props.list);
    const elementFrom = inventoryField[indexFrom];
    const elementTo = inventoryField[indexTo];

    inventoryField[indexFrom] = elementTo;
    inventoryField[indexTo] = elementFrom;

    emit('updateInventoryField', inventoryField);

    if (makeFocus) {
      const focusedElement = document.activeElement;
      setTimeout(() => {
        focusedElement.blur();
        focusedElement.focus();
      }, 1);
    }
  }
};

const moveItemKeyboardHandler = () => {
  inventoryListDOM.value.addEventListener('keydown', (event) => {
    const hotkeysIncrementValue = {
      ArrowLeft: -1,
      ArrowRight: 1,
      ArrowUp: -numberItemsInRow.value,
      ArrowDown: numberItemsInRow.value,
    };

    const pressedKeyboardButton = event.key;
    const indexIncrement = hotkeysIncrementValue[pressedKeyboardButton];
    if (indexIncrement) {
      const { activeElement } = document;

      const inventoryItemDOM = activeElement.parentNode;
      const indexFrom = getItemIndex(inventoryItemDOM);
      const indexTo = indexFrom + indexIncrement;

      moveItem(indexFrom, indexTo, true);
    }
  });
};

const moveItemMouseHandler = () => {
  let indexFrom = null;
  inventoryListDOM.value.addEventListener('focusin', (event) => {
    const li = event.target.parentNode;
    indexFrom = getItemIndex(li);
  });
  inventoryListDOM.value.addEventListener('focusout', () => {
    indexFrom = null;
  });
  inventoryListDOM.value.addEventListener('mouseup', (event) => {
    const { target } = event;
    const li = target.tagName === 'LI' ? target : target.parentNode;
    const hasIndexFrom = indexFrom !== null;
    if (hasIndexFrom) {
      const indexTo = getItemIndex(li);
      moveItem(indexFrom, indexTo, true);
    }
  });
};

const moveItemTouchScreenHandler = () => {
  let indexFrom = null;
  inventoryListDOM.value.addEventListener('focusin', (event) => {
    const li = event.target.parentNode;
    indexFrom = getItemIndex(li);
  });
  inventoryListDOM.value.addEventListener('focusout', () => {
    indexFrom = null;
  });
  inventoryListDOM.value.addEventListener('mousedown', (event) => {
    const { target } = event;
    const li = target.tagName === 'LI' ? target : target.parentNode;
    const hasIndexFrom = indexFrom !== null;
    if (hasIndexFrom) {
      const indexTo = getItemIndex(li);
      moveItem(indexFrom, indexTo, false);
      setTimeout(() => {
        document.activeElement.blur();
      }, 1);
    }
  });
};

const focusHandler = () => {
  inventoryListDOM.value.addEventListener('focusin', (event) => {
    const focusIndex = getItemIndex(event.target.parentNode);
    const item = props.list[focusIndex];
    item.dom = event.target;
    emit('changeFocus', item);
  });
};

const grabHandler = () => {
  let copy = null;
  const setPosition = (event) => {
    copy.style.top = `${event.clientY}px`;
    copy.style.left = `${event.clientX}px`;
  };

  let li;
  inventoryListDOM.value.addEventListener('mousedown', (event) => {
    if (event.target.tagName === 'BUTTON') {
      li = event.target.parentNode;
      copy = li.cloneNode(true);
      copy.classList.add('field__item_grabed');
      copy.style.width = `${li.offsetWidth}px`;
      copy.style.height = `${li.offsetHeight}px`;
      copy.style.left = '200vw';

      inventoryListDOM.value.appendChild(copy);

      window.addEventListener('mousemove', setPosition);

      li.classList.add('field__item_opacity');
    }
  });
  inventoryListDOM.value.addEventListener('mouseup', () => {
    if (copy !== null) {
      copy.remove();
      copy = null;
      li.classList.remove('field__item_opacity');
      window.removeEventListener('mousemove', setPosition);
    }
  });
};

const setNumberItemsInRow = () => {
  const listWidth = inventoryListDOM.value.offsetWidth;
  const itemWidth = inventoryListDOM.value.querySelector('li').getBoundingClientRect().width;
  numberItemsInRow.value = Math.round(listWidth / itemWidth);

  inventoryListDOM.value.style.setProperty('--grid-item-width', `${itemWidth}px`);
};

onMounted(() => {
  setNumberItemsInRow();
  window.addEventListener('resize', setNumberItemsInRow);

  const deviceHasTouch = window.matchMedia('(pointer: coarse)').matches;
  if (deviceHasTouch) {
    moveItemTouchScreenHandler();
  } else {
    moveItemKeyboardHandler();
    moveItemMouseHandler();
    grabHandler();
  }

  focusHandler();
});

onUpdated(() => {
  const inventoryListWithEmptyCells = props.list.map((inventoryItem, index) => {
    const hasProperties = inventoryItem.properties;
    if (hasProperties) {
      const id = index;
      const { ...properies } = inventoryItem.properties;
      return { id, ...properies };
    }
    return null;
  });

  const inventoryList = inventoryListWithEmptyCells.filter((isItem) => (isItem));
  localStorage.setItem('inventory-items', JSON.stringify(inventoryList));
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
        class="field__button"
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
  $grid-gap: 1px;

  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(clamp(4rem, 11.5vw, 6rem), 1fr));
  gap: $grid-gap;

  background-repeat: space;
  background-image:
    linear-gradient(to right, colors.$border $grid-gap, transparent 0px),
    linear-gradient(to bottom, colors.$border $grid-gap, transparent 0px);

  --grid-item-width: 100%;
  background-size:
    var(--grid-item-width)
    var(--grid-item-width);

  &__item {
    position: relative;
    display: flex;
    aspect-ratio: 1;

    --padding: 20%;

    @media (pointer: fine) {
      &_grabed {
        z-index: 100;
        position: fixed;
        top: 0;
        left: 0;
        transform: translate(-50%, -50%);

        background-color: inherit;
        box-shadow: 0 0 0 $grid-gap colors.$border;
        border-radius: 0.75rem;
        pointer-events: none;
      }
      &_opacity {
        opacity: 0.35;
        transition: 400ms;
      }
    }
  }

  &__button {
    @mixin focused-button-styles {
      outline-width: 4px;
      outline-offset: -1px;
      outline-color: colors.$border;
      outline-style: solid;
      z-index: 100;
    }
    @media (pointer: coarse) {
      &:focus {
        @include focused-button-styles;
      }
    }
    @media (pointer: fine) {
      &:focus-visible {
        @include focused-button-styles;
      }
    }
  }
  &__item_opacity &__button {
    @media (pointer: fine) {
      &:focus-visible {
        outline-style: dashed;
      }
    }
  }

  &__count {
    color: #7d7d7d;

    width: 1.25rem;
    height: 1.25rem;
    position: absolute;
    bottom: 0;
    right: 0;

    border-top-left-radius: 0.75rem;
    border-top: solid $grid-gap colors.$border;
    border-left: solid $grid-gap colors.$border;

    display: flex;
    align-items: center;
    justify-content: center;
  }
}
</style>
