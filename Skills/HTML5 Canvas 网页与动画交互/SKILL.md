---
name: "HTML5 Canvas 网页与动画交互"
description: "专精于 HTML5 Canvas 网页与动画交互开发。创建高性能、视觉丰富的 Canvas 应用，包括粒子系统、数据可视化、交互式动画、游戏渲染等。当用户需要创建、修改或优化 Canvas 网页时调用此技能。"
---

# HTML5 Canvas 网页与动画交互

本技能专注于 HTML5 Canvas API 的高性能网页与动画交互开发。创建流畅、视觉震撼的 Canvas 应用，涵盖粒子系统、数据可视化、物理模拟、交互式动画和 2D 游戏渲染等场景。

## 使用场景

当用户提供以下需求时调用此技能：
- Canvas 网页/应用创建或开发
- Canvas 动画与交互优化
- 粒子系统、物理模拟实现
- 数据可视化图表绘制
- Canvas 性能调优（帧率、内存、渲染效率）
- Canvas 与 DOM/SVG/WebGL 混合使用
- Canvas 响应式适配与高分屏支持

## 核心能力

### Canvas 基础
- Canvas 元素初始化与上下文获取（2D Context）
- 坐标系与变换（translate, rotate, scale, transform）
- 路径绘制（线条、曲线、贝塞尔曲线、圆弧）
- 形状绘制（矩形、多边形、自定义形状）
- 颜色与渐变（线性渐变、径向渐变）
- 图案与纹理（createPattern, 图像平铺）
- 阴影与模糊效果
- 合成模式（globalCompositeOperation）
- 裁剪与蒙版（clip）
- 像素级操作（getImageData, putImageData）
- 高分辨率屏幕适配（devicePixelRatio）

### 动画系统
- requestAnimationFrame 动画循环
- 时间增量计算（delta time）
- 缓动函数（easing functions）
  - 线性、二次、三次、正弦、指数、弹性
- 关键帧动画
- 状态机驱动的动画控制
- 动画暂停/恢复/倒带
- 插值动画（lerp, slerp）

### 粒子系统
- 粒子生命周期管理
- 粒子发射器（Emitter）设计
- 粒子属性（位置、速度、加速度、颜色、大小、透明度）
- 力的模拟（重力、风力、摩擦力、浮力）
- 粒子池与对象复用
- 粒子混合模式（additive, multiply, screen）
- 常见效果：火焰、烟雾、雨雪、爆炸、火花、星星、光晕

### 物理模拟
- 碰撞检测（AABB、圆形、多边形）
- 碰撞响应与反弹
- 弹性力学（弹簧、胡克定律）
- 刚体运动（速度、加速度、角速度）
- 约束系统（距离约束、角度约束）
- 流体模拟基础
- 布料模拟（Verlet 积分）
- 引力与轨道模拟

### 数据可视化
- 折线图、面积图
- 柱状图、条形图
- 饼图、环形图、玫瑰图
- 散点图、气泡图
- 雷达图、极坐标图
- 树状图（Treemap）
- 热力图、等高线图
- 网络图、力导向图
- 地图与地理数据可视化
- 自定义图表类型

### 交互处理
- 鼠标事件（点击、拖拽、悬停、滚轮）
- 触摸事件（移动端支持）
- 键盘事件
- 坐标转换（屏幕坐标 → Canvas 坐标）
- 命中检测（point-in-shape, 像素级检测）
- 手势识别
- 放大镜/缩放功能
- 拖放与画布平移

### 性能优化
- 离屏 Canvas 缓存（Offscreen Canvas）
- 分层渲染（静态层 + 动态层）
- 脏矩形渲染（Dirty Rectangles）
- 视口裁剪（只渲染可见区域）
- 对象池与内存复用
- 减少状态切换（批量绘制相同样式）
- 避免频繁 allocation（对象创建/销毁）
- 降采样与分辨率自适应
- Web Worker 辅助计算
- 帧率监控与性能分析

### 高级特性
- 文本渲染与字体处理
- 图像加载与绘制（drawImage）
- Canvas 导出（toDataURL, toBlob）
- Canvas 滤镜（filter 属性）
- 路径命中测试（isPointInPath, isPointInStroke）
- Canvas 与 WebGL 混合渲染
- Canvas 录屏与动画导出（MediaRecorder, gif.js）
- 响应式 Canvas 适配

## 架构模式

### 推荐架构
```
CanvasApplication
├── Renderer（渲染器）
│   ├── Canvas 上下文管理
│   ├── 高分屏适配
│   └── 渲染循环
├── Scene（场景）
│   ├── 实体列表
│   ├── 相机/视口
│   └── 层级管理
├── EntityManager（实体管理）
│   ├── 实体创建/销毁
│   ├── 组件系统（ECS）
│   └── 空间分区
├── InputHandler（输入处理）
│   ├── 鼠标/触摸事件
│   ├── 键盘事件
│   └── 手势识别
├── PhysicsEngine（物理引擎）[可选]
│   ├── 碰撞检测
│   ├── 力的模拟
│   └── 约束求解
└── AnimationController（动画控制器）
    ├── 动画状态
    ├── 缓动函数
    └── 时间管理
```

