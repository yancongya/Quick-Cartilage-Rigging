---
title: 扩展功能
---

# 扩展功能

本文档介绍如何为"快速软骨绑定"插件添加新功能或修改现有功能，包括自定义操作符、UI扩展和功能增强。

## 创建自定义操作符

### 基本操作符结构

```python
import bpy

class CUSTOM_OT_my_custom_operator(bpy.types.Operator):
    bl_idname = "armature.my_custom_function"
    bl_label = "我的自定义功能"
    bl_description = "自定义功能描述"
    bl_options = {'REGISTER', 'UNDO'}

    # 定义属性
    my_parameter: bpy.props.FloatProperty(
        name="参数",
        description="自定义参数",
        default=1.0,
        min=0.0,
        max=10.0
    )

    @classmethod
    def poll(cls, context):
        # 定义操作符可用条件
        return (context.mode == 'POSE' and 
                context.object and 
                context.object.type == 'ARMATURE')

    def execute(self, context):
        # 主要功能逻辑
        obj = context.object
        # 在这里添加您的逻辑
        self.report({'INFO'}, f"执行自定义功能，参数: {self.my_parameter}")
        return {'FINISHED'}

# 注册操作符
def register():
    bpy.utils.register_class(CUSTOM_OT_my_custom_operator)

def unregister():
    bpy.utils.unregister_class(CUSTOM_OT_my_custom_operator)
```

### 继承现有功能

可以继承现有操作符来扩展功能：

```python
from Quick_Cartilage_Rigging import SubdivideFibOperator

class ExtendedSubdivideOperator(SubdivideFibOperator):
    bl_idname = "armature.extended_subdivide"
    bl_label = "扩展细分功能"
    
    # 添加额外属性
    additional_param: bpy.props.BoolProperty(
        name="额外参数",
        default=False
    )
    
    def execute(self, context):
        # 调用父类功能
        result = super().execute(context)
        
        # 添加额外逻辑
        if self.additional_param:
            # 执行额外功能
            pass
            
        return result
```

## 扩展UI面板

### 自定义面板

```python
import bpy

class CUSTOM_PT_my_panel(bpy.types.Panel):
    bl_label = "我的扩展面板"
    bl_idname = "OBJECT_PT_my_custom_panel"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'UI'
    bl_category = 'Damped Track'  # 与主面板同类别

    def draw(self, context):
        layout = self.layout
        obj = context.object
        
        if obj and obj.type == 'ARMATURE':
            # 添加自定义UI元素
            row = layout.row()
            row.operator("armature.my_custom_function")
            
            # 添加属性控件
            box = layout.box()
            box.label(text="自定义设置")
            # 根据需要添加控件
```

### 扩展现有面板

可以通过UI预设钩子扩展现有面板：

```python
# 扩展主面板功能
def extend_damped_track_panel(self, context):
    layout = self.layout
    
    # 添加扩展功能
    col = layout.column()
    col.label(text="扩展功能", icon='PLUGIN')
    col.operator("armature.my_custom_function")

# 在注册时添加
def register():
    # 注册自定义操作符
    bpy.utils.register_class(ExtendedSubdivideOperator)
    
    # 扩展现有面板
    from Quick_Cartilage_Rigging import DampedTrackPanel
    original_draw = DampedTrackPanel.draw
    def extended_draw(self, context):
        original_draw(self, context)  # 调用原方法
        extend_damped_track_panel(self, context)  # 添加扩展
    DampedTrackPanel.draw = extended_draw
```

## 自定义约束和驱动器

### 添加新的约束类型

```python
def apply_custom_constraint(pose_bone, target_bone):
    """为骨骼应用自定义约束"""
    # 移除现有约束
    for const in pose_bone.constraints:
        if const.type == 'LIMIT_ROTATION':
            pose_bone.constraints.remove(const)
    
    # 添加新约束
    constraint = pose_bone.constraints.new('LIMIT_ROTATION')
    constraint.use_limit_x = True
    constraint.min_x = -0.5
    constraint.max_x = 0.5
```

### 自定义驱动器系统

```python
def setup_custom_driver(target_property, source_property_path, source_object):
    """设置自定义驱动器"""
    fcurve = target_property.driver_add("influence")
    driver = fcurve.driver
    driver.type = 'SCRIPTED'
    driver.expression = "var * 2.0"  # 自定义表达式
    
    var = driver.variables.new()
    var.name = "var"
    var.type = 'SINGLE_PROP'
    var.targets[0].id = source_object
    var.targets[0].data_path = source_property_path
```

## 集成到现有流程

### 钩子函数

可以在现有流程中添加钩子：

