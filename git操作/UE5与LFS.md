下面给你一套适合 **个人多设备开发 UE5 项目** 的完整流程。场景假设是：你有一台主力电脑和一台笔记本，想在两台设备之间同步同一个 UE5 项目。

推荐方案是：

```text
Git + Git LFS + 远程仓库
```

比如：

```text
GitHub / GitLab / Gitee / 自建 Git 服务器
```

------

# 一、整体思路

UE5 项目不要靠网盘实时同步整个文件夹，而应该这样做：

```text
本地 UE5 项目
    ↓
Git 管理代码和配置
    ↓
Git LFS 管理大资源文件
    ↓
远程仓库作为中转
    ↓
另一台设备 clone / pull
```

也就是：

```text
设备 A 修改项目 → 提交 → 推送到远程仓库
设备 B 拉取远程仓库 → 继续开发
```

------

# 二、需要同步哪些文件？

UE5 项目中应该进入 Git 的主要内容是：

```text
Config/
Content/
Source/
Plugins/
*.uproject
```

它们分别是：

| 文件 / 目录         | 作用                         | 是否同步             |
| ------------------- | ---------------------------- | -------------------- |
| `Config/`           | 项目配置                     | 是                   |
| `Content/`          | 蓝图、材质、模型、地图等资源 | 是                   |
| `Source/`           | C++ 源码                     | 是                   |
| `Plugins/`          | 项目插件                     | 看情况，一般建议同步 |
| `*.uproject`        | 项目入口文件                 | 是                   |
| `Binaries/`         | 编译产物                     | 否                   |
| `Intermediate/`     | 中间文件                     | 否                   |
| `Saved/`            | 日志、自动保存、缓存         | 否                   |
| `DerivedDataCache/` | UE 资源缓存                  | 否                   |
| `.vs/`              | Visual Studio 缓存           | 否                   |

------

# 三、第一台设备：初始化项目仓库

假设你的 UE5 项目目录是：

```text
D:/UEProjects/MyProject/
```

项目结构大概是：

```text
MyProject/
├─ Config/
├─ Content/
├─ Source/
├─ MyProject.uproject
├─ Binaries/
├─ Intermediate/
├─ Saved/
└─ DerivedDataCache/
```

------

## 1. 安装 Git 和 Git LFS

需要安装：

```text
Git
Git LFS
```

安装完成后，在项目根目录打开终端：

```bash
cd D:/UEProjects/MyProject
```

执行：

```bash
git --version
git lfs version
```

如果能看到版本号，说明安装成功。

------

## 2. 初始化 Git 仓库

在 UE5 项目根目录执行：

```bash
git init
```

然后初始化 Git LFS：

```bash
git lfs install
```

------

## 3. 创建 `.gitignore`

在项目根目录创建 `.gitignore` 文件。

内容可以使用这个：

```gitignore
# Unreal Engine generated folders
Binaries/
DerivedDataCache/
Intermediate/
Saved/

# Visual Studio
.vs/
*.suo
*.user
*.userosscache
*.sln.docstates
*.VC.db
*.VC.opendb

# Rider / JetBrains
.idea/

# VSCode
.vscode/

# Build files
*.opensdf
*.sdf
*.opendb

# Logs
*.log

# OS files
.DS_Store
Thumbs.db
```

注意，不要把这些目录忽略掉：

```text
Config/
Content/
Source/
Plugins/
*.uproject
```

------

## 4. 配置 Git LFS 管理 UE5 大文件

在项目根目录执行：

```bash
git lfs track "*.uasset"
git lfs track "*.umap"
git lfs track "*.fbx"
git lfs track "*.png"
git lfs track "*.tga"
git lfs track "*.jpg"
git lfs track "*.jpeg"
git lfs track "*.wav"
git lfs track "*.mp3"
git lfs track "*.mp4"
git lfs track "*.mov"
git lfs track "*.exr"
git lfs track "*.hdr"
git lfs track "*.obj"
git lfs track "*.blend"
```

执行后会生成一个文件：

```text
.gitattributes
```

这个文件也必须提交到 Git。

你可以查看：

```bash
git lfs track
```

------

## 5. 添加项目文件

执行：

```bash
git add .
```

查看将要提交的文件：

```bash
git status
```

你应该能看到：

```text
Config/
Content/
Source/
MyProject.uproject
.gitattributes
.gitignore
```

不应该看到大量这些目录：

```text
Binaries/
Intermediate/
Saved/
DerivedDataCache/
.vs/
```

如果这些目录出现在 `git status` 里，说明 `.gitignore` 没写对，先不要提交。

------

## 6. 第一次提交

确认没问题后执行：

```bash
git commit -m "Initial UE5 project"
```

------

# 四、创建远程仓库

你可以在 GitHub、GitLab、Gitee 上创建一个空仓库。

例如远程仓库地址类似：

