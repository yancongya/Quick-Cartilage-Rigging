---
title: API参考与开发接口
---

# API参考与开发接口

本文档详细介绍了"快速软骨绑定"插件提供的所有API接口，包括操作符、属性和开发接口。这些接口允许开发者编写脚本与插件交互或扩展插件功能。

## 概述

插件通过多种方式暴露API给开发者使用：

1. **操作符API**: 可通过 `bpy.ops` 调用的功能
2. **属性API**: 通过自定义属性访问的配置项
3. **偏好设置API**: 插件全局配置接口

## 操作符API (Operators)

### 主要绑定流程操作符

*   **`armature.subdivide_fib`**
    *   **标签**: 斐波那契细分
    *   **描述**: 使用斐波那契数列分割选中的骨骼，产生由疏到密的链条，适合做尾巴。
    *   **参数**:
        ```python
        segments: int (默认5, 范围1-100) # 细分段数
        coefficient: float (默认1.0, 范围0.01-10.0) # 斐波那契系数
        auto_execute: bool (默认False) # 自动执行完整流程
        ```

*   **`armature.subdivide_average`**
    *   **标签**: 平均细分
    *   **描述**: 将选中的骨骼平均分割为设定的段数。
    *   **参数**:
        ```python
        segments: int (默认5, 范围1-100) # 细分段数
        auto_execute: bool (默认False) # 自动执行完整流程
        ```

*   **`armature.setup_control_rig`**
    *   **标签**: 2.生成FK绑定
    *   **描述**: 为当前骨骼链生成一套FK控制器、父子关系和自定义图形。
    *   **注意**: 必须在编辑模式下运行

*   **`armature.apply_pose_setup`**
    *   **标签**: 3.生成软骨绑定
    *   **描述**: 应用所有姿态约束，包括复制旋转(FK)和软骨绑定，并设置驱动器。
    *   **注意**: 必须在姿态模式下运行

### 界面与辅助功能操作符

*   **`wm.check_addon_update`**
    *   **标签**: 检查更新
    *   **描述**: 从远程版本文件比对当前版本，必要时下载并覆盖更新。

*   **`armature.toggle_show_all_ctrl_bones`**
    *   **标签**: 切换显示所有控制骨骼
    *   **描述**: 显示或隐藏所有控制骨骼。

*   **`armature.toggle_show_first_only_ctrl_bone`**
    *   **标签**: 切换独显第一根控制骨骼
    *   **描述**: 控制是否独显控制骨骼的根骨骼。

### 模式切换操作符

*   **`wm.switch_object_mode`**
    *   **标签**: Object Mode
    *   **描述**: 切换到物体模式。

*   **`wm.switch_edit_mode`**
    *   **标签**: Edit Mode
    *   **描述**: 切换到编辑模式。

*   **`wm.switch_pose_mode`**
    *   **标签**: Pose Mode
    *   **描述**: 切换到姿态模式。

## 属性API (Properties)

### 插件偏好设置属性

这些属性储存在插件的偏好设置中，用于全局配置。

*   **访问路径**: `bpy.context.preferences.addons['Quick Cartilage Rigging.py'].preferences`

#### 属性列表

*   `show_in_n_panel: BoolProperty`
    *   是否在N面板显示。

*   `show_in_tool_panel: BoolProperty`
    *   是否在工具面板显示。

*   `enable_right_click_menu: BoolProperty`
    *   是否启用右键菜单。

*   `default_circle_scale: FloatProperty`
    *   新创建控制器的默认圆环缩放值 (范围0.0-5.0)。

*   `default_damped_track_influence: FloatProperty`
    *   新创建控制器的默认追踪强度（难崩系数）(范围0.0-1.0)。

### 姿态骨骼属性

这些属性被添加到每个姿态骨骼（PoseBone）上，但主要由控制器骨骼（`ctr_...`）使用。

*   **访问路径**: `pose_bone.my_tool_props`

#### 属性列表

*   `damped_track_influence: FloatProperty`
    *   难崩系数，控制整条链的阻尼追踪强度 (范围0.0-1.0)。

*   `circle_scale: FloatProperty`
    *   圆环缩放，通过驱动器控制整条链所有控制器的大小 (范围0.0-5.0)。

*   `show_all_ctrl_bones: BoolProperty`
    *   （内部属性）用于切换显示所有控制骨骼的逻辑。

*   `show_only_first_ctrl_bone: BoolProperty`
    *   （内部属性）用于切换独显第一根控制骨骼的逻辑。

### 场景属性

