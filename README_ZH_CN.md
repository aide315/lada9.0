# Lada v0.9.0 - 完整重构版本（含Re-mosaic功能）

本仓库包含Lada应用程序的完整重构版本，已集成**re-mosaic后处理功能**（来自PR #151）。

## 🚀 快速开始

### 下载已构建的EXE

**自动构建的EXE程序**：
- 地址：https://github.com/aide315/lada9.0
- 导航到 **Actions** 标签 → 选择最新的 **Build Windows EXE** 工作流
- 点击工作流运行 → 向下滚动到 **Artifacts** 部分
- 下载 `lada-windows-exe` 压缩包（包含编译好的exe文件）

**发布版本**（标签推送）：
- https://github.com/aide315/lada9.0/releases
- 查找 `lada-*.7z.001` 和 `lada-*.7z.002` 文件进行下载

---

## 📦 Re-mosaic 功能说明

此版本添加了**re-mosaic后处理**功能，允许在视频恢复后重新应用马赛克效果。

### 功能特性：
- **快速模式**：块大小 ≤7px 使用高效的OpenCV缩放算法
- **精确模式**：块大小 >7px 使用高精度马赛克算法
- **可配置块大小**：范围 0-32 像素
- **GUI/CLI 支持**：在图形界面和命令行中都可使用

### 使用方法：

#### GUI方式
1. 启动Lada应用程序
2. 在 **配置面板** 中找到 **Re-mosaic block size** 设置
3. 输入块大小（0=禁用，2-32=启用）
4. 在预览或导出时自动应用

#### CLI方式
```bash
lada-cli.exe input_video.mp4 --re-mosaic-block-size 4 --output output_video.mp4
```

**参数说明**：
- `--re-mosaic-block-size <N>`：设置马赛克块大小
  - 0 或不指定：禁用re-mosaic
  - 2-32：启用，设置块大小（像素）

---

## 🔨 本地构建

### 系统要求
- **Windows 10/11** (64-bit)
- **Python 3.13**
- **Visual Studio 2022 Build Tools**
- **7-Zip**（用于打包）
- **FFmpeg**、**MSYS2**、**Rust**

### 构建步骤

1. **安装依赖**
```powershell
# 使用 uv 包管理器（推荐）
pip install uv

# 创建虚拟环境
uv venv --python 3.13 venv
.\venv\Scripts\Activate.ps1
```

2. **运行构建脚本**
```powershell
cd lada-v0.9.0/lada

# 完整构建（包括GTK）
powershell -ExecutionPolicy Bypass ./packaging/windows/package_executable.ps1

# 仅CLI构建（快速）
powershell -ExecutionPolicy Bypass ./packaging/windows/package_executable.ps1 -cli-only

# 跳过系统依赖安装
powershell -ExecutionPolicy Bypass ./packaging/windows/package_executable.ps1 -skip-winget -skip-gvsbuild
```

3. **输出文件**
- CLI可执行文件：`lada-v0.9.0/lada/dist/lada/lada-cli.exe`
- GUI可执行文件：`lada-v0.9.0/lada/dist/lada/lada.exe`（完整构建）
- 7z压缩包：`lada-v0.9.0/lada/dist/lada-<version>.7z`

---

## ⚙️ CI/CD 工作流

本仓库配置了 **GitHub Actions** 自动构建Windows EXE。

### 工作流文件
- 位置：`.github/workflows/build-windows-exe.yml`
- 触发条件：
  - ✅ 推送到 `main` 或 `feat/re-mosaic` 分支
  - ✅ Pull Request 到 `main` 分支
  - ✅ 手动触发（Workflow dispatch）

### 访问构建结果

1. **自动构建工件**（最新）
   - https://github.com/aide315/lada9.0/actions
   - 找到最新工作流运行
   - 下载 **Artifacts** 中的 `lada-windows-exe`

2. **发布版本**（稳定）
   - https://github.com/aide315/lada9.0/releases
   - 查找标记为发布的版本

### 手动触发构建
```powershell
# 使用GitHub CLI
gh workflow run build-windows-exe.yml -R aide315/lada9.0
```

或在网页版本中：
1. 前往 GitHub Actions
2. 选择 "Build Windows EXE" 工作流
3. 点击 "Run workflow" 按钮

---

## 📂 项目结构

```
lada-v0.9.0/
├── lada/                          # 主源代码
│   ├── lada/
│   │   ├── cli/                  # 命令行接口
│   │   ├── gui/                  # 图形用户界面
│   │   ├── restorationpipeline/  # 核心恢复管道（包含re-mosaic）
│   │   └── utils/                # 工具函数
│   ├── packaging/
│   │   └── windows/              # Windows打包脚本
│   └── model_weights/            # AI模型权重
├── .github/
│   └── workflows/
│       └── build-windows-exe.yml  # GitHub Actions配置
└── README.md
```

---

## 🐛 故障排除

### 构建错误：Clock Skew
如果遇到时间戳相关错误，重写时间戳：
```powershell
$now = (Get-Date)
Get-ChildItem -Path ./build_gtk_release/build -Recurse | 
  ForEach-Object { $_.LastWriteTime = $now }
```

### 构建错误：缺少依赖
- 运行完整的 `package_executable.ps1` 而不使用 `-cli-only` 标志
- 确保已安装所有系统依赖（winget packages）

### GitHub Actions构建失败
- 检查工作流日志：GitHub Actions → 工作流运行 → 日志
- 常见原因：网络超时、模型权重下载失败

---

## 📝 版本信息

- **基础版本**：Lada v0.9.0
- **新增功能**：Re-mosaic后处理（PR #151）
- **修改文件**：
  - `lada/restorationpipeline/frame_restorer.py`
  - `lada/cli/main.py`
  - `lada/gui/frame_restorer_provider.py`
  - `lada/gui/config/config.py`
  - `lada/gui/config/config_sidebar.py`
  - `lada/gui/config/config_sidebar.ui`
  - `lada/gui/preview/preview_view.py`
  - `lada/gui/export/export_view.py`

---

## 📄 许可证

本项目遵循原项目许可证。详见 `lada-v0.9.0/lada/LICENSE.md`

---

## 🔗 相关链接

- **原项目仓库**：https://codeberg.org/ladaapp/lada
- **PR #151**：https://codeberg.org/ladaapp/lada/pulls/151
- **本项目fork**：https://github.com/aide315/lada9.0
- **开发分支**：https://github.com/aide315/lada/tree/feat/re-mosaic

---

## 💡 后续步骤

1. **测试re-mosaic功能**：
   - 使用CLI或GUI测试re-mosaic参数
   - 验证输出视频质量

2. **提交PR到主仓库**：
   - 向 https://codeberg.org/ladaapp/lada 提交PR
   - 获取原项目维护者的反馈

3. **发布版本**：
   - 创建GitHub Release标记（如 v0.9.0-re-mosaic）
   - 附加编译好的EXE文件
   - 更新下载链接

