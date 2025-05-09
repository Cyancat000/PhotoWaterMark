<template>
  <div class="canvas-container">
    <div class="canvas-panel">
      <div id='PhotoViewer'></div>
    </div>
    <div class="custom-panel">
      <n-space vertical>
        <n-space>
          <n-upload accept="image/*" @change="handleUpload" @before-upload="handleCleanStage" ref="uploader"
            :show-file-list="false">
            <n-button type="primary">
              上传图片
            </n-button>
          </n-upload>
          <n-button @click="handleExport" type="primary">
            下载图片
          </n-button>

        </n-space>
        <n-flex class="paramForm">
          <n-input placeholder="焦段" v-model:value="exifInfo.focal">
            <template #suffix>mm</template>
          </n-input>
          <n-input placeholder="光圈" v-model:value="exifInfo.f">
            <template #prefix>f/</template>
          </n-input>
          <n-input placeholder="曝光时间" v-model:value="exifInfo.exposure">
            <template #prefix>1/</template>
            <template #suffix>s</template>
          </n-input>
          <n-input placeholder="ISO" v-model:value="exifInfo.iso">
            <template #prefix>ISO</template>
          </n-input>
          <n-input placeholder="品牌" v-model:value="exifInfo.brand"/>
          <n-input placeholder="型号" v-model:value="exifInfo.model"/>
          <n-input placeholder="经度" v-model:value="exifInfo.longitude"/>
          <n-input placeholder="纬度" v-model:value="exifInfo.latitude"/>
          <n-input placeholder="日期" v-model:value="exifInfo.date"/>
          <n-input placeholder="时间" v-model:value="exifInfo.time"/>
        </n-flex>
      </n-space>

    </div>
  </div>
</template>


<script setup lang="ts">
import { ref, reactive, onMounted } from "vue"
import { NButton, NSpace, NUpload, NInput, NFlex } from "naive-ui"
import exifr from "exifr"
import Konva from 'konva';


// Type Const
interface exifInfo {
  iso?: string,
  exposure?: string,
  f?: string,
  focal?: string,
  latitude?: string,
  longitude?: string,
  brand?: string,
  model?: string,
  time?: string,
  date?: string
}

// Object Declaration & Definition Section
let stage: any
let photo: any
let canvasLimit: number = document.documentElement.clientWidth * 0.4
let photoScale: number
const exifInfo: exifInfo = reactive({})
let uploader = ref()

// Main Logic Section

onMounted(() => {

  // Create Stage
  stage = new Konva.Stage({
    container: 'PhotoViewer',
    width: canvasLimit,
    height: canvasLimit,
  });


});

// Handles Definition Section
const handleUpload = (file: any) => {

  let reader = new FileReader();
  reader.readAsDataURL(file.file.file);
  console.log(file);

  reader.onload = function () {
    if (typeof (reader.result) == 'string') {
      // Read Photo Exif Information
      exifr.parse(reader.result).then(output => {
        Object.assign(exifInfo, {
          iso: output.ISO,
          exposure: Math.round(1 / output.ExposureTime),
          f: output.FNumber,
          focal: output.FocalLength,
          latitude: convertToDMS(output.latitude, "latitude"),
          longitude: convertToDMS(output.longitude, "longitude"),
          brand: output.Make,
          model: output.Model,
          date: output.GPSDateStamp.replace(/:/g, "."),
          time: output.GPSTimeStamp
        })
        console.log("EXIF: ", output)
      })

      // Render Photo & Frame
      const img = new Image()
      img.onload = () => {
        console.log("图像宽高: ", img.naturalWidth, img.naturalHeight)

        // Render Photo
        const photoLayer = new Konva.Layer();
        stage.add(photoLayer);

        photo = new Konva.Image({
          x: 0,
          y: 0,
          image: img,
          width: 50,
          height: 50
        });
        photoLayer.add(photo);

        // 计算并使用自适应缩放
        photoScale = Math.min(canvasLimit / img.naturalWidth, canvasLimit / img.naturalHeight)
        stage.scale({ x: photoScale, y: photoScale })
        // 以照片分辨率设置元素宽高
        photo.width(img.naturalWidth)
        photo.height(img.naturalHeight)
        // 居中照片
        photo.move({ x: (canvasLimit / photoScale - img.naturalWidth) / 2, y: (canvasLimit / photoScale - img.naturalHeight) / 2 })
        console.log("舞台宽高: ", stage.width(), stage.height())
        createFrameLayer()
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

const handleCleanStage = () => {
  uploader.value.clear()
  stage.destroyChildren()
}

// Methods Definition Section

const createFrameLayer = () => {
  // Create Frame Layer
  const frameLayer = new Konva.Layer()
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

const convertToDMS = (value:number, type:string) => {
  // 获取绝对值
  const absolute = Math.abs(value);

  // 计算度、分、秒
  const degrees = Math.floor(absolute); // 整数部分作为度
  const minutesDecimal = (absolute - degrees) * 60; // 小数部分转为分钟
  const minutes = Math.floor(minutesDecimal); // 整数部分作为分
  const seconds = ((minutesDecimal - minutes) * 60).toFixed(2); // 小数部分转为秒（保留两位小数）

  // 确定方向
  let direction = "";
  if (type === "latitude") {
    direction = value >= 0 ? "N" : "S"; // 北纬 (N) 或 南纬 (S)
  } else if (type === "longitude") {
    direction = value >= 0 ? "E" : "W"; // 东经 (E) 或 西经 (W)
  } else {
    throw new Error("无效的类型(type)，请指定 'latitude' 或 'longitude'");
  }

  // 组合度分秒字符串
  return `${degrees}° ${minutes}' ${seconds}" ${direction}`;
}



</script>

<style lang=scss scoped>
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
  padding: 10px;
  box-sizing: border-box;
}

.canvas-panel {
  height: 40vw;
  width: 40vw;
}

.paramForm  .n-input{
  flex: 128px;
}
</style>