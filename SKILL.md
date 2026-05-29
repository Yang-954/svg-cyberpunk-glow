---
name: svg-cyberpunk-glow
description: 生成带赛博朋克动态流光效果的 SVG 图标/图形。采用 Mask + ForeignObject 解耦架构，支持中心正反转、外部广角扫射等多种动画模式。适用于项目看板、仪表盘、科技风 UI 中的图标动效。
triggers:
  - 赛博朋克风格 SVG
  - SVG 流光效果
  - 发光边框动画
  - 科技感图标动效
  - cyberpunk glow SVG
  - 霓虹旋转光效
  - mask foreignObject conic-gradient 动画
agent_created: true
---

# SVG 赛博朋克动态流光效果

## 什么时候用

用户需要制作带动态旋转流光/扫光效果的 SVG 图形时使用此技能。典型场景：
- 项目进度看板中的状态图标（科技风）
- 仪表盘/大屏可视化中的动态装饰图形
- 需要"活"的、有动感的 SVG，而不是静态矢量图

## 核心架构

```
SVG Mask（遮罩层） ── 控制形状 & 透光率
       +
CSS ForeignObject（发光层） ── 控制色彩 & 运动
```

- **SVG Mask**：`#FFFFFF` = 透光，`#000000` = 遮光。不要用 `linearGradient` 填充路径。
- **CSS ForeignObject**：内嵌一个 `<div>` + `conic-gradient` 圆盘，用 CSS `@keyframes` 旋转。

---

## AI 工作流（必须按顺序执行）

### Step 1: 确定图形需求

向用户确认或自行判断：
- 需要什么形状？（齿轮、盾牌、六边形、环形、自定义 Path 等）
- 需要几层流光？（单层 / 双层正反转 / 更多层）
- 动画风格？（中心旋转 / 外部扫射 / 混合模式）

### Step 2: 选择动画模式

根据用户意图选择模式，参考下表：

| 模式 | CSS 类名 | 效果描述 | 适用场景 |
|------|----------|----------|----------|
| 中心正转 | `mode-center-forward` | 从图形中心自转，150% 大小 | 单层流光 |
| 中心反转 | `mode-center-reverse` | 从图形中心反转，150% 大小 | 双层流光的内层 |
| 外部扫射 | `mode-sweep-external` | 500% 大盘偏心旋转，探照灯效果 | 需要"扫过"而非"旋转"的视觉 |

**双层常用组合**：外层 `mode-center-forward`（4.2s） + 内层 `mode-center-reverse`（5.0s），形成机械咬合感。

**扫射模式**：发光盘 500% 放大 + `top: -50%; left: -50%` + `translate(-50%, -50%) rotate(...)`，旋转轴心移出可视区。

### Step 3: 构造 SVG Path / Shape

- 简单图形（圆形、矩形）：直接使用 SVG 原生元素
- 复杂图形：使用 `<path>` 元素，可借助 `M L C Q A Z` 命令
- **关键**：Mask 中的形状必须 `stroke="#FFFFFF"` 或 `fill="#FFFFFF"`

如果需要遮挡镂空区域，用 `fill="#000000"` 的路径叠在上面，必要时加 `stroke="#000000"` + `stroke-width="X"` 封堵缝隙。

**多块拼合图形处理**（重要）：当图形由多个不连通的 Path 组成（如多个字母、散落几何块），推荐以下双层 Mask 策略：

*外层 Mask（轮廓 + 缝隙发光）—— 正转*：
1. 用图形整体的**外轮廓 Path**（`fill-rule="evenodd"`）作为白色透光基底
2. 将每个子块的 Path 以**黑色**（`fill="#000000" stroke="#000000" stroke-width="10~14" stroke-linejoin="round"`）叠在上面挖空内部
3. 必要时加一个大的黑色 polygon 覆盖中间过渡区域
4. 结果：只有轮廓边缘和多块之间的**缝隙**透光，形成干净的分割线光效

*内层 Mask（主体填充发光）—— 反转*：
1. 将每个子块的 Path 以白色 `fill="#FFFFFF"` 填入
2. 结果：各块内部透光，与外层缝隙光形成对比

