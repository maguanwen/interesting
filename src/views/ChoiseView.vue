<script setup lang="ts">
import Turntable from '../components/Turntable.vue'
import { ref, reactive, nextTick } from 'vue'
import func from '../../vue-temp/vue-editor-bridge';
// import { nextTick } from 'process';

let info = reactive({
  data: [],
  id: 5,
  input: '',
  result: ''
});

let test = reactive([1, 2, 3]);
let b = test;
test = [1];

info.data = [{ //可选项
  id: 1,
  name: "麻辣烫"
}, {
  id: 2,
  name: "炒饭"
}, {
  id: 3,
  name: "米线"
}, {
  id: 4,
  name: "黄焖鸡"
}
];

function addData() {
  let had = info.data.find(item => item.name == info.input);
  if (!had && info.input) {
    info.data.push({
      id: info.id,
      name: info.input
    });
    info.id++;
    info.input = '';
  }
}

function deleteItem(item) {
  if(info.data.length == 1) return;
  info.data = info.data.filter(i => i.id != item.id);
}
</script>

<template>
  <main>
    <h1 class="title">中午吃啥</h1>
    <div class="trun-table">
      <Turntable :data="info.data" />
    </div>

    <button @click="addData">增加选项</button>
    <input v-model="info.input" />
    <div>
      <div class="" v-for="item in info.data" :key="item.id">
        {{ item.name }}{{ item.id }}
        <span class="delete-item" @click="deleteItem(item)">x</span>
      </div>
    </div>
  </main>
</template>

<style>
  @media (min-width: 1024px) {
    .about {
      min-height: 100vh;
      display: flex;
      align-items: center;
    }
  }
  .title {
    text-align: center;
    margin-bottom: 24px;
  }
  .trun-table {
    margin-bottom: 24px;
  }
  .delete-item {
    margin-left: 8px;
    cursor: pointer;
  }
</style>
