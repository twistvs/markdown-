下面从**底层数据结构、离线构建、运行时选择、可见性剔除、光栅化、材质输出、流式加载**几个层面详细讲 Nanite。可以先记住一句话：

**Nanite 本质上是“虚拟化几何体系统”：它把传统按对象/LOD 渲染的模型，改造成按小三角形簇 cluster 管理、按屏幕像素误差选择细节、按需流式加载和 GPU 驱动渲染的几何数据库。**

Epic 官方文档把 Nanite 定义为用于实现“像素级细节”和“大量对象”的虚拟化几何系统；SIGGRAPH 2021 的 Nanite 技术分享也明确覆盖了从网格导入、数据结构构建、流送、解压、剔除、光栅化到最终着色的完整流程。([Epic Games Developers](https://dev.epicgames.com/documentation/unreal-engine/nanite-virtualized-geometry-in-unreal-engine?utm_source=chatgpt.com))

------

# 1. 传统渲染的问题：为什么需要 Nanite？

传统游戏模型通常要经历：

```text
高模雕刻 / 扫描模型
        ↓
低模拓扑
        ↓
法线贴图烘焙
        ↓
手工制作 LOD0 / LOD1 / LOD2 / LOD3
        ↓
运行时根据距离切换 LOD
```

这个流程有几个问题：

第一，**LOD 是粗粒度的**。
传统 LOD 通常是整个模型一起切换，比如一块岩石从 50 万面突然切到 5 万面，再切到 5000 面。即使有过渡，也容易出现 popping。

第二，**CPU 负担重**。
传统渲染常常以 mesh、section、draw call 为单位提交。场景中物体越多，CPU 组织渲染命令的压力越大。

第三，**GPU 会浪费很多计算**。
如果一个远处物体仍然用大量小于 1 像素的三角形绘制，GPU 光栅化阶段会处理大量“几乎看不见”的三角形。

第四，**美术资产需要大量人工优化**。
特别是摄影测量、雕刻资产、Megascans 资产，原始模型面数极高，传统流程需要大量减面、烘焙和 LOD 制作。

Nanite 的目标就是绕开这些问题：
**不再主要依赖人工 LOD，而是让引擎自动把几何体切成很多小块，根据屏幕可见性和像素误差动态选择合适细节。**

------

# 2. Nanite 的核心思想：不是渲染“模型”，而是渲染“cluster”

Nanite 的最小核心单位不是整个 Static Mesh，而是 **cluster**。

可以把一个高模拆成很多小块：

```text
原始高模
├── cluster 0
├── cluster 1
├── cluster 2
├── cluster 3
└── ...
```

每个 cluster 通常包含一小批三角形。Nanite 会围绕这些 cluster 构建层级结构，也就是类似树状结构的几何 LOD：

```text
高精度 cluster 层
        ↑
中精度 cluster 层
        ↑
低精度 cluster 层
```

可以理解为：

```text
一个复杂模型
    ↓
被切分成许多三角形簇
    ↓
这些簇又被组织成层级 DAG / hierarchy
    ↓
运行时根据相机距离、屏幕误差、可见性选择最合适的簇
```

这里的重点是：
**Nanite 的 LOD 不是整个模型切换，而是 cluster 级别的局部选择。**

例如一座雕像：

- 靠近摄像机的脸部，用高精度 cluster；
- 远离摄像机的背部，用低精度 cluster；
- 被遮挡的部分，完全不渲染；
- 小于像素级贡献的微小细节，自动省略。

这就是 Nanite 能同时保留高细节和高性能的关键。

------

# 3. 离线构建阶段：导入模型时 Nanite 做了什么？

当你在 UE5 中启用 Nanite 并导入 Static Mesh 时，Nanite 不会简单保存原始顶点和索引缓冲。它会进行一次专门的**离线预处理**。

大致过程如下：

```text
原始三角网格
    ↓
网格清理与分块
    ↓
生成 cluster
    ↓
对 cluster 进行简化
    ↓
构建 cluster hierarchy
    ↓
压缩几何数据
    ↓
生成用于流式加载的 page 数据
    ↓
保存为 Nanite 内部格式
```

## 3.1 Cluster 切分

Nanite 首先会把原始模型切成很多小的三角形簇。每个 cluster 都有：

- 一组三角形；
- 顶点数据；
- 包围盒 / 包围球；
- 法线锥或类似方向信息；
- 误差度量；
- 材质 section 信息；
- 层级父子关系。

为什么要有包围盒？
因为运行时可以快速判断这个 cluster 是否在视锥内、是否被遮挡、在屏幕上有多大。

## 3.2 Cluster 简化

Nanite 会自动生成不同精度版本。传统 LOD 是“整个模型生成几个版本”，而 Nanite 是对 cluster 层级进行连续化简。

可以想象成：

```text
100% 细节：很多小 cluster
50% 细节：合并/简化后的 cluster
10% 细节：更粗的 cluster
```

运行时不需要说“用 LOD2”，而是说：

> 对当前摄像机来说，哪些 cluster 的投影误差足够小？选它们。

## 3.3 构建层级结构

Nanite 会把 cluster 组织成层级结构。底层是最精细三角形，上层是更粗的近似表示。

类似这样：

```text
Root Cluster
├── Coarse Cluster A
│   ├── Fine Cluster A1
│   ├── Fine Cluster A2
│   └── Fine Cluster A3
└── Coarse Cluster B
    ├── Fine Cluster B1
    ├── Fine Cluster B2
    └── Fine Cluster B3
```

运行时会从较粗层级开始判断：
如果粗 cluster 已经足够精细，就直接用它；
如果误差太大，就继续展开到更细层级。

这和虚拟纹理有点像：
虚拟纹理按需加载 mip/page；
Nanite 按需加载几何 cluster/page。

所以它叫 **virtualized geometry**。

------

# 4. 屏幕空间误差：Nanite 如何决定“该用多细”？

Nanite 的 LOD 选择核心不是传统的“距离”，而是**屏幕空间误差**。

简单理解：

```text
如果某个几何简化造成的误差投影到屏幕上小于约 1 个像素，
那么这个简化版本就足够好。
```

也就是说，Nanite 关心的是：

```text
这个 cluster 的简化误差，在当前视角下会不会被玩家看出来？
```

如果看不出来，就用更粗版本。
如果看得出来，就继续细分。

举例：

同一个石头模型：

- 离摄像机 1 米：一个凹凸细节可能占 30 像素，需要高精度；
- 离摄像机 100 米：同样凹凸只占 0.2 像素，可以被简化；
- 被墙挡住：完全不用渲染。

这就避免了传统 LOD 的突变，也避免了远处微三角形浪费。

------

# 5. GPU 驱动渲染：为什么 Nanite 能处理大量几何？

传统渲染流程中，CPU 往往要准备很多 draw call：

```text
CPU 遍历物体
    ↓
判断可见性
    ↓
选择 LOD
    ↓
提交 draw call
    ↓
GPU 绘制
```

Nanite 更偏向 GPU 驱动：

```text
CPU 提供场景实例信息
    ↓
GPU 进行 cluster 剔除
    ↓
GPU 选择 LOD
    ↓
GPU 光栅化可见 cluster
    ↓
输出可见性/材质信息
```

这样可以减少 CPU 对大量物体和 LOD 的管理压力。
Nanite 的目标不是“每个三角形都免费”，而是把传统由 CPU 管理的许多细碎工作，转移到更适合并行处理的 GPU 上。

------

# 6. 可见性剔除：Nanite 为什么不渲染看不见的东西？

Nanite 的性能关键之一是大量剔除。它会尽量在绘制前排除无效 cluster。

主要包括：

## 6.1 视锥剔除 Frustum Culling

如果 cluster 的包围体完全在摄像机视锥外：

```text
不渲染
```

这很传统，但 Nanite 是 cluster 级别的，比物体级更细。

例如一座巨大建筑物，一半在屏幕外：

传统物体级剔除：建筑整体还在视野里，所以可能仍然提交。
Nanite cluster 级剔除：屏幕外那一半的 cluster 可以直接剔掉。

## 6.2 背面剔除 Backface / Cone Culling

Nanite 可以利用 cluster 的方向信息判断某些 cluster 是否背向相机。
如果一个 cluster 的所有三角形大致都朝向背面，就可以跳过。

这对封闭模型、岩石、雕塑等很有效。

## 6.3 遮挡剔除 Occlusion Culling

Nanite 使用深度信息判断 cluster 是否被前方物体挡住。UE 的控制变量参考中也能看到 Nanite 支持基于 hierarchical depth buffer 的遮挡剔除相关设置。([Epic Games Developers](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-console-variables-reference?utm_source=chatgpt.com))

可以理解为：

```text
上一阶段 / 上一帧 / 当前构建的深度层级
        ↓
用 cluster 包围体去测试
        ↓
如果完全在已有深度之后
        ↓
剔除
```

Hierarchical Z-Buffer，简称 HZB，可以理解成多级低分辨率深度图：

```text
原始深度图：1920 × 1080
第一级：960 × 540
第二级：480 × 270
第三级：240 × 135
...
```

越粗的层级越适合快速判断大块区域是否被遮挡。

------

# 7. Nanite 的光栅化：为什么不用完全传统的三角形管线？

Nanite 的三角形非常多，而且很多是微三角形。传统硬件光栅化管线对普通三角形很高效，但面对大量极小三角形时，会遇到一些问题：

- 顶点处理开销大；
- 小三角形调度成本高；
- 很多三角形小于一个像素；
- draw call / meshlet 管理复杂。

Nanite 因此采用了混合策略：
**对不同大小的 cluster / triangle 使用不同路径，包括硬件光栅化和软件光栅化。**

可以抽象成：

```text
较大的三角形
    → 走硬件光栅化路径

极小的微三角形
    → 走更适合 Nanite 数据结构的软件光栅化路径
```

注意这里的“软件光栅化”不是 CPU 软件渲染，而是**GPU compute shader 上的自定义光栅化**。

为什么这样做？

因为当三角形小到接近像素大小时，传统三角形管线的固定开销可能不划算。Nanite 的自定义路径可以更好地服务于 cluster 数据结构和可见性缓冲。

------

# 8. Visibility Buffer：Nanite 不是一开始就直接写完整 GBuffer

传统延迟渲染一般是：

```text
几何阶段
    ↓
直接写 GBuffer：
BaseColor / Normal / Roughness / Metallic / Depth / ...
```

Nanite 的流程更接近：

```text
Nanite cluster 光栅化
    ↓
写 Visibility Buffer
    ↓
根据可见像素解析材质
    ↓
输出 GBuffer / 参与后续光照
```

Visibility Buffer 可以理解成：
每个屏幕像素先不急着写完整材质，而是记录“这个像素看到了哪个三角形 / 哪个 cluster / 哪个 primitive”。

类似：

```text
Pixel (x, y) →
{
    cluster ID,
    triangle ID,
    primitive ID,
    barycentric info / depth info
}
```

然后在后续阶段，根据这个 ID 再去取顶点属性、插值法线、UV、材质参数，最终生成 GBuffer。

这样做的好处是：

第一，**先解决可见性，再做昂贵材质计算**。
看不见的三角形不需要做复杂材质。

第二，**减少过度着色 over-shading**。
传统流程中，被覆盖的像素可能先后被多个三角形着色。Visibility Buffer 可以先确定最终可见面，再进行材质解析。

第三，**适合海量微三角形**。
Nanite 的重点是几何量巨大，而不是让每个三角形都走传统完整材质流程。

------

# 9. 材质与着色：Nanite 解决的是几何，不是所有渲染问题

Nanite 的核心是几何虚拟化，不是材质虚拟化。
最终它仍然要和 UE 的材质系统、延迟渲染、阴影、Lumen、Virtual Shadow Maps 等系统配合。

大体流程可以理解为：

```text
Nanite 负责：
    哪些三角形可见？
    用多细的几何？
    哪个像素对应哪个三角形？

材质系统负责：
    Base Color
    Normal
    Roughness
    Metallic
    Emissive
    Opacity / Mask 等

光照系统负责：
    Direct Lighting
    Lumen GI
    Reflection
    Shadow
```

早期 Nanite 对材质类型限制较多，例如透明、变形、植被等支持有限。后续 UE5 版本逐步增强了 masked material、two-sided、foliage 等场景，但 Nanite 仍然不是“任何材质和任何变形都无成本支持”。

------

# 10. 流式加载：为什么 Nanite 可以处理超大资产？

Nanite 的另一个关键是**几何数据流式加载**。

高精度模型可能非常大，如果全部加载到显存，会爆显存。Nanite 会把几何数据组织成 page，类似虚拟纹理：

```text
完整 Nanite Mesh 数据
├── page 0
├── page 1
├── page 2
├── page 3
└── ...
```

运行时根据当前视角和需要的 LOD，只加载必要 page。

比如：

```text
摄像机靠近雕像头部
    → 加载头部高精度 page

摄像机远离雕像
    → 只需要低精度 page

雕像背面不可见
    → 背面高精度 page 不加载或不使用
```

这就是为什么 Nanite 很像“虚拟纹理”的几何版本：

| 虚拟纹理               | Nanite                   |
| ---------------------- | ------------------------ |
| 按需加载纹理 tile      | 按需加载几何 page        |
| 根据屏幕大小选 mip     | 根据屏幕误差选 cluster   |
| 避免一次性加载超大纹理 | 避免一次性加载超大几何   |
| 节省显存和带宽         | 节省显存、带宽和渲染成本 |

------

# 11. Fallback Mesh：为什么 Nanite 还需要备用网格？

Nanite 并不是所有渲染路径都直接使用 Nanite 内部格式。UE 中还会生成 fallback mesh，也就是回退网格。官方文档和发布说明中能看到 Nanite 相关的 fallback mesh、Fallback Relative Error 等设置。([Epic Games Developers](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-5.0-release-notes?application_version=5.0&utm_source=chatgpt.com))

Fallback Mesh 主要用于：

- 不支持 Nanite 的平台或渲染路径；
- 某些碰撞、光线追踪或工具流程；
- 兼容性场景；
- 编辑器预览或特殊系统。

也就是说，启用 Nanite 后，原始模型会变成：

```text
Nanite 内部几何数据
+
Fallback Mesh
```

Nanite 数据用于主要实时渲染；
Fallback Mesh 用于兼容路径。

------

# 12. Nanite 与传统 LOD 的根本区别

传统 LOD：

```text
以模型为单位
根据距离切换
LOD0 / LOD1 / LOD2 / LOD3
手工制作或自动生成
切换粒度粗
```

Nanite：

```text
以 cluster 为单位
根据屏幕空间误差选择
层级结构连续展开
自动构建
切换粒度细
```

对比表：

| 项目               | 传统 LOD         | Nanite                       |
| ------------------ | ---------------- | ---------------------------- |
| 单位               | 整个模型         | Cluster                      |
| 选择依据           | 距离 / 屏幕尺寸  | 屏幕空间误差                 |
| 制作方式           | 手工或自动 LOD   | 导入时自动构建               |
| 切换效果           | 容易 popping     | 更细粒度、更平滑             |
| CPU draw call 压力 | 较高             | 更偏 GPU 驱动                |
| 适合资产           | 优化后的游戏模型 | 高模、扫描资产、复杂静态几何 |
| 数据加载           | 通常按 mesh/LOD  | 按 page/cluster              |

------

# 13. Nanite 的运行时流程总结

可以把 Nanite 的一帧渲染流程简化为：

```text
1. CPU 提交 Nanite 实例信息
        ↓
2. GPU 遍历 Nanite cluster hierarchy
        ↓
3. 做视锥剔除、背面剔除、遮挡剔除
        ↓
4. 根据屏幕空间误差选择合适 cluster
        ↓
5. 检查所需几何 page 是否已在显存
        ↓
6. 未加载的数据发起流式请求
        ↓
7. 已可用 cluster 进入光栅化
        ↓
8. 大三角形走硬件光栅化，小三角形可走自定义软件光栅化
        ↓
9. 写入 Visibility Buffer
        ↓
10. 材质解析并输出 GBuffer
        ↓
11. 后续光照、阴影、Lumen、后处理
```

更抽象地说：

```text
Nanite = Cluster 化 + 层级 LOD + GPU 剔除 + 几何流送 + Visibility Buffer 渲染
```

------

# 14. Nanite 为什么特别适合“静态高精度几何”？

Nanite 对下面这些资产特别友好：

- 岩石；
- 山体；
- 建筑；
- 雕塑；
- 废墟；
- 硬表面模型；
- 摄影测量扫描资产；
- Megascans 资产；
- 高密度环境物件。

因为这些资产有几个共同点：

```text
三角形很多
形状复杂
大多是静态或刚体
材质相对稳定
不需要复杂骨骼变形
```

Nanite 可以把它们切成 cluster，然后自动管理细节和可见性。

------

# 15. Nanite 不擅长什么？

Nanite 很强，但不是万能。

## 15.1 透明材质

透明物体需要排序、混合和多层可见性，这和 Nanite 的 Visibility Buffer 思路不完全契合。
因此玻璃、水体、半透明粒子等通常不适合作为 Nanite 的主要目标。

## 15.2 大量顶点动画或骨骼变形

Nanite 的优势来自离线构建好的 cluster hierarchy。
如果网格每帧都大幅变形，原来的 cluster 包围体、误差度量、层级结构都会变得难以高效复用。

所以传统角色骨骼动画长期不是 Nanite 最典型的应用场景。后续版本在某些变形、材质和植被方向持续增强，但从原理上看，**越静态、越刚体、越高密度，越适合 Nanite**。

## 15.3 低面数简单物体

如果一个模型本来只有几百个三角形，Nanite 不一定带来收益。
Nanite 自身也有数据结构、流式管理和剔除开销。

因此：

```text
超高模复杂资产 → Nanite 很适合
极低模简单资产 → 传统渲染可能更合适
```

------

# 16. 一个直观例子：一块 100 万面的岩石如何被 Nanite 渲染？

假设有一块 100 万三角形的扫描岩石。

传统流程：

```text
100 万面高模
    ↓
减面到 2 万面
    ↓
烘焙法线贴图
    ↓
做 LOD0 / LOD1 / LOD2
    ↓
运行时按距离切换
```

Nanite 流程：

```text
100 万面模型导入 UE5
    ↓
启用 Nanite
    ↓
引擎离线切成 cluster
    ↓
构建层级 LOD
    ↓
压缩并分页
    ↓
运行时自动选择可见 cluster
```

当玩家靠近岩石：

```text
屏幕误差要求高
→ 展开到更细 cluster
→ 显示大量真实几何细节
```

当玩家远离岩石：

```text
屏幕误差要求低
→ 使用较粗 cluster
→ 不浪费微小三角形
```

当岩石被墙挡住：

```text
HZB 遮挡测试失败
→ cluster 不渲染
```

这就是 Nanite 的实际价值：
**它不是把 100 万面全部硬画出来，而是只画当前画面真正需要的那部分精度。**

------

# 17. Nanite 的本质总结

可以用四句话概括 Nanite 的底层原理：

**第一，资产导入时，Nanite 把高模切成大量 cluster，并构建层级化简结构。**

**第二，运行时，Nanite 根据屏幕空间误差，而不是简单距离，选择合适精度的 cluster。**

**第三，Nanite 在 GPU 上进行 cluster 级剔除、LOD 选择和光栅化，减少 CPU draw call 和无效几何处理。**

**第四，Nanite 通过 Visibility Buffer 和几何数据流式加载，把“超高面数模型”变成“按需可见、按需加载、按需着色”的虚拟化几何系统。**

一句更工程化的表达是：

```text
Nanite = 面向实时渲染的几何虚拟内存系统
       + cluster 级连续 LOD
       + GPU-driven visibility pipeline
       + visibility buffer shading
```

所以 Nanite 真正改变的不是“能不能放高模”，而是改变了引擎管理几何体的方式：
从过去的**模型级 LOD 渲染**，变成了**屏幕误差驱动的虚拟化微几何渲染**。