<template>
  <div class="particle-clock-container">
    <canvas ref="canvasRef" class="particle-clock-container-canvas"></canvas>
  </div>
</template>

<script setup>
import {onMounted, onUnmounted, reactive, ref} from 'vue';

const state = reactive({
  text: '',
  particles: [],
})

const canvasRef = ref(null);

class Particle {
  constructor() {
    this.size = getRandom(2 * devicePixelRatio, 7 * devicePixelRatio);
    const r = Math.min(canvasRef.value.width, canvasRef.value.height) / 2;
    const rad = (getRandom(0, 360) * Math.PI) / 180;
    const cx = canvasRef.value.width / 2;
    const cy = canvasRef.value.height / 2;
    this.x = cx + r * Math.cos(rad);
    this.y = cy + r * Math.sin(rad);
  }

  draw(ctx) {
    ctx.beginPath();
    ctx.fillStyle = '#5445544d';
    ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
    ctx.fill();
  }

  moveTo(tx, ty) {
    const duration = 500; // 500ms的运动时间
    const sx = this.x, sy = this.y;
    const xSpeed = (tx - sx) / duration;
    const ySpeed = (ty - sy) / duration;
    const startTime = Date.now();
    const _move = () => {
      const t = Date.now() - startTime;
      const x = sx + xSpeed * t;
      const y = sy + ySpeed * t;
      this.x = x;
      this.y = y;
      if (t >= duration) {
        this.x = tx;
        this.y = ty;
        return;
      }
      requestAnimationFrame(_move);
    }

    _move();
  }
}

const initCanvasSize = () => {
  canvasRef.value.width = window.innerWidth * devicePixelRatio;
  canvasRef.value.height = window.innerHeight * devicePixelRatio;
}

const getRandom = (min, max) => {
  return Math.floor(Math.random() * (max + 1 - min) + min);
}

const clear = (ctx) => {
  ctx.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height);
};

const drawParticles = (ctx) => {
  for (const p of state.particles) {
    p.draw(ctx);
  }
}

// 新函数用于绘制时间文本
const drawText = (ctx) => {

  const width = canvasRef.value.width;
  const height = canvasRef.value.height;
  ctx.fillStyle = '#000';
  ctx.textBaseline = 'middle';
  ctx.font = `${140 * devicePixelRatio}px 'DS-Digital', sans-serif`;
  ctx.textAlign = 'center';
  ctx.fillText(state.text, width / 2, height / 2);

  const points = getPoints()

  clear(ctx);

  for (let i = 0; i < points.length; i++) {
    const [x,y] = points[i];
    let p = state.particles[i];
    if (!p) {
      p = new Particle();
      state.particles.push(p);
    }
    p.moveTo(x, y);
  }

  if(points.length < state.particles.length) {
    state.particles.splice(points.length);
  }
}

const getPoints = () => {
  //拿到像素点信息
  const points = [];

  const {data} = ctx.getImageData(0, 0, canvasRef.value.width, canvasRef.value.height);

  const gap = 6;

  for (let i = 0; i < canvasRef.value.width; i += gap) {
    for (let j = 0; j < canvasRef.value.height; j++) {
      const index = (i + j * canvasRef.value.width) * 4;
      const r = data[index];
      const g = data[index + 1];
      const b = data[index + 2];
      const a = data[index + 3];
      if (r === 0 && g === 0 && b === 0 && a === 255) {
        points.push([i, j])
      }
    }
  }
  return points;
}

// 修改后的动画循环
const animate = () => {

  const currentText = getText(); // 获取当前时间

  if (currentText === state.text) {
    return;
  }

  state.text = currentText;

  clear(ctx);
  drawText(ctx);
  drawParticles(ctx);
};

const getText = () => {
  return new Date().toLocaleTimeString();
}

let ctx;

onMounted(() => {

  initCanvasSize();

  ctx = canvasRef.value.getContext('2d', {willReadFrequently: true});

  for (let i = 0; i < 1000; i++) {
    state.particles.push(new Particle());
  }

  setInterval(() => {
    animate();
  }, 1000);

  //animationFrameId = requestAnimationFrame(animate);
});

onUnmounted(() => {
  //cancelAnimationFrame(animationFrameId);
});

</script>

<style scoped>
.particle-clock-container {
  width: 100%;
  height: 100%;
}

.particle-clock-container-canvas {
  width: 100%;
  height: 100%;
  display: block;
  background: radial-gradient(#fff, #8c738c);
}
</style>
