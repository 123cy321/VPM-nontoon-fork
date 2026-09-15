# NonToon (Fork)

**[lilxyzw/NonToon](https://github.com/lilxyzw/NonToon) 的改进分支** —— 面向 VRChat 的卡通着色器。
官方停在 **0.1.3（2026-07-14）**之后一行未改，这个分支修掉了它的问题、补上了 lilToon 的迁移路径，
并加了「自带光源 / 自阴影」和**完整简体中文**。

| | |
|---|---|
| **一键添加页面** | https://catandling.github.io/VPM-nontoon-fork/ |
| **listing 地址** | `https://catandling.github.io/VPM-nontoon-fork/vpm.json` |
| 最新版本 | 着色器 **0.3.4** ｜ 工具包 **0.4.4** |
| 依赖 | `jp.lilxyzw.shadercore` ≥ 0.1.9（VPM 自动带） |
| 支持 | Unity 2022.3 ／ **BiRP**（VRChat）／ Quest 可用（着色器侧功能） |

> ⚠️ **账号改名公告**：GitHub 账号 `123cy321` → **`CatAndLing`**。
> GitHub Pages **不为改名做重定向**（实测旧地址直接 404），所以旧的 listing 地址已失效。
> **以前添加过旧地址的，请在 ALCOM / VCC 里重新添加上面的新地址**（仓库 id 未变，不会多出一条重复项）。

---

## 安装

**① 一键（推荐）**：打开 **[添加页面](https://catandling.github.io/VPM-nontoon-fork/)**，点上面的按钮 → 唤起 VCC 的「添加仓库」确认框。

> README 里**不能**直接放 `vcc://` 链接：GitHub 的 Markdown 渲染器会过滤非 http(s) 协议，
> 实测渲染后只剩文字、`href` 被删掉。所以按钮放在 GitHub Pages 上。
> 要手动触发的话，把下面这行粘到浏览器地址栏：

```text
vcc://vpm/addRepo?url=https://catandling.github.io/VPM-nontoon-fork/vpm.json
```

**② ALCOM / VCC 手动添加**：找到「添加仓库 / Add Repository」，粘贴 **listing 地址**（是 `vpm.json`，**不是** `.zip`）：

```text
https://catandling.github.io/VPM-nontoon-fork/vpm.json
```

添加后包里会出现两个（仓库里保留旧版本便于回退）：

| 包 | 装不装 | 内容 |
|---|---|---|
| **`NonToon (Fork)`** 0.3.4 | **必装** | 着色器本体 + 少量编辑器辅助（**纯库**，不含工具） |
| **`NonToon (Fork) Tools`** 0.4.4 | 可选 | lilToon→NonToon 材质转换器、SelfLight 烘焙器、Avatar 光源插件 |

> 依赖是**单向**的：装工具包会自动带着色器；只装着色器不会带工具。
> 工具包的 id 仍是 `com.123cy321.nontoon-converter`（历史原因，保持 id 才能原地升级），显示名已改为 Tools。

---

## 这个分支相对官方 0.1.3 做了什么

### 修掉的问题

| # | 问题 | 官方 0.1.3 | 本分支 |
|---|---|---|---|
| 1 | **半透明完全失效**（[#11](https://github.com/lilxyzw/NonToon/issues/11)） | Transparent 连续两次 `clip()` → 退化成 Cutout，没有 alpha 混合 | 真正做 alpha 混合（采纳未合并的 [PR #13](https://github.com/lilxyzw/NonToon/pull/13) 思路） |
| 2 | **透明排序错乱** | 队列 `2460`（落在不透明区间） | `3000`（Transparent 区间） |
| 3 | **侧光下渐变与直射光打架**（[#9](https://github.com/lilxyzw/NonToon/issues/9)） | 着色方向写死「真实光照 + 视线 × 1.5」 | `_ShadeDirectionBias`，设 `0` = 完全跟随真实光照 |
| 4 | **VR 里 MatCap 像贴纸**（[PR #12](https://github.com/lilxyzw/NonToon/pull/12)） | 双眼采样同一方向 | `VR Parallax Strength`，默认 `1` = 逐眼采样 |
| 5 | **大网格被近平面修正推出视锥**（[#8](https://github.com/lilxyzw/NonToon/issues/8)） | 只能全局关 | 逐材质 `Nearer → Enable` |
| 6 | **遮罩通道不够用**（[#10](https://github.com/lilxyzw/NonToon/issues/10)） | 只能 R/G/B/A | **8 档**：`R / G / B / A / 1-R / 1-G / 1-B / 1-A` |
| 7 | 环境光穿模 | BiRP 少乘 `cd.screenrim` | 与 URP 对齐 |
| 8 | 光照衰减偏平 | BiRP 少 `factor *= factor` | 与 URP 对齐 |
| 9 | 阴影里高光仍是全强度；描边 pass 里 8 次帧深度采样是死代码 | — | 高光受阴影衰减、死代码删除、新增 `_OutlineOffsetFactor/Units` |

### 加的能力

| 能力 | 官方 0.1.3 | 本分支 |
|---|---|---|
| 阴影颜色 | 只有渐变 ramp | **`ShadowColor` 模块**：lilToon 式 1/2/3 层阴影色 + 边界/模糊/强度/对比度，**含 lilToon 的逐像素阴影遮罩**（强度/边界/模糊） |
| 发光 | 没有（`Lighten` 只是亮度乘数） | **`Emission` 模块**：颜色/贴图/混合/混合遮罩/受主色影响/4 种混合模式，在 `postpixel` 应用 → 不受阴影衰减 |
| 光照调整 | 结果被硬编码 `saturate()` | `_LightMinLimit` / `_LightMaxLimit` / `_MonochromeLighting` / `_AsUnlit`（**属性名与 lilToon 相同**，可直接迁移） |
| **自带光源 / 自阴影** | 没有 | **`SelfLight` 模块**：角色私有光 + 烘焙深度图自阴影，**不挂实时光**（VRChat `Lights` 仍为 0）；PCSS 软阴影、阴影距离、接收遮罩、浓度、硬化 |
| **实时光源** | 没有 | 工具包里的 **Avatar 光源插件**（真 Spot Light，着色器无关，一键创建） |
| **lilToon 迁移** | 只能手工重做 | 工具包的**材质转换器**：生成新材质、**不动原 lilToon 材质** |
| **界面语言** | 日文 / 英文 | **完整简体中文**（着色器 146 条 + 工具包 151 条 `.po`，另**自动补齐 ShaderCore 的 31 个内置项**） |

### 体量（实测普查）

| | 官方 0.1.3 | 本分支 0.3.4 |
|---|---|---|
| 文件数（不含 `.meta`） | 54 | **83** |
| `SC_` 属性声明行 | 123 | **208** |
| 模块数 | 10 | **13** |
| Unity 实测面板属性 | — | **NonToon 155 / NonToonFur 141** |

> 自己 diff 时注意：**官方 release zip 是 CRLF、本分支源码是 LF**。
> 用 `diff -rq --strip-trailing-cr -x '*.meta' <官方0.1.3> <本分支>`，不加 `--strip-trailing-cr` 会多出一堆"换行符不同"的假差异。

---

## 四个主打能力

### ① 完整简体中文

ShaderCore 的材质面板、模块标题、枚举标签、ShaderLab 的渲染/模板属性、右键菜单与队列名，以及
工具包的窗口 / 报告 / 日志 / 组件检视面板**全部中文**（未命中的 key 自动回落英文）。

**ShaderCore 0.1.12 起不再附带 `zh-Hans.po`**，导致 `Main` / `贴图` / `共享遮罩` / `粗糙度` / `裁剪阈值`
这些**内置项永远是英文**（`L10n` 找不到语言文件就回落 `en-US.po`，在 core 表里命中英文直接返回，
自己写多少 po 都轮不到）。本包会在缺失时**自动补齐** ShaderCore 的 `zh-Hans.po`：
只在缺失时写、**绝不覆盖**上游自带的、幂等、`zh-CN` 也一并覆盖。

使用前提：材质编辑器右上角 **Language 选「简体中文」**（一次性，会记住）。

### ② lilToon → NonToon 转换器

菜单 **`Tools ▸ lilToon → NonToon 转换器`**，或右键 Hierarchy 里的模型 / Project 里的材质。
**生成新材质，不改动原 lilToon 材质**；转换报告会逐条说明哪些 1:1 搬过去了、哪些是近似、哪些没支持。

- 1:1：`_AsUnlit` / `_LightMinLimit` / `_LightMaxLimit` / `_MonochromeLighting`、1/2/3 层阴影色与边界/模糊、
  逐像素阴影遮罩（`_ShadowStrengthMask` / `_ShadowBorderMask` / `_ShadowBlurMask`，通道语义与 lilToon 一致）、
  Emission 全套、Main 2nd/3rd（烘焙进底图）
- 近似：Roughness、MatCap、RimLight、Backlight、Distance Fade、Fur、Outline
- 未支持：Decal / Dissolve / Glitter / Refraction / Gem / AudioLink / UV 动画 / `_Emission2nd*` 等
  （详见包内 `Mapping.md`）

### ③ 自带光源与自阴影（地图没有环境光也能用）

场景：**地图没灯 / 环境光乱来**，希望 avatar 靠自己的光 + 自己的阴影，且在任何世界里长得一样。

`Add Component ▸ NonToon ▸ 自有光源 Self Light`：

1. 指定一盏方向光/聚光灯（**烘焙只读它的方向/颜色，不会真的挂上去**）
2. 勾 **「只由它照亮」** → 世界光、环境光、lightmap、顶点光、天空盒反射**全部丢掉**
3. 点「烘焙自阴影并写入材质」

修过的坑：早期版本「只由它照亮」其实**没关掉环境光** —— SH 环境光是在 `customlight` 相位**之后**
才累加进 `env` 的，在那儿清零等于白清，而且 `sd.L` 被 SH 污染（同一颗 avatar 换个地图就换一副渐变）。
现在排他通路放在 `__SC_PHASE_modifylight__`（`sd.lightColor = env + lightSum.color` 之后），
同时清 `env` 并接管 `sd.L`。

> 想要**影子跟着姿势实时变**：用工具包的 Avatar 光源插件，或把组件切到「实时光源」模式。
> 代价是占 VRChat 的 Lights 计数、依赖观看者的 Shadow Quality、Quest 基本不可用。

### ④ 性能预算（NonToon 的本分就是低消耗）

**每像素自阴影采样**（默认全是最省那档，且只在光源包围盒内 + 阴影距离之内才发生）：

| 配置 | 采样/像素 |
|---|---|
| `_UseSelfLight = 0`（**默认**） | **0** |
| 自有光源开、强度 = 0（只要光不要影） | **0**（连 UV/包围盒都跳过） |
| 自有光源开、PCSS 关（硬阴影） | **1** |
| PCSS **低（默认）** | **20**（8 blocker + 12 PCF） |
| PCSS 中 / 高 / 极高 | 36 / 60 / 96 |
| 接收遮罩 | +1（**默认 0 = 不采样**） |
| ShadowColor 三张遮罩 | +3（**默认关 = 0 采样**） |

组件面板会直接把当前预算显示出来。这些默认值由自动化探针**逐条断言**锁死，被改贵会直接报红。

对比：实时光源在着色器侧是 0 采样，但 Unity 要**额外渲染一张阴影贴图**、每个角色多一个 ForwardAdd，
而且 `Lights = 1` → PC 上性能等级最高只能 **Poor**，**每个看到你的人都付这份开销**。要低消耗就用默认的烘焙那条。

---

## 升级注意

- **从官方版升级**：包 id 与官方相同，装上去就是**升级**官方版，不会并存。
- **从 ≤ 0.1.10 升级上来**：旧版本把转换器打包在着色包里，包管理器覆盖安装可能残留
  `Packages/jp.lilxyzw.nontoon/Editor/LilToonConverter/`，菜单里会出现**两个**转换器。
  遇到就把 `Packages/jp.lilxyzw.nontoon` 整个删掉重装。
- 新增的属性/模块都带默认值，**旧材质不需要重新转换**。

## 版本记录

**着色器包 `jp.lilxyzw.nontoon`**

| 版本 | 要点 |
|---|---|
| **0.3.4** | 消耗压回去：PCSS 默认档 中(36)→**低(20)**、接收遮罩默认不采样、强度 0 时**一次都不采**；面板显示采样预算 |
| 0.3.3 | 修「只由它照亮」**没真正关掉世界环境光**（排他通路挪到 `modifylight`，清 `env`、接管 `sd.L`） |
| 0.3.2 | 修「材质面板还有一半英文」：ShaderCore 0.1.12 起不再带 `zh-Hans.po`，本包缺失时自动补齐 |
| 0.3.1 | PCSS 画质四档、烘焙上限 2048；新增**实时光源模式** |
| 0.3.0 | **汉化补齐**（ShaderLab 渲染/模板属性、模块标题、枚举标签）；SelfLight 加 **PCSS 软阴影**；lilToon **逐像素阴影遮罩**可转换 |
| 0.2.0 | 新增 **SelfLight**（自有光源 + 烘焙自阴影）；转换器用上 Emission / ShadowColor |
| 0.1.11 | 转换器移出 → 独立工具包；着色包变纯库 |
| 0.1.10 | 转换器加拖放区；修「转换后阴影颜色模块没被打开」（Int 属性用 `SetFloat` 写是静默无效的） |
| 0.1.9 | 显示名 → `NonToon (Fork)`（**包 id 不变**） |
| 0.1.8 | 遮罩通道反向（4 档 → 8 档，[#10](https://github.com/lilxyzw/NonToon/issues/10)） |
| 0.1.7 | `_ShadeDirectionBias`（[#9](https://github.com/lilxyzw/NonToon/issues/9)）、MatCap VR 视差（[PR #12](https://github.com/lilxyzw/NonToon/pull/12)）、Nearer 开关（[#8](https://github.com/lilxyzw/NonToon/issues/8)） |
| 0.1.6 | Emission 模块、模块注册自愈、转换器汉化 |
| 0.1.5 | 修 0.1.4 的导入故障（`properties.hlsl` 里的注释会让 ShaderCore 解析抛异常） |
| 0.1.4 | 首次发布 |

**工具包 `com.123cy321.nontoon-converter`**

| 版本 | 要点 |
|---|---|
| **0.4.4** | 面板显示每像素采样预算、默认档位 Low |
| 0.4.3 | 新增独立 **Avatar 光源插件**（着色器无关、一键创建、默认只照 `PlayerLocal`、自动开 Receive Shadows） |
| 0.4.2 | 修读 ShaderCore 语言设置时的反射（泛型基类的静态属性要 `FlattenHierarchy`） |
| 0.4.1 | 实时光源创建/同步；PCSS 画质档位面板 |
| 0.4.0 | 工具包 UI **全中文**（带英文 fallback）、阴影遮罩转换、SelfLight v2 检视面板 |
| 0.3.x | 拖放区、检测规则重写、自诊断（0 检出时说明原因并列出实际着色器） |
| 0.2.0 / 0.1.x | 依赖抬到 `>=0.2.0`；转换器从着色包独立出来后的早期版本 |

> 全部版本的 zip 与 SHA-256 都在 [listing](https://catandling.github.io/VPM-nontoon-fork/vpm.json) 里，**22 个版本逐个下载比对过**。

## 依赖

需要 **ShaderCore `jp.lilxyzw.shadercore` ≥ 0.1.9**（新增模块用到它的 `keepPropertyNames`）。

装完后本包会**自愈**模块注册表：ShaderCore 把「每个着色器启用了哪些模块」冻结在
`ProjectSettings/jp.lilxyzw.shadercore.asset` 且只在首次导入时扫描一次，升级上来会**静默丢模块**；
本包内置的 `Editor/NTModuleRegistration.cs` 会在编辑器加载时补上缺失项并强制重导。

## 包 id 与显示名

| | 值 | 能不能改 |
|---|---|---|
| 包 **id**（`name`） | `jp.lilxyzw.nontoon`（与官方相同） | **不能改** —— 同 id 才会被当成官方版的**升级**；改成独立 id 会与官方版**并存**，工程里出现两个同为 `Shader "NonToon"` 的着色器 |
| **显示名** | `NonToon (Fork)` | 可以改，只影响 ALCOM/VCC 里的显示 |
| 着色器名 `Shader "NonToon"` | 未改 | 不改 —— 代码与第三方工具里有 `Shader.Find("NonToon")` 这类按名查找 |
| 工具包 id | `com.123cy321.nontoon-converter` | 不改 —— 改了老用户的包不会被升级，还会与旧包并存 |

## 验证状态

每次发版前都在 **Unity 2022.3.22f1 批处理**下跑三套自动化：

1. **AssetBundle 真编译**：为目标平台实际编译着色器变体，抓 `Shader error`。
   NonToon / NonToonFur **0 编译错误、0 警告**。
2. **负向对照**：故意写错的着色器必须被报出来，否则本套装置结论作废（实测有效）。
3. **端到端探针**：转换器（三种渲染模式、阴影色模块、Emission/光照 1:1、阴影遮罩、模型资源检测、
   VRC 反射剥离）、SelfLight 烘焙、实时光源、汉化 key、默认值预算 —— 全绿输出 `==== 全部通过 ====`。

发布后还会**从线上**逐个下载全部 22 个版本的 zip 并比对 SHA-256，与清单逐字符一致。

> 说清楚一个曾经的假绿：早期版本的「着色器无报错」只在 `-nographics` 批处理里查过，
> 而那种模式**根本不编译 shader 变体**（`ShaderUtil.GetShaderMessages` 对故意写错的着色器也返回 0 条）。
> 现在用的是上面第 1 条的真编译。

## 已知限制

- **URP 分支在 Unity 2022.3 上是死代码**（要 URP 17 / Unity 6），本分支只保证 **BiRP（VRChat）**。
- 转换是**近似**：贴图类细节、`_Emission2nd*`、渐变发光、闪烁、荧光、视差深度、Decal/Dissolve/Glitter/
  Refraction/Gem/AudioLink/UV 动画等未移植；`_OutlineZBias` 因单位不同刻意不映射。
- 两个 SubShader 都**没有声明 `Queue` 标签**，渲染队列依赖编辑器写入 `m_CustomRenderQueue`，
  脚本或复制出来的材质不会自动带正确队列。
- **自阴影是"烘焙那一刻的姿态"**（VRChat 内做不到实时自阴影）；想要实时就用实时光源，代价见上。
- 官方 issue [#7](https://github.com/lilxyzw/NonToon/issues/7)（Fur 在 Radeon / 老 N 卡上异常膨胀）**未处理**：
  上游补丁作者本人说明只能缓解不能根治，且改动几何着色器实例 ID 时序风险高。
- [#3](https://github.com/lilxyzw/NonToon/issues/3) 与 [#7](https://github.com/lilxyzw/NonToon/issues/7) 明确不做；
  [#5](https://github.com/lilxyzw/NonToon/issues/5)（选中别的着色器时 Render Queue 被清零）是 **Shader-Core 内部的问题**
  （[Shader-Core #21](https://github.com/lilxyzw/Shader-Core/issues/21)），与分支的改动无关、也改不动。

## 许可

修改部分基于 [lilxyzw/NonToon](https://github.com/lilxyzw/NonToon)，请遵守其原始 LICENSE（见包内 `LICENSE`）。
材质转换器源自 **LilToonToNonToonConverter**（MIT，保留文件头署名）。
