---
name: canon-g3090-web-scraping
description: Scraping Canon G3090 Remote UI web interface (HTTPS, CGI endpoints) for printer status, usage records, system info, LAN settings. 已合并入 canon-printer-monitor 技能。
---

# Canon G3090 Web Scraping（已合并）

本技能的全部内容已整合进统一技能 **`canon-printer-monitor`**（SNMPv3 + Web 双路径融合 + Web 密码页面填写）。

请改用 skill：加载 `canon-printer-monitor`，其中"路径二：Canon Remote UI Web 抓取"即本技能内容（登录点击序列、CGI 端点表、XML 健壮性、`_parse_cgi_xml` 拍平、关键数据标签）。

保留本文件仅为兼容历史引用；新工作一律使用 `canon-printer-monitor`。
