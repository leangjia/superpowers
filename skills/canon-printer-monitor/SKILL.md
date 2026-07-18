---
name: canon-printer-monitor
description: 用 SNMPv3(AuthPriv) + Canon Remote UI Web 双路径采集 Canon G3090（及类似 Canon 机型）打印机的打印计数、墨量、系统信息、使用记录、SNMPv3 设置。当用户要采集 Canon/打印机数据、遇到 pysnmp "Wrong SNMP PDU digest"/"Unknown USM user"/"notInTimeWindows"、或要用浏览器登录 Web 面板抓取 CGI 数据时调用。也覆盖"管理员密码由 Web 页面填写而非写死代码"的工程模式。
---

# Canon 打印机监控采集技能（SNMPv3 + Web 双路径融合）

本项目（D:\opencodeproject\printer）对 Canon G3090 系列打印机采用 **SNMPv3 + Web 双路径融合采集**：
SNMP 拿计数/部分信息，Web（Canon Remote UI CGI）补墨量/系统信息/使用记录/SNMPv3 用户名。
管理员密码 **由用户在 Web 页面填写**（存 settings 键值），不写死在 config。

本技能融合了两个前身：snmpv3-printer-auth（SNMPv3 认证）、canon-g3090-web-scraping（Web 抓取）。

## 适用场景
- 采集 Canon G3090（或类似 Canon Remote UI 打印机，如设备 A / 设备 B 脱敏示例）
- pysnmp 报 `Wrong SNMP PDU digest` / `Unknown SNMP security name` / `Unknown USM user` / `notInTimeWindows` / `No SNMP response`
- 用浏览器登录 Canon Remote UI，抓 CGI 端点（lan_snmpv3_data.cgi、ut_get_use_record.cgi 等）
- 让用户在前端页面自行填写管理员密码，后端据此启用完整 Web 采集

## 环境约束（本项目实测）
- Python 路径 `C:\Program Files\Python310`；pytest：`& "C:\Program Files\Python310\python.exe" -m pytest -q`
- Web UI 端口 5050
- 依赖锁定：`pysnmp==4.4.12` + `pyasn1<0.5`（**不要升级 pysnmp**，否则 v2c 采集 API 破坏）；scrapling 0.4.11（底层 playwright/patchright，chromium 可用）
- 永远说中文；UI/注释中文；时间格式 `fmt_time`（`2026-07-17 16:13`，空格无 T）
- Windows PowerShell 无 `&&`

## 设备实况（已实测，宝贵经验；以下为脱敏示例）
- **设备 A（Canon G3090）**：SNMPv1 `<community_v1>` 可用；SNMPv3 USM 用户名 `<usm_user>`，密码 `<usm_pass>`，engine ID 由 MAC `<AA:BB:CC:DD:EE:FF>` 派生（例：`80:00:06:42:03:80:03:<MAC 6字节>`）；config password 是 `<config_pw>`（仅 Web 只读登录，非管理员密码）
- **设备 B（Canon G3090）**：SNMPv3 user `<wrong_user>` 报 `Unknown USM user`；**真实 USM 用户名未知**——需用户在 Web 界面确认，或通过本技能的 Web 路径自动枚举
- **设备 C（Epson L15158）**：SNMPv2c community `<community_v2c>` 成功采集墨量+计数（非 Canon，不在此技能范围，仅对照）
- **计数口径**：直接取设备值，无勾稽计算；单面无设备直给字段时存 0
- **管理员密码未知时**：Web 完整采集降级（仍可 SNMP / Web 基础采集）
- 注：MAC 与 engine ID 的真实映射公式见上方"engine ID ↔ MAC 关系"，请勿在技能文件中写入真实设备地址/凭据

---

## 路径一：SNMPv3 认证采集（pysnmp）

