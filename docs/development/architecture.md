---
title: 插件架构
---

# 插件架构

本文档详细介绍了"快速软骨绑定"插件的内部架构和模块设计，帮助开发者理解插件的工作原理。

## 整体架构

插件采用模块化设计，主要包括以下几个核心部分：

### 1. 操作符层 (Operators)

操作符是插件功能的核心实现部分，负责执行具体的骨骼绑定任务：

- `SubdivideFibOperator`: 实现斐波那契细分算法
- `SubdivideAverageOperator`: 实现平均细分算法  
- `SetupControlRigOperator`: 创建FK控制器系统
- `ApplyPoseConstraintsOperator`: 应用阻尼追踪约束
- `WM_OT_*`: 模式切换和辅助操作符

### 2. UI层 (User Interface)

UI层负责与用户的交互，包括：

- `DampedTrackPanel`: 主面板，根据当前模式动态显示功能
- 右键菜单项：`VIEW3D_MT_damped_track_*_menu` 系列
- 属性面板：显示控制器属性

### 3. 数据模型层 (Data Model)

数据模型层管理插件的状态和配置：

- `MyArmatureProperties`: 骨骼属性，存储难崩系数等参数
- `DampedTrackAddonPreferences`: 插件偏好设置

## 核心工作流程

### 骨骼细分流程

1. **输入验证**: 检查当前模式和选中骨骼
2. **算法执行**: 根据选择的算法（平均/斐波那契）细分骨骼
3. **命名处理**: 使用 `get_unique_base_name` 函数确保名称唯一性
4. **父子关系重建**: 重建细分后骨骼的父子关系

### FK绑定流程

1. **识别骨骼链**: 从活动骨骼推断整个骨骼链
2. **创建控制器**: 为每个变形骨骼创建对应的控制器
3. **设置自定义图形**: 为控制器分配圆形自定义形状
4. **创建驱动器**: 建立控制器到形变骨骼的驱动关系

### 软骨绑定流程

1. **FK约束**: 为形变骨骼添加复制旋转约束
2. **阻尼追踪**: 为骨骼建立追踪约束链
3. **驱动器设置**: 将所有追踪强度连接到统一控制器

## 关键数据结构

### 骨骼命名约定

- **变形骨骼**: `basename.001`, `basename.002`, ...
- **控制器骨骼**: `ctr_basename.001`, `ctr_basename.002`, ...
- **末端骨骼**: `basename.000` (用于追踪目标)

### 骨骼集合

插件使用骨骼集合来管理控制器可见性：

- `ctrl_{basename}_all`: 包含所有控制器
- `ctrl_{basename}_first`: 仅包含第一个控制器

## 模块详解

### 骨骼细分模块

```python
# 斐波那契细分算法核心
fib = [1.0, 1.0]
for i in range(2, segments):
    fib.append(fib[-1] + coefficient * fib[-2])
```

### 控制器系统模块

- **统一属性管理**: 所有控制器的属性由第一个控制器统一管理
- **驱动器系统**: 使用驱动器实现参数的统一控制
- **自定义图形**: 圆形控制器便于操作

### 可见性控制系统

实现了一个双状态的控制器可见性管理系统：

```python
# 控制器可见性更新逻辑
if show_only_first_ctrl_bone:
    all_collection.is_visible = False
    first_collection.is_visible = True
elif show_all_ctrl_bones:
    all_collection.is_visible = True
    first_collection.is_visible = False
```

## 插件架构设计

### 设计原则

1. **模块化设计**: 功能按职责分离，易于维护和扩展
2. **用户友好**: 提供直观的UI界面和清晰的交互流程
3. **性能优先**: 优化算法和数据处理流程
4. **兼容性**: 支持不同版本的Blender

## 插件生命周期

### 注册流程 (`register()`)

1. **属性注册**: 添加自定义属性到 `Scene` 和 `PoseBone`
2. **类注册**: 注册所有操作符、面板和菜单
3. **面板注册**: 根据偏好设置注册面板
4. **UI注册**: 注册右键菜单项

### 注销流程 (`unregister()`)

1. **UI注销**: 注销右键菜单项
2. **面板注销**: 注销所有面板
3. **类注销**: 注销所有类
4. **属性注销**: 移除自定义属性

## 技术要点

### Blender API 使用

- **模式切换**: 使用 `bpy.ops.object.mode_set()` 进行模式切换
- **骨骼操作**: 在编辑模式下操作 `arm.edit_bones`，在姿态模式下操作 `obj.pose.bones`
- **UI更新**: 使用 `area.tag_redraw()` 强制UI重绘

### 驱动器系统

驱动器用于实现参数的统一控制：

- `circle_scale`: 控制所有控制器的大小
- `damped_track_influence`: 控制所有追踪约束的强度

### 内存管理

- **集合管理**: 动态创建和管理骨骼集合
- **临时对象**: 及时清理临时创建的网格对象
- **属性缓存**: 合理使用Blender的属性系统避免重复计算

## 扩展接口

插件提供了丰富的API供其他脚本调用：

- **操作符**: 可通过 `bpy.ops.armature.*` 调用
- **属性**: 可通过 `pose_bone.my_tool_props.*` 访问
- **偏好设置**: 可通过 `bpy.context.preferences.addons` 访问

## 性能优化

- **批量操作**: 在单次操作中完成多个骨骼的处理
- **缓存机制**: 使用缓存避免重复计算
- **事件驱动**: 响应式UI更新，仅在必要时重绘