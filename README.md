# 🧩 surge-modules

粥粥自用的 Surge (iOS) 去广告模块，**一个 App 一个模块**，重点处理开屏广告和信息流广告。

## 📦 模块列表

| App | 模块 | 适配版本 | 模块版本 | 状态 | 安装链接 |
|---|---|---|---|---|---|
| 🐟 闲鱼 | [`modules/xianyu.sgmodule`](modules/xianyu.sgmodule) | 7.28.90 | 1.1.1 | ✅ 2026-11-10 更新 | `https://raw.githubusercontent.com/Z46I4W5476SJA/surge-modules/main/modules/xianyu.sgmodule` |
| 🛒 京东 | [`modules/jd.sgmodule`](modules/jd.sgmodule) | 16.1.50 | 1.0.0 | ✅ 2026-11-09 已验证 | `https://raw.githubusercontent.com/Z46I4W5476SJA/surge-modules/main/modules/jd.sgmodule` |

## 📁 目录结构

```text
surge-modules/
├── modules/        # 每个 App 一个 .sgmodule，文件名用 App 英文/拼音
├── scripts/        # 自托管的 http-response 脚本（从上游固定版本后放这里）
├── docs/           # 每个 App 的调研记录与抓包验证清单
├── CHANGELOG.md
└── README.md
```

## 📝 模块约定

- 头部必须有：`#!name`、`#!desc`（App 版本｜日期｜验证状态）、`#!version`、`#!category=去广告`。
- 每条规则上方注释写：**来源**（上游仓库链接）和**用途**。
- 优先级：URL Rewrite `reject` → Map Local 空 JSON → 脚本删字段；按路径拦，不整域名。
- MITM 统一 `hostname = %APPEND% ...`，只加必要主机，**不加任何银行、支付域名**。
- 和网上模块混用时，同一接口只保留一个模块在改写。

## 🔁 失效后的处理流程

1. 记下 App 新版本号，先关掉对应模块确认广告来源。
2. 卸载重装 App（清掉开屏缓存），开启 Surge HTTP 捕获后冷启动 3 次。
3. 在「最近请求」搜 `splash`、`ad`、`cacheAd` 等关键词，定位新接口。
4. 修改模块，`#!version` 加一，`#!desc` 更新 App 版本与日期，写进 `CHANGELOG.md`。
5. 卸载重装再验证，通过后提交。

## ⏪ 回滚

在 Surge 模块里把安装链接里的 `main` 换成某个提交的 SHA 或 tag（如 `xianyu-v1.0.0`），即可固定到旧版本。
