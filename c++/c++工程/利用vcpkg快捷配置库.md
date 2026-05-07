#### 一、vcpkg的基本操作

vcpkg会自动使用windows的代理，所以**不需要额外设置代理**

使用git clone下载vcpkg的源码后，运行bootstrap-vcpkg.bat即可下载vcpkg.exe，这样在该目录下就可以执行vcpkg命令

------

查看vcpkg是否能够下载指定的库

```
.\vcpkg search 库的名字
```

安装指定的库，文件位置在vcpkg/packages中

```
.\vcpkg install [name]
```

------

#### 二、vcpkg的triplet概念

vcpkg 下载的库有一个非常核心的概念：**“triplet”**，也就是编译器 + 构建选项的组合。每个库在 vcpkg 中会根据 triplet 生成不同的版本。下面我详细讲清楚。

------

**1️⃣ vcpkg 的库版本分类**

每个库在 vcpkg 中通常有 **三维区分**：

| 维度                  | 含义             | 示例                                                         |
| --------------------- | ---------------- | ------------------------------------------------------------ |
| **平台架构**          | x86 或 x64       | x86 → 32 位，x64 → 64 位                                     |
| **链接类型**          | 静态库或动态库   | `-static` → `.lib`/`.a`，`-dynamic` → `.dll` + `.lib`/`.dll.a` |
| **运行时/编译器类型** | MSVC 或 MinGW 等 | MSVC → `x64-windows`，GCC → `x64-mingw-dynamic`              |

这三者组合就是一个 **triplet**。

------

**2️⃣ 常见 triplet 对照表**

**A. MSVC (Visual Studio)**

| Triplet 名称         | 构建方式 | 输出示例        | 说明                  |
| -------------------- | -------- | --------------- | --------------------- |
| `x64-windows`        | 动态库   | `.dll` + `.lib` | 默认动态链接 MSVC CRT |
| `x64-windows-static` | 静态库   | `.lib`          | 静态链接 MSVC CRT     |
| `x86-windows`        | 动态库   | `.dll` + `.lib` | 32 位版本             |
| `x86-windows-static` | 静态库   | `.lib`          | 32 位静态库           |

> MSVC 静态 triplet 会把 CRT 链接到 `.lib` 里，不依赖外部 DLL。动态 triplet 则依赖 MSVC 的 DLL。

------

**B. MinGW / GCC (MSYS2)**

| Triplet 名称        | 构建方式 | 输出示例          | 说明                         |
| ------------------- | -------- | ----------------- | ---------------------------- |
| `x64-mingw-dynamic` | 动态库   | `.dll` + `.dll.a` | 使用 MinGW 的 GCC + 动态链接 |
| `x64-mingw-static`  | 静态库   | `.a`              | 使用 MinGW 的 GCC + 静态链接 |
| `x86-mingw-dynamic` | 动态库   | `.dll` + `.dll.a` | 32 位动态链接                |
| `x86-mingw-static`  | 静态库   | `.a`              | 32 位静态链接                |

> MinGW 静态库用 `.a`，动态库生成 `.dll` 和对应 import `.dll.a`。

------

**C. UCRT / POSIX MSYS2（可选）**

MSYS2 提供多种运行库环境，例如 UCRT：

| Triplet 名称                   | 编译器            | 说明                                          |
| ------------------------------ | ----------------- | --------------------------------------------- |
| `x64-uwp` / `x64-uwp-static`   | MSVC              | Windows UWP 应用                              |
| `x64-ucrt` / `x64-ucrt-static` | GCC (MSYS2 UCRT)  | 使用 UCRT 的 MinGW GCC                        |
| `x64-msys`                     | GCC (MSYS2 POSIX) | 仅用于 MSYS2 环境，不推荐直接用于 Windows GUI |

> 你的 GAMES101 项目如果用 MSYS2 GCC + UCRT，应该选择 `x64-ucrt` 或 `x64-ucrt-static`。

------

**3️⃣ triplet 的选择原则**

1. **与编译器匹配**
   - MSVC → 用 `x64-windows` / `x64-windows-static`
   - GCC / MSYS2 → 用 `x64-mingw-dynamic` / `x64-mingw-static`
2. **动态/静态选择**
   - 动态库（`-dynamic`） → DLL，可减少二进制大小，但运行时依赖 DLL
   - 静态库（`-static`） → 单文件可执行，适合教学/小项目，但编译慢、体积大
3. **架构匹配**
   - 你的编译器是 64 位 → 选 x64
   - 32 位编译器 → 选 x86
4. **CRT 兼容**
   - MSVC 静态 → `/MT`
   - MSVC 动态 → `/MD`
   - triplet 会自动配置 CRT 类型，保持依赖库与项目一致

------

**4️⃣ 查看可用 triplet**

在 vcpkg 根目录：

```powershell
.\vcpkg list
```

查看已安装库及对应 triplet：

```text
eigen3:x64-windows-static           3.4.0
opencv:x64-windows-static           4.7.0
```

查看可用 triplet：

```powershell
.\vcpkg help triplets
```

------

**5️⃣ 总结**

- vcpkg 库是 **按 triplet 区分的版本**，triplet 决定了：
  - 编译器（MSVC / GCC / Clang）
  - 平台架构（x86 / x64）
  - 动态/静态链接
- **选择正确的 triplet 才能让项目正常编译**，尤其是依赖库要和项目编译器匹配。
- MSVC → `x64-windows` / `x64-windows-static`
- GCC / MSYS2 → `x64-mingw-dynamic` / `x64-mingw-static`