```text
https://github.com/你的用户名/MyProject.git
```

然后在本地项目目录执行：

```bash
git remote add origin https://github.com/你的用户名/MyProject.git
git branch -M main
git push -u origin main
```

如果项目里有 LFS 文件，推送时 Git LFS 会一起上传大文件。

------

# 五、第二台设备：拉取项目

在第二台电脑上，先安装：

```text
Git
Git LFS
UE5 相同版本
Visual Studio / Rider / VSCode，按你的开发环境选择
```

然后选择一个目录，例如：

```text
D:/UEProjects/
```

执行：

```bash
cd D:/UEProjects
git clone https://github.com/你的用户名/MyProject.git
```

进入项目目录：

```bash
cd MyProject
```

确保 LFS 文件拉取完整：

```bash
git lfs pull
```

然后双击：

```text
MyProject.uproject
```

打开项目。

------

# 六、C++ 项目需要额外做什么？

如果你的 UE5 项目是 C++ 项目，第二台电脑 clone 后，通常需要重新生成工程文件。

在 Windows 上可以右键：

```text
MyProject.uproject
```

选择：

```text
Generate Visual Studio project files
```

然后打开 `.sln` 文件编译。

如果你用的是 UE5 + Visual Studio，也可以直接双击 `.uproject`，UE 有时会提示你重新编译模块。

------

# 七、日常开发流程

这是最重要的部分。

## 设备 A 开始开发前

每次开始开发前，先拉取最新版本：

```bash
git pull
git lfs pull
```

然后再打开 UE5 项目。

------

## 设备 A 修改完成后

关闭 UE5 编辑器，或者至少保存所有资源。

然后查看改动：

```bash
git status
```

添加改动：

```bash
git add .
```

提交：

```bash
git commit -m "修改角色移动逻辑"
```

推送到远程：

```bash
git push
```

------

## 设备 B 继续开发前

在设备 B 上，先执行：

```bash
git pull
git lfs pull
```

然后再打开 UE5 项目。

------

# 八、个人多设备最推荐的工作习惯

## 1. 每次换设备前一定要 push

例如你在台式机开发完，要去笔记本继续做：

```bash
git add .
git commit -m "Save current progress"
git push
```

然后笔记本：

```bash
git pull
git lfs pull
```

不要出现这种情况：

```text
台式机改了一半没提交
笔记本又开始改同一个文件
```

这样很容易产生冲突。

------

## 2. 不要两台设备同时打开并修改同一个项目

尤其不要同时修改：

```text
.umap
.uasset
蓝图
材质
关卡
动画资源
```

因为这些大多是二进制文件，Git 很难合并。

对于个人多设备开发，最安全的习惯是：

```text
一台设备开发 → 提交推送 → 另一台设备拉取 → 再开发
```

------

## 3. 提交前先保存所有 UE 资源

在 UE5 编辑器里点击：

```text
File -> Save All
```

或者快捷键：

```text
Ctrl + Shift + S
```

否则可能出现：

```text
你以为改了资源
但实际文件没有保存
Git 也检测不到变化
```

------

## 4. 不要提交临时文件

如果你看到 Git 要提交这些内容，通常是不应该的：

```text
Binaries/
Intermediate/
Saved/
DerivedDataCache/
.vs/
```

这些文件可以在不同设备上重新生成，不需要同步。

------

# 九、推荐的提交粒度

不要一直堆到最后才提交。

比较好的提交方式是：

```text
完成一个小功能就提交一次
修复一个 bug 就提交一次
添加一批资源就提交一次
修改一个系统就提交一次
```

例如：

```bash
git commit -m "Add player movement blueprint"
git commit -m "Create main level lighting setup"
git commit -m "Fix camera rotation issue"
git commit -m "Add enemy patrol behavior"
```

不太推荐：

```bash
git commit -m "update"
git commit -m "test"
git commit -m "aaa"
```

------

# 十、推荐的个人分支方式

如果只是你一个人多设备开发，可以很简单：

```text
main
```

直接在 `main` 上开发也可以。

如果你想更规范一点，可以这样：

```text
main      稳定版本
dev       日常开发版本
```

流程是：

```bash
git checkout -b dev
```

平时在 `dev` 开发：

```bash
git add .
git commit -m "Add inventory system"
git push -u origin dev
```

当项目阶段稳定后，再合并到 `main`：

```bash
git checkout main
git merge dev
git push
```

对于个人项目，简单方案就够了：

```text
一个 main 分支 + 经常提交
```

------

# 十一、UE5 编辑器里连接 Git

UE5 编辑器右下角通常有：

```text
Source Control
```

可以选择：

```text
Connect to Source Control
```

然后选择 Git。

连接后，你可以在 UE5 里看到资源状态，比如：

```text
已修改
新增
需要提交
```

不过对个人开发来说，我更推荐：

```text
UE5 里保存资源
外部终端 / VSCode / Git GUI 提交
```

