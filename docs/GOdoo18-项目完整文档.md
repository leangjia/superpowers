# GOdoo18_CE 项目完整文档

> **项目路径**: `D:\GOdoo18_CE_20250305`
> **系统**: Odoo v18 社区版 + 康虎云报表集成
> **生成日期**: 2026-04-04

---

## 目录

1. [项目概述](#一项目概述)
2. [系统架构](#二系统架构)
3. [Odoo 核心架构](#三odoo-核心架构)
4. [自定义模块详解](#四自定义模块详解)
5. [打印系统工作原理](#五打印系统工作原理)
6. [开发指南](#六开发指南)
7. [部署与运维](#七部署与运维)
8. [故障排除](#八故障排除)

---

## 一、项目概述

### 1.1 项目简介

本项目是一个基于 Odoo 18 社区版的 ERP 系统，集成了**康虎云报表**打印解决方案。系统采用绿色版设计，无需安装，解压即用。

### 1.2 核心功能

| 功能 | 说明 |
|------|------|
| **ERP 系统** | 完整的 Odoo 18 社区版功能（CRM、销售、采购、库存、财务等） |
| **云报表打印** | B/S 架构精确打印，支持票据、套打、标签打印 |
| **可视化报表设计** | 在 Odoo 中直接定义报表字段和模板 |
| **模板管理** | 模板存储于数据库，支持版本历史 |

### 1.3 技术栈

| 类别 | 技术 |
|------|------|
| ERP 框架 | Odoo 18.0 (Python) |
| Python 版本 | 3.10.5 |
| 数据库 | PostgreSQL 15 (端口 65437) |
| Web 服务器 | Werkzeug (内置) + Nginx (可选) |
| 报表引擎 | FastReport (.fr3 格式) |
| 打印服务 | cfprint.exe (WebSocket 通信) |
| 加密 | AES-CBC (pycryptodome) |

---

## 二、系统架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                        浏览器 (用户界面)                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Odoo Web UI (Vue.js + OWL)                          │   │
│  └──────────────────────┬───────────────────────────────┘   │
│                         │                                    │
│  ┌──────────────────────▼───────────────────────────────┐   │
│  │  cfprint_ext.js (WebSocket 客户端)                    │   │
│  └──────────────────────┬───────────────────────────────┘   │
└─────────────────────────┼───────────────────────────────────┘
                          │ HTTP / WebSocket
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                     Odoo 服务器 (Python)                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  cfprint     │  │ cf_report_   │  │  cfreport_demo   │  │
│  │  (基础模块)   │  │ designer     │  │  (示例模块)       │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Odoo 18 核心框架                         │  │
│  │  ORM / HTTP / Module Loading / Services               │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                  PostgreSQL 15 数据库                         │
│  端口: 65437 | 用户: odoo | 密码: odoo                       │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│              cfprint.exe 打印服务器 (Windows)                 │
│  端口: 54321 | 协议: WebSocket                               │
│  功能: 接收数据 → 加载模板 → 渲染打印                         │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 目录结构

```
GOdoo18_CE_20250305/
├── bin/                    # 配置与工具脚本
│   └── odoo.conf           # 主配置文件
├── source/                 # Odoo 18 完整源码
│   ├── odoo-bin            # 启动脚本
│   └── odoo/               # 核心代码
├── runtime/                # 运行时环境
│   ├── python-3.10.5/     # Python 运行时
│   ├── pgsql/             # PostgreSQL 数据库
│   └── win32/             # Node.js, Nginx
├── myaddons/               # 自定义模块
│   ├── cfprint/           # 云报表基础模块
│   ├── cf_report_designer/# 报表设计器
│   ├── cf_sale_print_ext/ # 销售打印扩展
│   └── cfreport_demo/     # 报表示例
└── data/                   # 数据存储目录
```

### 2.3 端口配置

| 服务 | 端口 | 说明 |
|------|------|------|
| Odoo HTTP | 8869 | 主服务端口 |
| Odoo Gevent | 8771 | 长轮询/WebSocket |
| PostgreSQL | 65437 | 数据库服务 |
| cfprint.exe | 54321 | 打印服务器（默认） |

---

## 三、Odoo 核心架构

### 3.1 启动链路

```
odoo-bin → odoo.cli.main() → Command.run() → Server.main() → odoo.service.server.start()
```

### 3.2 ORM 系统

Odoo ORM 采用**双层类架构**：

```
Definition Class (模块源码中定义)
    ↓ 构建时
Registry Class (运行时动态生成)
    ↓ 实例化
Recordset (模型实例，即 env['model.name'])
```

**三大模型基类**:

| 类 | 用途 |
|---|------|
| `Model` | 常规持久化模型（数据库表） |
| `TransientModel` | 临时数据，自动清理（向导） |
| `AbstractModel` | 抽象基类（Mixin） |

### 3.3 字段类型

| 类型 | 说明 |
|------|------|
| `Char`, `Text`, `Html` | 文本字段 |
| `Integer`, `Float`, `Boolean` | 基本类型 |
| `Date`, `Datetime` | 日期时间 |
| `Selection` | 下拉选择 |
| `Binary`, `Json` | 二进制/JSON |
| `Many2one` | 外键（多对一） |
| `One2many` | 反向关系（一对多） |
| `Many2many` | 多对多（中间表） |

**特殊字段机制**:
- **计算字段** (`compute`): 通过 `@api.depends` 声明依赖
- **关联字段** (`related`): 自动穿透关系链
- **公司依赖字段** (`company_dependent`): 值按公司隔离

### 3.4 HTTP/路由系统

```python
class MyController(odoo.http.Controller):
    @odoo.http.route('/my/path', type='http', auth='user', methods=['GET'])
    def my_handler(self):
        return request.render('my_template')
```

**路由参数**:
- `type`: `'http'` (HTML) 或 `'json'` (JSON-RPC)
- `auth`: `'user'` / `'public'` / `'none'` / `'bearer'`

### 3.5 模块加载系统

```
load_modules(registry)
    ├── STEP 1: 加载 base 模块
    ├── STEP 3: 按依赖图加载模块
    │       ├── 导入Python模块
    │       ├── 实例化模型类
    │       ├── 初始化数据库表
    │       └── 加载XML/CSV数据文件
    └── STEP 9: 调用 _register_hook
```

### 3.6 扩展点

**模型扩展**:
```python
class ResPartner(models.Model):
    _inherit = 'res.partner'
    custom_field = fields.Char(string='Custom Field')
```

**控制器扩展**:
```python
class ExtendedController(OriginalController):
    @route(auth='user')
    def original_method(self):
        return super().original_method()
```

**模块钩子**:
| 钩子 | 时机 |
|------|------|
| `pre_init_hook` | 模型加载前 |
| `post_init_hook` | 模块安装后 |
| `uninstall_hook` | 模块卸载时 |

---

## 四、自定义模块详解

### 4.1 cfprint 模块（基础模块）

**模块信息**:
- **版本**: 18.0
- **依赖**: base, mail
- **路径**: `myaddons/cfprint/`

**核心功能**:
1. 打印模板管理（存储在数据库）
2. WebSocket 通信（浏览器 ↔ cfprint.exe）
3. QWeb 报表取值增强（HTML 过滤）
4. 金额中文大写转换
5. 打印服务器 IP 映射

**模型清单**:

| 模型 | 说明 |
|------|------|
| `cf.template` | 打印模板（Base64 编码 .fr3 文件） |
| `cf.template.category` | 模板分类 |
| `cf.template.history` | 模板历史版本 |
| `cf.print.server.user.mapping` | 用户→打印服务器映射 |

**控制器**:

| 路由 | 说明 |
|------|------|
| `/cfprint/template` | 模板下载接口 |

**菜单结构**:
```
CFPrint
├── Templates (模板管理)
├── Setting
│   └── Print Server IP (打印服务器配置)
└── Download CFPrint (下载客户端)
```

### 4.2 cf_report_designer 模块（报表设计器）

**模块信息**:
- **版本**: 18.0
- **依赖**: base, cfprint
- **路径**: `myaddons/cf_report_designer/`

**核心功能**:
1. 可视化定义报表字段
2. 自动生成 ir.actions.report
3. 自动生成 QWeb 模板
4. AES 加密报表数据
5. 许可证验证

**模型清单**:

| 模型 | 说明 |
|------|------|
| `cf.report.define` | 报表定义主表 |
| `cf.report.define.field` | 报表字段定义 |
| `cf.report.define.category` | 报表分类 |
| `cf.report.printer` | 打印机定义 |
| `cf.cfprint.license` | 许可证信息 |

**控制器**:

| 路由 | 说明 |
|------|------|
| `/report/<converter>/<reportname>` | 拦截报表请求 |
| `/cfreport/download` | 下载打印数据（JSON） |

**字段类型映射**:

| Odoo 类型 | 康虎云报表类型 |
|-----------|----------------|
| char/selection/reference | str |
| text/html | text |
| date/datetime | datetime |
| float/monetary | double |
| integer/many2one | int |
| binary | blob |

**许可证机制**:
- 机器码：跨平台获取设备 UUID
- 签名算法：SHA1（序列化数据 + 盐值）
- 类型：0=试用版, 1=单机版, 2=专业版, 3=企业版
- 试用版限制：每次 10 次、每天 50 次、总计 5000 次

### 4.3 cf_sale_print_ext 模块（销售打印扩展）

**模块信息**:
- **版本**: 18.0
- **依赖**: base, sale
- **路径**: `myaddons/cf_sale_print_ext/`

**功能**: 演示打印回调机制

**扩展内容**:
```python
class SaleOrder(models.Model):
    _inherit = 'sale.order'
    
    print_count = fields.Integer(string='打印次数', default=0)
    
    def after_print_cf(self, ids=[]):
        """打印后回调方法"""
        for doc in self.search([('id', 'in', ids)]):
            doc.update({'print_count': doc.print_count + 1})
```

### 4.4 cfreport_demo 模块（报表示例）

**模块信息**:
- **版本**: 18.0
- **依赖**: base, sale, product, cfprint
- **路径**: `myaddons/cfreport_demo/`

**示例场景**:
1. 销售订单/询价单打印
2. 产品标签打印

**模板文件**:
- `templates/report_saleorder.fr3` — 销售订单模板（A4）
- `templates/product_label.fr3` — 产品标签模板（3列布局）

---

## 五、打印系统工作原理

### 5.1 整体数据流

```
用户在 Odoo 中点击"打印"
        ↓
前端 JS 拦截报表动作
        ↓
后端生成报表 JSON 数据
        ↓
AES 加密报表数据
        ↓
嵌入 HTML 或通过 JSON RPC 返回
        ↓
前端 JS 读取数据
        ↓
WebSocket 发送到 cfprint.exe
        ↓
cfprint.exe 加载模板 (.fr3)
        ↓
渲染并输出到打印机
```

### 5.2 打印数据 JSON 格式

```json
{
  "template": "base64:...",
  "ver": 4,
  "Copies": 1,
  "Duplex": 0,
  "Tables": [{
    "table": {
      "Name": "Table1",
      "Cols": [
        {"type": "str", "size": 255, "name": "字段名", "required": false}
      ],
      "Data": [
        {"字段名": "值"}
      ]
    }
  }]
}
```

### 5.3 WebSocket 通信

- 默认连接: `ws://127.0.0.1:54321`
- 数据发送前进行 Base64 编码
- 支持 `ws` 和 `wss` 协议
- 自动重连机制（reconnectDecay=1.5）

### 5.4 模板获取方式

**方式 1: 数据库获取（推荐）**
```xml
<t t-esc="get_cf_template(user.env, 'templ_sale_order')" />
```

**方式 2: 直接 ORM 查询**
```xml
<t t-esc="user.env['cf.template'].search([('templ_id', '=', 'templ_sale_order')], limit=1).template" />
```

**方式 3: 本地文件**
模板文件放在客户端本地，通过文件名引用

---

## 六、开发指南

### 6.1 创建新模块

```bash
# 使用内置工具
bin\make_new_module.cmd

# 或使用 Odoo 脚手架
python -m odoo scaffold my_module myaddons/
```

### 6.2 模块结构

```
my_module/
├── __init__.py           # 模块入口
├── __manifest__.py       # 模块清单
├── models/               # 模型定义
│   ├── __init__.py
│   └── my_model.py
├── views/                # 视图定义
│   └── my_view.xml
├── security/             # 安全配置
│   └── ir.model.access.csv
└── data/                 # 数据文件
    └── my_data.xml
```

### 6.3 定义报表

```python
class MyReportDefine(models.Model):
    _name = 'cf.report.define'
    
    # 在 Odoo 中定义报表
    name = fields.Char('报表ID', required=True)
    comment = fields.Char('报表名称', required=True)
    model_id = fields.Many2one('ir.model', '数据源模型')
    template_id = fields.Many2one('cf.template', '打印模板')
```

### 6.4 升级模块

```bash
# 使用内置工具
bin\upgrade_module.bat

# 或手动升级
start-odoo.bat -u my_module -d MyERP16_CE
```

### 6.5 安装 Python 模块

```bash
bin\install_py_module.cmd <module_name>
```

---

## 七、部署与运维

### 7.1 系统要求

| 项目 | 要求 |
|------|------|
| 操作系统 | Windows 10/11 (x64) |
| 内存 | 最低 4GB，推荐 8GB+ |
| 磁盘 | 至少 2GB 可用空间 |
| 路径 | 不能包含中文或空格 |

### 7.2 启动服务

```bash
# 启动所有服务
start.bat

# 访问系统
http://localhost:8869
```

### 7.3 安装为 Windows 服务

```bash
bin\service_install.bat
```

### 7.4 备份数据库

在 Odoo 管理界面：
```
http://localhost:8869/web/database/manager
```

### 7.5 日志位置

- Odoo 日志: `logs/` 目录
- PostgreSQL 日志: `runtime/pgsql/data/log/`

---

## 八、故障排除

### 8.1 常见问题

**问题 1: DLL load failed**
```
ImportError: DLL load failed: 找不到指定的模块。
```
**解决**: 以管理员身份运行 CMD，进入 `runtime\python` 目录执行：
```bash
python.exe Scripts\pywin32_postinstall.py -install
```

**问题 2: 路径包含中文**
```
解压目录中不能包含中文，否则会出现各种异常
```

**问题 3: 端口冲突**
```
修改 bin\odoo.conf 中的 http_port 和 db_port
```

### 8.2 打印问题排查

| 问题 | 检查项 |
|------|--------|
| 打印无响应 | 确认 cfprint.exe 已启动 |
| 连接失败 | 检查 WebSocket 端口 54321 |
| 模板不显示 | 检查 cf.template 记录是否存在 |
| 数据不完整 | 检查字段定义是否正确 |

### 8.3 数据库问题

| 问题 | 解决 |
|------|------|
| 连接失败 | 检查 PostgreSQL 是否启动 |
| 密码错误 | 检查 odoo.conf 中的 db_password |
| 数据库不存在 | 使用数据库管理器创建 |

---

## 附录

### A. 快捷命令

| 命令 | 说明 |
|------|------|
| `start.bat` | 启动所有服务 |
| `stop.bat` | 停止所有服务 |
| `bin\start-cmd.cmd` | 打开命令行环境 |
| `bin\make_new_module.cmd` | 创建新模块 |
| `bin\upgrade_module.bat` | 升级模块 |

### B. 参考链接

- Odoo 官方文档: https://www.odoo.com/documentation/
- Odoo 源码: https://github.com/odoo/odoo
