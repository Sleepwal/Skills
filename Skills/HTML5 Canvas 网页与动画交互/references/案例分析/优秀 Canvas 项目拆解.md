# 优秀 Canvas 项目拆解

## 1. 粒子交互背景

### 项目特征
- 全屏 Canvas 背景
- 粒子随机运动并相互连线
- 鼠标交互影响粒子

### 技术拆解
```javascript
class ParticleNetwork {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.particles = [];
    this.maxDistance = 150;
    this.mouse = { x: null, y: null, radius: 200 };
    
    this.setup();
    this.setupEvents();
    this.animate();
  }
  
  setup() {
    this.resize();
    const count = Math.floor((this.width * this.height) / 10000);
    
    for (let i = 0; i < count; i++) {
      this.particles.push({
        x: Math.random() * this.width,
        y: Math.random() * this.height,
        vx: (Math.random() - 0.5) * 1,
        vy: (Math.random() - 0.5) * 1,
        size: Math.random() * 2 + 1
      });
    }
  }
  
  animate() {
    this.ctx.clearRect(0, 0, this.width, this.height);
    
    // 更新粒子位置
    this.particles.forEach(p => {
      p.x += p.vx;
      p.y += p.vy;
      
      // 边界检测
      if (p.x < 0 || p.x > this.width) p.vx *= -1;
      if (p.y < 0 || p.y > this.height) p.vy *= -1;
      
      // 鼠标交互
      if (this.mouse.x !== null) {
        const dx = this.mouse.x - p.x;
        const dy = this.mouse.y - p.y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        
        if (dist < this.mouse.radius) {
          const force = (this.mouse.radius - dist) / this.mouse.radius;
          p.x -= dx * force * 0.02;
          p.y -= dy * force * 0.02;
        }
      }
    });
    
    // 绘制连线
    for (let i = 0; i < this.particles.length; i++) {
      for (let j = i + 1; j < this.particles.length; j++) {
        const dx = this.particles[i].x - this.particles[j].x;
        const dy = this.particles[i].y - this.particles[j].y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        
        if (dist < this.maxDistance) {
          const alpha = 1 - dist / this.maxDistance;
          this.ctx.beginPath();
          this.ctx.strokeStyle = `rgba(100, 150, 255, ${alpha * 0.5})`;
          this.ctx.lineWidth = 1;
          this.ctx.moveTo(this.particles[i].x, this.particles[i].y);
          this.ctx.lineTo(this.particles[j].x, this.particles[j].y);
          this.ctx.stroke();
        }
      }
    }
    
    // 绘制粒子
    this.particles.forEach(p => {
      this.ctx.beginPath();
      this.ctx.fillStyle = '#6496ff';
      this.ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
      this.ctx.fill();
    });
    
    requestAnimationFrame(() => this.animate());
  }
  
  setupEvents() {
    window.addEventListener('resize', () => this.resize());
    
    this.canvas.addEventListener('mousemove', (e) => {
      const rect = this.canvas.getBoundingClientRect();
      this.mouse.x = e.clientX - rect.left;
      this.mouse.y = e.clientY - rect.top;
    });
    
    this.canvas.addEventListener('mouseleave', () => {
      this.mouse.x = null;
      this.mouse.y = null;
    });
  }
  
  resize() {
    this.width = window.innerWidth;
    this.height = window.innerHeight;
    this.canvas.width = this.width;
    this.canvas.height = this.height;
  }
}
```

### 关键学习点
- 粒子间距离计算与连线
- 鼠标排斥力实现
- O(n²) 优化（空间分区）
- 响应式 Canvas 适配

---

## 2. 音频可视化器

### 项目特征
- 实时音频频率分析
- 频谱柱状图/波形图
- 动态颜色变化

