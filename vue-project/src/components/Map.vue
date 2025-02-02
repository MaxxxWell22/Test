<template>
  <!-- Единственный блок в который будет помещена карта -->
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
let map;
const markers = {};
// Функция инициализации карты
function initMap() {
  map = new ymaps.Map(mapRef.value, {
    center: [props.center.lat, props.center.lng],
    zoom: props.zoom || 12,
  });
  // Проходим циклом по всем магазинам и добавляем маркеры
  props.stores.forEach((store) => {
    const marker = createMarker(store);
    markers[store.id] = marker;
    marker.events.add('click', () => handleMarkerClick(store));
    map.geoObjects.add(marker);
  });
}
// Функция создания маркера
function createMarker(store) {
  return new ymaps.Placemark(
    [parseFloat(store.lat), parseFloat(store.lng)],
    {
      // Баллон который открывается с информацией
      balloonContent: `
        <div class="balloon-content custom-balloon">
          <span class="address">${store.address}</span>
          <span class="schedule">График работы: ${store.schedule}</span>
          <button class="balloon-button">Подробнее</button>
        </div>
      `,
    },
    {
      hideIconOnBalloonOpen: false, // Кнопка о закрытие баллона
      iconLayout: "default#image", // Дефолтная картинка
      iconImageHref: pin, // Картинка меток
      iconImageSize: [30, 42], // Размер картинки для метки
      iconImageOffset: [-15, -42], // Расположение метки
      balloonOffset: [-85, 195], // Расположение баллона
    }
  );
}
// Функция для поднятия вверх события о клике на метку
function handleMarkerClick(store) {
  map.setCenter([parseFloat(store.lat), parseFloat(store.lng)], 17);
  emit('selectStore', store);
}

onMounted(() => {
  // Если при маунте компонента у нас ymaps == undefined, то вызываем ее
  if (typeof ymaps !== 'undefined') {
    ymaps.ready(initMap);
  } else {
    console.error('Yandex Maps API is not loaded');
  }
});
// Наблюдатель для изменения выбранного магазина. При изменении selectedStoreId обновляет центр карты и открывает баллон с информацией о магазине.
watch(() => props.selectedStoreId, async (newId) => {
  await nextTick();
  if (newId && map) {
    const store = props.stores.find(s => s.id === newId);
    if (store) {
      map.setCenter([parseFloat(store.lat), parseFloat(store.lng)], 17);
      const marker = markers[newId];
      if (marker) {
        marker.balloon.open();
      }
    }
  }
});

onUnmounted(() => {
  if (map) {
    map.destroy();
  }
});
</script>

<style lang="scss">
@import '../variables/variables.scss';

.map {
  width: $map-width;
  height: $map-height;

  @media (max-width: 1000px) {
    width: 100%
  }

  @media (max-width: 450px) {
    height: 528px !important;
  }
}

.ymaps-2-1-79-balloon {
  width: $balloon-width;
  border-radius: $border-radius;
  height: $balloon-height;

  .ymaps-2-1-79-balloon__layout {
    width: $balloon-width;
    border-radius: $border-radius;
    height: $balloon-height;

    .ymaps-2-1-79-balloon__content {
      padding: 16px;
      width: $balloon-width;
      box-sizing: border-box;
      display: flex;
      flex-wrap: wrap;
      height: $balloon-height;
    }
  }

  .ymaps-2-1-79-balloon__tail {
    left: 45% !important;
    bottom: 166px !important;
  }

  .custom-balloon {
    display: flex;
    flex-direction: column;
    height: 100%;
    justify-content: space-around;
    height: 153px;

    .address {
      color: $color-black;
      font-weight: 600;
      font-size: $font-size-small;
      line-height: 16.94px;
      letter-spacing: 0.3px;
    }

    .schedule {
      color: $color-dark-gray;
    }

    .balloon-button {
      width: 261px;
      height: 48px;
      background-color: $color-button-bg;
      color: white;
      border-radius: $border-radius;
      font-size: $font-size-medium;
      line-height: 20px;
      border: none;

      &:hover {
        cursor: pointer;
      }
    }
  }
}
</style>
