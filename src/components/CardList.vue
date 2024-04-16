<template>
  <section
    :style="`background: ${options.color}`"
    @drop="onDrop($event, options.id)"
    @dragover.prevent
    @dragenter.prevent
  >
    <v-tooltip :text="contentButtonSort.tooltip">
      <template v-slot:activator="{ props }">
        <v-btn
            v-bind="props"
            :icon="contentButtonSort.icon"
            density="compact"
            variant="tonal"
            class="sort-btn"
            color="white"
            @click="cardSort"
        />
      </template>
    </v-tooltip>
    <div class="title">
      <h2>
        {{ options.title }}
      </h2>
      <div class="counter">
        <span>{{ cardBySort.length }}</span>
      </div>
    </div>
    <v-btn
      icon="mdi-plus"
      variant="tonal"
      class="mt-5"
      color="white"
      @click="isNewCardDialogOpen = true"
    />

    <CardItem
      v-for="(card, index) in cardBySort"
      draggable="true"
      :key="index"
      :card="card"
      :options="props.options"
      @delete-card="deleteCard(card.id)"
      @dragstart="onDragStart($event, card)"
    />

    <CardForm
      title="Добавление новой карточки"
      v-model="isNewCardDialogOpen"
      :form="form"
      @save-card="addCard"
      @close-form="isNewCardDialogOpen = false"
    />
  </section>
</template>

<script setup>
  import {ref, inject, computed} from 'vue';
  import CardItem from './CardItem.vue';
  import CardForm from './CardForm.vue';

  const firstList = inject('firstList');
  const secondList = inject('secondList');
  const lastList = inject('lastList');

  const props = defineProps({
    options: {},
  });

  const isNewCardDialogOpen = ref(false);

  const form = ref({
    image: '',
    id: null,
    title: '',
    description: '',
    category: '',
    price: null,
    rating: {},
  });

  let cards = ref([]);

  function getLocalCards() {
    switch (props.options.id) {
      case 1:
        cards = firstList;
        break;
      case 2:
        cards = secondList;
        break;
      case 3:
        cards = lastList;
        break;
    }
  }

  function addCard() {
    cards.value.unshift(form.value);
    isNewCardDialogOpen.value = false;
  }
  function deleteCard(id) {
    cards.value = cards.value.filter((card) => card.id != id);
  }
  function onDragStart(event, item) {
    event.dataTransfer.dropEffect = 'move';
    event.dataTransfer.effectAllowed = 'move';
    event.dataTransfer.setData('itemId', item.id.toString());
  }
  function onDrop(event, optionsId) {
    const itemId = parseInt(event.dataTransfer.getData('itemId'));
    let otherLists = [];

    switch (optionsId) {
      case 1:
        cards.value = firstList.value;
        otherLists = secondList.value.concat(lastList.value);
        break;
      case 2:
        cards.value = secondList.value;
        otherLists = firstList.value.concat(lastList.value);
        break;
      case 3:
        cards.value = lastList.value;
        otherLists = secondList.value.concat(firstList.value);
        break;
    }

    const newCard = otherLists.find((card) => card.id === itemId);

    firstList.value = firstList.value.filter((card) => card.id !== newCard.id);
    secondList.value = secondList.value.filter((card) => card.id !== newCard.id);
    lastList.value = lastList.value.filter((card) => card.id !== newCard.id);
    cards.value.unshift(newCard);

    switch (optionsId) {
      case 1:
        firstList.value = cards.value;
        break;
      case 2:
        secondList.value = cards.value;
        break;
      case 3:
        lastList.value = cards.value;
        break;
    }
  }

  const sortBy = ref('default') // 'ascending' | 'descending' | 'default'

  const cardSort = () => {
    switch (sortBy.value) {
      case 'default':
        sortBy.value = 'ascending';
        break;
      case 'ascending':
        sortBy.value = 'descending';
        break
      default:
        sortBy.value = 'default';
    }
  }

  const contentButtonSort = computed(() => {
    switch (sortBy.value) {
      case 'descending':
        return { icon: 'mdi-sort-ascending', tooltip: 'Сортировка по низкому рейтингу' };
      case 'ascending':
        return { icon: 'mdi-sort-descending', tooltip: 'Сортировка по высокому рейтингу' };
      default:
        return { icon: 'mdi-sort', tooltip: 'Сортировка по умолчанию' };
    }
  })

  const cardBySort = computed(() => {
    getLocalCards();
    if (sortBy.value === 'ascending') {
      return cards.value.sort((a, b) => b.rating.rate - a.rating.rate);
    }

    if (sortBy.value === 'descending') {
      return cards.value.sort((a, b) => {
        if (a.rating.rate > b.rating.rate) {
          return 1;
        }
        if (a.rating.rate < b.rating.rate) {
          return -1;
        }
        return 0;
      });
    }
    return cards.value.sort((a, b) => a.id - b.id);
  })
</script>

<style lang="scss" scoped>
  section {
    padding: 10px;
    width: 400px;
    min-height: 500px;
    height: fit-content;
    border-radius: 10px;
    display: flex;
    flex-direction: column;
    align-items: center;
    .title {
      padding-bottom: 10px;
      width: 90%;
      display: flex;
      align-items: center;
      justify-content: center;
      border-bottom: 2px solid white;
      .counter {
        margin-left: 10px;
        background: white;
        border-radius: 50%;
        width: 30px;
        height: 30px;
        display: flex;
        justify-content: center;
        align-items: center;
      }
    }
  }
</style>
