<template>
  <!-- Общий родитель -->
  <div class="parent">
    <!-- Табы списка и карты в мобилке -->
    <div class="mobile-tabs">
      <span @click="setActiveTab('map')" :class="{ 'active-tab': activeTab === 'map' }">Карта</span>
      <span @click="setActiveTab('list')" :class="{ 'active-tab': activeTab === 'list' }">Список</span>
    </div>
    <!-- Список магазинов и карта в десктоп -->
    <template v-if="stores.length > 0">
      <StoreList v-show="isWideScreen || activeTab === 'list'" class="store-list" :stores="stores"
        :selected-store-id="selectedStoreId !== undefined ? selectedStoreId : null" @selectStore="handleStoreSelect"
        :selected-store="selectedStore" />
      <Map v-show="activeTab === 'map'" :stores="stores" :selected-store-id="selectedStoreId"
        :selected-store="selectedStore" :center="CENTER" :zoom="ZOOM" @selectStore="handleStoreSelect" />
    </template>
    <!-- Когда данные грузяться -->
    <div v-else>Загрузка данных...</div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import StoreList from "./components/StoreList.vue";
import Map from "./components/Map.vue";

// Изначальный пусстой массив для данных
const stores = ref([]);
// id ыбранного магазина
const selectedStoreId = ref(null);
// определение активного таба
const activeTab = ref('map');
// вычисляемое свойство для динамического вычисления id магазина
const selectedStore = computed(() => {
  return stores.value.find(store => store.id === selectedStoreId.value);
});
// центр карты
const CENTER = { lat: 45.037914, lng: 39.016233 };
// стандартный зум карты
const ZOOM = 12;
// вычисляемое свойство что бы понимать что экран уменьшился/увеличился
const isWideScreen = computed(() => window.innerWidth > 1000);

onMounted(async () => {
  // запрос данных
  try {
    const response = await fetch("https://cmstore.ru/rest-api/catalog/detail/41392/store-list?locationID=1168");
    if (!response.ok) {
      throw new Error('Network response was not ok');
    }
    const data = await response.json();
    stores.value = data;
  } catch (error) {
    console.error("Ошибка при загрузке данных:", error);
  }
});
// Функция для смены активных табов
function setActiveTab(tab) {
  activeTab.value = tab;
}
// Функция обработчик emits из дочерних компонентов
function handleStoreSelect(store) {
  if (store && store.id !== undefined) {
    selectedStoreId.value = store.id;
  }
  setActiveTab('map');
}
</script>

<style lang="scss">
@import './variables/variables.scss';

.parent {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;

  .mobile-tabs {
    display: none;
    justify-content: space-around;
    width: 100%;

    span {
      font-weight: 600;
      font-size: $font-size-small;
      line-height: 20px;
      color: $color-light-gray;
      width: 100%;
      text-align: center;
      border-bottom: 2px solid $color-light-gray;
      cursor: pointer;

      &.active-tab {
        color: $color-black;
        border-bottom: 2px solid black;
      }
    }

    @media(max-width: 1000px) {
      display: flex;
    }
  }

  .map {
    padding-top: 20px;
  }

  @media (max-width: 1000px) {
    flex-direction: column;
  }
}
</style>