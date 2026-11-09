# 闲鱼 7.28.90 去广告调研与抓包验证清单

调研日期：2026-11-09（Asia/Shanghai）  
目标 App：闲鱼 iPhone 7.28.90  
模块草稿：`/workspace/xianyu/闲鱼去广告.sgmodule`

---

## 1. 源文件一览（仅收录亲眼看到的规则）

| 来源 | 原始 URL | 文件内/Git 日期 | 7.28.90 状态 |
|------|----------|-----------------|--------------|
| ddgksf2013 GoofishAds | https://raw.githubusercontent.com/ddgksf2013/Rewrite/refs/heads/master/AdBlock/GoofishAds.conf | 头 `@UpdateTime 2025-09-06`；Git 最近提交 2025-12-12 | **用户实测失效** |
| Romeo 转 Surge | https://raw.githubusercontent.com/ifflagged/Romeo/main/Modules/Surge/ddgksf2013/GoofishAds.sgmodule | Git 2026-10-25 同步 | 同上墨鱼内容转换 |
| zirawell R-Store Surge | https://raw.githubusercontent.com/zirawell/R-Store/main/Rule/Surge/Adblock/App/X/闲鱼/goofish.sgmodule | 头 `#!date=2026-05-09`；Git 2026-09-14 | **用户实测失效** |
| zirawell 脚本 | https://raw.githubusercontent.com/zirawell/R-Store/main/Res/Scripts/AntiAd/goofish.js | 注明 checkout 自 fmz200 | 配合 R-Store |
| fmz200 XianYu.sgmodule | https://raw.githubusercontent.com/fmz200/wool_scripts/main/Surge/module/split/partX/XianYu.sgmodule | Git 2026-06-25 | 待验证（开屏手段类似 Map Local） |
| ishowshu goofish.js | https://raw.githubusercontent.com/ishowshu/qx/refs/heads/main/script/goofish.js | 文件头 `2026-10-24` | 待验证（信息流） |
| atopsecret 模块 | https://raw.githubusercontent.com/atopsecret/SurgeScripts/main/xianyu/xianyu-ad-block.sgmodule | Git 2026-11-05 | **重点候选**（cacheAd） |
| atopsecret 脚本 | https://raw.githubusercontent.com/atopsecret/SurgeScripts/main/xianyu/xianyu-ad-block.js | 同仓库 | 同上 |
| 可莉 / QingRex | https://raw.githubusercontent.com/QingRex/LoonKissSurge/refs/heads/main/Surge/闲鱼去广告.sgmodule | 头 `#!date=2025-12-26`；Git 2025-12-28 | 待验证；首页 jq 与墨鱼不同 |
| androidcn | https://raw.githubusercontent.com/androidcn/userscripts/main/goofish.sgmodule | Git 2024-01-24 | 仅首页/搜索脚本，无开屏 |
| blackmatrix7 XianYu.list | https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Surge/XianYu/XianYu.list | UPDATED 2025-06-06 | **分流域名**，非去广告 |
| Walvez StartUpAds | dist/StartUpAds_Selected.sgmodule | 含 `# ===== xianyu =====` 一行 splash reject | 等同墨鱼开屏 reject |
| kokoryh/Sparkle | — | 树内无 goofish/xianyu 路径 | **未收录** |
| RuCu6/QuanX | — | 树内无匹配路径 | **未收录** |
| app2smile/rules | — | README 无闲鱼 | **未收录** |

旧规则对照（历史，勿用）：`mtop.taobao.idle.home.welcome` reject（多份老 Advertising；blackmatrix7 Issue #391 报告会导致闲鱼搜索/页面失败）。

---

## 2. 接口对比（按用途）

### 2.1 开屏相关

