<template>
  <div class="content">
    <section id="sound-show">
      <img class="middle-img" src="../assets/img/banma-music.webp" alt="" />
    </section>
    <audio controls="controls" autoplay="autoplay">
      <source src="../assets/banma-music.mp3" type="audio/mpeg" />
    </audio>
  </div>
</template>

<script setup>
import { onMounted } from 'vue'
import p5 from 'p5'
// eslint-disable-next-line no-unused-vars
import p5Sound from '../libs/p5.sound.js'

const s = (sketch) => {
  //图案里，条的颜色默认值rgba
  let color1 = [255, 255, 255, 180]
  //图案里，面的颜色默认值rgba
  let color2 = [255, 255, 255, 150]

  const WHITE = 20
  const NUM = 48
  //图案的最长和最短
  const MIN_LENGTH = 112.5
  const MAX_LENGTH = 185
  //那个一条一条的图案的宽高
  const WID = 3
  const HIG = 70

  let fft
  let imgDom
  sketch.preload = () => {
    imgDom = document.getElementsByClassName('middle-img')[0]
    sketch.loadImage(imgDom.src, afterLoadImg)
  }
  sketch.setup = () => {
    const dom = document.getElementById('sound-show')
    sketch.createCanvas(dom.offsetWidth + MAX_LENGTH, dom.offsetHeight)
    sketch.audioContext_ = sketch.getAudioContext()
    const audioEl = document.querySelector('audio')
    const source = sketch.audioContext_.createMediaElementSource(audioEl)
    source.connect(p5.soundOut)

    //0.8是变化程度，越小图案越抖
    fft = new p5.FFT(0.8, sketch.ORI_NUM)
    fft.setInput(source)
    sketch.angleMode(sketch.DEGREES)
  }

  sketch.draw = () => {
    sketch.clear()
    const spectrum = fft.analyze()
    sketch.noStroke()
    sketch.fill(
      color2[0] + WHITE,
      color2[1] + WHITE,
      color2[2] + WHITE,
      color2[3]
    )
    sketch.translate(sketch.width / 2, sketch.height / 2)

    sketch.beginShape()

    for (let i = 0; i < NUM; i++) {
      // 获取音频大小与长度之间的映射
      const r = sketch.map(spectrum[i], 0, 255, MIN_LENGTH, MAX_LENGTH)
      // 条数与角度之间的映射
      const angle = sketch.map(i, 0, NUM - 1, -120, 60)
      sketch.push()
      sketch.fill(
        color1[0] + WHITE,
        color1[1] + WHITE,
        color1[2] + WHITE,
        color1[3]
      )

      sketch.rotate(angle - 30)
      sketch.rect(-WID / 2, r - HIG, WID, HIG, WID)
      sketch.pop()
      //面面的点
      let x = r * sketch.cos(angle)
      let y = r * sketch.sin(angle)
      sketch.curveVertex(x, y)
    }

    for (let i = NUM - 1; i >= 0; i--) {
      let r = sketch.map(spectrum[i], 0, 255, MIN_LENGTH, MAX_LENGTH)
      let angle = sketch.map(i, NUM - 1, 0, 60, 240)
      //另半个条条
      sketch.push()
      sketch.fill(
        color2[0] + WHITE,
        color2[1] + WHITE,
        color2[2] + WHITE,
        color2[3]
      )
      sketch.rotate(angle - 30)
      sketch.rect(-WID / 2, r - HIG, WID, HIG, WID)
      sketch.pop()
      //还是面面的点
      let x = r * sketch.cos(angle)
      let y = r * sketch.sin(angle)
      sketch.curveVertex(x, y)
    }
    sketch.endShape(sketch.CLOSE)
  }

  const afterLoadImg = (img) => {
    img.loadPixels()
    const twoColors = getTwoMainColors(img)
    color1[0] = twoColors[1].r
    color1[1] = twoColors[1].g
    color1[2] = twoColors[1].b
    color2[0] = twoColors[0].r
    color2[1] = twoColors[0].g
    color2[2] = twoColors[0].b
  }

  const getTwoMainColors = (img) => {
    // 缩小图片减少计算量
    img.resize(10, 10)
    const classifiedPix = new Array(7).fill([])

    for (let i = 0; i < 400; i += 4) {
      const r = img.pixels[i]
      const g = img.pixels[i + 1]
      const b = img.pixels[i + 2]
      const hue = getHue(r, g, b)
      classifiedPix[sketch.round(hue / 60)].push(r, g, b)
    }

    const mainColor = new Array(2).fill([])

    classifiedPix.forEach((colors) => {
      if (colors.length > mainColor[0].length) {
        mainColor[0] = colors
      } else if (colors.length > mainColor[1].length) {
        mainColor[1] = colors
      }
    })

    const mainRgbAver = new Array(2).fill({ r: 0, g: 0, b: 0 })
    // 计算主色平均值
    mainColor.forEach((main, index) => {
      const count = main.length / 3
      for (let i = 0; i < main.length; i += 3) {
        mainRgbAver[index].r += main[i]
        mainRgbAver[index].g += main[i + 1]
        mainRgbAver[index].b += main[i + 2]
      }
      mainRgbAver[index].r /= count
      mainRgbAver[index].g /= count
      mainRgbAver[index].b /= count
    })

    return mainRgbAver
  }

  //根据rgb的数值取得色调，也就是hsv里的h
  function getHue(red, green, blue) {
    let min = Math.min(Math.min(red, green), blue)
    let max = Math.max(Math.max(red, green), blue)

    if (min == max) {
      return 0
    }
    let hue
    if (max == red) {
      hue = (green - blue) / (max - min)
    } else if (max == green) {
      hue = 2 + (blue - red) / (max - min)
    } else {
      hue = 4 + (red - green) / (max - min)
    }
    hue = hue * 60
    if (hue < 0) hue += 360
    return Math.round(hue)
  }
}

onMounted(() => {
  new p5(s, 'sound-show')
})
</script>

<style scoped>
.content {
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
}
#sound-show {
  height: 400px;
  width: 300px;
  margin-bottom: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.middle-img {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  position: absolute
}
</style>
