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

const moveItem = (indexFrom, indexTo) => {
  const isValidMove = props.slotsCount > indexTo && indexTo >= 0;
  if (isValidMove) {
    const inventoryField = Array.from(props.list);
    const elementFrom = inventoryField[indexFrom];
    const elementTo = inventoryField[indexTo];

    inventoryField[indexFrom] = elementTo;
    inventoryField[indexTo] = elementFrom;

    emit('updateInventoryField', inventoryField);

    const focusedElement = document.activeElement;
    setTimeout(() => {
      focusedElement.blur();
      focusedElement.focus();
    }, 1);
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

      moveItem(indexFrom, indexTo);
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
      moveItem(indexFrom, indexTo);
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

  moveItemKeyboardHandler();
  moveItemMouseHandler();
  focusHandler();
  grabHandler();
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
  gap: --grid-gap;

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

    &_grabed {
      background-color: inherit;
      position: fixed;
      top: 0;
      left: 0;
      transform: translate(-50%, -50%);
      pointer-events: none;
      z-index: 100;
      box-shadow: 0 0 0 $grid-gap colors.$border;
    }
    &_opacity {
      opacity: 0.35;
    }
  }

  &__button:focus-visible {
    outline: 4px solid colors.$border;
    z-index: 100;
  }
  &__item_opacity &__button:focus-visible {
    outline: 4px dashed colors.$border;
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
