<template>
  <div class="canvas-container">
    <div class="canvas-panel">
      <div id='PhotoViewer'></div>
    </div>
    <div class="custom-panel">
      <n-space vertical>
        <n-flex class="button-group" justify="space-between">
          <div><n-upload accept="image/*" @change="handleUpload" @before-upload="handleCleanStage" ref="uploader"
              :show-file-list="false">
              <n-button type="primary">
                上传图片
              </n-button>
            </n-upload></div>
          <n-button @click="handleExport" type="primary">
            下载图片
          </n-button>
          <n-button @click="handleReRenderText" type="primary">
            保存修改
          </n-button>

        </n-flex>
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
          <n-input placeholder="品牌" v-model:value="exifInfo.brand" />
          <n-input placeholder="型号" v-model:value="exifInfo.model" />
          <n-input placeholder="经度" v-model:value="exifInfo.longitude" />
          <n-input placeholder="纬度" v-model:value="exifInfo.latitude" />
          <n-input placeholder="日期" v-model:value="exifInfo.date" />
          <n-input placeholder="时间" v-model:value="exifInfo.time" />
        </n-flex>
        <n-select placeholder="选择LOGO" v-model:value="logoSrc" :options="logoList"
          @update:value="handleReRenderLogo()" />
        <n-flex class="button-group" justify="space-between">
          <n-button @click="handleControlLogo('size', 0.1)">logo放大</n-button>
          <n-button @click="handleControlLogo('size', -0.1)">logo缩小</n-button>
          <n-button @click="handleControlLogo('location', -10)">logo左移</n-button>
          <n-button @click="handleControlLogo('location', 10)">logo右移</n-button>
        </n-flex>
      </n-space>

    </div>
  </div>
</template>


<script setup lang="ts">
import { ref, reactive, onMounted } from "vue"
import { NButton, NSpace, NUpload, NInput, NFlex, NSelect } from "naive-ui"
import exifr from "exifr"
import Konva from 'konva';
import dayjs from 'dayjs'

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

type textType = 'device' | 'param' | 'date' | 'location'

// Object Declaration & Definition Section
let stage: any
let photo: any
let canvasLimit: number = document.documentElement.clientWidth * 0.4
let photoScale: number
const exifInfo: exifInfo = reactive({})
let uploader = ref()
let textList: Array<any> = []
let logo: any
let logoProxyImage: HTMLImageElement = new Image()

const logoList = [
  { label: "Alpha(索尼)", value: "Alpha.svg" },
  { label: "Aquos(夏普)", value: "Aquos.svg" },
  { label: "ARRI(阿莱)", value: "ARRI.svg" },
  { label: "Canon(佳能)", value: "Canon.svg" },
  { label: "Casio(卡西欧)", value: "Casio.svg" },
  { label: "Fujifilm(富士)", value: "Fujifilm.svg" },
  { label: "Hasselblad(哈苏)", value: "Hasselblad.svg" },
  { label: "Huawei(华为)", value: "Huawei.svg" },
  { label: "Kodak(柯达)", value: "Kodak.svg" },
  { label: "Konica(柯尼卡)", value: "Konica.svg" },
  { label: "Leica(徕卡)", value: "Leica.svg" },
  { label: "Lumix(松下)", value: "Lumix.svg" },
  { label: "Minota(美能达)", value: "Minota.svg" },
  { label: "Nikon(尼康)", value: "Nikon.svg" },
  { label: "Olympus(奥林巴斯)", value: "Olympus.svg" },
  { label: "OPPO", value: "OPPO.svg" },
  { label: "Panasonic(松下)", value: "Panasonic.svg" },
  { label: "Pentax(宾得)", value: "Pentax.svg" },
  { label: "Phase One(飞思)", value: "Phase_One.svg" },
  { label: "Pixii", value: "Pixii.svg" },
  { label: "RED", value: "RED.png" },
  { label: "Ricoh(理光)", value: "Ricoh.svg" },
  { label: "Samsung(三星)", value: "Samsung.svg" },
  { label: "Sharp(夏普)", value: "Sharp.svg" },
  { label: "Sigma(适马)", value: "Sigma.svg" },
  { label: "Sony(索尼)", value: "Sony.svg" },
  { label: "VIVO", value: "VIVO.svg" },
  { label: "Xiaomi(小米)", value: "Xiaomi.svg" },
  { label: "Xperia(索尼)", value: "Xperia.svg" },
  { label: "YI(小蚁)", value: "YI.png" },
  { label: "Zeiss(蔡司)", value: "Zeiss.svg" },
]