### 关键事实（来自 pysnmp 源码与 RFC3414）
1. **engine ID 必须显式提供**。pysnmp 4.4.12 的自动 discovery 对"不回复 discovery 请求 / 不置 reportable 标志"的设备会卡死。用了 localized key（指定 engine ID）时，`securityEngineId` 必填。
2. **`UsmUserData` 必须同时传 `authKey` + `authProtocol` + `privKey` + `privProtocol`**。只 `addV3User` 不传 `UsmUserData` 会让 securityLevel 默认 `noAuthNoPriv` → 设备返回 `Wrong SNMP PDU digest`。**最常见坑。**
3. **engine ID ↔ MAC 关系**：`80`(fmt) + 厂商前缀 + MAC。Canon 实测前缀 `80000642038003` + 去冒号 MAC，即
   `80:00:06:42:03:80:03:<MAC 6字节>`。例：MAC `AA:BB:CC:DD:EE:FF` → engine ID `80000642038003aabbccddeeff`。
   也可从 report PDU 的 `msgAuthoritativeEngineID` 直接抓。

### 错误分级诊断
| 报错 | 含义 | 处理 |
|------|------|------|
| `Unknown SNMP security name` / `Unknown USM user` | USM 用户名错 | 确认设备 **SNMPv3 USM 用户名**（Web 界面"MIB 访问权限名/只读名"≠ USM 用户名，两套）；可用 Web 路径枚举 |
| `Wrong SNMP PDU digest` | 认证哈希不对 | ① `UsmUserData` 是否带 auth/priv；② 密码/协议(SHA1/AES)错；③ engine ID 未设为 bytes |
| `notInTimeWindows` | 时间窗口 | pysnmp 自动重发修复；仍卡则确认未写死 engineTime |
| `No SNMP response` | 完全无响应 | 确认可达(ping)、161 开放、v2c 是否真没开 |

### 可复用代码骨架
```python
import warnings
warnings.filterwarnings('ignore')
from pysnmp.hlapi import (SnmpEngine, UdpTransportTarget, ContextData,
                          ObjectType, ObjectIdentity, getCmd, UsmUserData)
from pysnmp.entity import config as snmp_config
from pysnmp.entity.config import usmHMACSHAAuthProtocol, usmAesCfb128Protocol

def snmp_v3_get(ip, engine_id_hex, user, auth_pass, priv_pass, oid,
                timeout=8, retries=3):
    engine_id = bytes.fromhex(engine_id_hex.replace(':', ''))
    eng = SnmpEngine()
    snmp_config.addV3User(eng, user, securityEngineId=engine_id,
        authKey=auth_pass, authProtocol=usmHMACSHAAuthProtocol,
        privKey=priv_pass,  privProtocol=usmAesCfb128Protocol)
    transport = UdpTransportTarget((ip, 161), timeout=timeout, retries=retries)
    ud = UsmUserData(user, authKey=auth_pass, authProtocol=usmHMACSHAAuthProtocol,
        privKey=priv_pass, privProtocol=usmAesCfb128Protocol,
        securityEngineId=engine_id)
    error_ind, error_st, _, var_binds = next(
        getCmd(eng, ud, transport, ContextData(), ObjectType(ObjectIdentity(oid))))
    if error_ind: raise RuntimeError(f'SNMPv3 读取失败: {error_ind}')
    if error_st:  raise RuntimeError(f'SNMPv3 错误状态: {error_st}')
    return var_binds[0][1].prettyPrint()
```

### 抓取未知 engine ID
用上函数但 engine_id 随便填，捕获异常后开启 debug：
```python
import pysnmp.debug
pysnmp.debug.setLogger(pysnmp.debug.Debug('msgproc'))  # 搜 msgAuthoritativeEngineID
```
或本项目已实现 `_snmp_v3_discover_engine_id`（按 Canon 公式 MAC 推导）——实用于已知 MAC 的 Canon 设备。

### 标准 OID（Printer-MIB RFC 3805）
- 总页数：`1.3.6.1.2.1.43.10.2.1.4.1.1`
- 墨量名/值/最大：`1.3.6.1.2.1.43.11.1.1.6 / .9 / .8`
- 序列号：`1.3.6.1.2.1.43.5.1.1.17.1`
- 墨量值 `-2`/`-3` = 未知/不支持（标准哨兵，当 unknown 处理）

