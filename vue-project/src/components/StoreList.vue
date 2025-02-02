<template>
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
  stores: {
    type: Array,
    required: true
  },
  selectedStoreId: {
    type: Number,
    required: true
  }
});

const emit = defineEmits(['selectStore']);

function selectStore(store) {
  emit('selectStore', store);
}
</script>

<style lang="scss" scoped>
.test {
  border: 1px solid #E1E1E3;
  border-radius: 14px;
  height: 684px;
  box-sizing: border-box;
  padding: 8px;
  margin-right: 20px;

  .wrapper-list {
    box-sizing: border-box;
    overflow-y: auto;
    border-radius: 14px;
    // width: 490px;
    height: 100%;
    border-radius: 8px;

    .ul-list {
      padding: 0;
      margin: 0;
      list-style: none;
      border-radius: 8px;

      .li-list {
        display: flex;
        flex-direction: column;
        padding: 13px;
        border: 1px solid transparent;
        border-radius: 8px;
        margin-bottom: 4px;
        margin-right: 4px;
        border-radius: 8px;

        .address {
          font-weight: 400;
          font-size: 19px;
          line-height: 20px;
        }

        .schedule {
          margin-top: 8px;
          font-weight: 400;
          font-size: 16px;
          line-height: 20px;
          color: #999999;
        }
      }

      .li-list:hover {
        cursor: pointer;
        border: 1px solid black;
        transition: border 0.3s ease;
      }

      .selected {
        border: 1px solid black;
        border-radius: 8px;
      }
    }

    /* Стилизация полосы прокрутки */
    &::-webkit-scrollbar {
      width: 4px;
      /* Ширина полосы прокрутки */
    }

    &::-webkit-scrollbar-track {
      background: #f1f1f1;
      /* Цвет фона полосы прокрутки */
      border-radius: 10px;
    }

    &::-webkit-scrollbar-thumb {
      background: #888;
      /* Цвет ползунка */
      border-radius: 10px;
    }

    &::-webkit-scrollbar-thumb:hover {
      background: #555;
      /* Цвет ползунка при наведении */
    }

    @media (max-width: 1000px) {
      border: none;
      border-radius: 0px;
      height: 100%;
      box-sizing: border-box;
      padding: 8px;
      margin-right: 0;
      overflow-y: auto;
    }
  }

  @media (max-width: 1000px) {
    width: 100%;
    border: none;
    border-radius: 0px;
    box-sizing: border-box;
    padding: 8px;
    margin-right: 0;
  }
}
</style>