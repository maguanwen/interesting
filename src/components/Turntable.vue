<script setup lang="ts">

import { onMounted, computed, ref } from 'vue'

const CIRCLE_ANGLE = 360;
const BIGSIZE = 24;

// 数据
const props = defineProps({
  data: {
    type: Array,
    default: () => [],
    readonly: true
  }
});

const emit = defineEmits(['result', 'delete']);

const bgDom = ref(null);
const divDom = ref(null);

// 数据列表
const prizeList = computed(() => formatPrizeList(props.data));


let angleList = []; // 记录每个奖的位置
let gift_id = 3; //中奖ID
let index = '';//抽中的是第几个奖品
let isRotating = false; //为了防止重复点击
let rotateAngle = 0;

let result = ref('');

//每个选择增加style
function formatPrizeList(list) {
  const l = list.length;
  // 计算单个选择所占的角度
  const average = CIRCLE_ANGLE / l; //60
  const half = average / 2; //30			  
  const rightBig: any = l == 2 ? '50' : '0';
  const heightBig = l <= 3 ? '100' : '50';
  const topBig = l == 3 ? '-50' : '0';
  const skewMain = l <= 2 ? 0 : -(l - 4) * 90 / l;
  // 循环计算给每个奖项添加style属性
  list.forEach((item, i) => {
    // 每个奖项旋转的位置为 当前 i * 平均值 + 平均值 / 2
    const angle = -(i * average + half);
    const bigAge = l > 2 ? i * 360 / l : '0';
    // 增加 style 这个是给每一个奖项增加的样式
    item.style = {
      '-webkit-transform': `rotate(${-angle}deg)`,
      'transform': `rotate(${-angle}deg)`,
      'width': `${100 / l * 2}%`,
      'margin-left': `-${100 / l}%`,
      'font-size': `${BIGSIZE - l}px`,
    };
    //这是给每一个转盘背景新增的样式
    item.style2 = {
      '-webkit-transform': `rotate(${bigAge}deg)`,
      transform: `rotate(${bigAge}deg) skewY(${skewMain}deg)`,
      right: `${rightBig * i}%`,
      height: `${heightBig}%`,
      top: `${topBig}%`,
      width: `${l == 1 ? 100 : 50}%`,
      background: `${item.color}`
    }
    // 记录每个奖项的角度范围
    angleList.push(angle);
  });
  return list;
};
//抽奖
function prizeRoll() {
  if (isRotating) return false;
  gift_id = Math.floor(1 + Math.random() * prizeList.value.length);
  result = prizeList.value[gift_id - 1].name;
  console.log(result);
  prizeList.value.forEach((item, i) => {
    if (item.id == gift_id) index = i; //判断中奖的位置
  });
  rotating();
};
//转盘转动角度
function rotating() {
  isRotating = true;
  const config = {
    duration: 5000,
    circle: 8,
    mode: "ease-in-out"
  }
  // 计算角度
  const angle =
    // 初始角度
    rotateAngle +
    // 多旋转的圈数
    config.circle * CIRCLE_ANGLE +
    // 奖项的角度
    angleList[index] -
    (rotateAngle % CIRCLE_ANGLE);
  rotateAngle = angle;
  bgDom.value.style.transform = `rotate(${rotateAngle}deg)`
  divDom.value.style.transform = `rotate(${rotateAngle}deg)`
  // 旋转结束后，允许再次触发
  setTimeout(() => {
    isRotating = false;
    console.log('旋转结束')
    emit('result', { result });
  }, config.duration + 500);
}
onMounted(() => { })

function deleteItem(item) {
  emit('delete', item)
}
</script>

<template>
  <div class="turntable">
    <div class="luckBg">
      <div class="luckWhellBg">
        <div ref="bgDom" class="luckWhellBgMain rotateStyle">
          <div
            class="luckWhellSector"
            :style="item.style2"
            v-for="item in prizeList"
            :key="item.id"
          ></div>
        </div>
        <div class="wheel-main">
          <div ref="divDom" class="prize-list rotateStyle">
            <div
              class="prize-item"
              :style="item.style"
              v-for="item in prizeList"
              :key="item.id"
            >
              <div class="prize-name" @click="deleteItem(item)">
                {{ item.name }}
              </div>
            </div>
          </div>
          <div class="prize_point" @click="prizeRoll()"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
  .luckBg {
    background: #ffcf49;
    width: 20rem;
    height: 20rem;
    border-radius: 50%;
    margin: 0 auto;
    padding: 5px;
  }
  .luckWhellBg,
  .luckWhellBgMain {
    background: #fff;
    width: 100%;
    height: 100%;
    border-radius: 50%;
    position: relative;
    overflow: hidden;
  }
  .rotateStyle {
    -webkit-transition: transform 5000ms ease-in-out;
    transition: transform 5000ms ease-in-out;
  }
  .luckWhellSector {
    position: absolute;
    top: 0;
    right: 0;
    transform-origin: 0% 100%;
    border: 1px solid #facd89;
    border-right: 0;
    border-top: 0;
    box-sizing: border-box;
  }

  .luckWhellSector:nth-child(2n) {
    position: absolute;
    background: #fff3d8;
  }
  .wheel-main {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    width: 100%;
    height: 100%;
    z-index: 2;
  }
  .prize-list {
    width: 100%;
    height: 100%;
    position: relative;
  }
  .prize-item {
    position: absolute;
    left: 50%;
    height: 50%;
    padding-top: 15px;
    box-sizing: border-box;
    text-align: center;
    transform-origin: 50% 100%;
    color: #f83c0e;
  }
  .prize-name {
    cursor: pointer;
  }
  .prize_point {
    position: absolute;
    left: 50%;
    background: #ff3a60;
    width: 4rem;
    height: 4rem;
    top: 50%;
    margin-left: -2rem;
    margin-top: -2rem;
    border-radius: 50%;
  }
  .prize_point::after {
    position: absolute;
    left: 50%;
    top: -1.5rem;
    width: 0;
    height: 0;
    margin-left: -1rem;
    border: 1rem solid;
    border-color: transparent transparent #ff3a60;
    content: '';
  }
</style>
