<template>
  <div class="parent">
    <div class="mobile-tabs">
      <span @click="setActiveTab('map')" :class="{ 'active-tab': activeTab === 'map' }">Карта</span>
      <span @click="setActiveTab('list')" :class="{ 'active-tab': activeTab === 'list' }">Список</span>
    </div>

    <template v-if="stores.length > 0">
      <StoreList v-if="isWideScreen || activeTab === 'list'" class="store-list" :stores="stores"
        :selected-store-id="selectedStoreId !== undefined ? selectedStoreId : null" @selectStore="handleStoreSelect"
        :selected-store="selectedStore" />
      <Map v-if="activeTab === 'map'" :stores="stores" :selected-store-id="selectedStoreId"
        :selected-store="selectedStore" :center="CENTER" :zoom="ZOOM" @selectStore="handleStoreSelect" />
    </template>
    <div v-else>Загрузка данных...</div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from "vue";
import StoreList from "./components/StoreList.vue";
import Map from "./components/Map.vue";

const stores = ref([]);
const selectedStoreId = ref(null); 
const activeTab = ref('map');
const selectedStore = computed(() => {
  return stores.value.find(store => store.id === selectedStoreId.value);
});
const CENTER = { lat: 45.037914, lng: 39.016233 };
const ZOOM = 12;

// Создаем реактивное свойство для ширины экрана
const isWideScreen = computed(() => window.innerWidth > 1000);

onMounted(async () => {
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

  // Добавляем обработчик события resize для обновления ширины экрана
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
});

function handleResize() {
  isWideScreen.value = window.innerWidth > 1000;
}

function setActiveTab(tab) {
  activeTab.value = tab;
}

function handleStoreSelect(store) {
  if (store && store.id !== undefined) {
    selectedStoreId.value = store.id; // Устанавливаем id выбранного магазина
  }
  setActiveTab('map'); // Переключаемся на карту
}
</script>

<style lang="scss">
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
      font-size: 14px;
      line-height: 20px;
      color: #E1E1E3;
      width: 100%;
      text-align: center;
      border-bottom: 2px solid #E1E1E3;
      /* Цвет неактивного таба */
      cursor: pointer;

      &.active-tab {
        color: #0E0E0E;
        border-bottom: 2px solid black;
        /* Цвет активного таба */
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