let logoSrc = ref("Leica.svg")

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
        console.log("EXIF: ", output)
        Object.assign(exifInfo, {
          iso: "ISO" in output ? output.ISO : "0",
          exposure: "ExposureTime" in output ? Math.round(1 / output.ExposureTime) : "0",
          f: "FNumber" in output ? output.FNumber : "0",
          focal: "FocalLength" in output ? output.FocalLength : "0",
          latitude: convertToDMS(output.latitude, "latitude"),
          longitude: convertToDMS(output.longitude, "longitude"),
          brand: "Make" in output ? output.Make : "Brand",
          model: "Model" in output ? output.Model : "Model",
          date: "DateTimeOriginal" in output ? dayjs(output.DateTimeOriginal).format("YYYY.MM.DD") : "2000.00.00",
          time: "DateTimeOriginal" in output ? dayjs(output.DateTimeOriginal).format("HH:mm:ss") : "00:00:00"
        })
        console.log(exifInfo);
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


        createFrameLayerNo1()
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

  const temp = stage.find('#frame-background')[0]

  console.log(photoScale)

  const dataURL = stage.toDataURL({
    mimeType: "image/jpeg",
    pixelRatio: 1 / photoScale,
    x: temp.x() * photoScale,
    y: temp.y() * photoScale,
    width: temp.width() * photoScale,
    height: temp.height() * photoScale
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
  textList.length = 0
}

const handleReRenderText = () => {
  setTextContent(['device', { type: 'param', align: 'right' }, 'date', { type: 'location', align: 'right' }])
}

const handleReRenderLogo = () => {
  logoProxyImage.src = "/src/images/"+logoSrc.value
}

const handleControlLogo = (type: "size" | "location", value: number) => {
  switch (type) {
    case "size": 
      logo.scaleX(logo.scaleX()+value)
      logo.scaleY(logo.scaleY()+value)
      break;
    case "location": logo.x(logo.x() + value)
      break;
  }
}

// Methods Definition Section

// 相框样式1号
const createFrameLayerNo1 = () => {
  const unitLength = photo.height() * 0.01

  // Create Frame Layer
  const frameLayer = new Konva.Layer()
  stage.add(frameLayer)

  // Create Frame
  const frameBackground = new Konva.Rect({
    x: photo.x(),
    y: photo.y(),
    width: photo.width(),
    height: unitLength * 110,
    fill: "white",
  })
  frameLayer.moveDown()
  frameLayer.add(frameBackground)
  frameBackground.id('frame-background')

  if (photoScale * frameBackground.height() > stage.height()) {
    console.log("超出范围！");
    photoScale = stage.height() / frameBackground.height()
    stage.scale({ x: photoScale, y: photoScale })
    // console.log(photo.x(),photo.y());
  }


  // Create Text Element & Set Text Style
  textList.push(new Konva.Text({
    x: unitLength * 4 + photo.x(),
    y: unitLength * 102 + photo.y(),
    fontSize: unitLength * 2.8,
    fontStyle: "bold",
  }))

  textList.push(new Konva.Text({
    x: photo.width() - (unitLength * 4) + photo.x(),
    y: unitLength * 102 + photo.y(),
    fontSize: unitLength * 2.8,
    fontStyle: "bold",
  }))

  textList.push(new Konva.Text({
    x: unitLength * 4 + photo.x(),
    y: unitLength * 105.6 + photo.y(),
    fontSize: unitLength * 2.8,
    fill: "#666666",
    fontStyle: "bold",
  }))

  textList.push(new Konva.Text({
    x: photo.width() - (unitLength * 4) + photo.x(),
    y: unitLength * 105.6 + photo.y(),
    fontSize: unitLength * 2.8,
    fill: "#666666",
    fontStyle: "bold",
  }))

  // 传参并渲染文本
  textList.forEach(i => {
    setTextContent(['device', { type: 'param', align: 'right' }, 'date', { type: 'location', align: 'right' }])
    frameLayer.add(i)
  })

  // 创建Logo分割线
  const splitLine = new Konva.Rect({
    x: photo.width() - Math.max(textList[1].getTextWidth(), textList[3].getTextWidth()) - unitLength * 7 + photo.x(),
    y: textList[0].y(),
    width: unitLength * 0.3,
    height: unitLength * 6,
    fill: "#BBBBBB"
  })
  frameLayer.add(splitLine)

  // 创建Logo对象
  logoProxyImage.onload = () => {
    if (logo === undefined) {
      logo = new Konva.Image({
        x: splitLine.x() - (unitLength * 2),
        y: unitLength * 105 + photo.y(),
        image: logoProxyImage,
      })
      frameLayer.add(logo)
    }
    logo.height(unitLength * 6)
    logo.width((logoProxyImage.naturalWidth / logoProxyImage.naturalHeight) * unitLength * 6)
    logo.offsetY(logo.height()/2)
    logo.offsetX(logo.width())

    console.log("LOGO宽高", logoProxyImage.naturalWidth, logoProxyImage.naturalHeight);

    console.log("图像已载入");
  }
  logoProxyImage.src = "/src/images/"+logoSrc.value
}

// 转换数字经纬度为度分秒
const convertToDMS = (value: number, type: string) => {

  // 确定方向
  let direction = "";
  if (type === "latitude") {
    direction = value >= 0 ? "N" : "S"; // 北纬 (N) 或 南纬 (S)
  } else if (type === "longitude") {
    direction = value >= 0 ? "E" : "W"; // 东经 (E) 或 西经 (W)
  } else {
    throw new Error("无效的类型(type)，请指定 'latitude' 或 'longitude'");
  }

  if (isNaN(value)) {
    return `0° 0' 0" ${direction}`
  } else {
    // 获取绝对值
    const absolute = Math.abs(value);

    // 计算度、分、秒
    const degrees = Math.floor(absolute); // 整数部分作为度
    const minutesDecimal = (absolute - degrees) * 60; // 小数部分转为分钟
    const minutes = Math.floor(minutesDecimal); // 整数部分作为分
    const seconds = ((minutesDecimal - minutes) * 60).toFixed(2); // 小数部分转为秒（保留两位小数）



    // 组合度分秒字符串
    return `${degrees}° ${minutes}' ${seconds}" ${direction}`;
  }

}

// 转换undefined为空字符串
const undefinedToBlankString = (obj: string | undefined) => {
  return obj === undefined ? "" : obj
}

// 获取所需要的格式化字符串
const getFormatString = (type: textType) => {
  switch (type) {
    case "param":
      return `${undefinedToBlankString(exifInfo.focal)}mm f/${undefinedToBlankString(exifInfo.f)} ISO${undefinedToBlankString(exifInfo.iso)} 1/${undefinedToBlankString(exifInfo.exposure)}s`
    case "device":
      return undefinedToBlankString(exifInfo.brand) + " " + undefinedToBlankString(exifInfo.model)
    case "date":
      return undefinedToBlankString(exifInfo.date) + " " + undefinedToBlankString(exifInfo.time)
    case "location":
      return undefinedToBlankString(exifInfo.latitude) + " " + undefinedToBlankString(exifInfo.longitude)
  }
}

// 设置文本元素的内容与偏移
const setTextContent = (textTypeList: Array<textType | { type: textType, align: "middle" | "right" }>) => {
  textTypeList.forEach((item, index) => {
    if (typeof (item) === 'string') {
      textList[index].text(getFormatString(item))
    } else {
      textList[index].text(getFormatString(item.type))
      textList[index].offsetX(textList[index].getTextWidth())
    }
  })
}





</script>

<style lang=scss scoped>
.canvas-container {
  width: 80vw;
  height: 40vw;
  display: flex;
}

.custom-panel {
  width: 40vw;
  height: 40vw;
  background-color: #FAFAFA;
  border-radius: 4px;
  box-shadow: 1px 1px 8px -4px #666;
  padding: 16px;
  box-sizing: border-box;
  margin-left: 8px
}

.canvas-panel {
  height: 40vw;
  width: 40vw;
  background: #EEE;
}

.paramForm .n-input {
  flex: 128px;
}

// .button-group>* {
//   flex: auto;
// }</style>