| 接口（路径中的 mtop 名） | 手段（源文件所见） | 来源 | 可信度 |
|--------------------------|--------------------|------|--------|
| `mtop.*splash.ads` / `mtop.taobao.idlecommerce.splash.ads` | QX reject-200；Surge Map Local `{}`；或脚本清字段 | 墨鱼、可莉、fmz200、Walvez、atopsecret | 多源一致**接口名**；手段上 reject/空 JSON **用户侧已失效** |
| `mtop.taobao.idlecommerce.splash`（无强制 .ads） | Map Local `{}` | zirawell | 用户实测失效 |
| `mtop.taobao.idlecommerce.splash.async.ads` | 脚本清 adMap/cacheAd | atopsecret 注释/脚本 | 旧版接口；脚本正则仍兼容 |
| `mtop.idle.ad.expose` | reject-200 或脚本回空成功 | 墨鱼、atopsecret | 两源；偏曝光上报 |
| `mtop.idle.idleadv.app.launch.report` | 脚本回无广告成功体 | **仅 atopsecret** | 单源，待抓包 |
| `mtop.idle.idleadv.scene.restore` | 同上 | **仅 atopsecret** | 单源，待抓包 |
| 响应字段 `cacheAd` / `cacheAdInfo` / `dspList` / `adMap` | 脚本清空 | atopsecret 脚本原文 | 解释「为何 reject 无效」的关键线索 |
| `ossgw.alicdn.com/creatives-assets/` | URL Rewrite reject | atopsecret | 单源素材路径 |
| `(bdlog\|df).tanx.com` | reject | atopsecret | 单源打点域 |

### 2.2 首页 / 信息流 / 搜索（节选）

| 接口 | 手段 | 多源一致？ |
|------|------|------------|
| `mtop.taobao.idlehome.home.nextfresh` | 脚本/jq 滤 sections（bizType≠item / 去 mamaAD） | ✅ 墨鱼、zirawell、fmz200、可莉、androidcn |
| `mtop.taobao.idle.local.home` | 脚本滤 AD | ✅ |
| `mtop.taobao.idle.user.strategy.list` | jq / Map Local / 脚本 | ✅（悬浮球） |
| `mtop.taobao.idlehome.home.circle.list` | jq/脚本 | ✅ |
| `mtop.taobao.idlemtopsearch.search` | 脚本/jq 滤广告结果 | ✅ |
| `mtop.idle.user.page.my.adapter` | jq/脚本精简「我的」 | ✅ |
| `mtop.taobao.idlehome.widget.refresh.get` | jq | 可莉独有（相对墨鱼/zirawell） |
| `mtop.taobao.idlehome.home.community` | jq | 可莉 |
| `mtop.taobao.idlehome.home.newitem.page` | jq | 可莉 |
| `mtop.taobao.idle.local.home.top` | jq | 可莉 |
| `mtop.taobao.idle.local.near.by.corner.info` | Map Local | 可莉 |
| `mtop.taobao.idle.playboy.recommend` | reject / Map Local | fmz200、可莉 |

### 2.3 MitM 主机名（源文件所见）

- 墨鱼 / Romeo / fmz200 / 可莉 / androidcn：`acs.m.goofish.com`, `g-acs.m.goofish.com`
- zirawell：另加 `h5.m.goofish.com`, `dinamicx.alibabausercontent.com`, `g.alicdn.com`
- atopsecret：`acs|g-acs|gw` × `goofish|taobao`（含 `acs.m.taobao.com` 等）

本草稿 MitM：**不加支付宝**；goofish 三主机 + 可选 taobao ACS + `ossgw.alicdn.com`。

---

## 3. 墨鱼 / zirawell 可能失效原因（区分「源文件证据」与「推测」）

**源文件证据：**

1. atopsecret 模块注释原文：开屏已从 `splash.async.ads/1.0` 变为 `splash.ads/2.0`，广告走 **`cacheAd` 本地缓存引用**；其脚本专门 `d.cacheAd=""` 并删 `cacheAdInfo`。
2. 墨鱼 GoofishAds 头注释：`使用前请卸载咸鱼，挂载脚本然后使用` → 作者也承认缓存/预装素材问题。
3. 墨鱼对开屏用 **reject-200**；业界常见结果是客户端回退本地已缓存开屏图（V2EX 等讨论同类行为，作背景参考）。
4. zirawell 对 `idlecommerce.splash` 回 **`{}`**，不处理 `cacheAd` 字段结构 → 与「缓存引用下发」模型不匹配。

**推测（需抓包确认，勿当事实）：**

- 7.28.90 开屏配置改走未写入规则的新 mtop 名，或只走 `g-acs` / `*.m.taobao.com` / `gw.m.*`。
- 开屏素材经 HTTP/2、ACCS 长连接或非 MitM 域名下发，Surge 最近请求里看不到对应 HTTPS。
- 仅拦配置、不拦素材 CDN，且本地已有 creatives 缓存。