```python
# 保存原始函数
from Quick_Cartilage_Rigging import SetupControlRigOperator
original_execute = SetupControlRigOperator.execute

def extended_execute(self, context):
    # 执行原始功能
    result = original_execute(self, context)
    
    # 在原始功能后执行额外逻辑
    if result == {'FINISHED'}:
        # 添加扩展逻辑
        self.custom_post_process(context)
    
    return result

# 替换execute方法
SetupControlRigOperator.execute = extended_execute

def custom_post_process(self, context):
    """自定义后处理逻辑"""
    # 在这里添加您的自定义逻辑
    print("执行扩展后处理")
```

## 自定义骨骼形状

### 创建新的自定义形状

```python
def create_custom_shape_object(obj, shape_name, shape_type="CIRCLE"):
    """创建自定义形状对象"""
    bpy.ops.object.mode_set(mode='OBJECT')
    
    # 删除现有对象（如果存在）
    if shape_name in bpy.data.objects:
        bpy.data.objects.remove(bpy.data.objects[shape_name], do_unlink=True)
    
    if shape_type == "CIRCLE":
        bpy.ops.mesh.primitive_circle_add(radius=0.5, vertices=32, 
                                         fill_type='NOTHING', location=obj.location)
    elif shape_type == "SQUARE":
        bpy.ops.mesh.primitive_cube_add(size=1.0, location=obj.location)
        # 调整为方形控制器
    
    shape_obj = context.active_object
    shape_obj.name = shape_name
    shape_obj.hide_viewport = True  # Blender 4.x兼容
    
    # 设为当前对象
    context.view_layer.objects.active = obj
    bpy.ops.object.mode_set(mode='POSE')
    
    return shape_obj
```

## 工具集成

### 添加到右键菜单

```python
def draw_custom_context_menu(self, context):
    """在右键菜单中添加自定义选项"""
    if (context.active_object and 
        context.active_object.type == 'ARMATURE' and 
        context.mode == 'POSE'):
        self.layout.separator()
        self.layout.operator("armature.my_custom_function", 
                           text="我的功能", 
                           icon='PLUGIN')

# 注册到菜单
def register():
    bpy.types.VIEW3D_MT_pose_context_menu.append(draw_custom_context_menu)

def unregister():
    bpy.types.VIEW3D_MT_pose_context_menu.remove(draw_custom_context_menu)
```

## 数据验证和错误处理

### 输入验证

```python
def validate_bone_chain(armature, base_name):
    """验证骨骼链是否存在"""
    required_bones = [f"{base_name}.{i:03d}" for i in range(1, 100)]
    existing_bones = [name for name in required_bones 
                     if name in armature.bones]
    
    if len(existing_bones) < 2:
        return False, "骨骼链太短，至少需要2个骨骼"
    
    return True, f"找到{len(existing_bones)}个骨骼"
```

### 错误处理

```python
def safe_execute_with_error_handling(func, *args, **kwargs):
    """安全执行函数，包含错误处理"""
    try:
        return func(*args, **kwargs)
    except Exception as e:
        print(f"执行出错: {str(e)}")
        bpy.ops.report({'ERROR'}, message=f"操作失败: {str(e)}")
        return None
```

## 性能优化技巧

### 批量操作

```python
def batch_process_bones(armature, bone_names, process_func):
    """批量处理骨骼，提高性能"""
    with bpy.context.temp_override(active_object=armature):
        for bone_name in bone_names:
            bone = armature.pose.bones.get(bone_name)
            if bone:
                process_func(bone)
```

### 缓存机制

```python
# 使用字典缓存复杂计算结果
calculation_cache = {}

def get_cached_calculation(key, calculation_func, *args):
    """获取缓存的计算结果"""
    if key not in calculation_cache:
        calculation_cache[key] = calculation_func(*args)
    return calculation_cache[key]
```

## 测试和调试

### 单元测试结构

```python
import unittest
import bpy

class TestCustomOperators(unittest.TestCase):
    def setUp(self):
        """测试前设置"""
        # 创建测试场景
        bpy.ops.object.select_all(action='SELECT')
        bpy.ops.object.delete(use_global=False)
        
        # 创建测试骨架
        bpy.ops.object.armature_add()
        self.armature_obj = bpy.context.active_object

    def test_custom_subdivide(self):
        """测试自定义细分功能"""
        # 切换到编辑模式
        bpy.ops.object.mode_set(mode='EDIT')
        
        # 执行自定义操作
        result = bpy.ops.armature.my_custom_function()
        
        # 检查结果
        self.assertEqual(result, {'FINISHED'})
        
    def tearDown(self):
        """测试后清理"""
        bpy.ops.object.select_all(action='SELECT')
        bpy.ops.object.delete(use_global=False)
```

## 最佳实践

### 代码组织

- 将相关功能组织到模块中
- 使用有意义的类和函数名称
- 添加适当的文档字符串

### 兼容性考虑

- 考虑不同Blender版本的API差异
- 提供向后兼容的代码路径
- 测试在不同Blender版本中的功能

### 用户体验

- 提供清晰的错误消息
- 实现撤销功能（{'REGISTER', 'UNDO'}）
- 在执行时间较长的操作时提供进度反馈