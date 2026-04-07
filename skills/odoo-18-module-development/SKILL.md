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

### 模块加载顺序规则（重要）

**核心原则**: `ir.model.access.csv` 在视图加载模型之前就被处理，导致新模型权限找不到。

**解决方案**: 将新模型的权限记录放到单独的 CSV 文件中，在 menus.xml 之后加载。

## 模块结构模板

```
module_name/
├── __manifest__.py              # 模块清单（必需）
├── models/
│   ├── __init__.py              # 模型入口（必需）
│   └── your_model.py            # 模型定义
├── views/
│   ├── your_model_views.xml     # 视图定义
│   └── menus.xml                # 菜单（最后加载）
├── data/
│   └── demo.xml                 # 演示数据
├── security/
│   ├── ir.model.access.csv      # 已有模型权限
│   └── ir.model.access.new.csv  # 新模型权限（后加载）
└── static/
    └── description/
        └── icon.png             # 模块图标
```

## `__manifest__.py` 正确结构

```python
{
    'name': "Module Name",
    'version': '1.0',
    'category': 'Industry',
    'author': 'Your Name',
    'license': 'LGPL-3',
    'depends': ['base'],
    'data': [
        # 已有模型的权限
        'security/ir.model.access.csv',
        'security/security.xml',
        # 数据文件
        'data/demo.xml',
        # 视图文件按依赖顺序
        'views/ps_basic_views.xml',
        'views/ps_sale_views.xml',
        # ... 其他视图
        # 菜单必须最后加载
        'views/menus.xml',
        # 新模型权限必须放在 menus.xml 之后
        'security/ir.model.access.new.csv',
    ],
    'demo': [
        'data/demo.xml',
    ],
    'installable': True,
    'application': True,
}
```

## 新增模型权限文件

如果模块已有权限 CSV，新增模型时创建新文件：

**security/ir.model.access.new.csv**:
```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_your_new_model_user,your.new.model.user,model_your_new_model,base.group_user,1,1,1,1
```

**错误做法**: 直接在原 ir.model.access.csv 中添加，会导致 "Missing required value for the field 'Model'" 错误。

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
    _order = 'name'

    name = fields.Char(string='名称', required=True)
    code = fields.Char(string='编码', index=True)
    active = fields.Boolean(string='启用', default=True)
    
    # Many2one 示例
    partner_id = fields.Many2one('res.partner', string='合作伙伴')
    
    # 计算字段
    total_amount = fields.Float(string='总金额', compute='_compute_total')
    
    # SQL 约束
    _sql_constraints = [
        ('code_unique', 'unique(code)', '编码不能重复!'),
    ]

    @api.depends('price', 'quantity')
    def _compute_total(self):
        for rec in self:
            rec.total_amount = (rec.price or 0) * (rec.quantity or 0)
```

## 视图定义模板

### 列表视图
```xml
<record id="view_your_model_tree" model="ir.ui.view">
    <field name="name">your.model.list</field>
    <field name="model">your.model</field>
    <field name="arch" type="xml">
        <list>
            <field name="name"/>
            <field name="code"/>
            <field name="active"/>
        </list>
    </field>
</record>
```

### 表单视图
```xml
<record id="view_your_model_form" model="ir.ui.view">
    <field name="name">your.model.form</field>
    <field name="model">your.model</field>
    <field name="arch" type="xml">
        <form>
            <sheet>
                <group>
                    <group string="基本信息">
                        <field name="name"/>
                        <field name="code"/>
                    </group>
                    <group string="设置">
                        <field name="active"/>
                    </group>
                </group>
                <notebook>
                    <page string="备注" name="remark">
                        <field name="remark"/>
                    </page>
                </notebook>
            </sheet>
        </form>
    </field>
</record>
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

### 已有模型（ir.model.access.csv）
```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_your_model_user,your.model.user,model_your_model,base.group_user,1,1,1,1
```

### 新模型（ir.model.access.new.csv）
```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_your_new_model_user,your.new.model.user,model_your_new_model,base.group_user,1,1,1,1
```

## 常见问题与解决方案

### 问题1: 模型未注册
**症状**: 模型定义存在但 Odoo 找不到
**原因**: 对应目录缺少 `__init__.py`
**解决**: 创建 `models/subdir/__init__.py`

### 问题2: ir.model.access.csv 模型找不到 ⚠️ 核心问题
**症状**: 
```
Missing required value for the field 'Model' (model_id)
在字段 'Model' 中没找到匹配的记录 外部ID 'model_your_new_model'
```
**原因**: `ir.model.access.csv` 在模型注册之前就被加载
**解决**: 
1. 创建单独的 CSV 文件 `security/ir.model.access.new.csv`
2. 在 `__manifest__.py` 中将此文件放在 `menus.xml` 之后

### 问题3: 菜单点击报错 action 不存在
**症状**: "Record does not exist or has been deleted"
**原因1**: `menus.xml` 加载顺序过早
**解决**: 在 `__manifest__.py` 中将 `menus.xml` 移到视图文件之后
**原因2**: action ID 引用格式错误
**解决**: 同模块引用不加模块前缀

### 问题4: 视图加载失败
**症状**: 视图页面空白或报错
**原因**: XML 语法错误或字段名错误
**解决**: 
1. 检查 XML 语法
2. 确认字段在模型中存在
3. 查看 Odoo 日志

### 问题5: 权限不足
**症状**: 操作被拒绝
**原因**: `ir.model.access.csv` 未正确配置
**解决**: 添加正确的权限记录（注意加载顺序）

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
    fb_con = fdb.connect(...)
    cursor = fb_con.cursor()
    
    cursor.execute("SELECT * FROM CUSTOMERS")
    
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

1. **创建模型** → `models/ps_basic/*.py`
2. **创建视图** → `views/ps_basic_*_views.xml`
3. **更新 __init__.py** → 添加新模型导入
4. **添加权限** → 创建 `security/ir.model.access.new.csv`
5. **更新 manifest** → 添加视图文件和新权限文件
6. **更新 menus.xml** → 添加新菜单项
7. **添加演示数据** → `data/demo.xml`
8. **同步到 Odoo** → `copy_modules_to_odoo.ps1`
9. **升级模块** → Settings → Apps → Upgrade

## 相关 Skills

- **superpowers:verification-before-completion** - 验证完成前必须执行验证
- **superpowers:systematic-debugging** - 系统性调试问题
