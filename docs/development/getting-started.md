---
title: 开发者入门
---

# 开发者入门

欢迎来到"快速软骨绑定"插件的开发指南！本文档将帮助您了解如何开始为插件贡献代码或进行自定义修改。

## 开发环境设置

### 系统要求

- **操作系统**: Windows 7+, macOS 10.14+, 或 Linux
- **Python版本**: 3.8+ (Blender自带)
- **Blender版本**: 4.5.0+ (推荐使用最新稳定版)

### 获取源代码

1. 克隆项目仓库：
   ```bash
   git clone https://github.com/yancongya/Quick-Cartilage-Rigging.git
   ```

2. 或者直接下载源代码压缩包并解压

### 开发工具推荐

- **代码编辑器**: VS Code、PyCharm 或 Sublime Text
- **Blender插件**: Blender Python Console、Text Editor 等用于调试
- **版本控制**: Git (建议使用图形化客户端如 GitKraken)

## 项目结构

```
Quick-Cartilage-Rigging/
├── Quick Cartilage Rigging.py     # 主插件文件
├── version.txt                   # 版本信息
├── .git/                         # Git仓库
├── 迭代/                         # 历史版本
└── docs/                         # 文档目录
    ├── index.md
    ├── guide/                    # 用户指南
    ├── features/                 # 功能详解
    ├── advanced/                 # 高级配置
    └── development/              # 开发指南 (当前目录)
```

## 设置开发环境

### 方法一：直接在Blender中开发

1. 打开Blender
2. 进入 `编辑(Edit)` > `偏好设置(Preferences)` > `插件(Add-ons)`
3. 点击 `安装(Install)`，选择项目目录中的 `Quick Cartilage Rigging.py`
4. 启用插件

### 方法二：通过符号链接开发

1. 在Blender的插件目录创建符号链接（便于开发时实时修改）
2. 这样可以直接修改源文件，保存后重新加载插件即可看到效果

## 插件交互方式

### 通过UI操作

最直接的方式是使用插件提供的用户界面：

1. 在3D视图侧边栏（N面板）中找到"Damped Track"选项卡
2. 根据当前模式（编辑/姿态）使用对应的功能
3. 在姿态模式下调节控制器属性

## 插件核心概念

### 主要组件

- `SubdivideFibOperator`: 斐波那契细分功能
- `SubdivideAverageOperator`: 平均细分功能
- `SetupControlRigOperator`: FK绑定生成
- `ApplyPoseConstraintsOperator`: 软骨绑定（阻尼追踪）
- `MyArmatureProperties`: 自定义属性存储
- `DampedTrackPanel`: 主面板UI

## 代码风格

### Python编码规范

- 遵循PEP 8 Python编码规范
- 使用有意义的变量和函数命名
- 添加适当的注释和文档字符串
- 使用中文注释（便于理解和维护）

### Blender API最佳实践

- 使用 `bpy.ops` 调用内置操作符
- 使用 `bpy.context` 访问当前上下文
- 遵循Blender的注册/注销模式

## 调试技巧

### 使用Blender Python控制台

1. 打开 `窗口(Window)` > `开发者(Developer)` > `Python控制台`
2. 可以直接运行Python代码测试功能

### 常见调试方法

1. 使用 `print()` 函数输出调试信息
2. 利用Blender内置的 `bpy.context.view_layer` 进行场景访问
3. 使用 `bpy.ops.wm.redraw_timer()` 强制重绘界面

## 贡献流程

1. Fork项目仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启Pull Request

## 下一步

- 阅读[插件架构](./architecture.md)了解详细设计
- 查看[API参考与开发接口](./api-reference.md)学习可用接口
- 参考[扩展功能](./extending.md)学习如何添加新特性
- 参考[贡献指南](./contributing.md)了解贡献规范