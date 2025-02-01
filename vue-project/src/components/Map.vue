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
  emit('selectStore', store.id);
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
.map {
  width: 996px;
  height: 684px;

  @media(max-width: 1000px) {
    width: 800px;
    height: 720px;
  }

  @media(max-width: 850px) {
    width: 660px;
    height: 720px;
  }

  @media(max-width: 700px) {
    width: 550px;
    height: 720px;
  }

  @media(max-width: 600px) {
    width: 460px;
    height: 720px;
  }

  @media(max-width: 500px) {
    width: 400px;
    height: 720px;
  }

  @media(max-width: 450px) {
    width: 351px;
    height: 528px;
  }

  @media(max-width: 450px) {
    width: 340px;
    height: 528px;
  }
}

.ymaps-2-1-79-balloon {
  width: 293px;
  border-radius: 8px;
  height: 173px;
}

.ymaps-2-1-79-balloon__layout {
  width: 293px;
  border-radius: 8px;
  height: 173px;

  .ymaps-2-1-79-balloon__content {
    padding: 16px;
    width: 293px;
    box-sizing: border-box;
    display: flex;
    flex-wrap: wrap;
    height: 173px;
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
    color: #0E0E0E;
    font-weight: 600;
    font-size: 14px;
    line-height: 16.94px;
    letter-spacing: 0.3px;
  }

  .schedule {
    color: #7D7D7D;
  }

  .balloon-button {
    width: 261px;
    height: 48px;
    background-color: black;
    color: white;
    border-radius: 8px;
    font-size: 16px;
    line-height: 20px;
    border: none;
  }

  .balloon-button:hover {
    cursor: pointer;
  }
}
</style>