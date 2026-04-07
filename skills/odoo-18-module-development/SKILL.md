---
name: odoo-18-module-development
description: Use when developing Odoo 18.0 modules, creating ERP extensions, or troubleshooting module loading issues
---

# Odoo 18.0 模块开发

## Overview

基于 Odoo 18.0 的模块开发规范和常见问题解决方案。

## Odoo 18.0 核心规范

### 视图语法变化
```xml
<!-- Odoo 17+ 使用 list 替代 tree -->
<list string="客户列表">
    <field name="name"/>
</list>
```

### 不使用的废弃功能
- chatter（讨论功能）
- `mail.thread` 基类（除非明确需要）
- `<tree>` 标签（使用 `<list>`）

### 外部ID引用规则

| 场景 | 正确格式 | 错误格式 |
|------|----------|----------|
| 同模块 action | `action="action_customer"` | `action="module.action_customer"` |
| 同模块 menu | `parent="menu_parent"` | `parent="module.menu_parent"` |
| 跨模块引用 | `action="other_module.action_id"` | `action="action_id"` |

## 模块结构模板

```
module_name/
├── __manifest__.py          # 模块清单（必需）
├── models/
│   ├── __init__.py          # 模型入口（必需）
│   └── your_model.py        # 模型定义
├── views/
│   ├── your_model_views.xml # 视图定义
│   └── menus.xml            # 菜单（最后加载）
├── data/
│   └── demo.xml             # 演示数据
├── security/
│   └── ir.model.access.csv  # 访问权限
└── static/
    └── description/
        └── icon.png         # 模块图标
```

## `__manifest__.py` 正确结构

```python
{
    'name': "模块中文名",
    'name': "Module English Name",
    'version': '1.0',
    'category': 'Manufacturing',
    'summary': '简短描述',
    'description': """
        详细描述
    """,
    'author': 'Your Name',
    'website': '',
    'license': 'LGPL-3',
    'depends': ['base'],
    'data': [
        # 安全文件优先
        'security/ir.model.access.csv',
        # 数据文件
        'data/demo.xml',
        # 视图文件按依赖顺序
        'views/parent_views.xml',
        'views/child_views.xml',
        # 菜单必须最后
        'views/menus.xml',
    ],
    'demo': [
        'demo/demo_data.xml',
    ],
    'installable': True,
    'application': True,
}
```

## `__init__.py` 模板

```python
# models/__init__.py
from . import your_model

# 如果有子模块
from . import sub_module
```

## 模型定义模板

```python
# models/your_model.py
from odoo import models, fields, api

class ModelName(models.Model):
    _name = 'your.model'
    _description = '模型描述'

    name = fields.Char(string='名称', required=True)
    active = fields.Boolean(string='活跃', default=True)
    
    # 其他字段...
    
    def action_do_something(self):
        for record in self:
            # 业务逻辑
            pass
        return True
```

## 视图定义模板

### 列表视图
```xml
<list string="标题">
    <field name="name"/>
    <field name="code"/>
    <field name="active"/>
</list>
```

### 表单视图
```xml
<form>
    <sheet>
        <group>
            <group string="分组1">
                <field name="name"/>
            </group>
            <group string="分组2">
                <field name="code"/>
            </group>
        </group>
        <notebook>
            <page string="详情" name="details">
                <field name="description"/>
            </page>
        </notebook>
    </sheet>
</form>
```

### 操作（Window Action）
```xml
<record id="action_your_model" model="ir.actions.act_window">
    <field name="name">菜单显示名称</field>
    <field name="res_model">your.model</field>
    <field name="view_mode">list,form</field>
</record>
```

### 菜单
```xml
<menuitem id="menu_parent" name="父菜单"/>
<menuitem id="menu_your_model" 
          name="菜单名称"
          parent="menu_parent"
          action="action_your_model"
          sequence="10"/>
```

## 安全权限 CSV

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_your_model_user,access.your.model.user,model_your_model,,1,1,1,1
```

## 常见问题与解决方案

### 问题1: 模型未注册
**症状**: 模型定义存在但 Odoo 找不到
**原因**: 对应目录缺少 `__init__.py`
**解决**: 创建 `models/subdir/__init__.py`

### 问题2: 菜单点击报错 action 不存在
**症状**: "Record does not exist or has been deleted"
**原因1**: `menus.xml` 加载顺序过早
**解决**: 在 `__manifest__.py` 中将 `menus.xml` 移到视图文件之后
**原因2**: action ID 引用格式错误
**解决**: 同模块引用不加模块前缀

### 问题3: 视图加载失败
**症状**: 视图页面空白或报错
**原因**: XML 语法错误或字段名错误
**解决**: 
1. 检查 XML 语法
2. 确认字段在模型中存在
3. 查看 Odoo 日志

### 问题4: 权限不足
**症状**: 操作被拒绝
**原因**: `ir.model.access.csv` 未正确配置
**解决**: 添加正确的权限记录

## 调试命令

```bash
# 升级模块
./odoo-bin -u your_module -d your_database

# 以开发者模式启动
./odoo-bin -d your_database --dev=all

# 查看已安装模块
./odoo-bin -u base -d your_database --stop-after-init
```

## Firebird 数据库集成

### 连接配置
```python
import fdb

con = fdb.connect(
    host='localhost',
    database='/path/to/database.fdb',
    user='sysdba',
    password='masterkey'
)
```

### 导入数据到 Odoo
```python
def import_from_firebird(self):
    # 连接 Firebird
    fb_con = fdb.connect(...)
    cursor = fb_con.cursor()
    
    # 查询数据
    cursor.execute("SELECT * FROM CUSTOMERS")
    
    # 转换为 Odoo 记录
    for row in cursor:
        self.env['your.model'].create({
            'name': row[0],
            'code': row[1],
        })
    
    fb_con.close()
```

## Odoo 18.0 绿色版路径

```
D:\GOdoo18_CE_20250305
```

## 开发工作流

1. **编辑源文件** → `D:\纸箱ERP功能模块\architecture-a-single-module\`
2. **同步到 Odoo** → 运行 `copy_modules_to_odoo.ps1`
3. **重启 Odoo**
4. **升级模块** → Settings → Apps → 搜索模块名 → Upgrade

## 相关 Skills

- **superpowers:verification-before-completion** - 验证完成前必须执行验证
- **superpowers:systematic-debugging** - 系统性调试问题