### 技术拆解
```javascript
class AudioVisualizer {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.audioContext = null;
    this.analyser = null;
    this.dataArray = null;
    
    this.setup();
  }
  
  async setup() {
    this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
    this.analyser = this.audioContext.createAnalyser();
    this.analyser.fftSize = 256;
    
    const bufferLength = this.analyser.frequencyBinCount;
    this.dataArray = new Uint8Array(bufferLength);
    
    // 获取音频源（麦克风/文件）
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    const source = this.audioContext.createMediaStreamSource(stream);
    source.connect(this.analyser);
    
    this.resize();
    this.animate();
  }
  
  animate() {
    this.analyser.getByteFrequencyData(this.dataArray);
    
    this.ctx.fillStyle = 'rgba(0, 0, 0, 0.2)';
    this.ctx.fillRect(0, 0, this.width, this.height);
    
    const barWidth = (this.width / this.dataArray.length) * 2.5;
    let x = 0;
    
    for (let i = 0; i < this.dataArray.length; i++) {
      const barHeight = (this.dataArray[i] / 255) * this.height;
      
      // 渐变色
      const gradient = this.ctx.createLinearGradient(0, this.height, 0, this.height - barHeight);
      gradient.addColorStop(0, '#ff0066');
      gradient.addColorStop(0.5, '#ff6600');
      gradient.addColorStop(1, '#ffcc00');
      
      this.ctx.fillStyle = gradient;
      this.ctx.fillRect(x, this.height - barHeight, barWidth, barHeight);
      
      x += barWidth + 1;
    }
    
    requestAnimationFrame(() => this.animate());
  }
  
  resize() {
    this.width = this.canvas.width = window.innerWidth;
    this.height = this.canvas.height = window.innerHeight;
  }
}
```

### 关键学习点
- Web Audio API 集成
- 频率数据分析
- 渐变柱状图绘制
- 拖尾效果实现（半透明覆盖）

---

## 3. 交互式星空

### 项目特征
- 3D 透视星空效果
- 鼠标控制视角
- 星星闪烁动画

### 技术拆解
```javascript
class StarField {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.stars = [];
    this.count = 800;
    this.speed = 2;
    this.mouse = { x: 0, y: 0 };
    
    this.init();
    this.setupEvents();
    this.animate();
  }
  
  init() {
    this.resize();
    
    for (let i = 0; i < this.count; i++) {
      this.stars.push({
        x: (Math.random() - 0.5) * this.width * 2,
        y: (Math.random() - 0.5) * this.height * 2,
        z: Math.random() * this.width,
        pz: 0, // 上一帧的 z
        twinkle: Math.random() * Math.PI * 2,
        twinkleSpeed: Math.random() * 0.02 + 0.01
      });
    }
  }
  
  animate() {
    this.ctx.fillStyle = '#000000';
    this.ctx.fillRect(0, 0, this.width, this.height);
    
    const centerX = this.width / 2 + this.mouse.x;
    const centerY = this.height / 2 + this.mouse.y;
    
    this.stars.forEach(star => {
      star.pz = star.z;
      star.z -= this.speed;
      star.twinkle += star.twinkleSpeed;
      
      if (star.z <= 0) {
        star.z = this.width;
        star.pz = star.z;
      }
      
      // 3D 透视投影
      const x = (star.x / star.z) * 300 + centerX;
      const y = (star.y / star.z) * 300 + centerY;
      
      const px = (star.x / star.pz) * 300 + centerX;
      const py = (star.y / star.pz) * 300 + centerY;
      
      const radius = Math.max(0.5, (1 - star.z / this.width) * 3);
      const alpha = 0.5 + 0.5 * Math.sin(star.twinkle);
      
      // 绘制星轨
      this.ctx.beginPath();
      this.ctx.strokeStyle = `rgba(255, 255, 255, ${alpha * 0.3})`;
      this.ctx.lineWidth = radius * 0.5;
      this.ctx.moveTo(px, py);
      this.ctx.lineTo(x, y);
      this.ctx.stroke();
      
      // 绘制星星
      this.ctx.beginPath();
      this.ctx.fillStyle = `rgba(255, 255, 255, ${alpha})`;
      this.ctx.arc(x, y, radius, 0, Math.PI * 2);
      this.ctx.fill();
    });
    
    requestAnimationFrame(() => this.animate());
  }
  
  setupEvents() {
    window.addEventListener('resize', () => this.resize());
    
    this.canvas.addEventListener('mousemove', (e) => {
      this.mouse.x = (e.clientX - this.width / 2) * 0.1;
      this.mouse.y = (e.clientY - this.height / 2) * 0.1;
    });
  }
  
  resize() {
    this.width = this.canvas.width = window.innerWidth;
    this.height = this.canvas.height = window.innerHeight;
  }
}
```

