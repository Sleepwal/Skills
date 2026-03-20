# CSS 技巧

本文档提供高级 CSS 技巧，增强前端美学实现。

## 现代 CSS 功能

### 1. CSS 容器查询
```css
/* 定义容器 */
.card-container {
  container-type: inline-size;
  container-name: card;
}

/* 容器内响应式 */
@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 2fr;
  }
}
```

### 2. :has() 伪类
```css
/* 父元素样式 */
.form:has(.input.error) .submit-btn {
  opacity: 0.5;
  pointer-events: none;
}

/* 条件样式 */
.card:has(.badge) {
  padding-top: 2rem;
}
```

### 3. @layer 层级
```css
/* 明确层叠顺序 */
@layer reset, base, components, utilities;

@layer reset {
  * { margin: 0; padding: 0; box-sizing: border-box; }
}

@layer base {
  body { font-family: var(--font-body); }
}

@layer components {
  .btn { padding: 1rem 2rem; }
}
```

### 4. color-mix() 函数
```css
/* 混合颜色 */
.btn-primary {
  background: color-mix(in srgb, var(--color-primary) 80%, white);
}

.btn-secondary {
  background: color-mix(in srgb, var(--color-primary) 80%, black);
}
```

### 5. 容器混合模式
```css
.text-on-image {
  position: relative;
}

.text-on-image::before {
  content: '';
  position: absolute;
  inset: 0;
  background: var(--color-overlay);
  mix-blend-mode: multiply;
}
```

## 创意背景效果

### 1. 多重渐变叠加
```css
.hero-bg {
  background:
    radial-gradient(ellipse at 20% 80%, var(--color-accent) 0%, transparent 50%),
    radial-gradient(ellipse at 80% 20%, var(--color-secondary) 0%, transparent 50%),
    linear-gradient(180deg, var(--color-bg) 0%, var(--color-bg-dark) 100%);
}
```

### 2. 网格图案
```css
.grid-pattern {
  background-image:
    linear-gradient(var(--color-line) 1px, transparent 1px),
    linear-gradient(90deg, var(--color-line) 1px, transparent 1px);
  background-size: 60px 60px;
}
```

### 3. 圆点图案
```css
.dot-pattern {
  background-image: radial-gradient(
    circle,
    var(--color-dot) 1px,
    transparent 1px
  );
  background-size: 24px 24px;
}
```

### 4. SVG 噪点纹理
```css
.noise {
  position: relative;
}

.noise::after {
  content: '';
  position: absolute;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)'/%3E%3C/svg%3E");
  opacity: 0.04;
  pointer-events: none;
}
```

## 创意边框和轮廓

### 1. 不规则边框
```css
.skewed-border {
  position: relative;
}

.skewed-border::before {
  content: '';
  position: absolute;
  inset: 0;
  background: var(--color-accent);
  transform: skewX(-3deg);
  z-index: -1;
}
```

### 2. 渐变边框
```css
.gradient-border {
  position: relative;
  background: var(--color-bg);
}

.gradient-border::before {
  content: '';
  position: absolute;
  inset: -2px;
  background: linear-gradient(
    45deg,
    var(--color-1),
    var(--color-2),
    var(--color-3)
  );
  border-radius: inherit;
  z-index: -1;
}
```

### 3. 阴影轮廓
```css
.shadow-outline {
  box-shadow:
    0 0 0 1px var(--color-outline),
    4px 4px 0 0 var(--color-outline);
}
```

## 排版高级技巧

### 1. 文字环绕
```css
.float-text {
  shape-outside: circle(50%);
  float: left;
}
```

### 2. 首字下沉
```css
.drop-cap::first-letter {
  float: left;
  font-size: 4rem;
  line-height: 0.8;
  padding-right: 0.5rem;
  font-family: var(--font-display);
  color: var(--color-accent);
}
```

### 3. 文本渐变
```css
.gradient-text {
  background: linear-gradient(
    135deg,
    var(--color-primary),
    var(--color-secondary)
  );
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
```

### 4. 文字描边
```css
.outline-text {
  color: transparent;
  -webkit-text-stroke: 2px var(--color-text);
}
```

## 滚动条样式
```css
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: var(--color-bg);
}

::-webkit-scrollbar-thumb {
  background: var(--color-primary);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: var(--color-accent);
}
```

## 选择文本样式
```css
::selection {
  background: var(--color-accent);
  color: var(--color-bg);
}
```

## 平滑滚动
```css
html {
  scroll-behavior: smooth;
}

/* 分段平滑滚动 */
.scroll-container {
  scroll-snap-type: y mandatory;
}

.scroll-section {
  scroll-snap-align: start;
}
```

## 焦点样式
```css
:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 2px;
}

/* 移除默认焦点环但保留可访问性 */
:focus:not(:focus-visible) {
  outline: none;
}
```