这种策略的优势：内部缝隙是有意被挖出来的独立发光通道，不会被主体流光吞没，层次分明。

### Step 4: 套用代码模板

直接使用下方模板，将 `<mask>` 中的 Path 替换为实际图形路径。

### Step 5: 输出完整可运行 HTML

最终产物必须是一个**完整的、可直接在浏览器打开的 HTML 文件**，包含：
- 完整的 `<svg>` 标签
- 所有 CSS 样式（内嵌在 `<defs><style>` 中）
- 至少一个实际可用的图形路径
- 不要留占位符

---

## 标准化代码模板

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300" viewBox="0 0 300 300">
  <defs>
    <style>
      .spin-container {
        width: 100%; height: 100%;
        position: relative; overflow: hidden;
      }
      .glow-source {
        position: absolute;
        border-radius: 50%;
        background: conic-gradient(from 0deg, #ff0055, #ff6600, #ffcc00, #00ff66, #00ccff, #9933ff, #ff0055);
      }

      /* 方案A：中心正转 */
      .mode-center-forward {
        width: 150%; height: 150%;
        top: -25%; left: -25%;
        animation: rot-fwd 4.2s linear infinite;
      }
      /* 方案B：中心反转 */
      .mode-center-reverse {
        width: 150%; height: 150%;
        top: -25%; left: -25%;
        animation: rot-rev 5.0s linear infinite;
      }
      /* 方案C：外部广角扫射 */
      .mode-sweep-external {
        width: 500%; height: 500%;
        top: -50%; left: -50%;
        animation: rot-sweep 4.2s linear infinite;
      }

      @keyframes rot-fwd {
        0%   { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
      }
      @keyframes rot-rev {
        0%   { transform: rotate(360deg); }
        100% { transform: rotate(0deg); }
      }
      @keyframes rot-sweep {
        0%   { transform: translate(-50%, -50%) rotate(0deg); }
        100% { transform: translate(-50%, -50%) rotate(360deg); }
      }
    </style>

    <mask id="layer-1">
      <!-- 这里放实际的 SVG 图形，fill/stroke 必须是 #FFFFFF -->
    </mask>
  </defs>

  <g mask="url(#layer-1)">
    <foreignObject x="0" y="0" width="300" height="300">
      <div xmlns="http://www.w3.org/1999/xhtml" class="spin-container">
        <div class="glow-source mode-center-forward"></div>
      </div>
    </foreignObject>
  </g>
</svg>
```

**模板使用时的关键检查清单**：
- [ ] `xmlns` 值不含方括号 `[]`
- [ ] `<mask>` 标签拼写正确（不是 `<mask_>`）
- [ ] `foreignObject` 的 `width/height` 用绝对值（匹配 `viewBox`），不用百分比
- [ ] `foreignObject` 内只有一个 HTML `<div xmlns="http://www.w3.org/1999/xhtml">` 根元素
- [ ] Mask 中透光部分是 `#FFFFFF`，遮光部分是 `#000000`
- [ ] 如需双层，用不同的 `mask id` 和不同的 `mode-*` 类名

---

## 完整示例：多块几何标识双层流光

以下是一个可直接保存为 `.html` 打开查看的完整示例，展示了一个由三块几何图形拼成的 Z 形标识。
**重点演示了外层 Mask 的「轮廓透光 + 黑色路径挖空内部 + 宽 stroke 制造缝隙」策略**。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>Cyberpunk Z Logo Glow</title>
<style>
  body { margin:0; padding:0; background:#0a0a0f;
    display:flex; justify-content:center; align-items:center; min-height:100vh; }
</style>
</head>
<body>
<svg xmlns="http://www.w3.org/2000/svg" width="330" height="327" viewBox="0 0 330.276 327.063">
  <defs>
    <style>
      .spin-container { width:100%; height:100%; display:flex;
        justify-content:center; align-items:center; position:relative; overflow:hidden; }
      .spin-glow-forward { width:150%; height:150%; position:absolute; border-radius:50%;
        background:conic-gradient(from 0deg,#ff0055,#ff6600,#ffcc00,#00ff66,#00ccff,#9933ff,#ff0055);
        animation:rotate-forward 4.2s linear infinite; }
      .spin-glow-reverse { width:150%; height:150%; position:absolute; border-radius:50%;
        background:conic-gradient(from 0deg,#ff0055,#ff6600,#ffcc00,#00ff66,#00ccff,#9933ff,#ff0055);
        animation:rotate-reverse 5s linear infinite; }
      @keyframes rotate-forward { 0%{transform:rotate(0deg)} 100%{transform:rotate(360deg)} }
      @keyframes rotate-reverse { 0%{transform:rotate(360deg)} 100%{transform:rotate(0deg)} }
    </style>

    <!-- 外层：轮廓透光 + 黑色挖空 + 宽stroke制造干净缝隙 -->
    <mask id="mask-outer">
      <path fill="#FFFFFF" fill-rule="evenodd" d="M329.184,198.549L290.984,132.382L286.492,124.583L306.722,89.565C307.424,88.364,307.767,87.022,307.767,85.665C307.767,84.308,307.424,82.967,306.722,81.766L284.245,42.833C283.543,41.632,282.561,40.664,281.391,39.978C280.237,39.307,278.88,38.933,277.491,38.933L192.076,38.933L171.845,3.9C170.457,1.497,167.883,0,165.107,0L120.153,0C118.764,0,117.423,0.374,116.269,1.045C115.099,1.716,114.116,2.699,113.414,3.9L75.199,70.083L70.707,77.851L30.245,77.851C28.857,77.851,27.515,78.225,26.361,78.896C25.191,79.566,24.193,80.549,23.506,81.75L1.045,120.683C0.343,121.884,0,123.226,0,124.583C0,125.94,0.343,127.281,1.045,128.483L43.753,202.449L23.522,237.483C22.82,238.684,22.477,240.025,22.477,241.382C22.477,242.739,22.82,244.08,23.522,245.282L45.999,284.215C46.701,285.416,47.684,286.383,48.854,287.069C50.023,287.74,51.365,288.114,52.753,288.114L138.153,288.114L158.384,323.148C159.772,325.55,162.346,327.032,165.122,327.047L210.076,327.047C211.464,327.047,212.806,326.673,213.976,326.002C215.13,325.331,216.128,324.349,216.83,323.148L259.538,249.181L300,249.181C301.388,249.181,302.729,248.807,303.899,248.136C305.053,247.465,306.052,246.483,306.754,245.282L329.231,206.349C329.932,205.147,330.276,203.806,330.276,202.449C330.276,201.091,329.932,199.75,329.231,198.549L329.184,198.549ZM120.168,7.799L142.645,46.732L120.168,85.65L299.984,85.65L277.507,124.567L102.184,124.567L77.445,81.75L120.168,7.799ZM138.137,280.331L52.737,280.299L75.214,241.366L120.168,241.366L30.26,85.65L75.214,85.65L97.691,124.583L162.86,237.498L138.153,280.331L138.137,280.331ZM277.476,202.449L255.014,163.516L165.107,319.232L142.63,280.299L165.122,241.382L230.307,128.498L279.753,128.498L322.445,202.449L277.476,202.449Z"/>
      <path fill="#000000" stroke="#000000" stroke-width="12" stroke-linejoin="round" d="M120.168,7.799L142.645,46.732L120.168,85.665L299.984,85.665L277.507,124.582L102.184,124.582L77.445,81.766L120.168,7.799Z"/>
      <path fill="#000000" stroke="#000000" stroke-width="12" stroke-linejoin="round" d="M30.26,85.65L75.214,85.65L162.86,237.498L138.137,280.331L52.737,280.3L75.214,241.367L120.168,241.382L30.26,85.65Z"/>
      <path fill="#000000" stroke="#000000" stroke-width="12" stroke-linejoin="round" d="M142.63,280.299L165.107,319.232L255.015,163.516L277.476,202.449L322.446,202.449L279.754,128.482L230.307,128.498L142.63,280.299Z"/>
      <polygon fill="#000000" points="85,115 245,115 165,255"/>
    </mask>

    <!-- 内层：三块主体填充透光 -->
    <mask id="mask-inner">
      <path fill="#FFFFFF" d="M120.168,7.799L142.645,46.732L120.168,85.665L299.984,85.665L277.507,124.582L102.184,124.582L77.445,81.766L120.168,7.799Z"/>
      <path fill="#FFFFFF" d="M30.26,85.65L75.214,85.65L162.86,237.498L138.137,280.331L52.737,280.3L75.214,241.367L120.168,241.382L30.26,85.65Z"/>
      <path fill="#FFFFFF" d="M142.63,280.299L165.107,319.232L255.015,163.516L277.476,202.449L322.446,202.449L279.754,128.482L230.307,128.498L142.63,280.299Z"/>
    </mask>
  </defs>
  <g mask="url(#mask-outer)">
    <foreignObject x="0" y="0" width="100%" height="100%">
      <div xmlns="http://www.w3.org/1999/xhtml" class="spin-container">
        <div class="spin-glow-forward"></div>
      </div>
    </foreignObject>
  </g>
  <g mask="url(#mask-inner)">
    <foreignObject x="0" y="0" width="100%" height="100%">
      <div xmlns="http://www.w3.org/1999/xhtml" class="spin-container">
        <div class="spin-glow-reverse"></div>
      </div>
    </foreignObject>
  </g>
</svg>
</body>
</html>
```

---

## 常见坑与解决方案

| 问题 | 原因 | 解决 |
|------|------|------|
| 流光不显示 | Mask 中图形没有 `fill="#FFFFFF"` 或 `stroke="#FFFFFF"` | 确保透光路径是纯白 |
| 流光从缝隙漏出 | 相邻图形的 Mask 边界未封闭 | 在遮光路径上加 `stroke="#000000" stroke-width="2"` |
| `foreignObject` 布局错乱 | 使用百分比宽高 | 改用与 `viewBox` 匹配的绝对值 |
| Safari / 旧 WebKit 无效果 | `conic-gradient` 兼容性问题 | Safari 15.4+ 已支持；更低版本需降级方案 |
| 扫射模式边缘穿帮 | 发光盘不够大 | 增大到 `500%` 以上 |
| Firefox 中 `foreignObject` 位置偏移 | Firefox 对 `foreignObject` 的渲染差异 | 改用 `<embed>` 或在外层加 `transform` 微调 |

---

## 自定义配色

`conic-gradient` 中的色彩序列决定了流光颜色。**模板默认使用彩虹色**。以下是一些预设配色：

```
/* 彩虹（默认）—— 红 → 橙 → 黄 → 绿 → 蓝 → 紫 */
conic-gradient(from 0deg, #ff0055, #ff6600, #ffcc00, #00ff66, #00ccff, #9933ff, #ff0055)

/* 经典赛博朋克（偏粉青） */
conic-gradient(from 0deg, #ff0055, #ffaa00, #00ff66, #00ddff, #aa00ff, #ff0055)

/* 冷蓝科技 */
conic-gradient(from 0deg, #0033cc, #0066ff, #00ccff, #33ffff, #0066ff, #0033cc)

/* 暖金 */
conic-gradient(from 0deg, #ff6600, #ff9900, #ffcc00, #ffdd44, #ff9900, #ff6600)

/* 紫金 */
conic-gradient(from 0deg, #6b00ff, #cc00ff, #ff0099, #ff6600, #cc00ff, #6b00ff)
```

---

## 兼容性说明

- **conic-gradient**：Chrome 69+、Edge 79+、Safari 15.4+、Firefox 83+。如遇不兼容环境，可用多个 `linear-gradient` 拼接近似替代（性能较差）。
- **foreignObject**：主流浏览器均支持，但某些 SVG 查看器（如部分 Markdown 预览器）可能禁止 HTML 嵌入。最终输出建议为独立 HTML 文件。
- **CSS animation in SVG**：在 `<style>` 标签中定义，兼容所有现代浏览器。

---

## 参考文件

- `references/z-logo-demo.html`：Z 形标识双层流光示例（演示轮廓挖空策略，可直接浏览器打开）
- `references/color-presets.css`：配色预设合集
