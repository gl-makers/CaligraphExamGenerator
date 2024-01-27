<template>
  <div>
    <n-h1 prefix="bar" align-text>
      <n-text> 书法练习生成器 </n-text>
    </n-h1>
    <n-space vertical>
      <n-space>
        选择字体
        <n-select v-model:value="font" :options="fonts" style="width: 150px;"/>
      </n-space>
      <n-space>
        <n-button type="primary" @click="count++"> + </n-button>
        <n-button type="primary" @click="count--"> - </n-button>
      </n-space>
      <n-space style="width: 400px">
        <template v-for="(_, index) in new Array(count)">
          <n-input
            v-model:value="text[Number(index)]"
            @change="render"
            type="text"
            placeholder="字"
            style="width: 60px"
          />
        </template>
      </n-space>
      <n-button type="primary" @click="exportImg">导出为图片</n-button>
    </n-space>
    <div class="paper-box">
      <canvas ref="canvas" style="width: 900px; height: 1600px"></canvas>
    </div>
  </div>
</template>

<script setup lang="ts">
import { NButton, NH1, NText, NSpace, NInput, NSelect } from "naive-ui";
import { onMounted, ref, watch } from "vue";

const count = ref(0);
const text = ref<Array<string>>([]);
const font = ref("楷体");

const canvas = ref<HTMLCanvasElement>();

const fonts = [
  { label: "楷体", value: "楷体" },
  { label: "田英章行楷", value: "TianyinzhangXK" },
];

function createHiDPICanvas(w: number, h: number, can: HTMLCanvasElement, ratio: number = 1) {
  can.width = w * ratio;
  can.height = h * ratio;
  can.style.width = w + "px";
  can.style.height = h + "px";
  // @ts-ignore
  can.getContext("2d").setTransform(ratio, 0, 0, ratio, 0, 0);
  return can;
}

// 当 canvas 准备好
onMounted(() => {
  const ctx = canvas.value?.getContext("2d");
  if (!ctx) return;
  createHiDPICanvas(900, 1600, canvas.value!, 5);
  render()
});

const render = () => {
  const ctx = canvas.value?.getContext("2d");
  if (!ctx) return;
  ctx.clearRect(0, 0, 900, 1600);
  console.log(text.value);
  // 生成田字格
  ctx.beginPath();
  // 大小
  const size = 70;
  const gap = 20;
  // 计算能塞多少个一行
  const countX = Math.floor((900 - 20) / (size + gap));
  // 计算能塞多少行
  // const countY = Math.floor((1600 - 20) / (size + gap));
  let pos = {
    x: (900 - (countX*size + (countX-1)*gap))/2,
    y: 30,
  };
  ctx.strokeStyle = "red";
  ctx.lineWidth = 1;

  const drawBox = (x: number, y: number, text: string) => {
    ctx.beginPath()
    ctx.moveTo(x, y);
    ctx.setLineDash([5, 0]);
    // 框框
    ctx.lineTo(x + size, y);
    ctx.lineTo(x + size, y + size);
    ctx.lineTo(x, y + size);
    ctx.lineTo(x, y);
    ctx.stroke();
    ctx.closePath()
    // 十字
    ctx.beginPath()
    ctx.setLineDash([5, 5]);
    ctx.moveTo(x + size / 2, y);
    ctx.lineTo(x + size / 2, y + size);
    ctx.moveTo(x, y + size / 2);
    ctx.lineTo(x + size, y + size / 2);
    // ×(虚线)
    ctx.moveTo(x, y);
    ctx.lineTo(x + size, y + size);
    ctx.moveTo(x + size, y);
    ctx.lineTo(x, y + size);
    ctx.stroke();
    ctx.closePath()
    // 绘制文字
    ctx.fillStyle = "#ff000070";
    ctx.font = (size - 10) + "px " + font.value;
    ctx.fillText(text, x + size / 2 - 28, y + size / 2 + 23);
    
  };

  console.log(countX);
  for (let i = 0; i < text.value.length; i++) {
    for (let j = 0; j < countX; j++) {
      drawBox(pos.x, pos.y, text.value[i]);
      pos.x += size + gap;
    }
    pos.x = (900 - (countX*size + (countX-1)*gap))/2;
    pos.y += size + gap;
  }
  ctx.stroke();
};

watch(font, render);

const exportImg = () => {
  const link = document.createElement("a");
  link.download = `书法练习-${text.value.join("")}.png`;
  // @ts-ignore
  link.href = canvas.value?.toDataURL("image/png");
  link.click();
};
</script>

<style scoped>
.paper-box {
  padding: 8px;
  background-color: rgb(255, 217, 0);
  width: fit-content;
  canvas {
    background-color: white;
  }
}
</style>