### 关键学习点
- 3D 透视投影公式
- 星轨拖尾效果
- 正弦函数闪烁动画
- 鼠标控制视角

---

## 4. 流体模拟

### 项目特征
- 基于网格的流体模拟
- 鼠标添加染料和力
- 实时渲染

### 技术拆解要点
```javascript
// 简化版流体模拟
class FluidSimulation {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.resolution = 128; // 网格分辨率
    this.iterations = 4;
    
    this.size = this.resolution * this.resolution;
    this.density = new Float32Array(this.size);
    this.velocityX = new Float32Array(this.size);
    this.velocityY = new Float32Array(this.size);
    
    this.setupEvents();
    this.animate();
  }
  
  // 添加染料
  addDensity(x, y, amount) {
    const i = Math.floor(x * this.resolution);
    const j = Math.floor(y * this.resolution);
    const index = i + j * this.resolution;
    this.density[index] += amount;
  }
  
  // 添加力
  addForce(x, y, fx, fy) {
    const i = Math.floor(x * this.resolution);
    const j = Math.floor(y * this.resolution);
    const index = i + j * this.resolution;
    this.velocityX[index] += fx;
    this.velocityY[index] += fy;
  }
  
  // 扩散步骤
  diffuse(b, x, x0, diff, dt) {
    const a = dt * diff * this.resolution * this.resolution;
    this.linSolve(b, x, x0, a, 1 + 4 * a);
  }
  
  // 线性方程求解
  linSolve(b, x, x0, a, c) {
    const reciprocal = 1.0 / c;
    for (let k = 0; k < this.iterations; k++) {
      for (let j = 1; j < this.resolution - 1; j++) {
        for (let i = 1; i < this.resolution - 1; i++) {
          x[this.IX(i, j)] = (
            x0[this.IX(i, j)] +
            a * (
              x[this.IX(i + 1, j)] +
              x[this.IX(i - 1, j)] +
              x[this.IX(i, j + 1)] +
              x[this.IX(i, j - 1)]
            )
          ) * reciprocal;
        }
      }
    }
  }
  
  IX(i, j) {
    return i + j * this.resolution;
  }
  
  // 渲染密度场
  render() {
    const imageData = this.ctx.createImageData(this.resolution, this.resolution);
    const data = imageData.data;
    
    for (let i = 0; i < this.size; i++) {
      const density = this.density[i];
      const index = i * 4;
      
      data[index] = Math.floor(density * 255);
      data[index + 1] = Math.floor(density * 200);
      data[index + 2] = Math.floor(density * 255);
      data[index + 3] = 255;
    }
    
    this.ctx.putImageData(imageData, 0, 0);
    this.ctx.drawImage(this.canvas, 0, 0, this.resolution, this.resolution, 0, 0, this.width, this.height);
  }
  
  animate() {
    // 模拟步骤
    this.densityStep();
    this.velocityStep();
    
    // 渲染
    this.render();
    
    requestAnimationFrame(() => this.animate());
  }
}
```

### 关键学习点
- Navier-Stokes 方程简化
- 网格-based 流体模拟
- 扩散、平流、投影步骤
- 密度场可视化

---

## 5. 数据可视化仪表板

