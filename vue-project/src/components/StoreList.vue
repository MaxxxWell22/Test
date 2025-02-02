<template>
  <!-- Блок с классом .test нужен для того, что бы сделать полосу прокрутки какв дизайне, иначе она всегда будет прилипать к нашему блоку со списком -->
  <div class="test">
    <div class="wrapper-list">
      <ul class="ul-list">
        <li class="li-list" v-for="store in stores" :key="store.id" @click="selectStore(store)"
          :class="{ selected: selectedStoreId === store.id }">
          <span class="address">{{ store.address }}</span>
          <span class="schedule">{{ store.schedule }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from "vue";

const props = defineProps({
  stores: { type: Array, required: true },
  selectedStoreId: { type: [Number, null], required: true },
});
const emit = defineEmits(['selectStore']);
// Emit для всплытия события родителю
function selectStore(store) {
  emit('selectStore', store);
}
</script>

<style lang="scss" scoped>
@import '../variables/variables.scss';

.test {
  border: 1px solid $color-light-gray;
  border-radius: $border-radius;
  height: 684px;
  box-sizing: border-box;
  padding: 8px;
  margin-right: 20px;

  .wrapper-list {
    box-sizing: border-box;
    overflow-y: auto;
    height: 100%;
    border-radius: $border-radius;

    .ul-list {
      padding: 0;
      margin: 0;
      list-style: none;

      .li-list {
        display: flex;
        flex-direction: column;
        padding: 13px;
        border: 1px solid transparent;
        border-radius: $border-radius;
        margin-bottom: 4px;
        margin-right: 4px;

        .address {
          font-weight: 400;
          font-size: $font-size-large;
          line-height: 20px;
        }

        .schedule {
          margin-top: 8px;
          font-weight: 400;
          font-size: $font-size-medium;
          line-height: 20px;
          color: $color-border;
        }
      }

      .li-list:hover {
        cursor: pointer;
        border: 1px solid black;
        transition: border 0.3s ease;
      }

      .selected {
        border: 1px solid black;
      }
    }

    &::-webkit-scrollbar {
      width: 4px;
    }

    &::-webkit-scrollbar-track {
      background: #f1f1f1;
      border-radius: 10px;
    }

    &::-webkit-scrollbar-thumb {
      background: #888;
      border-radius: 10px;
    }

    &::-webkit-scrollbar-thumb:hover {
      background: #555;
    }

    @media (max-width: 1000px) {
      border: none;
      border-radius: 0;
      height: 100%;
      padding: 8px;
      margin-right: 0;
      overflow-y: auto;
    }
  }

  @media (max-width: 1000px) {
    width: 100%;
    height: 760px;
    border: none;
    border-radius: 0;
    padding: 8px;
    margin-right: 0;
  }

  @media (max-width: 450px) {
    height: 548px;
  }
}
</style>
