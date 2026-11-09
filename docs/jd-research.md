# 京东去广告调研（iPhone 16.1.50｜草稿）

> 整理日期：2026-11-09（Asia/Shanghai）  
> 目标：`com.360buy.jdmobile` 开屏为主，兼顾首页/我的/搜索推广  
> 模块草稿：同目录 `jd.sgmodule`（v0.1.0，**待粥粥实测**）

---

## 1. 关键结论（先看这里）

| 项 | 结论 | 置信度 |
|---|---|---|
| 16.1.50 是否仍可用 URL `functionId=start` 匹配 | **大概率否**。≥15.9.50 起 functionId 进 base64 POST body，开屏改名 `startup` | 源仓库自述（zbsdsb，2026-08-12 验证） |
| 仅 reject / Map Local `functionId=start` | 对新协议 **通常无效**（正则 miss） | 同上 |
| 推荐主路径 | **响应脚本按内容特征清洗**：`images[]` / `floorList` / `floors` | 源已核实 + HAR 自述 |
| 是否像闲鱼有 `cacheAd` 字段 | **未见**公开脚本清「本地缓存引用字段」；开屏清洗点是 `images` + `showTimesDaily`。zirawell 仍提示「可能需重装」 | 部分核实 / 部分推测 |
| MITM 最小集 | `api.m.jd.com`（+ 可选 `bdsp-x.jd.com,dsp-x.jd.com`） | 多源一致 |
| 支付/金融域 | **禁止加入** MitM / 规则 | 用户硬性要求 |

---

## 2. 协议与开屏接口（源已核实）

### 2.1 ≥15.9.50 协议升级

