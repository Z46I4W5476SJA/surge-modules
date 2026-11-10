# 📜 CHANGELOG

## 2026-11-10

### 🐟 闲鱼 `xianyu.sgmodule` v1.2.1
- 回退：停用 AMDC 调度拦截。实测拦截后 App 仍用内置/缓存策略连 `tls-goofish.dingtalk.com`，mtop 不走 `acs.m.goofish.com`，开屏照出，且 AMDC 反复重试。
- 当前行为等同 v1.0.0：刚卸载重装时有效，切到长连接后开屏可能复现。

### 🐟 闲鱼 `xianyu.sgmodule` v1.2.0（测试中）
- 新增 URL Rewrite：拒绝闲鱼 AMDC 调度 `http://<IP>/amdc/mobileDispatch?appkey=12431167`，期望 mtop（含开屏）回退到可 MitM 的 HTTPS 网关。
- 只匹配闲鱼 appkey，不影响其他阿里系 App。若聊天异常，回退 v1.1.1。

### 🐟 闲鱼 `xianyu.sgmodule` v1.1.1
- 回滚：停用 `DOMAIN,tls-goofish.dingtalk.com,REJECT`。实测拒绝后聊天记录加载失败，且 App 没有回退到 HTTP mtop 网关。

### 🐟 闲鱼 `xianyu.sgmodule` v1.1.0
- 抓包发现：App 使用一段时间后，mtop 接口（含开屏 `splash.ads`）改走 ACCS 长连接 `tls-goofish.dingtalk.com`，无法 MitM，cacheAd 脚本不生效，冷启动和后台切回都会出现开屏。
- 新增 `DOMAIN,tls-goofish.dingtalk.com,REJECT`，让 App 回退到 HTTPS mtop 网关。
- 风险：长连接可能承载聊天消息推送，如消息不实时可注释该行。

## 2026-11-09

### 🛒 京东 `jd.sgmodule` v1.0.0
- 适配京东 iPhone 16.1.50。
- 京东 ≥15.9.50 把 `functionId` 移进 base64 POST body，开屏接口 `start` 改为 `startup`，按 URL 匹配的旧规则失效。
- 开屏/首页/我的：用 zbsdsb `JD_remove_ads_v2.js` 按响应内容清除 `images[]` 并置 `showTimesDaily=0`，兼容明文与 base64。
- 辅助：纯广告/监测域 REJECT、旧协议 functionId Map Local 空 JSON、DSP `adx` 路径 Map Local。
- MITM 仅 `api.m.jd.com, bdsp-x.jd.com, dsp-x.jd.com`，不含任何金融/支付域名。
- 验证：卸载重装后冷启动多次，无开屏广告。

### 🐟 闲鱼 `xianyu.sgmodule` v1.0.0
- 适配闲鱼 iPhone 7.28.90。
- 开屏：用 atopsecret 脚本清除 `idlecommerce.splash.ads` 响应中的 `cacheAd`，解决 reject / 空 JSON 拦不掉缓存开屏的问题。
- 信息流：首页、同城、我的、搜索使用 ishowshu `goofish.js` 脚本；搜索底纹、推荐等走 Map Local 空 JSON。
- 验证：卸载重装后冷启动 4 次，无开屏广告。
