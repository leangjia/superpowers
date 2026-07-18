---
name: snmpv3-printer-auth
description: 用 pysnmp 对接只支持 SNMPv3(AuthPriv) 的打印机/网络设备时，解决 engine ID 发现失败、WrongDigest、notInTimeWindows 等问题。已合并入 canon-printer-monitor 技能。
---

# SNMPv3 打印机认证采集（已合并）

本技能的全部内容已整合进统一技能 **`canon-printer-monitor`**（SNMPv3 + Web 双路径融合 + Web 密码页面填写）。

请改用 skill：加载 `canon-printer-monitor`，其中"路径一：SNMPv3 认证采集"即本技能内容（engine ID 推导、UsmUserData 完整参数坑、错误分级诊断、`-2` 哨兵、Canon 私有 MIB 计数器）。

保留本文件仅为兼容历史引用；新工作一律使用 `canon-printer-monitor`。
