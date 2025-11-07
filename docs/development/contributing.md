---
title: 贡献指南
---

# 贡献指南

感谢您有兴趣为"快速软骨绑定"插件做出贡献！本文档将指导您如何参与项目的开发、修复问题或添加新功能。

## 行为准则

请在参与项目时遵循我们的行为准则：

- 尊重其他贡献者和用户
- 提供建设性的反馈
- 保持专业和友好的交流

## 贡献方式

### 报告问题

如果您发现了bug或有功能建议，请：

1. **搜索现有问题**: 检查是否已有相同或类似的问题报告
2. **创建新问题**: 包含以下信息：
   - Blender版本和插件版本
   - 重现步骤
   - 预期行为 vs 实际行为
   - 相关的错误信息或日志

### 提交代码

1. **Fork项目**: 在GitHub上Fork项目仓库
2. **创建分支**: 为您的功能或修复创建专用分支
3. **提交代码**: 遵循编码规范和提交信息格式
4. **创建PR**: 提交Pull Request到主仓库

### 改进文档

文档的改进同样重要，包括：

- 修复错别字或语法错误
- 完善功能说明
- 添加使用示例
- 优化组织结构

## 开发环境设置

请参考[开发者入门](./getting-started)文档设置开发环境。

## 代码规范

### Python编码规范

- 遵循PEP 8 Python编码规范
- 使用有意义的变量和函数命名
- 每行代码长度不超过80字符
- 使用适当的注释和文档字符串

### 命名约定

- **类名**: 使用PascalCase（如`SetupControlRigOperator`）
- **函数名**: 使用snake_case（如`update_ctrl_bone_visibility`）
- **变量名**: 使用snake_case（如`damped_track_influence`）
- **常量名**: 使用UPPER_CASE（如`DEFAULT_INFLUENCE`）

### 代码结构

```python
# 示例：符合规范的操作符
class CustomOperator(bpy.types.Operator):
    """操作符的简短描述"""
    bl_idname = "armature.custom_operator"
    bl_label = "自定义操作符"
    bl_description = "自定义操作的描述"
    bl_options = {'REGISTER', 'UNDO'}

    # 属性定义
    my_property: bpy.props.FloatProperty(
        name="属性名称",
        description="属性描述",
        default=1.0,
        min=0.0,
        max=10.0
    )

    @classmethod
    def poll(cls, context):
        """定义操作符是否可用的条件"""
        return (context.mode == 'EDIT_ARMATURE' and 
                context.object and 
                context.object.type == 'ARMATURE')

    def execute(self, context):
        """主要执行逻辑"""
        obj = context.object
        # 功能实现...
        self.report({'INFO'}, "操作完成")
        return {'FINISHED'}
```

### 注释规范

- 添加函数和类的文档字符串
- 用中文编写注释
- 在复杂的逻辑旁边添加解释性注释
- 避免冗余注释

## Git工作流

### 分支命名

- `feature/`: 新功能开发 (如 `feature/custom-shape`)
- `bugfix/`: 问题修复 (如 `bugfix/subdivision-error`)
- `docs/`: 文档改进 (如 `docs/api-reference`)
- `refactor/`: 重构 (如 `refactor/ui-layout`)

### 提交信息格式

使用以下格式编写提交信息：

```
type(scope): description

body

footer
```

其中type可以是：
- `feat`: 新功能
- `fix`: 问题修复
- `docs`: 文档更新
- `style`: 代码格式调整
- `refactor`: 代码重构
- `test`: 测试相关
- `chore`: 构建过程或辅助工具变动

示例：
```
feat(operator): 添加自定义骨骼形状选项

- 添加圆形、方形、十字形等形状选项
- 实现形状切换功能
- 添加相应的UI控件
```

## 代码审查流程

### PR要求

- 代码遵循项目规范
- 包含适当的测试（如果适用）
- 更新相关文档
- 描述清楚变更内容和原因

### 审查标准

- 代码功能正确性
- 代码质量和可维护性
- 性能和内存使用
- 用户体验
- 文档完整性

## 测试要求

### 单元测试

为新功能添加适当的测试：

```python
import unittest
import bpy

class TestFeature(unittest.TestCase):
    def setUp(self):
        # 设置测试环境
        pass

    def test_functionality(self):
        # 测试功能
        pass

    def tearDown(self):
        # 清理测试环境
        pass
```

### 手动测试

- 在不同Blender版本上测试
- 测试不同使用场景
- 验证撤销/重做功能

## 文档贡献

### 文档规范

- 使用Markdown格式
- 保持一致的标题层级
- 使用适当的代码块和语法高亮
- 提供清晰的示例

### 示例格式

```markdown
## 功能名称

简短描述功能。

### 使用方法

1.  打开Blender
2.  选择插件面板
3.  点击功能按钮

### 参数说明

- **参数1**: 参数描述
- **参数2**: 参数描述
```

## 发布流程

### 版本号规范

遵循语义化版本控制 (SemVer):

- `MAJOR.MINOR.PATCH`
- `MAJOR`: 不兼容的API更改
- `MINOR`: 向后兼容的功能添加
- `PATCH`: 向后兼容的问题修复

### 发布前检查

- 所有测试通过
- 文档已更新
- 代码已审查
- 版本号已更新

## 社区参与

### 讨论方式

- GitHub Issues: 问题报告和功能请求
- GitHub Discussions: 一般讨论和想法分享
- PRs: 代码贡献

### 反馈处理

- 保持开放的心态
- 提供清晰的解释
- 尊重不同意见

## 常见问题

### 代码风格问题

如果您的代码不符合风格要求，CI/CD系统会报告问题。请根据反馈进行调整。

### 测试失败

确保所有测试在提交前通过。如果有问题，请分析测试输出并修复。

### 合并冲突

如果出现合并冲突，请：

1. 更新本地分支
2. 解决冲突
3. 测试功能
4. 重新提交

## 联系方式

如果需要帮助或有疑问：

- 通过GitHub Issues提问
- 在Pull Request中留言
- 查看相关文档

感谢您的贡献！您的努力让这个插件变得更好。