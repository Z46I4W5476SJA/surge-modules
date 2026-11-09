# 📜 CHANGELOG

## 2026-11-09

### 🐟 闲鱼 `xianyu.sgmodule` v1.0.0
- 适配闲鱼 iPhone 7.28.90。
- 开屏：用 atopsecret 脚本清除 `idlecommerce.splash.ads` 响应中的 `cacheAd`，解决 reject / 空 JSON 拦不掉缓存开屏的问题。
- 信息流：首页、同城、我的、搜索使用 ishowshu `goofish.js` 脚本；搜索底纹、推荐等走 Map Local 空 JSON。
- 验证：卸载重装后冷启动 4 次，无开屏广告。
