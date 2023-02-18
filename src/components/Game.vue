<script setup lang="ts">
import { onMounted, ref } from 'vue'

const props = defineProps<{ width: number, height: number }>()
const gamediv = ref()
let players = ['#000', '#fff']
let current = 0

function getArrayIndex(arr1: any[] | HTMLCollection, obj: any) {
  let arr: any[] = [];
  for (let i of arr1) {
    arr.push(i)
  }
  let i = arr.length;
  while (i--) {
    if (arr[i] === obj) {
      return i;
    }
  }
  return -1;
}

function getAround(block: HTMLDivElement): HTMLDivElement[] {
  return [
    block.previousElementSibling,
    block.nextElementSibling,
    gamediv.value.children[
      getArrayIndex(
        gamediv.value.children,
        block.parentElement
      ) - 1]?.children[
    getArrayIndex(
      block.parentElement?.children || [],
      block
    )
    ] || null,
    gamediv.value.children[
      getArrayIndex(
        gamediv.value.children,
        block.parentElement
      ) + 1]?.children[
    getArrayIndex(
      block.parentElement?.children || [],
      block
    )
    ] || null,
  ]
}

function canPlace(block: HTMLDivElement) {
  const around = getAround(block)
  console.log(around)
  let temp1 = [];
  for (let i of around) {
    if (i === null) {
      continue
    }
    if (i.dataset.color !== players[current] && i.dataset.color !== '0') {
      temp1.push(i)
    }
  }
  return true
}

onMounted(() => {
  console.log('初始化棋盘')
  gamediv.value.innerHTML = ''
  for (let i = 0; i < props.height; i++) {
    console.log('----------')
    console.log('行', i)
    const line = document.createElement('div')
    line.classList.add('line')
    for (let j = 0; j < props.width; j++) {
      console.log('列', j)
      const block = document.createElement('div')
      block.classList.add('block')
      block.addEventListener('click', function () {
        if (!this.classList.contains('filled')) {
          console.log('玩家', current, '颜色', players[current])
          this.classList.add('filled')
          this.style.setProperty('--color', players[current])
          if (!canPlace(this)) {
            this.style.removeProperty('--color')
          }
          current++
          if (current >= players.length) {
            current = 0
          }
        }
      })
      line.appendChild(block)
    }
    gamediv.value.appendChild(line)
  }
})
</script>

<template>
  <div class="game" ref="gamediv">Loading...</div>
</template>

<style scoped>
:deep(*) {
  user-select: none;
  -webkit-user-drag: none;
}

:deep() {
  overflow: hidden;
  cursor: none;
  padding: 1em;
  background-color: rgb(211, 169, 105);
}

:deep(.line) {
  display: flex;
}

:deep(.block) {
  display: inline-block;
  position: relative;
  width: 2em;
  height: 2em;
}

:deep(.block::before) {
  /* 为什么不用制表符? */
  /* 因为制表符太粗了 */
  /* content: '┼'; */
  content: '+';
  color: #fff;
  font-weight: lighter;
  /* 为什么不用font-size? */
  /* font-size的缩放导致文字往下偏移 */
  /* scale和zoom都是从中心点缩放，所以不会偏移 */
  /* 考虑到兼容性问题，这里不用zoom */
  /* font-size: 500%; */
  /* zoom: 500%; */
  transform: scale(500%);
  overflow: hidden;
  top: 5.5px;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 1;
  position: absolute;
  /* 禁用鼠标事件 */
  /* 防止框框往右下角偏移 */
  pointer-events: none;
}

:deep(.line:first-child .block::before) {
  /* 上边 */
  content: '┬';
  transform: scale(282%);
}

:deep(.line:first-child .block:first-child:before) {
  /* 左上 */
  content: '┌';
  transform: scale(282%);
  margin-top: 0;
}

:deep(.block:first-child:before) {
  /* 左边 */
  content: '├';
  transform: scale(300%);
  margin-top: 0.5px;
}

:deep(.line:last-child .block:first-child:before) {
  /* 左下 */
  content: '└';
  transform: scale(320%);
  margin-top: 0.9px;
}

:deep(.line:last-child .block::before) {
  /* 下边 */
  content: '┴';
  transform: scale(325%);
  margin-top: 0.8px;
}

:deep(.line:last-child .block:last-child:before) {
  /* 右下 */
  content: '┘';
  transform: scale(320%);
  margin-top: 0.9px;
}

:deep(.block:last-child:before) {
  /* 右边 */
  content: '┤';
  transform: scale(305%);
  margin-top: 0.4px;
}

:deep(.line:first-child .block:last-child:before) {
  /* 右上 */
  content: '┐';
  transform: scale(285%);
  margin-top: 0.5px;
}

:deep(.block::after) {
  content: '';
  border-radius: 50%;
  z-index: 2;
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}

:deep(.block:hover::after) {
  border: 3px solid #000;
}

:deep(.block.filled::after) {
  border: none;
  background-color: var(--color);
  z-index: 3;
}

:deep(.block.filled:hover::after) {
  filter: invert(20%);
}
</style>