### 项目特征
- 多种图表类型组合
- 实时数据更新
- 交互式tooltip

### 技术拆解要点
```javascript
class Dashboard {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.charts = [];
    this.tooltip = null;
    this.hoveredChart = null;
    
    this.initCharts();
    this.setupEvents();
    this.animate();
  }
  
  initCharts() {
    // 折线图
    this.charts.push(new LineChart({
      x: 20, y: 20,
      width: 400, height: 200,
      data: this.generateTimeSeriesData(100)
    }));
    
    // 柱状图
    this.charts.push(new BarChart({
      x: 440, y: 20,
      width: 400, height: 200,
      data: this.generateCategoryData(10)
    }));
    
    // 饼图
    this.charts.push(new PieChart({
      x: 20, y: 240,
      width: 200, height: 200,
      data: this.generatePieData()
    }));
  }
  
  animate() {
    this.ctx.clearRect(0, 0, this.width, this.height);
    
    // 绘制背景
    this.ctx.fillStyle = '#f5f5f5';
    this.ctx.fillRect(0, 0, this.width, this.height);
    
    // 绘制图表
    this.charts.forEach(chart => chart.render(this.ctx));
    
    // 绘制tooltip
    if (this.tooltip) {
      this.renderTooltip();
    }
    
    requestAnimationFrame(() => this.animate());
  }
  
  setupEvents() {
    this.canvas.addEventListener('mousemove', (e) => {
      const pos = this.getCanvasPosition(e);
      
      this.hoveredChart = this.charts.find(chart => chart.isPointInside(pos));
      this.canvas.style.cursor = this.hoveredChart ? 'pointer' : 'default';
      
      if (this.hoveredChart) {
        this.tooltip = this.hoveredChart.getTooltipAt(pos);
      } else {
        this.tooltip = null;
      }
    });
  }
  
  renderTooltip() {
    this.ctx.save();
    this.ctx.fillStyle = 'rgba(0, 0, 0, 0.8)';
    this.ctx.fillRect(this.tooltip.x, this.tooltip.y - 30, 150, 25);
    
    this.ctx.fillStyle = '#ffffff';
    this.ctx.font = '12px sans-serif';
    this.ctx.fillText(this.tooltip.text, this.tooltip.x + 5, this.tooltip.y - 13);
    this.ctx.restore();
  }
}
```

### 关键学习点
- 多图表布局
- 命中检测与交互
- 实时数据更新
- Tooltip 渲染

---

## 共同设计模式

### 1. 动画循环
```javascript
class CanvasApp {
  animate() {
    this.update();
    this.render();
    requestAnimationFrame(() => this.animate());
  }
}
```

### 2. 响应式适配
```javascript
resize() {
  const dpr = window.devicePixelRatio || 1;
  this.canvas.width = this.width * dpr;
  this.canvas.height = this.height * dpr;
  this.canvas.style.width = this.width + 'px';
  this.canvas.style.height = this.height + 'px';
  this.ctx.scale(dpr, dpr);
}
```

### 3. 事件封装
```javascript
getCanvasPosition(event) {
  const rect = this.canvas.getBoundingClientRect();
  return {
    x: event.clientX - rect.left,
    y: event.clientY - rect.top
  };
}
```

### 4. 对象池
```javascript
class ObjectPool {
  constructor(factory, size) {
    this.pool = [];
    this.active = [];
    
    for (let i = 0; i < size; i++) {
      this.pool.push(factory());
    }
  }
  
  acquire() {
    return this.pool.pop() || this.factory();
  }
  
  release(obj) {
    this.pool.push(obj);
  }
}
```

## 学习建议

1. **从简单开始**：先实现基础功能，再添加特效
2. **性能优先**：保持 60fps，必要时降低质量
3. **模块化设计**：分离渲染、逻辑、输入
4. **真实项目**：模仿优秀案例，逐步提高复杂度
5. **性能分析**：使用 DevTools Profiler 优化瓶颈