---

## 4. 抓包验证清单（针对 7.28.90，重点找新开屏接口）

### 4.0 准备

1. Surge 已安装并信任 CA；MitM 开启。  
2. 临时关闭其它去广告模块，避免重复改写干扰观察。  
3. MitM 先只加：`acs.m.goofish.com`, `g-acs.m.goofish.com`, `gw.m.goofish.com`, `acs.m.taobao.com`, `g-acs.m.taobao.com`（验证阶段可临时加宽，确认后收窄）。  
4. 打开 **最近请求**，过滤器准备好。

### 4.1 如何逼出「真·开屏配置请求」（应对缓存）

按优先级做，每做一步后杀进程再开闲鱼：

1. 设置 → 闲鱼 → 清除缓存 / 卸载重装（墨鱼与多份模块都强调）。  
2. 关掉 Wi‑Fi 用蜂窝冷启动一次，再开 Wi‑Fi 冷启动一次。  
3. 调系统时间跨天后冷启动（部分 App 会过期开屏缓存）。  
4. 若仍无 splash 相关 HTTPS：在最近请求搜 `idleadv`、`expose`、`creative`、`tanx`、`alicdn`，看是否只上报/只拉图。

### 4.2 最近请求建议搜索关键词

**开屏必搜：**

- `splash`
- `idlecommerce`
- `idleadv`
- `cacheAd`（看**响应体**，不一定在 URL）
- `ad.expose` / `idle.ad`
- `launch.report` / `scene.restore`
- `creatives-assets`
- `tanx`

**首页/信息流：**

- `idlehome.home.nextfresh`
- `idle.local.home`
- `user.strategy.list`
- `home.circle.list`
- `idlemtopsearch.search`
- `my.adapter`
- `playboy.recommend`
- `widget.refresh` / `home.community` / `newitem.page`（可莉差异接口）

**网关/域名（确认是否漏 MitM）：**

- 主机含：`goofish.com`、`m.taobao.com`、`alicdn.com`
- 路径含：`/gw/mtop.` 、`/h5/mtop.`

### 4.3 判定表（填空）

| 检查项 | 预期（若旧规则仍适用） | 你在 7.28.90 的结果 |
|--------|------------------------|---------------------|
| 冷启动是否出现 URL 含 `splash` 的请求 | 有 | |
| 该请求主机是 `acs` 还是 `g-acs` 还是 `*.taobao.com` | 记下完整 host | |
| 完整 mtop 名与版本（如 `...splash.ads/2.0`） | 记下 | |
| 响应 JSON 是否含 `cacheAd` / `adMap` / `dspList` | 有则截字段名 | |
| Map Local 回 `{}` 后是否仍出开屏 | 若仍出 → 缓存或未命中 | |
| atopsecret 脚本命中后 `cacheAd` 是否被清空 | 看改写后响应 | |
| 无 splash URL 但仍有开屏 | 极可能纯本地缓存或非 HTTPS 通道 | |
| `nextfresh` 是否仍有 `bizType=AD` / `mamaAD` | 有则信息流脚本仍有价值 | |

### 4.4 若发现「新接口」

记录并回传（可打码 Cookie）：

1. 完整 URL（含 `/gw/mtop.xxx/版本`）  
2. Method、是否有 body  
3. 响应里广告相关字段路径（不要整包隐私）  
4. Surge 是否已对该 host MitM（证书是否成功）  
5. 用 Map Local `data="{}"` 试一次 vs 脚本改字段试一次的差异  

然后才能把草稿里「待抓包验证」改成「已验证 7.28.90」。

### 4.5 安全注意

- 不要 MitM 支付宝/支付域名。  
- 不要整域 REJECT `acs.m.goofish.com`（会杀业务）。  
- 旧 `idle.home.welcome` reject 不要加回去。

---

## 5. 模块策略说明

草稿路径：`/workspace/xianyu/闲鱼去广告.sgmodule`

- **不把**墨鱼 reject-200 / zirawell 原样规则当作有效解。  
- **开屏优先验证** atopsecret 脚本（清 cacheAd）；Map Local 空 JSON 保留为对照。  
- **信息流**用 ishowshu 真实脚本 URL（fmz200 在用）。  
- **可莉**独有 Map Local / 接口作补充候选。  
- 所有规则上方注释带来源；状态保持「待抓包验证」。
