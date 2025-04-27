<template>
  <div class="canvas-container-wrapper">
    <button @click="handleExport" style="margin-bottom: 10px"></button>
    <v-stage ref="stageRef" :config="stageSize" @dragstart="handleDragstart" @dragend="handleDragend">
      <v-layer ref="layer">
        <v-star v-for="item in list" :key="item.id" :config="{
          x: item.x,
          y: item.y,
          rotation: item.rotation,
          id: item.id,
          numPoints: 5,
          innerRadius: 30,
          outerRadius: 50,
          fill: '#89b717',
          opacity: 0.8,
          draggable: true,
          scaleX: dragItemId === item.id ? item.scale * 1.2 : item.scale,
          scaleY: dragItemId === item.id ? item.scale * 1.2 : item.scale,
          shadowColor: 'black',
          shadowBlur: 10,
          shadowOffsetX: dragItemId === item.id ? 15 : 5,
          shadowOffsetY: dragItemId === item.id ? 15 : 5,
          shadowOpacity: 0.6
        }" />
      </v-layer>
    </v-stage>
  </div>
</template>


<script setup lang="ts">
import { ref, onMounted } from "vue"
const stageSize = {
  width: window.innerWidth,
  height: window.innerHeight
};

interface star {
  x: number;
  y: number;
  rotation: number;
  id: string;
  scale: number;
}
const list = ref<star[]>([]);
const dragItemId = ref(null);



const handleDragstart = (e:any) => {
  // save drag element:
  dragItemId.value = e.target.id();
  // move current element to the top:
  const item = list.value.find(i => i.id === dragItemId.value);
  if (item!=undefined) {
    const index = list.value.indexOf(item);
    list.value.splice(index, 1);
    list.value.push(item);
  }
};

const handleDragend = () => {
  dragItemId.value = null;
};

  const stageRef:any = ref(null)

  const handleExport = () => {
  // get data URL with default settings
  const dataURL = stageRef.value.getNode().toDataURL();
  
  // open in new window
  const win:any = window.open();
  win.document.write(`<img src="${dataURL}" alt="Stage"/>`);
  
  // you can also save with different settings
  const jpegURL = stageRef.value.getNode().toDataURL({
    mimeType: 'image/jpeg',
    quality: 0.8
  });
  console.log('JPEG URL:', jpegURL);
};

onMounted(() => {
  for (let n = 0; n < 30; n++) {
    list.value.push({
      id: Math.round(Math.random() * 10000).toString(),
      x: Math.random() * stageSize.width,
      y: Math.random() * stageSize.height,
      rotation: Math.random() * 180,
      scale: Math.random()
    });
  }


});
</script>

<style scoped>
.canvas-container-wrapper {
  /* 容器样式，可以控制画布外层布局 */
  width: 100%;
  max-width: 800px;
  /* 示例：设置最大宽度 */
  margin: 20px auto;
  /* 居中 */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  background-color: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
}

#canvas-element {
  /* Canvas 元素本身的样式 */
  border: 1px solid #ccc;
  display: block;
  /* 避免底部的空白间隙 */
}
</style>