因为这样更清楚每次提交了什么。

------

# 十二、Git GUI 工具推荐

如果你不想每次都打命令，可以用图形界面工具。

常见选择：

```text
GitHub Desktop
SourceTree
Fork
GitKraken
VSCode Git 面板
Visual Studio Git Changes
```

但注意：
即使用 GUI，也要先正确配置：

```text
.gitignore
.gitattributes
Git LFS
```

否则 UE5 大文件仍然可能管理不正确。

------

# 十三、Git LFS 锁定文件，个人多设备也有用

虽然你是个人开发，但如果担心两台设备同时改同一个资源，可以使用 LFS lock。

例如锁定主地图：

```bash
git lfs lock Content/Maps/Main.umap
```

解锁：

```bash
git lfs unlock Content/Maps/Main.umap
```

查看锁定状态：

```bash
git lfs locks
```

不过个人多设备开发时，更简单的做法是：

```text
不要两台设备同时改
换设备前一定 commit + push
换设备后一定 pull + lfs pull
```

------

# 十四、常见问题

## 1. 第二台电脑打开项目后资源丢失

可能是 LFS 文件没有拉下来。

执行：

```bash
git lfs pull
```

如果 `.uasset` 文件很小，比如只有几百字节，而且内容像这样：

```text
version https://git-lfs.github.com/spec/v1
oid sha256:...
size ...
```

说明它只是 LFS 指针文件，不是真正的资源文件。

------

## 2. clone 很慢

UE5 项目资源大是正常的。

可以检查：

```bash
git lfs ls-files
```

看看哪些文件在 LFS 里。

如果项目巨大，可以考虑：

```bash
git clone --depth 1 <仓库地址>
```

不过个人项目一般不必一开始就这样。

------

## 3. GitHub 提示 LFS 配额不够

GitHub 的 LFS 有容量和流量限制。UE5 项目资源多时可能会很快用完。

可选方案：

```text
购买 GitHub LFS 配额
使用 GitLab
使用 Gitee
使用自建 Git LFS 服务
改用 Perforce
```

如果项目资源很大，Perforce 会更适合。

------

## 4. 换电脑后 C++ 编译失败

一般检查这些：

```text
UE5 版本是否一致
Visual Studio 是否安装了 C++ 桌面开发组件
是否安装了对应 Windows SDK
是否重新生成了 Visual Studio project files
插件是否齐全
```

可以尝试删除本地生成目录：

```text
Binaries/
Intermediate/
Saved/
```

然后重新生成工程文件并编译。

------

## 5. 两台设备产生冲突怎么办？

如果冲突发生在代码文件，比如：

```text
.cpp
.h
.ini
```

通常可以手动合并。

如果冲突发生在：

```text
.uasset
.umap
```

通常不能手动合并，只能选择保留其中一个版本。

例如保留远程版本：

```bash
git checkout --theirs Content/Maps/Main.umap
git add Content/Maps/Main.umap
git commit -m "Resolve map conflict"
```

保留本地版本：

```bash
git checkout --ours Content/Maps/Main.umap
git add Content/Maps/Main.umap
git commit -m "Resolve map conflict"
```

所以最好的办法不是解决冲突，而是避免冲突。

------

# 十五、一个完整示例

假设你在台式机创建了 UE5 项目：

```text
D:/UEProjects/MyGame
```

完整初始化流程：

```bash
cd D:/UEProjects/MyGame

git init
git lfs install

git lfs track "*.uasset"
git lfs track "*.umap"
git lfs track "*.fbx"
git lfs track "*.png"
git lfs track "*.tga"
git lfs track "*.jpg"
git lfs track "*.wav"
git lfs track "*.mp4"

git add .
git commit -m "Initial UE5 project"

git remote add origin https://github.com/你的用户名/MyGame.git
git branch -M main
git push -u origin main
```

然后在笔记本：

```bash
cd D:/UEProjects
git clone https://github.com/你的用户名/MyGame.git
cd MyGame
git lfs pull
```

之后每次开发：

```bash
git pull
git lfs pull
```

开发完成后：

```bash
git add .
git commit -m "描述这次修改"
git push
```

------

# 十六、推荐你的实际使用流程

对于你个人多设备开发 UE5，我建议直接采用这个流程：

```text
1. 用 GitHub / GitLab / Gitee 创建私有仓库
2. 本地 UE5 项目初始化 Git
3. 配置 .gitignore
4. 配置 Git LFS 管理 .uasset 和 .umap
5. 第一次 commit
6. push 到远程仓库
7. 另一台设备 clone
8. 每次换设备前 push
9. 每次换设备后 pull + git lfs pull
10. 不要两台设备同时修改同一个项目
```

最关键的三件事是：

```text
.gitignore 忽略临时文件
Git LFS 管理 UE5 资源文件
换设备前后严格 push / pull
```