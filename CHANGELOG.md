# 📜 CHANGELOG

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
