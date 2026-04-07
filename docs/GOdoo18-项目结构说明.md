# GOdoo18_CE 项目结构说明

> **项目路径**: `D:\GOdoo18_CE_20250305`
> **系统**: Odoo v18 社区版（绿色版，Windows）
> **作者**: 康虎软件工作室

---

## 一、目录结构总览

```
GOdoo18_CE_20250305/
├── bin/                          # 配置与工具脚本
│   ├── odoo.conf                 # Odoo 主配置文件
│   ├── start-cmd.cmd             # 打开命令行
│   ├── start-venv.cmd            # 激活虚拟环境
│   ├── make_new_module.cmd       # 创建新模块
│   ├── upgrade_module.bat        # 升级模块
│   ├── service_install.bat       # 安装为 Windows 服务
│   ├── service_remove.bat        # 移除 Windows 服务
│   ├── nginx-start.cmd           # 启动 Nginx
│   ├── nginx-stop.cmd            # 停止 Nginx
│   ├── nginx-reload.cmd          # 重载 Nginx 配置
│   ├── nginx-kill.cmd            # 强制关闭 Nginx
│   ├── install_py_module.cmd     # 安装 Python 模块
│   ├── make_python_venv.cmd      # 创建 Python 虚拟环境
│   ├── recreate-venv.cmd         # 重建虚拟环境
│   ├── logo.bat                  # 显示 Logo
│   └── title.bat                 # 设置窗口标题
│
├── data/                         # Odoo 数据目录（filestore）
├── logs/                         # 日志目录
├── whl/                          # Python 依赖包（.whl 文件）
│
├── runtime/                      # 运行时环境
│   ├── python-3.10.5.amd64/     # Python 3.10.5 运行时
│   ├── pgsql/                   # PostgreSQL 15 数据库
│   └── win32/                   # Windows 工具
│       ├── nodejs/              # Node.js（LESS 编译）
│       └── nginx/               # Nginx 反向代理
│
├── source/                       # Odoo 18 完整源码
│   ├── odoo-bin                  # Odoo 启动脚本
│   ├── odoo/                     # Odoo 核心代码
│   │   ├── __init__.py           # 包初始化
│   │   ├── cli/                  # 命令行接口
│   │   ├── service/              # 服务层
│   │   ├── models.py             # ORM 模型基类
│   │   ├── fields.py             # 字段定义
│   │   ├── http.py               # HTTP 路由
│   │   ├── modules/              # 模块加载系统
│   │   ├── addons/               # 官方插件模块
│   │   └── ...
│   ├── requirements.txt          # Python 依赖列表
│   └── setup.py                  # 安装脚本
│
├── myaddons/                     # 自定义模块（康虎云报表）
│   ├── readme.txt                # 模块说明
│   ├── cfprint/                  # 云报表基础模块
│   ├── cf_report_designer/       # 报表设计器
│   ├── cf_sale_print_ext/        # 销售打印扩展
│   └── cfreport_demo/            # 报表示例
│
├── start.bat                     # 启动数据库 + Odoo
├── start-odoo.bat                # 仅启动 Odoo
├── start-pg.bat                  # 仅启动数据库
├── stop.bat                      # 停止所有
├── stop-odoo.bat                 # 仅停止 Odoo
├── stop-pg.bat                   # 仅停止数据库
├── nginx-start.cmd               # 启动 Nginx
├── nginx-stop.cmd                # 停止 Nginx
├── README.md                     # 项目说明
├── odoo_v18康虎绿色版使用说明.txt # 使用说明
└── 相关问题解决办法.txt           # 故障排除
```

---

## 二、核心配置文件

### odoo.conf

```ini
[options]
admin_passwd = www.khcloud.net        # 数据库管理密码
addons_path = myaddons                # 自定义模块路径
data_dir = data                       # 数据存储目录
db_host = 127.0.0.1                   # 数据库主机
db_port = 65437                       # 数据库端口
db_user = odoo                        # 数据库用户
db_password = odoo                    # 数据库密码
db_maxconn = 64                       # 最大数据库连接数
dbfilter = .*                         # 数据库过滤正则
list_db = True                        # 允许列出数据库

http_enable = True                    # 启用 HTTP
http_port = 8869                      # Odoo 服务端口
gevent-port = 8771                    # 长轮询端口

pg_path = .\runtime\pgsql\bin          # PostgreSQL 路径
max_cron_threads = 2                  # 定时任务线程数
log_level = info                      # 日志级别
log_handler = :INFO                   # 日志处理器
```

---

## 三、启动脚本说明

| 脚本 | 功能 |
|------|------|
| `start.bat` | 同时启动 PostgreSQL 数据库和 Odoo 服务 |
| `start-odoo.bat` | 仅启动 Odoo（需数据库已运行） |
| `start-pg.bat` | 仅启动 PostgreSQL 数据库 |
| `stop.bat` | 同时停止 Odoo 和数据库 |
| `stop-odoo.bat` | 仅停止 Odoo 服务 |
| `stop-pg.bat` | 仅停止 PostgreSQL 数据库 |
| `nginx-start.cmd` | 启动 Nginx 反向代理 |
| `nginx-stop.cmd` | 停止 Nginx |

---

## 四、自定义模块目录 (myaddons)

### 模块依赖关系

```
cfprint (基础模块)
    ↑
    ├── cf_report_designer (报表设计器)
    ├── cfreport_demo (报表示例)
    └── cf_sale_print_ext (销售打印扩展，独立模块)
```

### 模块清单

| 模块 | 版本 | 依赖 | 说明 |
|------|------|------|------|
| **cfprint** | 18.0 | base, mail | 云报表基础模块，模板管理、WebSocket 通信 |
| **cf_report_designer** | 18.0 | base, cfprint | 可视化报表设计器，字段定义、数据生成 |
| **cf_sale_print_ext** | 18.0 | base, sale | 销售单打印回调示例 |
| **cfreport_demo** | 18.0 | base, sale, product, cfprint | 销售订单/产品标签打印示例 |

---

## 五、端口配置

| 服务 | 端口 | 说明 |
|------|------|------|
| Odoo HTTP | 8869 | 主服务端口 |
| Odoo Gevent | 8771 | 长轮询/WebSocket |
| PostgreSQL | 65437 | 数据库服务 |
| cfprint.exe | 54321 | 打印服务器（默认） |

---

## 六、登录信息

| 项目 | 值 |
|------|-----|
| 访问地址 | http://localhost:8869 |
| 数据库管理员密码 | www.khcloud.net |
| 默认数据库 | MyERP16_CE |
| 默认用户 | admin |
| 默认密码 | 123456 |

---

## 七、技术栈

| 类别 | 技术 |
|------|------|
| ERP 框架 | Odoo 18.0 (Python) |
| Python 版本 | 3.10.5 |
| 数据库 | PostgreSQL 15 |
| Web 服务器 | Werkzeug (内置) + Nginx (可选) |
| 前端 | jQuery, QWeb 模板引擎 |
| 报表引擎 | FastReport (.fr3 格式) |
| 打印服务 | cfprint.exe (WebSocket 通信) |
| 加密 | AES-CBC (pycryptodome) |
| 代码保护 | pycfloader (自定义字节码) |
