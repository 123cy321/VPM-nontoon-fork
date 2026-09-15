# NonToon（自定义构建）— VPM 仓库

这是一个 [VPM](https://vcc.docs.vrchat.com/vpm/) 包仓库，用于分发 **NonToon 的自定义构建**。

## 怎么用（ALCOM / VCC）

1. 打开 ALCOM（或 VCC），找到**添加仓库 / Add Repository** 的入口
2. 粘贴这个地址：

   ```
   https://123cy321.github.io/VPM-nontoon-fork/vpm.json
   ```

3. 之后在工程的包管理页里，`NonToon` 会出现 **0.1.5** 版本，安装即可

> ⚠️ 注意粘贴的是上面这个 **`vpm.json` 的地址**，不是 `.zip` 的地址。

## 这个构建相对官方 NonToon 0.1.3 改了什么

**七项缺陷修复**

1. BiRP 补 `factor *= factor` —— 光照衰减曲线与 URP 对齐（原 BiRP 过渡偏平）
2. BiRP 环境光假边缘补 `* cd.screenrim` —— 修背景光穿模
3. 描边 pass 跳过 8 次深度采样 —— 原本是彻头彻尾的死代码，而描边是每个网格额外一次 draw
4. 修 URP `(SCCustomData)cd` 自引用笔误
5. 镜面高光受阴影衰减 —— 原先被投影的阴影里也有全强度高光
6. 暴露镜面菲涅耳 F0 属性（原先硬编码 `0.04`）
7. 描边新增可选深度偏移 `_OutlineOffsetFactor` / `_OutlineOffsetUnits`（默认 `0,0`，即中性）

**新增功能（默认关闭，不改变原有观感）**

- **光照调整**：`_LightMinLimit` / `_LightMaxLimit` / `_MonochromeLighting` / `_AsUnlit`
  —— 属性名与 lilToon 完全一致，可从 lilToon 材质直接迁移
- **阴影颜色模块 `ShadowColor`**：lilToon 风格的 1st / 2nd / 3rd 阴影颜色 + 边框 / 模糊 / 强度
  —— 属性名与默认值抄自 lilToon，可直接迁移
  > 使用时请把 `_ShadeGradientIndex` 设为 `-1`，否则会与渐变 ramp 叠加

**中文界面**：内置 `zh-Hans` 汉化（11 个 `.po`）

### 让中文生效

ShaderCore 的语言默认取系统区域（如 `zh-CN`），而语言文件叫 `zh-Hans`，**名字对不上，默认不会加载**。

打开任意 NonToon 材质 → ShaderCore 材质编辑器里的 **Language** 下拉框 → 选 **简体中文**（一次性，会持久化）。

## 依赖

需要 **ShaderCore `jp.lilxyzw.shadercore` ≥ 0.1.9**（新增模块用到了它的 `keepPropertyNames` 字段）。
通常 NonToon 的 VPM 依赖会自动带上。

## 关于包 id

本构建**沿用官方的包 id `jp.lilxyzw.nontoon`**，只是版本号更高（0.1.5 > 0.1.3）。

这样做是刻意的：它会被视为官方版的**升级**，安装后即替换官方版，不会出现两个同名
`Shader "NonToon"` 共存（Unity 材质按 GUID 引用 shader，本构建刻意保留了官方的全部资源 GUID，
所以已有材质不会断引用）。

## 已知限制

- **未经编译与渲染验证**：本包在没有可用 Unity 编辑器的环境下修改。若编译报错请反馈具体信息。
- 阴影颜色的**贴图类遮罩**（lilToon 的 `_ShadowStrengthMask` / `_ShadowBorderMask` / `_ShadowBlurMask`）
  **未移植**；本构建改用 NonToon 既有的共享遮罩通道约定（`_ShadowStrengthMaskChannel`）。
- 新模块 `ShadowColor` 只有中文与英文，**没有日文**。
- `Nearer` 模块名仍是英文（上游本来就没有该模块的语言文件）。

## 许可

修改部分基于 [lilxyzw/NonToon](https://github.com/lilxyzw/NonToon)，请遵守其原始 LICENSE（见包内 `LICENSE`）。
