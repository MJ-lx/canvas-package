<template>
  <div></div>
</template>

<script setup>
import p5 from 'p5'
const round = (sketch) => {
  sketch.setup = () => {
    sketch.createCanvas(200, 200) // 创建一个 200 * 200 像素的画布
    sketch.background(180, 180, 180) // 画布背景色 background(r, g, b)
  }

  sketch.draw = () => {
    sketch.background(100, 180, 180)
    sketch.circle(sketch.mouseX, sketch.mouseY, 40, 40) // 画一个圆形
    // sketch.point(100, 100) // 点
    // // sketch.strokeWeight(10) // 更大的点
    // sketch.line(60, 30, 130, 140) // 线
  }
}
new p5(round, 'p5Round')
// s.draw()
</script>

