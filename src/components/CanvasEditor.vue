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
import { onMounted } from "vue"
import { NButton, NSpace, NUpload } from "naive-ui"
import exifr from "exifr"
import Konva from 'konva';


// Object Declaration Part
console.log("Konva输出", Konva)
console.log("exif输出", exifr)

let imageObj: HTMLImageElement
let stage: any
let photo: any
let canvasLimit:number = document.documentElement.clientWidth * 0.4
console.log("宽高限制: ", canvasLimit)
let photoScale:number

// Logic Part

onMounted(() => {

  // Create Stage
  stage = new Konva.Stage({
    container: 'PhotoViewer', // id of container <div>
    width: canvasLimit,
    height: canvasLimit,
  });

  console.log(stage)

  // Create Layer
  var layer = new Konva.Layer();
  stage.add(layer);

  // Create Image
  imageObj = new Image();
  imageObj.onload = function () {
    photo = new Konva.Image({
      x: 0,
      y: 0,
      image: imageObj,
      width: 50,
      height: 50
    });

    layer.add(photo);
  };
});

// Handles Declaration Part
const handleUpload = (file: any) => {

  let reader = new FileReader();
  reader.readAsDataURL(file.file.file);

  reader.onload = function () {
    if (typeof (reader.result) == 'string') {
      imageObj.src = reader.result
      exifr.parse(reader.result, true).then(output => console.log("EXIF: ", output))

      // 计算并重设图像宽高
      const img = new Image()
      img.onload = () => {
        console.log("图像宽高: ", img.naturalWidth, img.naturalHeight)
        photoScale = Math.min(canvasLimit/img.naturalWidth, canvasLimit/img.naturalHeight)
        stage.scale({x:photoScale, y:photoScale})
        photo.width(img.naturalWidth)
        photo.height(img.naturalHeight)
        photo.move({x:(canvasLimit/photoScale-img.naturalWidth)/2, y:(canvasLimit/photoScale-img.naturalHeight)/2})
        console.log("舞台宽高: ", stage.width(), stage.height())
      }
      img.src = reader.result
    } else {
      // todo: 上传失败的用户感知
    }
  }

  reader.onerror = function () {
    // todo: 上传失败的用户感知
    console.log(reader.error);
  };
}

const handleExport = () => {
  const dataURL = stage.toDataURL({
    pixelRatio: 6 // double resolution
  });

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
  background: #3333;
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