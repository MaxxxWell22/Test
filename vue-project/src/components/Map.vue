<template>
  <div ref="mapRef" class="map"></div>
</template>

<script setup>
import { defineProps, defineEmits, ref, onMounted, onUnmounted, watch, nextTick } from "vue";
import pin from "../assets/pin.svg";

const props = defineProps({
  stores: Array,
  center: Object,
  zoom: Number,
  selectedStoreId: Number,
  selectedStore: Object,
});

const emit = defineEmits(['selectStore']);
const mapRef = ref(null);
let map; // Объявляем переменную для карты
const markers = {};

function initMap() {
  map = new ymaps.Map(mapRef.value, {
    center: [props.center.lat, props.center.lng],
    zoom: props.zoom || 12,
  });

  // Добавление маркеров на карту
  props.stores.forEach((store) => {
    const marker = createMarker(store);
    markers[store.id] = marker;
    marker.events.add('click', () => handleMarkerClick(store));
    map.geoObjects.add(marker);
  });
}

function createMarker(store) {
  return new ymaps.Placemark(
    [parseFloat(store.lat), parseFloat(store.lng)],
    {
      balloonContent: `
        <div class="balloon-content custom-balloon">
          <span class="address">${store.address}</span>
          <span class="schedule">График работы: ${store.schedule}</span>
          <button class="balloon-button">Подробнее</button>
        </div>
      `,
    },
    {
      hideIconOnBalloonOpen: false,
      iconLayout: "default#image",
      iconImageHref: pin,
      iconImageSize: [30, 42],
      iconImageOffset: [-15, -42],
      balloonOffset: [-85, 195],
    }
  );
}

function handleMarkerClick(store) {
  map.setCenter([parseFloat(store.lat), parseFloat(store.lng)], 17);
  emit('selectStore', store);
}

onMounted(() => {
  if (typeof ymaps !== 'undefined') {
    ymaps.ready(initMap);
  } else {
    console.error('Yandex Maps API is not loaded');
  }
});

onUnmounted(() => {
  if (map) {
    map.destroy();
  }
});

// Watch for changes in selectedStoreId prop
watch(() => props.selectedStoreId, async (newId) => {
  await nextTick(); // Ждем обновления DOM
  if (newId && map) { // Проверяем, что карта инициализирована
    const store = props.stores.find(s => s.id === newId);
    if (store) {
      map.setCenter([parseFloat(store.lat), parseFloat(store.lng)], 17); // Устанавливаем центр карты
      const marker = markers[newId];
      if (marker) {
        marker.balloon.open(); // Открываем баллон
      }
    }
  }
});
</script>

<style lang="scss">
@import '../variables/variables.scss';

.map {
  width: $map-width; // Используем переменную для ширины карты
  height: $map-height; // Используем переменную для высоты карты

  @media (max-width: 1000px) {
    width: 100%; // Адаптивная ширина для мобильных устройств
  }

  @media (max-width: 450px) {
    height: 528px !important;
  }
}

.ymaps-2-1-79-balloon {
  width: $balloon-width; // Используем переменную для ширины балуна
  border-radius: $border-radius; // Используем переменную для радиуса границы
  height: $balloon-height; // Используем переменную для высоты балуна

  .ymaps-2-1-79-balloon__layout {
    width: $balloon-width; // Используем переменную для ширины
    border-radius: $border-radius; // Используем переменную для радиуса границы
    height: $balloon-height; // Используем переменную для высоты

    .ymaps-2-1-79-balloon__content {
      padding: 16px;
      width: $balloon-width; // Используем переменную для ширины
      box-sizing: border-box;
      display: flex;
      flex-wrap: wrap;
      height: $balloon-height; // Используем переменную для высоты
    }
  }

  .ymaps-2-1-79-balloon__tail {
    left: 45% !important; // Позиционирование хвоста балуна
    bottom: 166px !important; // Позиционирование хвоста балуна
  }

  .custom-balloon {
    display: flex;
    flex-direction: column;
    height: 100%;
    justify-content: space-around;
    height: 153px;

    .address {
      color: $color-black; // Используем переменную для цвета текста
      font-weight: 600;
      font-size: $font-size-small; // Используем переменную для размера шрифта
      line-height: 16.94px;
      letter-spacing: 0.3px;
    }

    .schedule {
      color: $color-dark-gray; // Используем переменную для цвета текста
    }

    .balloon-button {
      width: 261px; // Можно также вынести в переменные, если используется в нескольких местах
      height: 48px;
      background-color: $color-button-bg; // Используем переменную для цвета фона кнопки
      color: white;
      border-radius: $border-radius; // Используем переменную для радиуса границы
      font-size: $font-size-medium; // Используем переменную для размера шрифта
      line-height: 20px;
      border: none;

      &:hover {
        cursor: pointer; // Курсор при наведении
      }
    }
  }
}
</style>