来源：[`zbsdsb/loon-adblock-plugins` README](https://github.com/zbsdsb/loon-adblock-plugins)

1. `https://api.m.jd.com/client.action` 的 `functionId` 从 **URL query** 迁到 **base64 编码的 POST body**  
2. 响应体从明文 JSON 变为 **整体 base64**  
3. 开屏广告接口：`start` → **`startup`**

因此：所有依赖  
`pattern=...client.action?functionId=start`  
或  
`url.includes("functionId=start")`  
的旧规则/旧脚本，在 16.1.50 上 **预期失配**。

### 2.2 开屏响应特征（脚本侧）

| 来源 | 清洗动作 | 是否依赖 URL functionId |
|---|---|---|
| zbsdsb `JD_remove_ads_v2.js` | 有 `images` 数组则清空；`showTimesDaily=0` | **否**（内容特征） |
| RuCu6 / fmz200 / zirawell 旧脚本 | 同上，但先要求 URL 含 `functionId=start` | **是**（旧协议） |
| blackmatrix7 `startup.js` | 改 `images[*].showTimes/onlineTime` + `showTimesDaily=0` | **是**（URL=`start`） |
| ddgksf / 多数合集 | `functionId=(start\|queryMaterialAdverts) reject` | **是** |

**与闲鱼对照**：闲鱼失效点是 `cacheAd` 本地缓存引用；京东公开脚本未见等同字段名。若清 `images` 后仍播开屏，优先怀疑 **本地已缓存素材**（zirawell 注释原文：*may need to reinstall app*）或请求未走 HTTP MitM。

---

## 3. 各仓库对照

### 3.1 启用 / 重点参考

| 仓库 | 文件 | 日期/验证 | 开屏做法 | 对本草稿 |
|---|---|---|---|---|
| **zbsdsb/loon-adblock-plugins** | [`JD_remove_ads_v2.lpx`](https://raw.githubusercontent.com/zbsdsb/loon-adblock-plugins/main/JD_remove_ads_v2.lpx) / [`JD_remove_ads_v2.js`](https://raw.githubusercontent.com/zbsdsb/loon-adblock-plugins/main/JD_remove_ads_v2.js) | 2026-08-12 验证 ≥15.9.50 | 拦全部 `client.action`；base64/明文双解；清 `images` | **主脚本** |
| **zirawell/R-Store** | [`jd.sgmodule`](https://raw.githubusercontent.com/zirawell/R-Store/main/Rule/Surge/Adblock/App/J/京东/jd.sgmodule) / [`jd.js`](https://raw.githubusercontent.com/zirawell/R-Store/main/Res/Scripts/AntiAd/jd.js) | #!date=2026-05-20 | URL `functionId=start` + 脚本清 images；另有大量 DOMAIN REJECT、Map Local | 借 DOMAIN / Map Local；**脚本对 16.1.50 预期失效** |
| **QingRex / iKeLee** | [`京东去广告.sgmodule`](https://raw.githubusercontent.com/QingRex/LoonKissSurge/refs/heads/main/Surge/京东去广告.sgmodule) | #!date=2025-05-13 | URL 匹配 `start\|welcomeHome|...` + kelee `JD_remove_ads.js`；Map Local 搜索/推荐 | 旧协议；basicConfig jq 关 socket/httpdns（本草稿未搬，防副作用） |
| **fmz200/wool_scripts** | `blockAds.module`（#!date=2026-10-30）+ [`jingdong.js`](https://raw.githubusercontent.com/fmz200/wool_scripts/main/Scripts/jingdong/jingdong.js) | 脚本头 2024-04-18（与 RuCu6 同源） | DOMAIN 广告域 + 旧 URL 脚本 + `functionId=start` Map Local | DOMAIN 列表参考；脚本不单用 |
| **RuCu6（jqyisbest 镜像）** | [`Scripts/jingdong.js`](https://raw.githubusercontent.com/jqyisbest/RuCu6/main/Scripts/jingdong.js) | 2024-04-18 | URL `functionId=start` 清 images；welcomeHome 删悬浮楼层 | v2 逻辑祖先；不直接启用 |

### 3.2 对照后未作为主方案

| 仓库 | 情况 |
|---|---|
| **ddgksf2013/Rewrite** | `AdBlock/` 下 **无** `JingDong.conf`；仅在第三方收录的 `startingad.module` 见 `start\|queryMaterialAdverts` reject + 热词脚本 |
| **blackmatrix7** | `script/startup` 仍匹配 `functionId=start`（旧协议） |
| **kokoryh/Sparkle** | 仓库树检索 **未见** 京东去广告文件 |
| **app2smile/rules** | 检索 **未见** 京东模块 |
| **Walvez/surge-startup-ads** | 主要为墨鱼开屏合集转换；未单独发现适配 base64 的京东专用模块 |

### 3.3 接口清单（按来源标注）

#### 开屏 / 启动相关

| functionId / 路径 | 用途（来源描述） | 处理建议 | 状态 |
|---|---|---|---|
| `start` | 旧开屏配置 | 旧脚本清 `images`；新版改名 | 源核实（旧） |
| `startup` | 新开屏配置（≥15.9.50） | v2 按 `images` 特征清 | 源核实（zbsdsb） |
| `queryMaterialAdverts` | 素材广告 | Map Local `{}` / reject（需 URL 带参） | 多源有规则 |

#### 首页 / 我的 / 订单 / 物流（脚本清洗，非整接口 reject）

| functionId | 用途 | 来源 |
|---|---|---|
| `welcomeHome` | 首页楼层（悬浮、通栏、二楼等） | RuCu6 / zirawell / iKeLee / v2 |
| `personinfoBusiness` | 我的页推广楼层、弹窗 | 同上 |
| `myOrderInfo` | 订单页 banner/plus 楼层 | 同上 |
| `deliverLayer` / `orderTrackBusiness` | 物流页 banner | 同上 |
| `getTabHomeInfo` | 新品页悬浮/二楼 | 同上 |
| `readCustomSurfaceList` | 底栏自定义表面（zirawell 另有） | zirawell |

#### Map Local / reject（辅；依赖 URL functionId）

| functionId | 用途 | 来源 |
|---|---|---|
| `searchBoxWord` | 搜索底纹/热词 | iKeLee / zirawell |
| `stationPullService` | 站内拉取 | iKeLee / zirawell |
| `uniformRecommend0/6/7` | 推荐 | iKeLee / zirawell |
| `xview2Config` / `jdaPreDownload` | 投放配置 / 预下载 | zirawell |
| `(bdsp-x\|dsp-x).jd.com/adx/` | DSP 广告 | zirawell / fmz200 / ddgksf |

#### 明确不碰（支付 / 金融）

| 主机 / 路径 | 原因 |
|---|---|
| `ms.jr.jd.com`（含 `getLoadingPicture`） | 京东**金融**开屏；用户禁金融域 |
| `*.jr.jd.com` / `pay.m.jd.com` / jdpay / wallet | 支付/金融 |
| 整域 `api.m.jd.com` REJECT | 购物主网关 |

---

## 4. 已知风险

1. **证书固定 / 非 HTTP 通道（推测，勿当事实）**  
   部分广告素材或配置可能走私有通道、已固定证书的连接或本地缓存。若 MitM 最近请求里完全看不到开屏相关 `client.action`，脚本无法生效。  
2. **签名参数 `sign` / body 校验**  
   改响应体一般不触发请求签名校验；若日后客户端校验响应完整性，脚本会失效（目前各活跃仓库仍在用改响应方案）。  
3. **拦全部 `client.action` 的开销**  
   v2 对每个 `client.action` 响应跑脚本，但无广告特征则原样放行，并跳过 >2MB body。仍可能略增 CPU；属新协议下的折中。  
4. **DOMAIN REJECT 误伤**  
   - `mapi.m.jd.com`、`dns.jd.com`、`img-x.jd.com`：存在「广告域 vs 业务 CDN/API」争议 → **草稿默认不启用**。  
5. **与其它京东模块叠用**  
   请禁用旧版 iKeLee / zirawell / fmz200 京东去广告，避免双重改写。  
6. **本地缓存开屏**  
   若脚本已生效但冷启动仍闪一次广告：卸载重装或清京东 App 数据后再验（类比闲鱼）。

---

## 5. 本草稿策略（与 `jd.sgmodule` 一致）

```
升级顺序（符合「先 reject/Map Local，再脚本」）：
1. [Rule] 纯广告 DOMAIN REJECT（无 MitM 依赖）
2. [Map Local] 仍带 URL functionId 的广告接口 → {}
3. [Script] zbsdsb v2 拦 client.action（新协议主路径）
4. MITM 仅 api.m.jd.com + 可选 DSP 主机
```

**不**默认启用：旧 URL 脚本、金融域、`dns.jd.com` / `mapi.m.jd.com` REJECT、basicConfig 关 httpdns。

---

## 6. 粥粥验证清单（开屏仍在时）

### 6.1 安装与互斥

1. Surge 导入本模块；**关掉**其它京东去广告模块。  
2. 确认 MitM 已信任，且 `api.m.jd.com` 在解密列表。  
3. **完全杀掉**京东 → 再冷启动（仅切后台不够；zbsdsb README 同述）。

### 6.2 Surge「最近请求」里要找的关键字

在 **api.m.jd.com** 上筛选（新协议 functionId 可能不在 URL，看 **请求体 / 脚本是否命中**）：

| 优先看 | 说明 |
|---|---|
| `client.action` 且路径无 `functionId=` | 新协议典型；应被「京东-去广告-v2」命中 |
| URL 仍含 `functionId=startup` 或 `start` | 若出现，记录是明文还是旧式 |
| `queryMaterialAdverts` | 素材广告辅接口 |
| `welcomeHome` / `personinfoBusiness` | 首页/我的推广 |
| `bdsp-x.jd.com` / `dsp-x.jd.com` `/adx/` | DSP；可对照 Map Local 是否命中 |
| **看不到**上述请求但仍有开屏 | 可能本地缓存 / 非 HTTP；尝试卸载重装后再冷启动 3～4 次 |

### 6.3 抓包时建议记录

- 请求 URL 是否含 `functionId=`  
- 请求方法（POST？）与 Content-Type  
- 响应是否整段 base64（非 `{` 开头）  
- 解码后 JSON 是否含 `images` / `showTimesDaily` / `floorList`  
- 模块脚本是否显示已执行、改写前后 body 长度变化  

### 6.4 判定

| 现象 | 可能原因 | 下一步 |
|---|---|---|
| 开屏消失 | v2 生效 | 标为已验证，升 v1.0.0 入库 |
| 脚本命中但开屏仍在 | 本地缓存或字段变更 | 卸载重装；或把解码后 JSON 关键点发回 |
| 最近请求无 client.action | MitM 未生效 / 证书问题 / 非该通道 | 查 Surge MitM 与证书 |
| 购物车/下单异常 | 误伤 | 立刻禁用模块；反馈具体页面 |

---

## 7. 源 URL 速查

```
# 主脚本（启用）
https://raw.githubusercontent.com/zbsdsb/loon-adblock-plugins/main/JD_remove_ads_v2.js
https://raw.githubusercontent.com/zbsdsb/loon-adblock-plugins/main/JD_remove_ads_v2.lpx

# 对照（旧协议，默认不启用）
https://raw.githubusercontent.com/zirawell/R-Store/main/Rule/Surge/Adblock/App/J/京东/jd.sgmodule
https://raw.githubusercontent.com/zirawell/R-Store/main/Res/Scripts/AntiAd/jd.js
https://raw.githubusercontent.com/QingRex/LoonKissSurge/refs/heads/main/Surge/京东去广告.sgmodule
https://raw.githubusercontent.com/fmz200/wool_scripts/main/Scripts/jingdong/jingdong.js
https://raw.githubusercontent.com/jqyisbest/RuCu6/main/Scripts/jingdong.js
https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/script/startup/startup.sgmodule
```

本地抓取副本（便于 diff）：`/workspace/jd/src/`。

---

## 8. 未证实 / 勿当事实

- 京东开屏是否使用与闲鱼 `cacheAd` 同构的「缓存指针」字段名  
- 开屏是否部分走 ACCS/长连接或证书固定私有协议  
- 16.1.50 相对 15.9.50 是否又改字段（需粥粥 HAR）  
- `img-x.jd.com` / `kepler*.jd.com` 一律算广告域  

以上需抓包后再升级模块版本。