### Canon 私有 MIB 计数器（G3090 实测）
- `sysObjectID = enterprises.1602.4.7`，**实现 Canon 私有 MIB `1.3.6.1.4.1.1602`**（G3090 有，与部分资料相反）
- 名称表 `1.3.6.1.4.1.1602.1.11.2.1.1.2.x` / 值表 `1.3.6.1.4.1.1602.1.11.2.1.1.3.x`（x 对应：1=Total1, 4=Total(Small), 9=Black/Small, 13=FullColor+SingleColor/Small, 10=2-Sided, 25=Print Total1）
- 标准 MIB `hrPrinterTotalPages` 只给总数；分项（黑白/彩色/单面/双面）仅在私有 MIB。私有 MIB 无独立"单面"字段 → 单面 = 总 - 双面
- **墨量**：G3090 喷墨 `prtMarkerSuppliesLevel` 普遍返回 `-2`（unknown）——Canon 墨量根本不通过 SNMP 暴露，只走 Web 或标记未知

### 各品牌 SNMP 版本
- Canon MF/LBP/iR/iR-ADV：仅 **v1**（不是 v2c！）
- HP/Xerox/Brother/Kyocera/Ricoh/Konica/Epson：v2c
- G3090 消费/SOHO：**v3 AuthPriv**
- 某 Canon 超时先确认版本：v1 用 `CommunityData('public', mpModel=0)`

---

## 路径二：Canon Remote UI Web 抓取（scrapling 真实浏览器）

### 登录流程（实测验证完整点击序列）
```
index.html → #logonBtnScreen_1020 (Log in) → #YNBtn00Screen_1050 (OK 确认重定向)
  → 在 login_manager.html 用 page.type('#passdata', pw) 输入
  → input[type=submit] 提交
  → 取 #SBID.value 作为会话 token
密码错时设备返回 "The entered values are invalid"
```
**关键洞察**：JS 框架（URI_TrFW/send_xmlhttp）可能加载失败，但 `#SBID` 会话 token 可用 → 用原始 XHR + SBID 调 CGI 即可。

### CGI 端点表（相对 `/rui/`，POST + SBID）
| 类别 | 端点 | POST |
|------|------|------|
| 状态 | `prninfo_data.cgi` | GETINFO=0 |
| 系统 | `sysinfo_data.cgi` | GETINFO=0 |
| 使用记录 | `ut_get_use_record.cgi` | GETINFO=0-5 |
| SNMPv3 设置 | `lan_snmpv3_data.cgi` | GETINFO=0 |
| SNMPv3 用户枚举 | `lan_snmpv3_get_user.cgi` | GETINFO=0-N（0=用户总数，再逐个 i 取 `SNMPV3_USER.USER_NAME`） |
| 墨量 | `ink_level.cgi` 等 | GETINFO=0 |

### XML 响应健壮性（融合实战）
- 顶层可能返回 `RW_OK` 带 `RESULT` 属性，或 `SES_ERR_URL`（会话失效需重登）
- 内层 `RW_OK=NG` 表示操作失败
- 解析需容错 `parsererror`、空字段、`HEADER_2PAIN.*`/`MENU_2PAIN.*` boilerplate 过滤
- 本项目 `_parse_cgi_xml` 将 XML 拍平成 `KEY.SUBKEY=val` 字典，便于按 `SNMPV3_USER.USER_NAME` 取用户名

### scrapling 实现骨架
```python
from scrapling.fetchers.stealth_chrome import StealthySession

def web_login_with_sbid(ip, password):
    """登录并返回 SBID；无 password 时干净跳过返回 None"""
    session = StealthySession()
    # page_action 内执行上面登录点击序列，最后 evaluate 取 #SBID.value
    # 识别 "invalid" 判断密码错误
    ...

def call_cgi(page, endpoint, post='GETINFO=0', sbid=None):
    return page.evaluate("""([url, post, sbid]) => new Promise(res => {
        const xhr = new XMLHttpRequest();
        xhr.open('POST', url, true);
        xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
        xhr.onload = () => res(xhr.responseText);
        xhr.send(post + (sbid ? '&SBID=' + sbid : ''));
    })""", [endpoint, post, sbid])
```