### 设计模式
- **游戏循环模式**：update + render 分离
- **对象池模式**：粒子/实体复用
- **策略模式**：不同渲染策略切换
- **观察者模式**：事件驱动交互
- **状态模式**：动画/应用状态管理
- **ECS（实体-组件-系统）**：复杂场景管理

## 实现要求

- **性能优先**：稳定 60fps，复杂场景 30fps 以上
- **代码模块化**：清晰分离渲染、逻辑、输入
- **响应式设计**：自适应屏幕尺寸和 DPI
- **可访问性**：提供 DOM fallback 或 ARIA 标签
- **浏览器兼容**：支持现代浏览器（Chrome, Firefox, Safari, Edge）
- **内存管理**：避免内存泄漏，合理释放资源
- **可维护性**：注释清晰，结构合理

## 代码规范

### Canvas 初始化
```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

// 高分屏适配
const dpr = window.devicePixelRatio || 1;
canvas.width = width * dpr;
canvas.height = height * dpr;
canvas.style.width = width + 'px';
canvas.style.height = height + 'px';
ctx.scale(dpr, dpr);
```

### 动画循环
```javascript
let lastTime = 0;

function animate(currentTime) {
  const deltaTime = (currentTime - lastTime) / 1000;
  lastTime = currentTime;
  
  update(deltaTime);
  render();
  
  requestAnimationFrame(animate);
}

requestAnimationFrame(animate);
```

### 实体类结构
```javascript
class Entity {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.vx = 0;
    this.vy = 0;
    this.rotation = 0;
    this.scale = 1;
    this.opacity = 1;
    this.active = true;
  }
  
  update(dt) {
    this.x += this.vx * dt;
    this.y += this.vy * dt;
  }
  
  render(ctx) {
    ctx.save();
    ctx.translate(this.x, this.y);
    ctx.rotate(this.rotation);
    ctx.scale(this.scale, this.scale);
    ctx.globalAlpha = this.opacity;
    // 绘制逻辑
    ctx.restore();
  }
}
```

## 常见应用场景

### 创意交互
- 鼠标跟随效果
- 粒子互动背景
- 流体/波浪效果
- 星空/宇宙效果
- 抽象艺术生成器
- 音频可视化（Web Audio API 集成）

### 数据可视化仪表板
- 实时数据监控面板
- 股票/金融图表
- 网络流量可视化
- 地理信息地图
- 统计分析报告

### 游戏开发
- 2D 游戏引擎
- 平台跳跃游戏
- 射击游戏
- 益智游戏
- 物理沙盒模拟

### 教育工具
- 数学函数可视化
- 物理实验模拟
- 几何图形教学
- 算法可视化
- 分形与混沌理论

### UI 增强
- 自定义图表组件
- 动画背景
- 加载动画/进度指示器
- 交互式教程
- 签名/绘图工具

## 技术选型指南

### 何时使用 Canvas
- 需要绘制大量动态图形（1000+ 对象）
- 需要像素级控制
- 复杂动画/粒子系统
- 游戏/实时渲染

### 何时使用 SVG/DOM
- 静态或半静态图形
- 需要 SEO/可访问性
- 简单图标/图表
- 需要 CSS 样式控制

### 何时使用 WebGL
- 3D 渲染
- 超大规模粒子（10000+）
- GPU 计算加速
- 复杂着色器效果

## 工具与库推荐

### 原生 Canvas API
优先使用原生 API，减少依赖，提高性能和控制力。

### 辅助库（按需选用）
- **PixiJS**：高性能 2D 渲染引擎（WebGL fallback）
- **Konva.js**：Canvas 交互库，适合应用开发
- **Fabric.js**：Canvas 对象模型，适合编辑器
- **p5.js**：创意编码库，适合艺术/教育
- **Three.js**：3D 渲染（WebGL）
- **D3.js**：数据可视化（可结合 Canvas）
- **Chart.js**：图表库（Canvas 渲染）

## 输出格式

完成开发后，提供：
1. **架构说明**（模块划分与数据流）
2. **完整的、可工作的代码实现**（HTML + CSS + JS）
3. **性能优化说明**（如适用）
4. **使用示例与 API 文档**（如适用）
5. **浏览器兼容性说明**（如使用了高级特性）

## 参考资料

参考资料目录 `references/` 包含可选的参考文档和资源文件，用于增强技能的细节层次，并为实现提供额外的上下文或支持信息。

### 目录结构（待创建）

```
references/
├── 性能优化/
│   ├── 渲染优化技巧.md
│   └── 内存管理最佳实践.md
├── 动画与物理/
│   ├── 缓动函数参考.md
│   ├── 物理模拟公式.md
│   └── 粒子系统设计模式.md
├── 数据可视化/
│   ├── 图表类型选择指南.md
│   └── 色彩与数据可视化.md
├── 交互模式/
│   ├── 鼠标交互处理.md
│   └── 触摸与移动端适配.md
└── 案例分析/
    ├── 优秀 Canvas 项目拆解.md
    └── 常见问题与解决方案.md
```

### 使用方式

- 查阅 `references/` 目录获取技术参考和最佳实践
- 参考文档提供更深入的实现细节和性能优化技巧
- 案例分析帮助理解架构设计和交互设计原则
