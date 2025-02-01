<template>
  <div class="parent">
    <div class="mobile-tabs">
      <span @click="setActiveTab('map')" :class="{ active: activeTab === 'map' }">Карта</span>
      <span @click="setActiveTab('list')" :class="{ active: activeTab === 'list' }">Список</span>
    </div>

    <!-- Условие для отображения StoreList -->
    <StoreList v-if="isWideScreen || activeTab === 'list'" class="store-list" :stores="stores" :selected-store-id="selectedStoreId"
      @selectStore="handleStoreSelect" :selectedStore="selectedStore" />

    <Map v-if="activeTab === 'map' && stores.length > 0" :stores="stores" :selected-store-id="selectedStoreId"
      :selectedStore="selectedStore" :center="center" :zoom="zoom" @selectStore="handleStoreSelect" />

    <div v-if="stores.length === 0">Загрузка данных...</div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import StoreList from "./components/StoreList.vue";
import Map from "./components/Map.vue";

const stores = ref([]);
const selectedStoreId = ref(null);
const center = ref({ lat: 45.037914, lng: 39.016233 });
const zoom = ref(12);
const activeTab = ref('map');
const selectedStore = ref(null);

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
  window.addEventListener('resize', () => {
    isWideScreen.value = window.innerWidth > 1000;
  });
});

function setActiveTab(tab) {
  activeTab.value = tab;
}

// Метод для обработки выбора магазина
function handleStoreSelect(store) {
  selectedStore.value = store; // Устанавливаем выбранный магазин
  selectedStoreId.value = store.id; // Устанавливаем id выбранного магазина
  activeTab.value = 'map'; // Переключаемся на карту
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

      &.active {
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
    padding: 20px;
  }

  @media (max-width: 1000px) {
    flex-direction: column;
  }
}
</style>