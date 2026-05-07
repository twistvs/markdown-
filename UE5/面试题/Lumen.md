下面以 **Unreal Engine 5 的 Lumen** 为对象介绍。

## 1. Lumen 是什么？

**Lumen 是 UE5 的全动态全局光照与反射系统**，主要负责两件事：

1. **Global Illumination，全局光照 / 间接光照**
   - 光线打到墙面、地面、物体后继续反弹，影响周围环境。
   - 例如红色墙面会把红色光反射到附近白色物体上。
2. **Reflections，动态反射**
   - 金属、玻璃、水面、光滑地板等材质上的实时反射。
   - 可以随着场景、光源、摄像机变化而变化。

Epic 官方文档称 Lumen 是 UE 的 fully dynamic global illumination and reflections system，并且在 UE5 中默认用于新一代主机/高端平台的动态光照方案。([Epic Games Developers](https://dev.epicgames.com/documentation/unreal-engine/lumen-global-illumination-and-reflections-in-unreal-engine?utm_source=chatgpt.com))

------

## 2. 它解决了什么问题？

在 UE4 或传统实时渲染流程中，如果想获得比较真实的间接光照，常用做法是：

- 烘焙 Lightmap；
- 手动布置 Reflection Capture；
- 对静态场景提前计算光照；
- 动态物体和动态时间变化则需要额外处理。

Lumen 的目标是让这些过程更自动化、更实时。

例如：

- 太阳角度变化，室内光照实时改变；
- 打开一扇门，外面的光照能动态进入室内；
- 移动一个大物体，它对周围的遮挡和反射会更新；
- 不需要等待 Lightmap 烘焙；
- 编辑器中看到的效果更接近最终运行效果。

Epic 在 UE5 特性介绍中也强调，Lumen 可以让间接光照随着直接光照或几何体变化而实时适应，并减少手动制作 lightmap UV、等待烘焙和放置反射捕获的工作。([Unreal Engine](https://www.unrealengine.com/unreal-engine-5?utm_source=chatgpt.com))

------

## 3. Lumen 的基本工作原理

可以把 Lumen 理解成一种 **实时近似路径追踪方案**。它不是真正完整的离线路径追踪，但会用多种技术组合，尽可能在实时帧率下模拟光线反弹和反射。

它主要会用到：

### 3.1 屏幕空间信息

Lumen 会先利用当前屏幕上已经渲染出来的像素信息。

优点是快，缺点是只能看到屏幕里已有的东西。比如镜子里要反射摄像机背后的物体，单靠屏幕空间信息就不够。

### 3.2 Mesh Distance Fields 软件光线追踪

在软件光追模式下，Lumen 可以使用 Mesh Distance Fields 来进行光线求交。这个方式对硬件要求较低，适合更广泛的设备，但精度和支持的几何类型不如硬件光追。Epic 官方技术文档也说明 Lumen 支持软件和硬件两种 ray tracing 路径。([Epic Games Developers](https://dev.epicgames.com/documentation/unreal-engine/lumen-technical-details-in-unreal-engine?utm_source=chatgpt.com))

### 3.3 Hardware Ray Tracing 硬件光线追踪

如果显卡支持 DXR / 硬件光追，Lumen 可以使用硬件光线追踪获得更准确的反射和遮挡效果。

一般来说：

- **软件 Lumen**：性能更友好，适合大多数场景；
- **硬件 Lumen**：质量更好，尤其是镜面反射、复杂几何、动态物体方面，但成本更高。

UE 5.6 还对硬件光追系统做了优化，以提升 Lumen GI 性能并降低 CPU 瓶颈。([Unreal Engine](https://www.unrealengine.com/news/unreal-engine-5-6-is-now-available?utm_source=chatgpt.com))

### 3.4 Surface Cache 表面缓存

Lumen 不会每一帧都对所有表面进行完整光照计算，而是会缓存场景表面的光照信息。这样可以降低实时计算成本。

简单说，它会给场景中的物体建立一种“可用于间接光照查询的简化表示”。

------

## 4. Lumen 和传统烘焙光照的区别

| 对比项       | Lightmass / 烘焙光照       | Lumen                         |
| ------------ | -------------------------- | ----------------------------- |
| 光照类型     | 主要适合静态光照           | 动态全局光照                  |
| 是否需要烘焙 | 需要                       | 通常不需要                    |
| 时间变化     | 不灵活                     | 支持昼夜变化                  |
| 动态物体     | 处理复杂                   | 更自然                        |
| 性能         | 运行时成本低               | 运行时成本更高                |
| 画质稳定性   | 静态场景非常稳定           | 动态场景强，但可能有噪点/闪烁 |
| 适用场景     | 移动端、低端平台、固定场景 | PC、主机、高质量实时场景      |

------

## 5. Lumen 的优点

### 5.1 开发效率高

不用频繁烘焙 Lightmap，改灯光、改场景后能立即看到结果。

这对游戏开发、虚拟制片、建筑可视化、仿真场景非常有用。

### 5.2 动态场景表现好

比如：

- 昼夜循环；
- 开关灯；
- 门窗开合；
- 大型物体移动；
- 可破坏场景；
- 室内外光照切换。

这些都是传统烘焙光照比较难处理的情况。

### 5.3 间接光照更真实

Lumen 能表现颜色反弹、暗部补光、遮挡变化，使场景不再像“只有直接光打亮”。

### 5.4 与 Nanite 搭配效果好

UE5 常见组合是：

> **Nanite 负责高精度几何体，Lumen 负责动态全局光照。**

这也是 UE5 视觉效果提升的核心组合之一。

------

## 6. Lumen 的缺点和限制

### 6.1 性能开销较大

Lumen 比传统静态光照更消耗 GPU/CPU，尤其是在：

- 大型开放世界；
- 高分辨率；
- 大量反射材质；
- 多个动态光源；
- 室内复杂遮挡；
- 硬件光追开启时。

### 6.2 反射不是绝对完美

Lumen 反射是实时近似方案，不等同于离线渲染级别的路径追踪。镜子、玻璃、金属等高反射材质中可能出现：

- 反射细节缺失；
- 远处物体不准确；
- 画面边缘反射消失；
- 闪烁或噪点。

### 6.3 对场景建模有要求

为了让 Lumen 工作稳定，场景模型最好：

- 避免极薄墙体；
- 室内墙、地、顶有一定厚度；
- 不要大量使用过小、过碎的细节物体参与 GI；
- 注意 Mesh Distance Field 的生成质量；
- 对镜面反射场景考虑硬件光追。

### 6.4 移动端和低端平台不适合

Lumen 主要面向现代 PC、当前世代主机和较高端设备。移动端或低端设备通常仍然需要传统烘焙光照、SSGI、Reflection Capture 等方案。

------

## 7. 在 UE5 中如何开启 Lumen？

通常路径是：

**Project Settings → Rendering → Dynamic Global Illumination Method**

设置为：

```text
Lumen
```

然后：

**Reflection Method**

设置为：

```text
Lumen
```

常见相关选项包括：

```text
Dynamic Global Illumination Method: Lumen
Reflection Method: Lumen
Generate Mesh Distance Fields: Enabled
Support Hardware Ray Tracing: Enabled / Disabled
Use Hardware Ray Tracing when available: Enabled / Disabled
```

如果使用硬件光追，需要显卡、RHI、项目设置都支持。

------

## 8. 常用优化建议

### 8.1 优先控制反射质量

高质量反射很贵。对于非关键材质，可以降低粗糙度表现要求，减少镜面大面积反射。

### 8.2 减少小物体参与 GI

大量小物体会增加 Lumen 场景表示成本。可以通过剔除、合并、简化模型来优化。

### 8.3 使用合理的后处理设置

在 Post Process Volume 中可以调节：

```text
Lumen Global Illumination Quality
Lumen Reflection Quality
Final Gather Quality
Scene Lighting Quality
```

质量越高，越接近真实，但性能消耗也越大。

### 8.4 室内场景注意墙体厚度

太薄的墙可能导致漏光。室内场景建议使用有厚度的墙、顶、地板结构。

### 8.5 必要时混合使用传统方案

不是所有项目都必须全用 Lumen。例如：

- 静态建筑展示：可以考虑烘焙；
- 移动端项目：通常不用 Lumen；
- 高端 PC 影视化展示：可用 Lumen 或 Path Tracing；
- 游戏项目：根据目标帧率选择 Lumen 质量级别。

------

## 9. Lumen 适合哪些项目？

比较适合：

- UE5 高质量游戏；
- 第一/第三人称写实场景；
- 建筑可视化；
- 虚拟制片；
- 室内外动态光照场景；
- 有昼夜循环的开放世界；
- 需要快速迭代光照的项目；
- 仿真、数字孪生、训练场景展示。

不太适合：

- 低端移动端；
- 极端性能敏感项目；
- 完全静态、追求最低运行成本的场景；
- 大量镜面反射且要求物理级准确的场景；
- 需要完全无噪声、离线电影级光追的最终渲染。

------

## 10. 一句话总结

**Lumen 是 UE5 中用于实时全局光照和动态反射的核心技术，它让场景光照可以像现实一样随着光源、物体和环境变化实时更新，大幅减少烘焙和手动布光工作，但代价是更高的运行性能开销和一定的近似误差。**