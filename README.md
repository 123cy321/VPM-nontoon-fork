# NonToon (Fork) — VPM 仓库

这是一个 [VPM](https://vcc.docs.vrchat.com/vpm/) 包仓库，用于分发 **NonToon (Fork)** ——
[lilxyzw/NonToon](https://github.com/lilxyzw/NonToon) 0.1.3 的改进分支。

> **为什么显示名带 `(Fork)`**：包 **id 仍是官方的 `jp.lilxyzw.nontoon`**（这样才是「升级官方版」而不是并存的第二套），
> 但**显示名**刻意加了后缀，避免你在 ALCOM/VCC 的包列表里分不清哪个是这个分支。
> 两件事都要保持：**id 不能改，显示名要能区分**。

## 一键添加

**▶ [点我打开添加页面](https://123cy321.github.io/VPM-nontoon-fork/)**

那个页面上的按钮就是 `vcc://` 深链，点一下会唤起 VCC 的「添加仓库」确认框。

> **为什么 README 里不直接放 `vcc://` 链接**：GitHub 的 Markdown 渲染器会过滤非 http(s) 协议的链接。
> 实测把 `[文字](vcc://vpm/addRepo?…)` 写进 README 后，渲染出来**只剩加粗文字、`href` 被删掉**
> （本次已在该仓库线上页面核实）。自定义协议只有在真正的 HTML 页面里点才有效，
> 所以按钮放在 GitHub Pages 上。

需要手动添加时，把下面这行复制到浏览器地址栏，效果和点按钮相同：

```text
vcc://vpm/addRepo?url=https://123cy321.github.io/VPM-nontoon-fork/vpm.json
```

**ALCOM** 直接粘贴 listing 地址（注意是 `vpm.json`，不是 `.zip`）：

```text
https://123cy321.github.io/VPM-nontoon-fork/vpm.json
```

## 怎么用（ALCOM / VCC）

1. 打开 ALCOM（或 VCC），找到**添加仓库 / Add Repository** 的入口
2. 粘贴 listing 地址：

   ```
   https://123cy321.github.io/VPM-nontoon-fork/vpm.json
   ```

3. 之后在工程的包管理页里会出现**两个**包（仓库里同时保留旧版本便于回退）：

   | 包 | 装不装 | 说明 |
   |---|---|---|
   | **`NonToon (Fork)` 0.3.1** | 必装 | 着色器本体，**纯库**（只有着色器 + 少量编辑器辅助） |
   | **`NonToon (Fork) Tools` 0.4.1** | 可选 | 工具包：材质转换器 + SelfLight（自有光源）烘焙器 / 实时光源 |

   > 依赖方向是**单向的**：装工具包会自动带着色器；装着色器**不会**带工具包（工具是可选件）。
   > 工具包的 **id 仍是 `com.123cy321.nontoon-converter`**（历史原因，保持 id 才能原地升级），显示名已改成 Tools。

> ⚠️ 注意粘贴的是上面这个 **`vpm.json` 的地址**，不是 `.zip` 的地址。

## lilToon → NonToon 转换器怎么用

> **0.1.11 起转换器已从着色包移出，改为独立辅助包 `com.123cy321.nontoon-converter`。**
> 这样主着色包保持**纯库**：不掺工具、不掺 VRChat 相关逻辑。
> 本包**不依赖 VRChat SDK**（`VRC.Core.PipelineManager` 走反射），所以在非 VRChat 工程里也能用。
>
> ⚠️ **从 0.1.10 及更早升级上来的话**：如果包管理器只在旧包目录上覆盖文件，
> 旧的 `Packages/jp.lilxyzw.nontoon/Editor/LilToonConverter/` 可能残留，
> 于是菜单里会出现**两个**转换器。看到这种情况就把 `Packages/jp.lilxyzw.nontoon` 整个删掉，
> 让包管理器重新安装一次。

菜单：`Tools ▸ LilToon to NonToon Converter`
（也可以右键 Hierarchy 里的模型 → `Convert lilToon to NonToon`）

**本分支的能力已经被转换器用上了（0.2.0）** —— 不再是"只能近似"：

- **发光 1:1**：`_EmissionColor` / `_EmissionMap` / `_EmissionBlend` / `_EmissionBlendMask` /
  `_EmissionMainStrength` / `_EmissionBlendMode` 全套 → 原生 `Emission` 模块，且**不被阴影衰减**
  （原版工具只能把强度近似成 `LightBoost`，其 `Mapping.md` 明确写着「Emission Map は未対応」）
- **lilToon 光照调整 1:1**：`_AsUnlit` / `_LightMinLimit` / `_LightMaxLimit` / `_MonochromeLighting`
  —— 本分支这几个属性与 lilToon **同名同义**，直接搬（原版工具只把它们写进诊断日志）
- **阴影 1:1**：lilToon 的 1/2/3 层阴影色走原生 `ShadowColor` 模块（保真 border/blur/strength），
  并**自动关掉 Shade ramp** 避免两层叠加
- **受光方向**：自动把 `_ShadeDirectionBias` 置 **0**，让着色**跟光**（贴近 lilToon 的行为）；
  想要 NonToon 原味就把它调回 `1.5`
- 发光遮罩不再占用共享遮罩通道（原生模块直接挂贴图）

转换后请过一遍包内 **`Mapping.md`** 的「转换后的注意事项」（12 条：阴影叠加、受光方向、未移植项、
描边深度偏移为何没自动映射、报告文件在哪……）。

**两种用法，任选一种**

1. **拖进去**（0.1.10 起）—— 把模型（Hierarchy 里的，或 Project 里的 fbx / prefab）、材质、文件夹
   拖到窗口**最上面的方框**里，再点「转换拖入的对象」
2. **选中再转** —— 在 Project（材质 / 文件夹 / fbx / prefab）或 Hierarchy（模型）里选中，点「转换所选对象」

> ⚠️ **0.1.9 及更早没有拖放功能**，只能走第 2 种；那时候往窗口里拖是**没有反应**的。

**会发生什么**

- 生成**新的** NonToon 材质，**原 lilToon 材质一个都不动**
- 默认输出到 `Assets/NonToonConverted/`；报告写到
  `Assets/NonToonConverted/LilToonToNonToonReport.txt`，并在 `Assets/NonToonConversionLogs/` 留一份带环境信息的日志
- 选 Hierarchy 里的模型时，默认会**复制一份** `模型名_NonToon` 并把原对象禁用（`Ctrl+Z` 可撤销）；
  想要就地替换就取消勾选「复制 Hierarchy 对象并禁用原对象」
- 失败的项会写进报告，**把报告发我就能定位**

**转换前先看窗口里「检测到的 lilToon 材质」这个数字**：是 0 就说明拖/选的对象里没有 lilToon 材质
（只认「着色器名里含 lilToon」的材质），转换器不会做任何事 —— **此时转换按钮是灰色的（禁用状态），不是没有按钮**。

> **0.1.1 修了一个会让上面这个数字永远是 0 的 bug**：从 Project 窗口拖入**模型资源**（fbx / prefab）时，
> 旧版 `CollectMaterials` 会在那条判定路径上一个分支都不进（实测），于是收集到 0 个材质、按钮一直灰着 ——
> 看起来就像"窗口里根本没有转换按钮"。现在改成：先收对象自身的 Renderer 材质，资源再补「依赖」和
> 「场景实例」两路（fbx 常常不自带材质，材质挂在场景实例上）。拖 Project 里的 fbx 现在也能正确识别。

## SelfLight —— 角色「自己的光源 + 自阴影」（0.2.0 新增）

给 avatar 一路**只照自己**的光，并让它给自己投影。**不挂任何真实 Unity Light。**

为什么不用真实光源（有官方出处）：

- VRChat 性能等级里 **Lights 在 PC 上要求 0**（Excellent/Good/Medium），只有 Poor 允许 1 —— 挂一盏实时光直接掉档（[Performance Ranks](https://creators.vrchat.com/avatars/avatar-performance-ranking-system/)）
- 真实光会**照亮世界与其他玩家**；VRChat 的层里 `Player(9)` 是"除本地玩家以外的玩家"、`PlayerLocal(10)` 才是本地玩家，而且 Unity 的 light culling mask 对自定义层本就不可靠（[Unity Layers in VRChat](https://creators.vrchat.com/worlds/layers/)）
- Quest 端实时光基本不可用

所以全部做在**着色器**里：私有光 + **烘焙好的光照空间阴影图** ⇒ 不进 Lights 计数、不外溢、Quest 可用，
运行时只多 1 次贴图采样 + 1 个 `dot(N,L)`。

**用法**：avatar 根节点 → `Add Component → NonToon → Self Light` → 指定那盏光源 →
点「烘焙自阴影并写入材质」。烘焙是**纯 CPU 光线投射**（不需要相机/RT/GPU），
组件**运行时无行为**，烘焙完可删。

> ⚠️ 要点：阴影是**烘焙那一刻的姿态**（换姿势/改网格/改光源方向都要重烘）；数据写在**材质**里
> （多对象共享材质就会共享结果）；贴图必须保持"非 sRGB / 不压缩 / 无 mipmap"。
> 完整注意事项（10 条）见包内 **`SelfLight.md`**。

## 0.3.0 新增

### 1. 汉化补齐（这次修的是"看着还有一半是英文"）

ShaderCore 的显示名靠 `lang/*.po` 查表，而**手工写在 `.scshader` 里的 ShaderLab 属性**
（模板测试 / 渲染 / 描边偏移那一整块）之前根本没进 po —— 折叠标题是中文、**里面的
`Ref / Comp / Pass / Cull / SrcBlend / ZWrite / AlphaToMask / Outline Offset…` 全是英文**。现在：

- 补全这些 + `Off/Front/Back` 枚举标签 + **模块折叠标题**（ShaderCore 用 `module.name` 当标签）
  + ShaderCore 会在当前表里查的通用键（右键菜单 `复制/粘贴/重置/还原`、渲染队列 `几何体/镂空/半透明`、锁定提示）
- 13 个模块的 po 都补了通用键：**着色器侧 146 条**
- 工具包新增 `NTL10n` + `lang/zh-Hans.po`：**151 条**，覆盖窗口 / 报告 / 日志 / 进度条 / 安装状态 /
  SelfLight 检视面板（字段标签也是自己画的）；**未命中的 key 原样显示英文**（英文 fallback）
- 菜单项无法走 po（`[MenuItem]` 要编译期常量），直接写成中英双语

### 2. SelfLight v2：PCSS 软阴影（参考 nHaruka 的 PCSS4VRC）

| | PCSS4VRC「真实影システム」 | 本分支 SelfLight v2 |
|---|---|---|
| 手段 | 真·Spot Light + 定制着色器 | 私有光 + 烘焙深度图，**没有 Light 组件** |
| VRChat 性能等级 | Lights = 1 → PC 上最高 **Poor** | Lights 保持 **0** |
| 外溢 | 会照亮世界/他人 | 完全不外溢 |
| Quest | 官方说明**不支持** | 可用（固定采样、无动态循环） |

移植过来的是：**PCSS**（blocker search → 变半径 PCF，固定 8+12 次采样）、**Shadow Distance**
（默认 10m 自动关闭）、**ReceiveMask**（逐像素控制哪里接收阴影），另外加了 **阴影浓度** 与
**Shadow Clamp**（把软边压成硬边，动画风）。PCSS 关掉就是 1 次采样的硬阴影。
**CastMask 没做**（我们的深度图是烘焙时 CPU 光栅化的），详见 `SelfLight.md`。

### 3. lilToon 逐像素阴影遮罩可转换

以前只能对 `_ShadowStrengthMask` / `_ShadowBorderMask` / `_ShadowBlurMask` **发警告**，现在
**同名属性、同通道语义**直接迁移（Strength 用 `.r`；Blur/Border 用 `.rgb` 对应第 1/2/3 层），
转换器自动拷贴图 + 打开 `Use Shadow Masks`。开关默认**关**，关着时一次采样都不做。

## 0.3.1 新增（性能不再当约束）

VRChat 性能等级不重要时（avatar 本来就是极高负载），之前为 `Lights = 0` 与省算力放弃的东西都拿回来：

- **PCSS 画质四档**：低 8+12 / 中 12+24 / 高 20+40 / **极高 32+64** 次采样；烘焙分辨率上限 1024 → **2048**。
  深度图采样改成显式 LOD 0，顺带消掉了 `gradient instruction used in a loop` 的编译警告。
- **实时光源模式**（PCSS4VRC 走的那条路）：给 avatar 真挂一盏 **Spot Light**，影子**实时**（改姿势立刻变、不用烘焙）。
  「创建 / 同步实时光源」会自动：
  1. 建 Spot Light（软阴影；角度/范围/颜色/强度/阴影强度来自组件字段）
  2. **Culling Mask 默认第 10 层 `PlayerLocal`** → 不照世界、不照其他玩家（每端只有自己的 avatar 在这个层上）
  3. 把层级里所有 Renderer 的 **Receive Shadows / Cast Shadows** 打开（有些 avatar 默认是关的，关了就不出影子）
  4. 关掉材质里的 `_UseSelfLight`，避免"真光源 + 着色器私有光"双份打光
  5. 配置不合理（没选 PlayerLocal / 范围 > 8m）时面板给警告

  代价（面板上也写了）：占 Lights 计数；影子能不能看到取决于**观看者**的 Shadow Quality；多盏这种光会叠加照白；
  Quest 基本不可用。**默认仍是烘焙模式**，两条路可以随时切换。

### 顺带补上一个验证缺口

0.3.0 的「shader 无报错」当时是**假绿** —— `-nographics` 批处理不编译 shader 变体，
`ShaderUtil.GetShaderMessages` 对故意写错的着色器也报 0 条。现在改成**构建 AssetBundle**
（真正调 `UnityShaderCompiler`）并加**负向对照**：故意写错的着色器必须被报错，否则本探针结论作废。
结果：对照被报错 ✓，NonToon / NonToonFur **0 编译错误、0 警告**。

## 这个构建相对官方 NonToon 0.1.3 改了什么

> 上游自 0.1.3（2026-07-14）之后**一行未改**，本仓库的 issue 修复均领先于官方。

### 对照表（官方 0.1.3 → 本 fork 0.1.9）

| 能力 | 官方 0.1.3 | 本 fork |
|---|---|---|
| **半透明** | **完全失效**：Transparent 被连续两次 `clip()` 打成 Cutout，无 alpha 混合（#11） | 真正做 alpha 混合（采纳未合并 PR #13） |
| **透明排序** | 队列 `2460`（不透明区间） | `3000`（Transparent 区间） |
| **阴影颜色** | 只有渐变 ramp | `ShadowColor` 模块：lilToon 式 1/2/3 层阴影色 + 边框/模糊/强度/对比度 |
| **发光** | 没有（`Lighten` 只是亮度乘数，不是自发光） | `Emission` 模块，4 种混合模式，在 `postpixel` 阶段 → 不受阴影衰减 |
| **光照调整** | 结果被硬编码 `saturate()` | `_LightMinLimit` / `_LightMaxLimit` / `_MonochromeLighting` / `_AsUnlit`（同 lilToon 名与默认值） |
| **受光方向** | 写死「真实光照 + 视线×1.5」，侧光下渐变与直射光会打架（#9） | `_ShadeDirectionBias`，设 `0` 即完全跟随真实光照 |
| **VR 里的 MatCap** | 双眼贴图相同，反射像"贴"在模型上（PR #12） | `VR Parallax Strength`，默认 `1` = 逐眼采样 |
| **大网格 + 近平面修正** | 只能全局关，覆盖屏幕的大网格会整个消失（#8） | 逐材质 `Enable` 开关 |
| **遮罩通道** | 只能 R/G/B/A | **8 档**，含 `1-R / 1-G / 1-B / 1-A`（#10） |
| **lilToon 迁移** | 只能手工重做 | 内置转换器（生成新材质、不动原材质） |
| **界面语言** | 日文 / 英文 | 新增简体中文（11 个 `.po`，含转换器 UI） |
| **环境光穿模 / 光照衰减 / 镜面高光** | 各有偏差 | 与 URP 对齐、高光受阴影衰减、F0 可调 |
| **描边** | 无可调深度偏移；pass 里 8 次帧深度采样是死代码 | `_OutlineOffsetFactor` / `_OutlineOffsetUnits`；死代码已删 |

**体量对比**：文件 132 → **188**；属性声明 95 → **134**（Unity 实测每着色器属性数：NonToon **125** / NonToonFur **111**）。

> 想自己 diff 的话注意：**官方 release zip 是 CRLF、本 fork 是 LF**（源自上游 Git 仓库源码）。
> 用 `diff -rq --strip-trailing-cr -x '*.meta' <官方0.1.3> <本fork>` 只会看到 **21 个**内容变更文件；
> 不加 `--strip-trailing-cr` 会看到 51 个 —— 多出来的 30 个只是换行符不同（功能无影响）。

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
- **遮罩通道反向（对应官方 issue #10）** —— 10 个遮罩通道选择器从 4 档扩到 **8 档**：
  `R / G / B / A / 1-R / 1-G / 1-B / 1-A`。
  这样「要遮罩」和「要反向遮罩」的两个功能可以共用同一条遮罩通道，
  缓解共享遮罩只有 RGBA 四条通道的拥挤。**值 0–3 与旧行为完全等价**，已有材质观感不变
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

## 关于包 id 与显示名

| | 值 | 能不能改 |
|---|---|---|
| 包 **id**（`name`） | `jp.lilxyzw.nontoon`（与官方相同） | **不能改** —— 相同 id 才会被当成官方版的**升级**；改成独立 id 会与官方版**并存**，工程里出现两个同为 `Shader "NonToon"` 的着色器，还可能让材质指向错的那个 |
| **显示名**（`displayName`） | `NonToon (Fork)`（0.1.9 起） | 可以改，只影响 ALCOM/VCC 里的显示，不影响解析与安装 |
| 着色器名（`Shader "NonToon"`） | `NonToon`（未改） | 不改 —— 代码与第三方工具里有 `Shader.Find("NonToon")` 之类的按名查找，改名会连带出问题 |

所以本分支的做法是：**id 沿用官方、显示名加 `(Fork)` 后缀**。
这样它会被视为官方版的升级，同时你在包列表里一眼能看出装的是哪一个。

> 0.1.6 / 0.1.7 / 0.1.8 的显示名仍是 `NonToon`（与它们 zip 内的 `package.json` 保持一致），
> 从 **0.1.9** 起才是 `NonToon (Fork)`。

## 验证状态

打包前已在 Unity 2022.3.22f1 批处理模式下实测：C# 与着色器**零错误**，
`NonToon` / `NonToonFur` 属性表分别为 **125 / 111** 项（0.1.7 新增的三个属性、0.1.8 的 8 档
遮罩通道枚举均导入正常），两个 `.scshader` 预编译 `ok=1`，
zip 内 188 个文件全部通过 CRC 校验、条目名全部为正斜杠。

**未验证**：shader **变体编译**与实际渲染观感（批处理环境不编译变体、也没有画面）。
若控制台出现 `Shader error in 'NonToon'`，请把报错原文发我；否则请以你在编辑器/VR 里的目视为准。

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
