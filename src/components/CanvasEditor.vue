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
        <n-code :code="JSON.stringify(exifInfo)" word-wrap></n-code>
      </n-space>
    </div>
  </div>
</template>


<script setup lang="ts">
import { reactive, onMounted } from "vue"
import { NButton, NSpace, NUpload, NCode } from "naive-ui"
import exifr from "exifr"
import Konva from 'konva';


// Type Const
interface exifInfo {
  iso?: number,
  exposure?: number,
  f?: number,
  focal?: number,
  latitude?: number,
  longitude?: number,
  brand?: string,
  model?: string,
  time?: Date
}

// Object Declaration & Definition Section
let imageObj: HTMLImageElement
let stage: any
let photo: any
let canvasLimit: number = document.documentElement.clientWidth * 0.4
let photoScale: number
let exifInfo: exifInfo = reactive({})

// Main Logic Section

onMounted(() => {

  // Create Stage
  stage = new Konva.Stage({
    container: 'PhotoViewer', 
    width: canvasLimit,
    height: canvasLimit,
  });

  console.log(stage)

  // Create Photo Layer
  var photoLayer = new Konva.Layer();
  stage.add(photoLayer);

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
    photoLayer.add(photo);
  };
});

// Handles Definition Part
const handleUpload = (file: any) => {

  let reader = new FileReader();
  reader.readAsDataURL(file.file.file);

  reader.onload = function () {
    if (typeof (reader.result) == 'string') {
      imageObj.src = reader.result
      // Read Photo Exif Information
      exifr.parse(reader.result).then(output => {
        Object.assign(exifInfo, {
          iso: output.ISO,
          exposure: 1/output.ExposureTime,
          f: output.FNumber,
          focal: output.FocalLength,
          latitude: output.latitude,
          longitude: output.longitude,
          brand: output.Make,
          model: output.Model,
          time: output.DateTimeOriginal
        })
        console.log("EXIF: ", exifInfo)
      })


      // Render Photo & Frame
      const img = new Image()
      img.onload = () => {
        console.log("图像宽高: ", img.naturalWidth, img.naturalHeight)

        // 计算并使用自适应缩放
        photoScale = Math.min(canvasLimit / img.naturalWidth, canvasLimit / img.naturalHeight)
        stage.scale({ x: photoScale, y: photoScale })
        // 以照片分辨率设置元素宽高
        photo.width(img.naturalWidth)
        photo.height(img.naturalHeight)
        // 居中照片
        photo.move({ x: (canvasLimit / photoScale - img.naturalWidth) / 2, y: (canvasLimit / photoScale - img.naturalHeight) / 2 })
        console.log("舞台宽高: ", stage.width(), stage.height())
        handleCreateFrame()
      }
      img.src = reader.result

    } else {
      // todo: 上传成功读取失败的用户感知
    }
  }

  reader.onerror = function () {
    // todo: 上传失败的用户感知
    console.log(reader.error);
  };
}

const handleExport = () => {
  const dataURL = stage.toDataURL({
    pixelRatio: 6
  });

  const link = document.createElement('a');
  link.download = 'stage.png';
  link.href = dataURL;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);

};

const handleCreateFrame = () => {
  // Create Frame Layer
  var frameLayer = new Konva.Layer()
  stage.add(frameLayer)

  // Create Frame
  const frameBackground = new Konva.Rect({
    x: 0,
    y: photo.y(),
    width: photo.width(),
    height: photo.height() * 1.1,
    fill: "white",
  })
  frameLayer.moveDown()
  frameLayer.add(frameBackground)
}




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