这些属性被添加到场景（Scene）中，主要用于在UI和操作符之间传递参数。

*   **访问路径**: `bpy.context.scene`

#### 属性列表

*   `fib_segments: IntProperty`
    *   UI上"段数"输入框的值 (范围1-100)。

*   `fib_coefficient: FloatProperty`
    *   UI上"系数"输入框的值 (范围0.01-10.0)。

## 使用示例

### 调用操作符

```python
import bpy

# 调用斐波那契细分
bpy.ops.armature.subdivide_fib(segments=8, coefficient=1.2)

# 调用FK绑定
bpy.ops.armature.setup_control_rig()

# 调用软骨绑定
bpy.ops.armature.apply_pose_setup()
```

### 访问和修改属性

```python
import bpy

# 获取活动对象
obj = bpy.context.active_object
if obj and obj.type == 'ARMATURE':
    # 获取第一个控制器骨骼
    controller_bone = obj.pose.bones.get("ctr_bone.001")
    if controller_bone:
        # 读取难崩系数
        current_influence = controller_bone.my_tool_props.damped_track_influence
        
        # 设置难崩系数
        controller_bone.my_tool_props.damped_track_influence = 0.8
        
        # 读取圆环缩放
        current_scale = controller_bone.my_tool_props.circle_scale
```

### 获取偏好设置

```python
import bpy

# 访问插件偏好设置
addon_prefs = bpy.context.preferences.addons.get("Quick Cartilage Rigging.py")
if addon_prefs:
    # 读取偏好设置
    show_in_n = addon_prefs.preferences.show_in_n_panel
    default_scale = addon_prefs.preferences.default_circle_scale
    
    # 修改偏好设置
    addon_prefs.preferences.default_damped_track_influence = 0.7
```

## 最佳实践

### 脚本中使用插件API的建议

1. **验证前提条件**: 在调用操作符前验证当前模式和选中对象
2. **错误处理**: 包含适当的异常处理
3. **上下文检查**: 确保在正确的Blender模式下运行操作符

### 自动化脚本示例

```python
import bpy

def automated_cartilage_rig():
    """自动化软骨绑定流程"""
    # 验证当前上下文
    if bpy.context.mode != 'EDIT_ARMATURE':
        print("请在编辑模式下运行此脚本")
        return
        
    obj = bpy.context.active_object
    if not obj or obj.type != 'ARMATURE':
        print("请选择骨架对象")
        return
    
    # 执行细分
    bpy.ops.armature.subdivide_average(segments=8)
    
    # 切换到姿态模式并应用绑定
    bpy.ops.object.mode_set(mode='POSE')
    bpy.ops.armature.apply_pose_setup()
    
    print("自动化软骨绑定完成")

# 运行自动化流程
# automated_cartilage_rig()
```

## 高级API用法

### 创建自定义操作符集成

```python
import bpy

class OBJECT_OT_enhanced_cartilage_setup(bpy.types.Operator):
    """增强版软骨绑定设置操作符"""
    bl_idname = "object.enhanced_cartilage_setup"
    bl_label = "增强版软骨绑定"
    bl_description = "执行软骨绑定并应用额外设置"
    
    # 添加自定义参数
    use_auto_stages: bpy.props.BoolProperty(
        name="自动执行阶段",
        description="自动执行细分、FK和软骨绑定",
        default=True
    )
    
    def execute(self, context):
        obj = context.active_object
        
        if self.use_auto_stages:
            # 使用插件API自动执行全流程
            bpy.ops.armature.subdivide_average(
                segments=8, 
                auto_execute=True  # 自动执行完整流程
            )
        else:
            # 分步执行
            bpy.ops.armature.subdivide_average(segments=8)
            bpy.ops.object.mode_set(mode='POSE')
            bpy.ops.armature.apply_pose_setup()
        
        return {'FINISHED'}

def register():
    bpy.utils.register_class(OBJECT_OT_enhanced_cartilage_setup)

def unregister():
    bpy.utils.unregister_class(OBJECT_OT_enhanced_cartilage_setup)
```

### 与插件深度集成

```python
import bpy

def setup_custom_control_properties():
    """设置自定义控制属性"""
    obj = bpy.context.active_object
    if not obj or obj.type != 'ARMATURE':
        return
    
    # 找到控制器骨骼
    for bone in obj.pose.bones:
        if bone.name.startswith("ctr_"):
            # 增强控制器属性
            props = bone.my_tool_props
            # 可以访问和修改难崩系数和圆环缩放
            props.damped_track_influence = min(props.damped_track_influence + 0.1, 1.0)
```