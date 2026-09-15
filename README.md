# NonToon（自定义构建）— VPM 仓库

这是一个 [VPM](https://vcc.docs.vrchat.com/vpm/) 包仓库，用于分发 **NonToon 的自定义构建**。

## 怎么用（ALCOM / VCC）

1. 打开 ALCOM（或 VCC），找到**添加仓库 / Add Repository** 的入口
2. 粘贴这个地址：

   ```
   https://123cy321.github.io/VPM-nontoon-fork/vpm.json
   ```

3. 之后在工程的包管理页里，`NonToon` 会出现 **0.1.7** 版本，安装即可
   （仓库里同时保留 0.1.6，方便回退）

> ⚠️ 注意粘贴的是上面这个 **`vpm.json` 的地址**，不是 `.zip` 的地址。

## 这个构建相对官方 NonToon 0.1.3 改了什么

> 上游自 0.1.3（2026-07-14）之后**一行未改**，本仓库的 issue 修复均领先于官方。

**缺陷修复**

1. BiRP 补 `factor *= factor` —— 光照衰减曲线与 URP 对齐（原 BiRP 过渡偏平）
2. BiRP 环境光假边缘补 `* cd.screenrim` —— 修背景光穿模
3. 描边 pass 跳过 8 次深度采样 —— 原本是彻头彻尾的死代码，而描边是每个网格额外一次 draw
4. 修 URP `(SCCustomData)cd` 自引用笔误
5. 镜面高光受阴影衰减 —— 原先被投影的阴影里也有全强度高光
6. 暴露镜面菲涅耳 F0 属性（原先硬编码 `0.04`）
7. 描边新增可选深度偏移 `_OutlineOffsetFactor` / `_OutlineOffsetUnits`（默认 `0,0`，即中性）
8. **Transparent 模式不再做 alpha 硬裁剪**（官方 issue #11 / 未合并 PR #13）——
   官方实现里 Transparent 会连续执行两次 `clip()`，像素在到达渲染目标前就被丢弃，
   导致它**退化成 Cutout、完全没有 alpha 混合**
9. **BiRP 透明渲染队列 `2460` → `3000`** —— 原先落在不透明区间（Geometry `2000+460`），
   透明件被当不透明排序，头发/衣服的透明部分排序错乱

**新增功能**

- **光照调整**：`_LightMinLimit` / `_LightMaxLimit` / `_MonochromeLighting` / `_AsUnlit`
  —— 属性名与 lilToon 完全一致，可从 lilToon 材质直接迁移
- **阴影颜色模块 `ShadowColor`**：lilToon 风格的 1st / 2nd / 3rd 阴影颜色 + 边框 / 模糊 / 强度
  —— 属性名与默认值抄自 lilToon，可直接迁移
  > 使用时请把 `_ShadeGradientIndex` 设为 `-1`，否则会与渐变 ramp 叠加
- **发光模块 `Emission`**：`_UseEmission` / `_EmissionColor` / `_EmissionMap` / `_EmissionBlend` /
  `_EmissionBlendMask` / `_EmissionMainStrength` / `_EmissionBlendMode`
  —— 算法逐字对应 lilToon 的 `lilBlendColor`，在 `postpixel` 阶段应用，**不会被阴影衰减**
- **受光方向偏移（`_ShadeDirectionBias`）** —— 对应官方 issue #9。
  NonToon 原本把着色方向写死为「真实光照 + **视线方向 × 1.5**」，视线权重压过光照，
  于是侧光场景里渐变认为「正对相机的面」受光、直射光却认为「正对光源的面」受光，两者打架。
  把它设成 **0** 即可让渐变完全跟随真实光照。默认 `1.5` = 原行为，**不改变现有材质外观**
- **MatCap VR 视差强度（`VR Parallax Strength`）** —— 对应官方 PR #12。
  原先 MatCap 用双眼中间方向采样，左右眼看到完全相同的贴图，反射看起来是「贴」在模型上的；
  现在可切到逐眼采样，恢复真实视差（默认 `1`；设 `0` 回到旧行为）
- **Nearer 逐材质开关（`Enable`）** —— 对应官方 issue #8。
  近平面修正对「覆盖整个屏幕的大网格」会把顶点推出视锥、导致整个网格消失；
  现在可以在出问题的那个材质上单独关掉，不必全局禁用。默认开启（原行为）
- **内置 lilToon → NonToon 转换器**（原名 LilToonToNonToonConverter，MIT 许可）
  —— 生成**新的**材质，不改动原 lilToon 材质

**中文界面**：内置 `zh-Hans` 汉化（含转换器 UI）

### 让中文生效

ShaderCore 的语言默认取系统区域（如 `zh-CN`），而语言文件叫 `zh-Hans`，**名字对不上，默认不会加载**。

打开任意 NonToon 材质 → ShaderCore 材质编辑器里的 **Language** 下拉框 → 选 **简体中文**（一次性，会持久化）。

## 依赖

需要 **ShaderCore `jp.lilxyzw.shadercore` ≥ 0.1.9**（新增模块用到了它的 `keepPropertyNames` 字段）。
通常 NonToon 的 VPM 依赖会自动带上。

> 装完后本包会**自愈**模块注册表：ShaderCore 会把「每个着色器启用了哪些模块」冻结在
> `ProjectSettings/jp.lilxyzw.shadercore.asset` 里，只在首次导入时扫描一次；
> 本包内置的 `Editor/NTModuleRegistration.cs` 会在编辑器加载时补上缺失的模块并强制重导着色器。

## 关于包 id

本构建**沿用官方的包 id `jp.lilxyzw.nontoon`**，只是版本号更高（0.1.7 > 0.1.3）。

这样做是刻意的：它会被视为官方版的**升级**，安装后即替换官方版，不会出现两个同名
`Shader "NonToon"` 共存（Unity 材质按 GUID 引用 shader，本构建刻意保留了官方的全部资源 GUID，
所以已有材质不会断引用）。

## 验证状态

打包前已在 Unity 2022.3.22f1 批处理模式下实测：C# 与着色器**零错误**，
`NonToon` / `NonToonFur` 属性表分别为 **125 / 111** 项，本版新增的三个属性均已出现，
两个 `.scshader` 预编译 `ok=1`，zip 内 188 个文件全部通过 CRC 校验。

**未验证**：实际渲染观感（无 GUI 环境），请以你在编辑器/VR 里的目视为准。

## 已知限制

- 阴影颜色的**贴图类遮罩**（lilToon 的 `_ShadowStrengthMask` / `_ShadowBorderMask` / `_ShadowBlurMask`）
  **未移植**；本构建改用 NonToon 既有的共享遮罩通道约定（`_ShadowStrengthMaskChannel`）。
- `Emission` 未移植 lilToon 的 `_Emission2nd*`、渐变发光、闪烁、荧光、视差深度、UV 模式。
- 两个 SubShader 都**没有声明 `Queue` 标签**，渲染队列依赖编辑器写入 `m_CustomRenderQueue`；
  脚本或复制产生的材质不会自动带上正确队列。
- 官方 issue #7（Fur 在 Radeon / 老 N 卡上异常膨胀）**未处理**：上游补丁作者本人说明只能缓解、
  不能根治，且改动几何着色器的实例 ID 时序风险较高。

## 许可

修改部分基于 [lilxyzw/NonToon](https://github.com/lilxyzw/NonToon)，请遵守其原始 LICENSE（见包内 `LICENSE`）。
内置转换器来自 **LilToonToNonToonConverter 1.1.4**（MIT，已保留文件头署名）。
