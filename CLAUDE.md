# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

蛐蛐 (QuQu) 是一个基于 Electron + React 的桌面端语音转文字应用，专为中文优化，使用阿里巴巴 FunASR 模型进行本地语音识别，支持可配置的 AI 模型进行文本优化。

## 开发命令

### 核心开发命令
- `pnpm run dev` - 启动开发环境 (同时运行前端和主进程)
- `pnpm run dev:renderer` - 仅启动前端开发服务器 (在 src 目录)
- `pnpm run dev:main` - 仅启动 Electron 主进程开发模式
- `pnpm run start` - 启动生产环境应用

### 构建命令
- `pnpm run build` - 完整构建 (前端构建 + Electron 打包)
- `pnpm run build:renderer` - 仅构建前端 (在 src 目录)
- `pnpm run pack` - 构建但不打包 (用于测试)
- `pnpm run dist` - 完整分发构建 (包含嵌入式 Python 环境)

### 平台特定构建
- `pnpm run build:mac` - macOS 构建
- `pnpm run build:win` - Windows 构建
- `pnpm run build:linux` - Linux 构建

### Python 环境管理
- `pnpm run prepare:python` - 准备嵌入式 Python 环境
- `pnpm run prepare:python:uv` - 使用 uv 准备 Python 环境 (推荐)
- `pnpm run test:python` - 测试嵌入式 Python 环境
- `python download_models.py` - 下载 FunASR 模型

### 代码质量
- `pnpm run lint` - 运行 ESLint 检查 (在 src 目录)
- `pnpm run preview` - 预览构建结果 (在 src 目录)
- `pnpm run clean` - 清理构建文件和 Python 环境

## 架构概述

### 技术栈
- **前端**: React 19, TypeScript, Tailwind CSS, shadcn/ui, Vite
- **桌面端**: Electron 36.5.0
- **语音识别**: FunASR (Paraformer-large, FSMN-VAD, CT-Transformer)
- **数据库**: better-sqlite3
- **Python 环境**: 支持系统 Python、uv 管理环境、嵌入式环境

### 核心架构

#### Electron 主进程 (main.js)
- 管理应用生命周期和全局快捷键 (F2)
- 协调各个管理器模块:
  - `EnvironmentManager` - 环境配置管理
  - `WindowManager` - 窗口管理
  - `DatabaseManager` - SQLite 数据库管理
  - `ClipboardManager` - 剪贴板管理
  - `FunASRManager` - FunASR 服务管理
  - `TrayManager` - 系统托盘管理
  - `HotkeyManager` - 全局快捷键管理
  - `IPCHandlers` - IPC 通信处理

#### 前端渲染进程
- **主界面** (`src/App.jsx`): 录音控制和实时转录
- **设置页面** (`src/settings.jsx`): AI 模型配置和应用设置
- **历史记录** (`src/history.jsx`): 转录历史管理

#### 核心钩子函数
- `useRecording` - 录音状态管理
- `useHotkey` - 快捷键处理
- `useTextProcessing` - 文本处理和 AI 优化
- `useModelStatus` - 模型状态监控
- `usePermissions` - 权限管理

#### FunASR 服务
- `funasr_server.py` - Python 后端服务，保持模型在内存中
- 通过 stdin/stdout 与 Electron 主进程通信
- 支持三种模型: ASR (语音识别)、VAD (语音活动检测)、PUNC (标点符号)

### 关键文件结构
```
ququ/
├── main.js                    # Electron 主进程入口
├── funasr_server.py          # FunASR Python 服务
├── download_models.py        # 模型下载脚本
├── src/                      # 前端代码
│   ├── App.jsx              # 主界面组件
│   ├── settings.jsx         # 设置页面组件
│   ├── history.jsx          # 历史记录组件
│   ├── components/          # UI 组件
│   ├── hooks/               # React 钩子
│   ├── helpers/             # Electron 辅助模块
│   └── vite.config.js       # Vite 配置
├── assets/                  # 静态资源
└── python/                  # 嵌入式 Python 环境
```

### 数据流程
1. 用户按下 F2 开始录音
2. `FunASRManager` 启动 `funasr_server.py` 服务
3. 音频数据发送到 Python 服务进行实时转录
4. 转录文本返回到前端显示
5. 可选择发送到 AI 模型进行优化
6. 最终文本通过 `ClipboardManager` 自动粘贴到光标位置

### 环境配置
应用支持三种 Python 环境配置方式:
1. **uv 管理** (推荐): 自动管理 Python 版本和依赖
2. **系统 Python + 虚拟环境**: 使用现有 Python 环境
3. **嵌入式 Python**: 完全隔离的环境，用于生产构建

### AI 模型集成
- 支持任何兼容 OpenAI API 的服务
- 优先适配国产模型 (通义千问、Kimi、智谱AI等)
- 配置存储在本地 SQLite 数据库中