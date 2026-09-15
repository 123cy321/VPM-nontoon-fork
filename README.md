# NonToon (Fork)

给 **VRChat avatar** 用的卡通着色器，基于 [lilxyzw/NonToon](https://github.com/lilxyzw/NonToon) 改进。

上游版本停在 0.1.3 之后未再更新。本分支修复了它遗留的问题（半透明、透明排序、侧光下渐变冲突等），
补上了 **lilToon 材质一键迁移**，增加了**自带光源 / 自带阴影**（地图无光源时仍可正常显示），
并完成**界面汉化**（165 条面板属性中 117 条为中文；0.3.7 起无需手动切换语言）。

| 项 | 值 |
|---|---|
| 一键添加 | https://catandling.github.io/VPM-nontoon-fork/ （点页面上的按钮） |
| 手动添加用地址 | `https://catandling.github.io/VPM-nontoon-fork/vpm.json` |
| 当前版本 | 着色器 **0.3.7** ／ 工具包 **0.4.8** |
| 环境 | Unity 2022.3 ／ **BiRP（VRChat）** ／ PC 与 Quest |

> **关于账号改名**：作者账号由 `123cy321` 改为 `CatAndLing`。GitHub Pages 不做重定向，旧地址已失效。
> 曾经添加过旧地址的，请在 ALCOM / VCC 里重新添加上表中的地址（仓库 id 未变，不会产生重复项）。

## 文档导航

本文分两部分，按读者区分：

**第一部分　普通用户** —— 面向使用 avatar 的玩家与改模作者

- [1. 安装](#1-安装)
- [2. 五分钟上手](#2-五分钟上手)
- [3. 光源组件详解](#3-光源组件详解)
- [4. 材质面板功能总览](#4-材质面板功能总览)
- [5. 性能与代价](#5-性能与代价)
- [6. 已知限制](#6-已知限制)
- [7. 常见问题](#7-常见问题)

**第二部分　专业开发者** —— 面向着色器/工具开发者与需要深入排障的人

- [8. 包结构与依赖](#8-包结构与依赖)
- [9. 着色器技术细节](#9-着色器技术细节)
- [10. 实测性能数据](#10-实测性能数据)
- [11. 平台与运行时约束](#11-平台与运行时约束)
- [12. 兼容性与集成](#12-兼容性与集成)
- [13. 验证方法与质量标准](#13-验证方法与质量标准)
- [14. 修改指引与已知实现坑](#14-修改指引与已知实现坑)

**附录**：[版本记录](#附录-a-版本记录) ／ [依赖与许可](#附录-b-依赖与许可) ／ [数据与引用来源](#附录-c-数据与引用来源)

---

# 第一部分　普通用户

## 1. 安装

**方式一（推荐）：一键添加**

打开 https://catandling.github.io/VPM-nontoon-fork/ ，点页面上的按钮，VCC 会弹出「添加仓库」确认框。

> 为什么 README 里不直接放可点的 `vcc://` 链接：GitHub 的 Markdown 渲染器会过滤非 http(s) 协议，
> 只保留文字、删除链接。需要手动触发时，把下面这行粘到浏览器地址栏：

```text
vcc://vpm/addRepo?url=https://catandling.github.io/VPM-nontoon-fork/vpm.json
```

**方式二：ALCOM / VCC 手动添加**

在「添加仓库 / Add Repository」里粘贴 **listing 地址**（`vpm.json`，不是 zip 文件地址）：

```text
https://catandling.github.io/VPM-nontoon-fork/vpm.json
```

仓库添加后，包列表里有两个包：

| 包 | 是否需要 | 内容 |
|---|---|---|
| `NonToon (Fork)` | 必装 | 着色器本体 |
| `NonToon (Fork) Tools` | 建议 | lilToon 材质转换器 + 光源工具 + 自阴影烘焙。可选，不影响着色器 |

依赖是单向的：装工具包会自动带着色器；只装着色器不会带工具包。

## 2. 五分钟上手

### 2.1 界面语言

0.3.7 起无需任何操作，中文会自动生效。

原因说明（供排查用）：中文 Windows 上 ShaderCore 的默认语言是 `zh-CN`，而旧版本包只附带 `zh-Hans.po`，
两者文件名对不上，导致 165 条面板属性中只有 35 条显示中文。0.3.7 起本包会把中文语言表补齐成当前语言的那一份，
覆盖率提升到 117 条。

### 2.2 从 lilToon 迁移材质

菜单：`Tools/lilToon → NonToon 转换器`（也可右键 Hierarchy 里的模型或 Project 里的材质）。

- 生成**新材质**，不会修改原有的 lilToon 材质
- 转换报告逐条列出：哪些是 1:1 迁移、哪些是近似、哪些未支持
- 1:1 迁移项：光照上下限 / 单色 / 无光照、1/2/3 层阴影色与边界模糊、逐像素阴影遮罩、Emission 全套
- 近似项：MatCap、边缘光、距离淡出、毛发、描边。转换后建议目视检查并微调

### 2.3 地图太黑或地图光照不合适时的三种方案

| | Avatar 光源 | 自有光源 | 亮度自适应（插件） |
|---|---|---|---|
| 一句话 | 挂一盏真实 Unity 灯 | 着色器内加一路私有光 | 诊断 + 亮度预设 + 游戏内径向 |
| 是否需要改材质 | 不需要（任何着色器都吃） | 需要 | 插件自动写入 |
| 阴影 | 实时，跟随姿势 | 烘焙，固定姿态 | 沿用自有光源 |
| 代价 | 占用 VRChat `Lights` 计数，性能等级最高 Poor | 几乎为零 | 无额外采样代价 |

三种方案的详细参数与适用场景见下一节。

## 3. 光源组件详解

### 3.1 前提：VRChat 中挂在 avatar 上的自定义脚本不会运行

VRChat 官方文档 [Allowed Avatar Components](https://creators.vrchat.com/avatars/whitelisted-avatar-components/whitelisted-avatar-components/) 原文：

> "Any component on the following list can be used in VRChat.
> Other components or custom scripts won't work in VRChat and may stop you from uploading your avatar."

白名单只包含 `Animator`、`Animation`、`Light`、`Renderer`、`VRCAvatarDescriptor`、PhysBone 等，**不包含自定义脚本**。

这决定了本包所有光源工具的设计方式：

| 你看到的东西 | 实际身份 | 进入 VRChat 后 |
|---|---|---|
| `NTAvatarLight`（第一种组件） | 编辑器期的设置载体 | 组件本身不运行，但它创建的 `Light` 照常工作（`Light` 在白名单内） |
| `NTSelfLight`（第二种组件） | 编辑器期的设置载体，把参数与烘焙结果写入材质 | 组件本身不运行，材质里的参数与阴影贴图照常工作 |
| 插件生成的 Animator 参数 / 动画剪辑 / 径向菜单 | 均为白名单内资产 | 正常工作。这是"游戏内可调"唯一可行的实现方式 |

由此得到两条实用结论：

- 上传时若 SDK 提示存在不允许的组件，**移除这些组件不影响效果**，插件面板中有「上传前清理」可一键移除
- 想在游戏内调亮度，只能通过 Animator + 动画剪辑 + Expression 参数实现

### 3.2 Avatar 光源（不需要改材质）

菜单：`GameObject/NonToon/① 创建 Avatar 光源（不用改材质）`

它会在指定节点下创建一盏真实的 Unity `Light`：

- **与着色器无关**：lilToon、Poiyomi、NonToon、Standard 都会被照到
- **Culling Mask 默认第 10 层（PlayerLocal）**：VRChat 中只有本地玩家自己的 avatar 在该层，
  因此它不会照亮世界，也不会照亮其他玩家
- 自动开启层级内所有 Renderer 的 `Receive Shadows`（部分 avatar 默认关闭，关闭时不出影子）
- 支持色温（直接使用 Unity 的 `Light.colorTemperature`，6500K 约为白色）

| 参数 | 说明 |
|---|---|
| 光源组件 | 由创建器生成的那盏灯，一般无需手动修改 |
| 颜色 / 使用色温 / 色温(K) | 光色。启用色温后 `color` 作为乘算 |
| 光强 | 默认 1.5 |
| 聚光灯角度 | 越小越集中，影子越干净 |
| 照射范围（米） | 只照自己时调小，避免影响他人 |
| 阴影类型 / 浓度 / 偏移 | 推荐 `Soft`；出现条纹状噪点（阴影痤疮）时增大偏移 |
| Culling Mask | 默认 `1<<10`（PlayerLocal） |
| 渲染模式 | `ForcePixel` 阴影质量最好 |
| 光的方向来源 | 留空时使用本节点朝向；需要 PhysBone 控制方向时，把本节点挂到被驱动的那根骨骼下 |

代价（组件面板中同样会提示）：

- 占用 VRChat 性能等级的 `Lights` 计数。PC 端 Excellent / Good / Medium 均要求 `Lights = 0`，
  仅 Poor 允许 1，因此**挂一盏实时光后性能等级最高只能到 Poor**
- 需额外渲染一张阴影贴图，每个能看到你的人都承担这份开销
- 影子是否可见取决于观看者的 Shadow Quality 设置；超出 Unity 的 Shadow Distance 后没有影子
- 多人同时使用这类光源会互相叠加，拥挤时可能过曝
- Quest 端基本不可用

### 3.3 自有光源（需要在材质启用）

菜单：`GameObject/NonToon/② 创建自有光源（需要在材质启用）`

实现方式不同：**在 NonToon 着色器内增加一路私有光**，只有该材质可见。

- 他人看不到你的光，也不会照亮世界
- 阴影来自**烘焙的深度图**（非实时阴影），始终可见，Quest 可用
- 代价接近于零：默认 0 采样（强度为 0 或未启用 PCSS 时）

材质面板中对应「Self Light」分组，组件面板可调整同一批参数（组件会写入材质）：

| 参数 | 说明 |
|---|---|
| 自有光源（开关） | 开启后才生效；第二种组件会自动开启 |
| 只接受光源编号（Light ID） | 0 表示任意。同一模型可并存多路自带光，用编号让材质只接受指定那一路；组件写入时会自动打上编号 |
| 颜色 / 使用色温 / 色温(K) | 1000–20000K，6500K 约为中性白 |
| 强度 / 方向（世界空间，指向光源） | 方向即"光从哪来" |
| 只由它照亮 | 丢弃世界光、环境光、光照贴图、天空盒反射，avatar 在任何地图中观感一致 |
| 屏蔽地图环境光（0–1） | 只想压掉环境光、保留地图太阳时使用（近似处理） |
| 匹配世界光颜色 / 方向 | 探测地图环境光颜色与主光方向，让自带光跟随。读取的是着色器已有的数据，不增加采样 |
| 阴影强度 / 阴影颜色 / 阴影强度遮罩 | 阴影处的染色，以及逐像素控制哪里影子深 |
| 接收遮罩 | 逐像素控制哪里接收阴影（例如眼白不希望被刘海投影，可涂黑） |
| PCSS 软阴影 + 画质四档 | 低 20 / 中 36 / 高 60 / 极高 96 次采样每像素 |
| 阴影硬化（Clamp） | 把软边压成硬边，适合动画风格 |
| 阴影距离 | 超出设定距离后自动关闭 |
| 自备阴影贴图（Map 组） | 自行提供灰度深度图，并填写原点/三轴/半宽高/远近平面 |

组件面板上有两个按钮：

- **只同步参数（不烘焙）**：只把参数写入材质
- **烘焙自阴影并写入材质**：用 CPU 光线投射生成深度图后写入（PCSS 可到极高，贴图可到 2048）

> 影子对应的是"烘焙那一刻的姿态"。VRChat 内无法做到实时自阴影（avatar 不能带相机与渲染纹理），
> 更换姿势后重新烘焙即可。

### 3.4 亮度自适应与防煤（编辑器插件）

菜单：`Tools/NonToon/③ 亮度自适应与防煤（编辑器插件）`

解决的问题有两个：**在全黑地图里 avatar 变成一块煤**，以及亮度偏好因人而异需要**在游戏内随手调整**。

该插件是一个编辑器窗口，不会往 avatar 上挂任何组件。

**（1）场景光照诊断**

读取当前场景的环境光（三种模式）、平行光数量与亮度、着色器实际使用的 SH L0，给出 0–1 的世界亮度估算，
并判断"在这种图里会不会变煤"。同时提供 **「模拟全黑地图」**：临时把主光压到 0.02、环境光压到 0.01，
便于目视验证自己的 avatar 在极暗环境中的表现；查看后点「还原原来的光照设置」。

> 该读数来自你当前打开的场景。VRChat 中各地图光照不同、脚本又不运行，因此真正的保障是下面的
> 亮度预设与游戏内径向。

**（2）亮度预设**

一键把选中材质写成以下配置：

| 按钮 | 写入内容 |
|---|---|
| 防煤（下限 0.35） | `_LightMinLimit = 0.35`，上限 1 |
| lilToon 推荐（0.15） | `_LightMinLimit = 0.15` |
| 明亮（0.5） | `_LightMinLimit = 0.5` |
| 压过曝（上限 0.85） | `_LightMaxLimit = 0.85`，用于地图过亮/白飞 |
| 还原官方默认（0 / 1） | 恢复默认值 |

此外可直接调节：单色化（地图灯光颜色异常时去掉彩度）、Unlit 化（越接近 1 越接近 Unlit、底色更亮；
需注意自己变亮的同时，相对地周围的人会显得更暗）。

**（3）兜底：自有光源（可选）**

需要"完全不受地图影响"时使用：开启着色器自有光、设置强度，可选"只由它照亮"或"屏蔽地图环境光"。
代价参见 3.3。

**（4）生成游戏内可调的 VRChat 参数**

点击按钮后在 `Assets/NonToonLightAdjuster/` 下生成：

| 生成物 | 内容 |
|---|---|
| Animator Float 参数 | 默认名 `NT_Light` |
| 两条曲线剪辑 | `..._低.anim` / `..._高.anim`，曲线绑定 Renderer 的 `material.<所选属性>` |
| 一层 Animator 层 + 1D BlendTree | 由 `NT_Light` 在两条剪辑间混合（阈值 0 / 1） |
| VRC Expression Parameter | Float，默认 0，占 8 bit 同步内存 |
| Expression Menu 径向控件 | 名称「光照强度」，类型 Radial Puppet，指向 `NT_Light` |

可选择驱动哪个属性：

| 选项 | 适用场景 |
|---|---|
| `_LightMinLimit`（默认） | 防煤主力：径向即亮度下限，0 为官方默认（暗图会黑），1 为很亮。零采样代价 |
| `_LightMaxLimit` | 压制过曝 |
| `_SelfLightIntensity` | 使用自有光源方案时（需先开启自有光源） |

生成后，玩家戴上 avatar 便可在 Action Menu 中拉动径向自行调节亮度。

参数开销（依据官方文档）：`Float` 占 8 bit，VRChat 自定义同步参数上限 256 bit；
关闭「同步给别人看」可省下 8 bit，但只有自己能看到变化。`Radial Puppet` 的取值范围是 0.0–1.0。

> 未安装 VRCSDK 时，插件只生成 Animator 部分，并会逐条列出手动创建参数与菜单的步骤。

**（5）上传前清理**

统计模型上本包助手组件的数量，并可一键移除。移除不影响效果（那盏 `Light` 与材质中的烘焙结果均保留）。

### 3.5 三种方案怎么选

| 你的情况 | 建议 |
|---|---|
| 不用 NonToon 着色器（例如 Poiyomi），只想补一盏灯 | Avatar 光源 |
| 只用 NonToon，地图有时很黑 | 先做亮度预设（零代价），仍不满意再加自有光源 |
| 在意性能等级，或要上 Quest | 亮度预设 + 自有光源（烘焙阴影）；不要用实时光源 |
| 希望玩家自己能调亮度 | 用插件生成径向参数 |
| 需要 avatar 在任何地图里观感完全一致 | 自有光源并勾选「只由它照亮」 |

## 4. 材质面板功能总览

打开任意 NonToon 材质即可看到以下分组。

**主色与基础**

| 功能 | 说明 |
|---|---|
| 渲染模式 | 不透明 / 镂空（Cutout）/ 半透明 |
| 主贴图 + 共享遮罩 + 共享渐变 | 一张遮罩图可供所有模块按通道取用 |
| 法线贴图 + 粗糙度 + 裁剪阈值 | 支持从法线贴图中取粗糙度 |
| 反照率遮罩（RGBA 四通道） | 单个模块控制四个区域 |

**阴影（1/2/3 层）**

- 三层阴影色，每层包含颜色、边界（Border）、模糊（Blur）
- 逐像素阴影遮罩：一张图的三个通道分别控制三层阴影的强度/边界/模糊
- 阴影主色强度、边界颜色与范围、AA 强度
- 上述项目可从 lilToon 材质 1:1 转换

**光照调整**

| 项 | 作用 |
|---|---|
| 光照上下限（Min/Max Limit） | 把受光强度限制在区间内，控制整体明暗对比 |
| 单色光照（Monochrome） | 去掉光的颜色、只保留明暗，避免被地图灯光染色 |
| 无光照（As Unlit） | 完全不受光，始终满亮 |
| 阴影方向偏置 | 修正"侧光下渐变冲突" |

**其他模块**

描边（含 Offset Factor/Units、从顶点色取宽度）、边缘光、背面光、MatCap（含 VR 视差）、
发丝高光、边缘阴影（RimShade）、细节贴图（4 通道）、距离淡出、Nearer（VR 内防穿模）、
Emission 全套（颜色/贴图/混合模式/混合遮罩/主色强度）、毛发（NonToonFur）。

## 5. 性能与代价

### 5.1 默认均为最低开销档位

每像素自阴影采样数：

| 配置 | 采样 / 像素 |
|---|---|
| 自有光源关闭（默认） | 0 |
| 自有光源开启、强度 0（只要光不要影） | 0 |
| 自有光源开启、PCSS 关闭（硬阴影） | 1 |
| PCSS 低（默认） | 20 |
| PCSS 中 / 高 / 极高 | 36 / 60 / 96 |
| 接收遮罩 / 阴影强度遮罩 | 各 +1（默认 0，不采样） |

采样仅在光源包围盒内且位于阴影距离之内时发生。组件面板会直接显示当前预算。

### 5.2 实时光源的代价

- 额外渲染一张阴影贴图，每个能看到你的人都承担这份开销
- 占用 VRChat 的 `Lights` 计数，PC 端性能等级最高只能到 Poor
- Quest 端基本不可用

### 5.3 使用注意事项

- 第一种（Avatar 光源）与第二种（自有光源）**不要同时开启**，会双份打光。
  第二种组件的实时光源模式会自动关闭材质里的自有光源并给出警告
- 自有光源的阴影是静态的：更换姿势或服装后需重新烘焙
- 自备阴影贴图的约定：灰度为"离光源近平面的归一化距离"；贴图需为非 sRGB、不压缩、Clamp、无 mip
- 渲染队列：`NonToon.scshader` 的两个 SubShader 都没有声明 `Queue` 标签（`NonToonFur` 有 `AlphaTest`），
  队列由编辑器写入。因此通过脚本创建或复制的材质不会自带队列，半透明可能显示异常，需手动设置 Queue
- 面板中仍有约 48 条英文：`SrcBlendRGB`、`ZWrite`、`Ref`、`Comp`、`Pass`、`Cull`、`AlphaToMask` 等。
  这些属于 ShaderCore 的底层属性，本包无法覆盖
- 与优化工具（AAO / MA）共用时的注意事项见 5.4

### 5.4 与 AAO / MA 等工具共用时的注意事项

**AAO（Avatar Optimizer）**

- 可以一起使用，外观不会出问题
- 使用插件生成的亮度径向时，请不要手动把两条剪辑改成"每个材质不同的值"：
  AAO 的网格合并不支持差异化材质动画，会给出警告且可能失效。插件默认对每个 Renderer 写入同一条曲线、同一个值，属于受支持的情况
- 使用 AAO 的材质合并 / UV 打包时，NonToon 目前不会参与这些优化（外观不受影响，只是少一层优化）。
  愿意用图集的话，该部位可以改用 lilToon
- AAO 的材质合并本身不支持视差（Parallax）与 UV 滚动类功能

**MA（Modular Avatar）**

- `Remove Vertex Color`：若同时开启了「从顶点色取描边宽度」（默认关闭），顶点色被移除后描边方向会走形
- `Merge Animator`：其路径默认相对组件本身，因此不要用它来装载插件生成的 FX 控制器。
  插件生成的控制器是直接写入 avatar descriptor 的 FX 层的，这是正确用法
- `Material Swap / Material Setter`：可以共用。运行时亮度径向仍会写当前材质的属性，
  但换上的材质不会保留其面板中的亮度值
- 菜单与参数：插件生成的 Expression Parameter 与菜单控件是标准资产，可与 MA 的菜单系统共存（注意不要重复添加同名参数）

### 5.5 Quest

| 项 | 结论 |
|---|---|
| 着色器本体 / 采样器 | 可以使用。采样器去重后仅 7 个，远低于 GLES3 保证的 16 个 |
| 毛发（NonToonFur） | 仅限 PC。其包含 7 个几何着色器 pass，GLES3 不支持几何着色器 |
| 实时光源 | 基本不可用 |
| 自有光源的烘焙自阴影 | 可用 |
| 插件生成的 Animator 参数 | 可用（均为白名单内资产） |

## 6. 已知限制

- **只在 BiRP（VRChat）上保证可用**。URP 分支在 Unity 2022.3 上属于不生效的代码路径
- **VRChat 内无法做到实时自阴影**：avatar 不能带相机与渲染纹理，自阴影只能是"烘焙那一刻的姿态"
- 转换是近似的，以下功能**未移植**：贴花（Decal）、溶解（Dissolve）、闪烁、荧光、宝石（Gem）、
  折射、AudioLink、UV 动画、视差深度、`_Emission2nd*`、渐变发光。转换器会警告并跳过
- `NonToon.scshader` 未声明 `Queue` 标签（见 5.3）
- 上游 issue #7（毛发在 Radeon / 较老 N 卡上异常膨胀）**未处理**；上游作者说明只能缓解、无法根治
- 面板中仍有约 48 条英文（ShaderCore 的底层属性，见 5.3）
- 实时光源与 VRChat 性能等级存在冲突：PC 端最高只能到 Poor。这是 VRChat 的规则，非本包缺陷
- 插件生成的材质属性曲线，其**运行时行为未在本机验证**（原因见 13），需要在 VRChat 内实测

## 7. 常见问题

**问：装上去会不会和官方 NonToon 并存？**
不会。包 id 与官方相同，因此是升级官方版，不会同时出现两个 `Shader "NonToon"`。

**问：升级到 0.3.7 后，场景里某个组件变成 `Missing (Mono Script)`？**
0.3.6 及更早的包中，Avatar 光源的脚本文件漏带了 `.meta`，以致 Unity 在每台机器上生成不同的 GUID；
挂好组件后换机器打开或重装该包，引用就会断开。0.3.7 已修复。
处理方式：删除变成 Missing 的组件，重新挂一次并重填参数。自 0.3.7 起 GUID 已固定，后续升级不会再断。

**问：上传时 SDK 提示存在「不允许的组件」？**
本包的第一、二种组件是编辑器期的设置载体，VRChat 中不会运行（官方白名单不允许自定义脚本）。
它们的效果已经落在"那盏 Light"和材质中的参数/贴图上，移除组件不影响效果。
插件面板中有「上传前清理」可一键移除。

**问：我已经选了简体中文，为什么面板里还有英文？**
0.3.7 起无需手动选择。剩余约 48 条属于 ShaderCore 自身的底层属性，本包无法覆盖。

**问：转换后观感不太一样？**
转换是近似的（MatCap、边缘光、距离淡出、毛发、描边尤为明显）。请查看转换报告中标记为「近似」的条目，
或参考工具包中的 `Mapping.md`。

**问：老是提示 ShaderCore 模块缺失？**
本包会在编辑器加载时自动补注册并重导着色器（ShaderCore 的模块表只扫描一次，升级后可能静默丢失模块）。
控制台出现 `[NonToon] ... 补注册模块` 即表示该机制在工作。

**问：地图里的光很难看，不想每次都去调材质？**
用插件：先点「防煤（下限 0.35）」写入材质，再生成径向参数，游戏内可随手调整。
也可以使用自有光源的「匹配世界光颜色 / 方向」，或直接勾选「只由它照亮」与地图光照完全解耦。

**问：从很老的版本升级后，菜单里出现两个转换器？**
请将 `Packages/jp.lilxyzw.nontoon` 整个删除后重新安装一次。

---

# 第二部分　专业开发者

## 8. 包结构与依赖

本仓库发布两个包，依赖为单向：

| 包 | id | 内容 | 依赖 |
|---|---|---|---|
| 着色器 | `jp.lilxyzw.nontoon` | `Shaders/`（ShaderCore 着色器定义、模块、本地化）+ `Editor/`（模块注册自愈、ShaderCore 中文补全） | `jp.lilxyzw.shadercore >= 0.1.9` |
| 工具包 | `com.123cy321.nontoon-converter` | `Runtime/`（光源组件）+ `Editor/`（lilToon 转换器、光源与自阴影工具、亮度自适应插件） | `jp.lilxyzw.nontoon >= 0.2.0` |

设计约束（长期有效的决定，改动前请先评估）：

- **包 id、着色器名（`Shader "NonToon"`）、VPM 仓库 id 均保持不变**。前两者影响已有材质与按名查找的代码；
  仓库 id 是 VPM 识别"同一仓库"的键，改动会让已添加该仓库的用户多出一条失效条目
- 工具包**不引入 VRCSDK 硬依赖**：所有对 VRC 类型的访问都通过反射完成，未安装 SDK 时降级为提示手动步骤
- 亮度自适应插件是**编辑器窗口**，不向场景添加组件。原因见 3.1：avatar 上的自定义脚本不会运行

## 9. 着色器技术细节

**ShaderCore 模块与相位**

着色器由 ShaderCore 在导入时把模块内容拼入 `NonToon.scshader` 中的占位符，编译的是拼接后的结果：

- 模块位于 `Shaders/Modules/<Name>/`，包含 `*.scmodule`（`name` / `uniqueID` / `keepPropertyNames`）、
  `properties.hlsl`、`phase_<相位>.hlsl`，可选 `includes.hlsl`
- 相位占位符：`light`、`customlight`、`base`、`modifylight`、`shade`、`reflection`、`add`、`postpixel`、`postvertex`
- **同一相位内的注入顺序由 `module.name` 的字符串序决定**。因此模块名不能随意改成中文：
  `shade` 相位内有 RimShade、Shade、ShadowColor 三个模块，顺序敏感
- 当前相位占用：`base` = Details；`postvertex` = Nearer；`light` = Specular；`reflection` = MatCaps；
  `customlight` / `modifylight` = SelfLight（后者同时含 Lighten）；`shade` = RimShade + Shade + ShadowColor；
  `add` = HairSpecular + RimLight；`postpixel` = DistanceFade + Emission
- `NonToon/Shaders/birp.hlsl` 的 fragment 被 ForwardBase、ForwardAdd、Outline、OutlineAdd 共用，
  任何"加光"代码都必须用 `UNITY_PASS_FORWARDADD` 分流，否则会在每个 forward pass 重复叠加

**亮度公式**（与 lilToon 一致，见附录 C 的 lilToon 文档）：

```
RGB = clamp(RGB, _LightMinLimit, _LightMaxLimit);
RGB = lerp(RGB, Mono, _MonochromeLighting);
RGB = lerp(RGB, 1.0,  _AsUnlit);
```

NonToon 的 `_LightMinLimit` 默认为 0，因此无光环境下 clamp 结果为纯黑。这是"全黑地图变煤"的直接原因，
也解释了为什么防煤方案以抬高亮度下限为主（零采样代价），而不是新增光源。

**渲染队列**

`NonToon.scshader` 的两个 SubShader 均未声明 `Queue` 标签，队列由编辑器写入材质。
`NonToonFur.scshader` 声明了 `Queue = AlphaTest`。

**本地化机制**

- 属性标签、折叠标题、枚举标签等通过 ShaderCore 的 po 表本地化；查找发生在**绘制时**
  （`L10n.L(Property.displayName)`），因此 `MaterialProperty.displayName` 始终保留原始键名
- `__` 开头的内置键只在 `Packages/jp.lilxyzw.shadercore/lang/` 下按**精确语言码**查找，
  找不到 `<语言>.po` 时回退 `en-US.po`
- 中文 Windows 默认语言为 `zh-CN`，因此本包会把 `zh-Hans.po` 复制为 `<当前语言>.po`
  并铺入本包全部 lang 目录（仅在缺失时写入，不覆盖上游自带文件，目录只读时静默降级）

## 10. 实测性能数据

以下数据来自对**拼接后的生成源码**的静态审计（脚本化统计，非估算）：

| 指标 | NonToon | NonToonFur |
|---|---|---|
| SubShader / Pass | 2 / 14（每个 SubShader 7 个） | 2 / 14 |
| `#pragma target` 分布 | 2.0 × 2，5.0 × 12 | 2.0 × 2，5.0 × 12 |
| 关键字指令 | 83 条（关键字名 54 个） | 83 条 |
| `SamplerState` 声明 | 112 次，**去重后 7 个** | 同左 |
| 采样调用点 | 150 处（跨 pass 与分支） | 150 处 |
| 几何着色器 | 无 | **7 个 pass** |

说明：

- **采样器去重后仅 7 个**：ShaderCore 让各纹理共用采样器，不随属性数量线性增长。
  OpenGL ES 3.0 规范保证片元着色器至少有 16 个纹理单元，因此采样器数量对 Quest 是安全的
- **URP 分支不会带来额外编译成本**：第一个 SubShader（URP）带有
  `PackageRequirements { "com.unity.render-pipelines.universal": "17.0" }` 与
  `"RenderPipeline" = "UniversalPipeline"`，在 BiRP/VRChat 工程中会被跳过
- 变体总量受 Unity 内置关键字组（shadow / lightmap / instancing 等）影响，实际数量取决于工程设置。
  VRChat 上传时会使用其自身的剥离配置，本机无法模拟其最终体积
- 本条目的数据可由仓库内的脚本复算，方法见 13

## 11. 平台与运行时约束

**VRChat**

- 白名单机制见 3.1：自定义脚本不会运行。本包的光源组件因此只承担编辑器期设置载体的角色
- 性能等级：PC 端 Excellent / Good / Medium 要求 `Lights = 0`，仅 Poor 允许 1
- 实时光源的额外开销：一张阴影贴图的渲染 + 每个观看者各自的 ForwardAdd
- 常见坑：创建实时光源时需要自动开启 `Receive Shadows`（部分 avatar 默认关闭，关闭时没有影子）

**Quest / GLES3**

| 项 | 结论 |
|---|---|
| 采样器数量 | 安全（去重 7 个，规范下限 16） |
| `SampleLevel`、`ddx/ddy`、`Texture2DArray` | 均为 ES 3.0 核心能力 |
| 几何着色器 | GLES3 不支持。NonToonFur 有 7 个几何着色器 pass，因此毛发仅限 PC |
| `#pragma target 5.0` | **未验证**。本体 12 个 pass 使用 5.0（该值继承自上游 0.1.3，非本分支引入）。Unity 中 5.0 要求 DX11 / ES3.1+AEP / Vulkan；Quest 的 GLES3 路径是否存在编译风险，需要在安装 Android 构建支持的环境中验证。对照：lilToon 主要使用 target 3.5 |

**Unity**

- 调试用批处理环境：Unity 2022.3.22f1
- `-nographics` 批处理不会编译着色器变体，`ShaderUtil.GetShaderMessages` 在该模式下也不可靠
  （对故意写错的着色器同样返回 0 条），因此着色器正确性验证依赖 AssetBundle 真编译，见 13
- `Environment.ExitCode` 在 batchmode 下会被忽略（实测设为 7 得到进程退出码 0），
  探针必须使用 `EditorApplication.Exit(code)`，否则失败对命令行不可见

## 12. 兼容性与集成

### 12.1 与官方 NonToon

包 id 相同，因此属于升级关系而非并存。以下为实测依据（2026-09）：

| 项 | 结果 |
|---|---|
| 着色器名 | 仍为 `Shader "NonToon"` / `"NonToonFur"`，`Shader.Find` 等按名查找不受影响 |
| 资源 GUID | 与官方 0.1.3 共有的 78 个带 `.meta` 文件，GUID 全部一致；`Shaders/NonToon.scshader` 为 `78361b0b760724141a2d2c09100cf00f`，与官方逐字符相同 |
| 材质参数 | 官方 0.1.3 的 92 条属性全部保留，另新增 75 条 |
| 模块 | 官方 10 个模块全部保留（details、distancefade、hairspecular、lighten、matcaps、nearer、rimlight、rimshade、shade、specular），另新增 emission、selflight、shadowcolor |
| 新增项默认值 | 均为不影响原有观感的值：自有光源默认关闭、PCSS 默认低、遮罩默认不采样 |

因此升级不会导致已有材质、预制体、场景断引用，也不会丢失参数。

升级路径：

| 起点 | 操作 |
|---|---|
| 官方 0.1.3 或本分支 0.1.4 及以后 | 直接在 ALCOM / VCC 中升级 |
| 本分支 0.1.10 及更早 | 该版本的着色包内嵌了转换器，覆盖安装可能残留两份。将 `Packages/jp.lilxyzw.nontoon` 整个删除后重装 |
| 0.3.6 及更早升级到 0.3.7 及以后 | 可能出现一次组件变为 Missing Script，见第 7 节 |

回退：包 id 相同，安装官方版即替换本分支；listing 保留全部历史版本（着色包自 0.1.6 起）可供回退。
已发布版本不会被删除，VPM 依赖版本列表做解析。

### 12.2 与 AAO（Avatar Optimizer）

以下结论依据 AAO 官方文档与一次真实构建实测（AAO 1.9.16 + NDMF 1.14.3）。

**网格合并与材质动画**

AAO《Merge Skinned Mesh》文档：

> "material-related animations will work without modification."

AAO changelog（PR #769）给出的边界：

> "Merge Skinned Mesh does not support animating material properties differently...
> If you animated all materials from same animations, your animation will not be warned."

本包插件生成的剪辑对每个 Renderer 写入同一条曲线、同一个值，属于受支持的情况。
若手动改成每个材质不同值，则落入 AAO 明确不支持的范围。

**材质合并与贴图优化**

AAO《Shader Information API》文档：

> "Without Shader Information, Avatar Optimizer treats a shader conservatively and cannot perform some of these optimizations."

已由 AAO 内置支持 ShaderInformation 的着色器包括 Standard、ToonLit、ToonStandard 与 **lilToon**，**不含 NonToon**。
因此在 AAO 环境下，NonToon 走保守路径：纹理图集、UV 打包、移除被关闭功能所占用的贴图这几项优化不会执行，
但外观不受影响。

如需补齐，实现方式为：编写 `ShaderInformation` 子类，在 `[InitializeOnLoad]` 静态构造中调用
`ShaderInformationRegistry.RegisterShaderInformationWithGUID("<shader GUID>", instance)`；
asmdef 引用 `com.anatawa12.avatar-optimizer.api.editor`，并用 Version Defines（符号 `AVATAR_OPTIMIZER`，
建议区间 `[1.8,2.0)`）保证未安装 AAO 时仍可编译。
需要注意：该注册要求逐个纹理属性正确声明 UV 通道与变换，**声明错误会导致 AAO 删除实际在用的贴图**，
其后果比不注册更严重，因此尚未实现。

另外，AAO 的材质合并本身不支持 Parallax 与 UV 滚动类功能，本包的 MatCap VR 视差属于这一类。

**Trace and Optimize 对生成资产的影响（已实测）**

用带 `TraceAndOptimize` 组件、且包含插件真实产物的 avatar 调用
`AvatarProcessor.ProcessAvatar` 完成一次完整构建，共 23 项断言全部通过：

| 检查项 | 结果 |
|---|---|
| 动画层 `NonToon LightMinLimit` | 构建后仍存在 |
| Float 参数 `NT_Light` 与 BlendTree 驱动 | 仍存在 |
| `material._LightMinLimit` 曲线 | 仍存在，且 2/2 条绑定均可解析到处理后的 avatar 对象 |
| Expression Parameter `NT_Light` | 仍存在 |
| NonToon 材质属性与贴图 | 未被修改（材质资产、`_BaseTexture`、贴图文件均在） |

构建过程中 AAO 将两个相同材质合并为一个。这正是"材质动画"警告的触发场景，而整个过程没有任何材质动画警告，
与前述 changelog 中"所有材质使用同一条动画则不会警告"一致。

其它观察：

- NDMF 构建会检查 avatar 中所有贴图是否启用 **Mip Streaming**，未启用会在构建报告中报错
  （与本包无关，但排障时需要知道）
- AAO 的 `Optimization Metrics` pass 在 `VRCAvatarDescriptor.specialAnimationLayers` 为 null 时抛出
  `ArgumentNullException`（调用点 `VRCSDKUtils.GetAvatarLayerControllers`）。通过 SDK 正常创建的 avatar
  该字段为 3 条记录，一般不会遇到

### 12.3 与 MA（Modular Avatar）

| 组件 | 与本包的关系 |
|---|---|
| Remove Vertex Color | 与「从顶点色取描边宽度」冲突：本包描边在启用该选项时用 `vertex.color.rgb * 2 - 1` 作为法线（`sc_common.hlsl`），顶点色被移除后描边方向会走形。该选项默认关闭 |
| Merge Animator | 其路径默认相对组件本身。不要用它装载插件生成的 FX 控制器，否则写死的 `material._LightMinLimit` 绑定会指向错误对象。插件生成的控制器直接写入 avatar descriptor 的 FX 层 |
| Material Swap / Material Setter | 可以共用。运行时亮度径向仍会写当前材质的属性，但换上的材质不会保留其面板中的亮度值 |
| Menu Item / Parameters | 插件生成的是标准 Expression Parameter 与菜单资产，可与 MA 菜单系统共存（避免重复添加同名参数） |

### 12.4 与 lilToon 及亮度调整类工具

- **lilToon**：工具包提供单向转换器，生成新材质，不修改原材质
- **Poiyomi / Standard 等**：Avatar 光源与着色器无关，任何着色器均可使用；自有光源仅对 NonToon 材质有效
- **Light Limit Changer（LLC）**：本包亮度插件的设计思路与其一致，即"为亮度上下限生成动画"；
  区别在于插件面向 NonToon，并把诊断、预设与生成集中在同一窗口内

## 13. 验证方法与质量标准

发版前在 Unity 2022.3.22f1 批处理下执行以下检查，**判据是进程退出码**，不是阅读输出文件：

| 检查 | 内容 |
|---|---|
| 离线预检 | `SCProperty` 语法、`.scmodule` JSON、`.po` 重复与畸形行；零命中同样判为失败 |
| 本地化审计 | 抽取全部可本地化字符串与 po 对比；当前为 115 键、缺失 0 |
| 装置 A：AssetBundle 真编译 | 为目标平台实际编译着色器变体，并放入一个**故意写错的着色器作为负向对照**。对照未被报错则整轮结论作废 |
| 装置 B：端到端探针 | 三种渲染模式、阴影色模块、Emission 与光照 1:1、阴影遮罩、自阴影烘焙、两类光源组件、汉化键、默认值预算 |
| 装置 C：亮度插件探针 | 生成资产的结构断言，以及全黑场景下的真实 GPU 渲染对照 |
| 发布物审计 | 线上 listing 与本地逐字节比对；线上 zip 与源码逐文件比对；`.meta` 配对检查；与官方版的 GUID 对齐 |
| 线上核验 | 逐个版本下载并比对 SHA-256，同时校验「版本号 ↔ URL 文件名 ↔ zip 内 package.json」三者一致 |

质量控制中已经踩过并记录在案的失效模式（这些都会造成"通过"的假象）：

- `-nographics` 批处理不编译着色器变体，`ShaderUtil.GetShaderMessages` 对错误着色器返回 0 条
- AssetBundle 增量构建在"无变化"时直接跳过，整轮等于没有编译
- 包目录未同步时，装置 A 仍可能打印"全部通过"（已改为：找不到着色器即判失败，并断言探针材质数量）
- 探针若不预先删除旧输出，崩溃或超时后会残留上一轮的"全部通过"
- `Environment.ExitCode` 在 batchmode 下被忽略
- 发布脚本若版本号取自 `package.json`、而 zip 名另行硬编码，可能出现"版本号与内容不一致仍发布成功"，
  且线上核验因哈希同源而判定一致（已改为三重一致性校验）

已知未验证项（如实标注）：

- 亮度插件生成的材质属性曲线在**运行时**是否确实修改材质：批处理环境下动画预览整体失效
  （连 Transform 位置曲线也无法采样），需在 VRChat 内用 Action Menu 实测
- Quest 上 `#pragma target 5.0` 的编译结果（见 11）
- 注册 AAO ShaderInformation 之后的实际优化效果（见 12.2）

## 14. 修改指引与已知实现坑

**目录**

| 路径 | 内容 |
|---|---|
| `Shaders/NonToon.scshader`、`NonToonFur.scshader` | ShaderCore 着色器定义，含手工 ShaderLab 属性 |
| `Shaders/*_properties.hlsl` | 主属性文件，两个文件内容需保持一致 |
| `Shaders/birp.hlsl`、`urp.hlsl`、`sc_common.hlsl` | 光照与公共函数 |
| `Shaders/Modules/<Name>/` | 模块：`properties.hlsl`、`phase_<相位>.hlsl`、可选 `includes.hlsl`、`lang/*.po` |
| `Shaders/lang/*.po` | 主着色器本地化 |
| `NonToon/Editor/NTModuleRegistration.cs` | 模块注册自愈 |
| `NonToon/Editor/NTShaderCoreLocalization.cs` | 补齐 ShaderCore 与本包的中文语言表 |
| `nontoon-converter/Editor/LilToonToNonToonConverter.cs` | lilToon → NonToon 转换实现 |
| `nontoon-converter/Editor/NTLightAdjusterWindow.cs` | 亮度自适应插件（窗口） |
| `nontoon-converter/Editor/NTVrcParameterBuilder.cs` | 生成 Animator 参数、曲线剪辑与 VRC 参数/菜单 |
| `nontoon-converter/Runtime/` | 两种光源组件（编辑器期设置载体） |

**必须遵守的实现约束**

- `properties.hlsl` **不能包含任何注释**：`SCProperty.Parse` 会抛出异常，导致整个着色器导入失败
- `SC_uint` / ShaderLab `Integer` 属性**必须使用 `Material.SetInteger`**：
  `SetFloat` 与 `SetInt` 均为静默无效（实测：`SetInt` 后 `GetInteger` 读到 1，但着色器中仍为 0）。
  本包 165 条属性中有 36 条为整数型。读取同样需按类型（`GetInteger`）
- 宏体内不能出现 `#if` 等预处理指令（d3d11 会报 `syntax error: unexpected token '#'`），守卫需放到调用点
- 模块 `properties.hlsl` 中不要使用 `SC_Foldout`：其标签不允许包含括号，且会导致着色器整体不导入
  （日志无提示）。分组请使用 `SC_Box`
- `.po` 中不允许重复键（`POParser` 会抛异常）；值中不要出现英文双引号或反斜杠转义
- ShaderCore 的模块注册表只扫描一次（`ProjectSettings/jp.lilxyzw.shadercore.asset`），
  新模块会被静默忽略，由 `NTModuleRegistration` 自愈
- `AssetDatabase.CreateAsset` 会重置整数型属性，模块开关需在其之后重放
- AnimationClip 资产必须使用 `.anim` 扩展名：使用 `.clip` 时 `AssetDatabase.CreateAsset` 会拒绝创建
- 包内每个文件都必须附带 `.meta`。缺失时 Unity 会在每台机器生成不同的 GUID，
  用户挂好的组件在重装或换机后会变成 Missing Script（0.3.6 及更早版本即存在此问题，0.3.7 修复）
- 修改验证流程或发布脚本前，请先阅读第 13 节的失效模式清单

---

## 附录 A. 版本记录

| 版本 | 要点 |
|---|---|
| 工具 0.4.8 | 新增亮度自适应与防煤插件：场景光照诊断、「模拟全黑地图」、亮度预设（抬高亮度下限，零采样防煤）、生成 VRChat 可调参数（Float 8 bit + 径向菜单，可驱动 `_LightMinLimit` 等）、上传前清理助手组件 |
| 0.3.7 | 修复 `.meta` 缺失（此前重装或换机后 Avatar 光源组件会变为 Missing Script）；修复默认语言下汉化不生效（中文 Windows 默认 `zh-CN`，此前 165 条中仅 35 条为中文，现为 117 条） |
| 0.3.6 | 组件拆分为两类（不需要改材质 / 需要材质启用）；作用范围与光源编号；屏蔽环境光；阴影颜色、阴影强度遮罩、可自备阴影贴图；修复自有光被每盏附加光各叠加一次导致的过曝 |
| 0.3.5 | 面板整理（内部参数不再堆叠在面板上）；色温；环境光匹配 |
| 0.3.4 | 开销下调：默认档 20 采样、强度 0 时零采样、面板显示采样预算 |
| 0.3.3 | 修复「只由它照亮」未真正关闭世界环境光 |
| 0.3.2 | 修复材质面板仍有大量英文（自动补齐 ShaderCore 的 `zh-Hans.po`） |
| 0.3.1 | PCSS 画质四档、烘焙上限 2048、实时光源模式 |
| 0.3.0 | 汉化补齐、PCSS 软阴影、lilToon 逐像素阴影遮罩可转换 |
| 0.2.0 | 新增自有光源（私有光 + 烘焙自阴影） |
| 0.1.4 – 0.1.11 | 半透明修复、透明排序、受光方向、MatCap VR 视差、Nearer 开关、8 档遮罩通道、转换器拆分为独立包 |

每个版本的 zip 与 SHA-256 见 [listing](https://catandling.github.io/VPM-nontoon-fork/vpm.json)。

## 附录 B. 依赖与许可

- 依赖 **ShaderCore `jp.lilxyzw.shadercore >= 0.1.9`**（由 VPM 自动安装）
- 修改部分基于 [lilxyzw/NonToon](https://github.com/lilxyzw/NonToon)，遵循其原始 LICENSE
- 材质转换器源自 **LilToonToNonToonConverter**（MIT，保留署名）

## 附录 C. 数据与引用来源

本文中的结论分两类：有官方文档依据的，以及本机实测的。来源如下。

官方文档与规范：

- VRChat：[Allowed Avatar Components](https://creators.vrchat.com/avatars/whitelisted-avatar-components/whitelisted-avatar-components/) ·
  [Animator Parameters](https://creators.vrchat.com/avatars/animator-parameters/) ·
  [Expression Menu and Controls](https://creators.vrchat.com/avatars/expression-menu-and-controls)
- AAO（Avatar Optimizer）文档：《Merge Skinned Mesh》《Merge Material》《Trace And Optimize》
  《Shader Information API》《System Assumptions》以及 changelog（PR #769）
- MA（Modular Avatar）文档：《Remove Vertex Color》《Merge Animator》《Material Swap》
- lilToon：[ライティング・明るさ設定](https://lilxyzw.github.io/lilToon/ja_JP/base/lighting.html)（亮度公式与建议取值范围）
- Light Limit Changer：[Azukimochi/LightLimitChangerForMA](https://github.com/Azukimochi/LightLimitChangerForMA)

本机实测（Unity 2022.3.22f1）：

- 属性、GUID、模块对比：与官方 0.1.3 逐文件比对
- 渲染数据：离屏渲染至 RenderTexture 后回读像素（真实 GPU，Direct3D 11）
- 着色器静态数据：对 ShaderCore 拼接后的生成源码做脚本化统计
- 生成资产与 AAO 兼容性：调用 NDMF 的 `AvatarProcessor.ProcessAvatar` 完成构建后检查结果
