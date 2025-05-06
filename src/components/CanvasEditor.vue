<template>
  <div class="canvas-container">
    <div class="canvas-panel">
      <div id='PhotoViewer'></div>
    </div>
    <div class="custom-panel">
      <n-space>
        <n-upload accept="image/*" @change="handleUpload">
          <n-button type="primary">
            上传图片
          </n-button>
        </n-upload>
        <n-button @click="handleExport" type="primary">
          下载图片
        </n-button>
      </n-space>
    </div>
  </div>
</template>


<script setup lang="ts">
import { ref, reactive, onMounted } from "vue"
import { NButton, NSpace, NUpload } from "naive-ui"
import exifr from "exifr"
import Konva from 'konva';


// Object Declaration Part
console.log("Konva输出", Konva)
console.log("exif输出", exifr)

let imageObj: HTMLImageElement
let stage: any

// 逻辑执行

onMounted(() => {

  // 创建舞台
  stage = new Konva.Stage({
    container: 'PhotoViewer', // id of container <div>
    width: document.documentElement.clientWidth * 0.4,
    height: document.documentElement.clientWidth * 0.4,
  });

  console.log(stage)

  // 创建并添加图层
  var layer = new Konva.Layer();
  stage.add(layer);

  var circle = new Konva.Circle({
    x: stage.width() / 2,
    y: stage.height() / 2,
    radius: 70,
    fill: 'red',
    stroke: 'black',
    strokeWidth: 4,
  });
  layer.add(circle);

  imageObj = new Image();
  imageObj.onload = function () {
    const yoda = new Konva.Image({
      x: 50,
      y: 50,
      image: imageObj,
      width: 106,
      height: 118
    });

    layer.add(yoda);
  };
  imageObj.src = 'https://konvajs.org/assets/yoda.jpg';
});

// Handles Declaration Part
const handleUpload = (file: any) => {

  let reader = new FileReader();
  reader.readAsDataURL(file.file.file);

  reader.onload = function () {
    if (typeof (reader.result) == 'string') {
      imageObj.src = reader.result
      exifr.parse(reader.result, true)
        .then(output => console.log(output))
    } else {
      // 待办: 上传失败的用户感知
    }
  }

  reader.onerror = function () {
    // 待办: 上传失败的用户感知
    console.log(reader.error);
  };
}

const handleExport = () => {
  const dataURL = stage.toDataURL({
    pixelRatio: 4 // double resolution
  });

  // 下载
  const link = document.createElement('a');
  link.download = 'stage.png';
  link.href = dataURL;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);

};




</script>

<style scoped>
.canvas-container {
  width: 80vw;
  height: 40vw;
  background: #8888;
  display: flex;
}

.custom-panel {
  width: 40vw;
  height: 40vw;
  background-color: aquamarine;
}

.canvas-panel {
  height: 40vw;
  width: 40vw;
}
</style>