### 关键数据标签
- 系统：`PAGE_SYSTEMINFO.IPV4_ADD` / `.ROM_VER` / `.SERIAL_NUM` / `.PRINTER_NAME`
- 使用：`PRODUCT_PRINT_COUNT.PRODUCT_TOTAL` / `COPY_PRINT_COUNT.COPY_TOTAL` / `SCAN_COUNT.SCAN_TOTAL`
- 墨量：`PAGE_PRNINFO.CARTRIDGE_MODEL`（如 `<CARTRIDGE_MODEL>`）
- 网络：`CONFIRM_LAN_SETTINGS.WIRELESS_SSID` / `.WIRELESS_LAN_SECURITY`

---

## 路径三：管理员密码 Web 页面填写（工程模式）

**核心原则**：管理员密码（Remote UI 登录用）不写死在 config，由用户在 Web 页面填写，存于 `settings` 键值 `admin_password_<printer_id>`。

### 后端
- `web_ui.py`：`GET/POST /api/printer/<id>/admin_password`
  - POST：存 settings `admin_password_<id>`（空串 = 清除）；返回 `{success, configured}`
  - GET：返回 `{configured}`（**不返回明文**）
- `printer_scraper.py scrape_printer`：开头按 IP 查 `printer_id`，从 `database.get_all_settings()` 读 `admin_password_<id>`，注入 effective_config 传给 `get_scraper`（**Web 填写优先于 config 占位**）
- `config.py` 中 166/217 保留 `"admin_password": ""` 占位（注释说明可选）
- 填入真实密码后，登录成功 → 调 `lan_snmpv3_get_user.cgi` 枚举所有 SNMPv3 用户名 → 自动补全 `self.snmp['user']`（解决 217 用户名未知问题）

### 前端（templates/printer_detail.html）
- 仅 Canon 显示"Canon 管理员密码（可选）"卡片：说明 + password 输入框 + 保存按钮 + 状态提示
- 加载时 GET 显示是否已配置；保存调 POST；空串清除；不回显明文
- 未配置时自动降级为 SNMP / Web 基础采集

### 实测验证（本次会话）
- Web 填密码 → settings 存储 → `scraper.admin_password` 正确注入（实测值为示例密码，已脱敏）
- 新增测试 `TestAdminPasswordWebForm`：API 存取 + 注入验证，全量 92 测试通过
- 注意：测试间 `database.DATABASE_PATH` 被 test_charts.py 改指向临时库，需 `setUp` 时 `database.DATABASE_PATH = config.DATABASE_PATH` 恢复

---

## 融合串联逻辑（scrape_printer）
1. 按 IP 查 printer_id，从 settings 读 `admin_password_<id>` 注入 effective_config
2. 直连 SNMP：v1/v2c 优先；v3 用 engine ID（166 已知 / 217 由 MAC 推导）+ 用户名（config 或 Web 枚举补全）
3. Web 完整采集（需 admin_password）：登录 → `lan_snmpv3_get_user.cgi` 枚举用户名补全 → `sysinfo_data.cgi` 系统信息 → `ut_get_use_record.cgi` 使用记录 → 墨量
4. 墨量 SNMP 全 `-2` 时回退 Web；耗材全量替换（`replace_consumables`）
5. 计数直接取设备值，无勾稽

## 依赖与版本陷阱
- 锁定 `pysnmp==4.4.12` + `pyasn1<0.5`；升级 pysnmp 报 `No module named 'pyasn1.compat.octets'` 时还原 `pyasn1==0.4.8 pyasn1-modules==0.2.8`
- scrapling 0.4.11 底层 playwright/patchright，chromium 已可用

## 参考资源
- 前身技能：snmpv3-printer-auth、canon-g3090-web-scraping（已合并入本技能）
- GitHub：dsorlov/snmp_printer、jsammarco/printer_levels、pemitic/e765c540、Rustem2003/ws-printer-monitoring、Char0n-1/snmp_printer_tool
- 本项目参考脚本：`D:\opencodeproject\Scrapling\scrape_printer.py` / `scrape_snmp.py` / `scrape_lan.py`
