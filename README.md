# SVG Cyberpunk Glow

动态赛博朋克流光效果的 SVG 技能包。采用 **Mask + ForeignObject** 解耦架构，只需替换 Path 路径即可为任意 SVG 图形添加旋转流光/扫光动画。

## 效果预览

彩虹色双层流光，轮廓与主体独立动画通道：

![preview](references/z-logo-demo.html)

## 快速开始

1. 复制 `SKILL.md` 到你的 WorkBuddy skills 目录（`~/.workbuddy/skills/svg-cyberpunk-glow/`）
2. 或打开 `references/z-logo-demo.html` 直接在浏览器查看效果
3. 在 WorkBuddy 中说"帮这个 SVG 加个赛博朋克流光效果"

## 文件结构

```
svg-cyberpunk-glow/
├── SKILL.md                    # 主技能文件（AI 工作流 + 代码模板 + 完整示例）
└── references/
    ├── z-logo-demo.html         # Z 形标识双层流光演示
    └── color-presets.css        # 9 组配色预设
```

## 核心架构

```
SVG Mask（遮罩层） ── 控制形状 & 透光率
       +
CSS ForeignObject（发光层） ── 控制色彩 & 运动
```

- **SVG Mask**：`#FFFFFF` = 透光，`#000000` = 遮光
- **CSS ForeignObject**：`conic-gradient` + CSS `@keyframes` 旋转
- **双层策略**：外层轮廓 + 黑色挖空制造缝隙光，内层填充反转形成层次

## 动画模式

| 模式 | 效果 | 适用场景 |
|------|------|----------|
| 中心正转 | 150% 大小中心自转 | 主体流光 |
| 中心反转 | 150% 大小中心反转 | 双层流光的内层 |
| 外部扫射 | 500% 大盘偏心旋转 | 探照灯/雷达波效果 |

## 配色预设

- 🌈 彩虹（默认）：红 → 橙 → 黄 → 绿 → 蓝 → 紫
- 🎮 经典赛博朋克：品红-橙-绿-青-紫
- ❄️ 冷蓝科技
- 🔥 暖金 / 熔岩
- 🟣 紫金 / 霓虹粉
- 🖥️ 绿屏终端

详见 `references/color-presets.css`

## 兼容性

- conic-gradient：Chrome 69+、Edge 79+、Safari 15.4+、Firefox 83+
- foreignObject：主流浏览器均支持
- 建议输出为独立 HTML 文件

## License

MIT
