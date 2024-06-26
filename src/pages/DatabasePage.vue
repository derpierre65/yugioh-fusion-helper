<template>
  <q-page class="q-pa-lg">
    <q-card class="q-mb-md">
      <q-card-section>
        <form @submit.prevent="reset">
          <span class="text-h6">Filter</span>

          <q-input v-model.number="filter.id" type="number" label="Search by ID"/>

          <div class="row q-gutter-x-md">
            <div class="col-grow">
              <q-input v-model.number="filter.minCosts" type="number" label="Min Cost"/>
            </div>
            <div class="col-grow">
              <q-input v-model.number="filter.maxCosts" type="number" label="Max Cost"/>
            </div>
          </div>
          <div class="row q-gutter-x-md">
            <div class="col-grow">
              <q-input v-model.number="filter.minAtk" type="number" label="Min Atk"/>
            </div>
            <div class="col-grow">
              <q-input v-model.number="filter.minDef" type="number" label="Min Def"/>
            </div>
          </div>
          <div class="row q-gutter-x-md">
            <div class="col-grow">
              <q-select v-model="filter.sort" :options="sortValues" label="First Sort Level" map-options emit-value/>
            </div>
            <div class="col-grow">
              <q-select
                  v-model="filter.secondSort"
                  :options="sortValues"
                  label="Second Sort Level"
                  map-options
                  emit-value
              />
            </div>
          </div>

          <q-btn label="Filter" color="primary" type="submit"/>
        </form>
      </q-card-section>
    </q-card>

    <q-infinite-scroll ref="infiniteScroll" @load="loadMore">
      <div class="flex">
        <PlayCard v-for="id in cardList" :id="id" />
      </div>
    </q-infinite-scroll>
  </q-page>
</template>

<script setup lang="ts">
import useCardStore from 'stores/card';
import {computed, nextTick, ref, watch} from 'vue';
import PlayCard from 'components/PlayCard.vue';
import {QInfiniteScroll} from 'quasar';

//#region Composable & Prepare
const cardStore = useCardStore();
//#endregion

//#region Data
const filter = ref({
  id: 0,
  minCosts: 0,
  maxCosts: 0,
  minAtk: 0,
  minDef: 0,
  sort: 'cost',
  secondSort: 'atk',
});
const sortValues = Object.freeze([
  {label: 'Attack', value: 'atk'},
  {label: 'Defense', value: 'def'},
  {label: 'Cost', value: 'cost'},
]);
const cardList = ref<string[]>([]);
const cardIds = Object.keys(cardStore.cards);
const infiniteScroll = ref<QInfiniteScroll>();
//#endregion

//#region Computed
const filteredCardIds = computed(() => {
  const minCosts = filter.value.minCosts;
  const maxCosts = filter.value.maxCosts;
  const minAtk = filter.value.minAtk;
  const minDef = filter.value.minDef;
  const firstSort = filter.value.sort;
  const secondSort = filter.value.secondSort;
  const id = filter.value.id;
  let cardListIds = cardIds;

  if (minCosts || maxCosts || minAtk || minDef || id > 0) {
    cardListIds = cardListIds.filter((cardId) => {
      const card = cardStore.cards[cardId];
      if (id && cardId !== id.toString()) {
        return false;
      }
      if (minCosts && (!card.cost || card.cost < minCosts)) {
        return false;
      }
      if (maxCosts && (!card.cost || card.cost > maxCosts)) {
        return false;
      }
      if (minAtk && (!card.atk || card.atk < minAtk)) {
        return false;
      }

      return !(minDef && (!card.def || card.def < minDef));
    });
  }

  return cardListIds.sort((a, b) => {
    const aCard = cardStore.cards[a];
    const bCard = cardStore.cards[b];

    if (aCard[firstSort] === bCard[firstSort]) {
      return aCard[secondSort] < bCard[secondSort] ? 1 : -1;
    }

    return aCard[firstSort] < bCard[firstSort] ? 1 : -1;
  });
});

//#endregion

//#region Watch
watch(filter, reset, {
  deep: true,
});
//#endregion

//#region Lifecycle Events
//#endregion

//#region Methods

function reset() {
  nextTick(() => {
    cardList.value = [];
    const scroll = infiniteScroll.value!;
    scroll.setIndex(0);
    scroll.resume();
    scroll.trigger();
  });
}

function loadMore(index: number, done: (stop: boolean) => void) {
  const nextCards: string[] = filteredCardIds.value.slice((index - 1) * 50, index * 50);
  cardList.value.push(...nextCards);
  done(nextCards.length !== 50);
}

//#endregion

//#region Created
//#endregion
</script>