<template>
  <q-page class="q-pa-lg">
    <!--region add card container-->
    <div class="row q-col-gutter-md">
      <div class="col-grow">
        <q-input
            v-model="newCards"
            label="New Cards"
            hint="Add cards to your deck. Split multiple Card ids with , or space. example: 604,102,123,101,586"
            @keyup.enter="addCards"
        />
      </div>
      <div class="flex items-center">
        <q-btn label="Add Cards" color="primary" @click="addCards"/>
      </div>
    </div>
    <!--endregion-->

    <q-banner class="bg-yellow-9 q-my-md text-black">Fusions are overall fusions not only available fusions within your deck.</q-banner>

    <div class="text-h6">Your deck {{ savegameStore.deckCards.length }}/40:</div>
    <div class="flex">
      <PlayCard v-for="card in cardList" :id="card.id">
        <template #before-name>
          <span>Fusions: {{card.possibleFusions}}</span><br>
          <span>Fusions done: {{savegameStore.deckFusions[card.id] || 0}}</span><br>
        </template>
        <template #action-buttons>
          <q-btn icon="fas fa-times" color="negative" size="sm" dense @click="removeCard(card.id)">
            <q-tooltip>Remove</q-tooltip>
          </q-btn>
        </template>
      </PlayCard>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import {computed, ref} from 'vue';
import {fusionList, getIdsByString} from 'src/lib/fusions';
import PlayCard from 'components/PlayCard.vue';
import useSavegameStore from 'stores/savegame';

//#region Composable & Prepare
const savegameStore = useSavegameStore();
//#endregion

//#region Data
const newCards = ref('');
//#endregion

//#region Computed
const allFusions = computed(() => {
  const keys = Object.keys(fusionList);

  return keys.reduce((prev, value) => prev.concat(...fusionList[value]), []);
});

const cardList = computed(() => {
  const cardFusions = Object.create(null);
  for ( const id of savegameStore.deckCards ) {
    if (!cardFusions[id] ) {
      cardFusions[id] = allFusions.value.filter((fusionId: string) => fusionId.startsWith(`${id}+`) || fusionId.endsWith(`+${id}`)).length;
    }
  }

  return savegameStore.deckCards.map((id) => ({
    id,
    possibleFusions: cardFusions[id] || 0,
  }));
})
//#endregion

//#region Watch
//#endregion

//#region Lifecycle Events
//#endregion

//#region Methods
function addCards() {
  if (savegameStore.deckCards.length === 40) {
    return false;
  }

  const ids = getIdsByString(newCards.value);
  if (savegameStore.deckCards.length + ids.length > 40) {
    return false;
  }

  savegameStore.deckCards.push(...ids);
  savegameStore.deckCards = savegameStore.deckCards.sort((a, b) => {
    return parseInt(a) > parseInt(b) ? 1 : -1;
  });
  newCards.value = '';
}

function removeCard(id: string) {
  savegameStore.deckCards.splice(savegameStore.deckCards.indexOf(id), 1);
}

//#endregion

//#region Created
//#endregion
</script>