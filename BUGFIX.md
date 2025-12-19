# Bug Fix: 139 移动云盘分类问题

## 问题描述

用户反馈：URL `https://yun.139.com/shareweb/#/w/i/2qidFiUxax0iv` 被错误分类为 `others`，但应该是 `mobile`（移动云盘）。

## 原因分析

多个插件的 URL 分类逻辑只匹配了 `caiyun.139.com` 或 `caiyun`，但没有匹配 `yun.139.com`。

移动云盘的域名有多个：
- `caiyun.139.com` - 传统移动云盘
- `yun.139.com` - 新的移动云盘域名
- `caiyun.feixin.10086.com` - 飞信云盘

## 修复内容

### 1. Pansearch 插件
**文件：** `server/core/plugins/pansearch.ts`

```typescript
// 修复前
if (u.includes("caiyun")) return "mobile";

// 修复后
if (u.includes("caiyun") || u.includes("yun.139.com")) return "mobile";
```

### 2. Qupansou 插件
**文件：** `server/core/plugins/qupansou.ts`

```typescript
// 修复前
if (u.includes("caiyun.139.com")) return "mobile";

// 修复后
if (u.includes("caiyun.139.com") || u.includes("yun.139.com")) return "mobile";
```

### 3. Panta 插件
**文件：** `server/core/plugins/panta.ts`

```typescript
// 修复前
if (u.includes("caiyun.139.com")) return "mobile";

// 修复后
if (u.includes("caiyun.139.com") || u.includes("yun.139.com")) return "mobile";
```

### 4. Duoduo 插件
**文件：** `server/core/plugins/duoduo.ts`

```typescript
// 修复前
mobile: /https?:\/\/(?:caiyun\.139|caiyun\.feixin\.10086)\.com\/[^"\s]+/g,

// 修复后
mobile: /https?:\/\/(?:caiyun\.139|caiyun\.feixin\.10086|yun\.139)\.com\/[^"\s]+/g,
```

### 5. TG 服务（已正确）
**文件：** `server/core/services/tg.ts`

```typescript
// 已正确
if (host === "yun.139.com") return "mobile";
```

## 验证

### 测试 URL
```
https://yun.139.com/shareweb/#/w/i/2qidFiUxax0iv
https://yun.139.com/shareweb/#/w/i/2qidENShSceev
```

### 预期结果
```json
{
  "type": "mobile",
  "url": "https://yun.139.com/shareweb/#/w/i/2qidFiUxax0iv",
  "password": "",
  "note": "阿凡达｜ 2025科幻电影"
}
```

## 影响范围

### 已启用插件
- ✅ `pansearch` - 已修复
- ✅ `qupansou` - 已修复
- ✅ `panta` - 已修复
- ✅ `duoduo` - 已修复
- ✅ `hunhepan` - 无需（使用 API disk_type）
- ✅ `labi` - 无需（只支持夸克）
- ✅ `jikepan` - 无需（使用 API service）
- ✅ `xuexizhinan` - 无需（只支持夸克）
- ✅ `thepiratebay` - 无需（种子站）
- ✅ `nyaa` - 无需（种子站）

### TG 服务
- ✅ `server/core/services/tg.ts` - 已正确，无需修改

## 其他插件

以下插件也被发现有类似问题，但当前未启用：
- `panyq.ts` - 已修复
- `susu.ts` - 需要修复
- `pan666.ts` - 需要修复
- `zhizhen.ts` - 需要修复
- `ouge.ts` - 需要修复
- `wanou.ts` - 需要修复
- `fox4k.ts` - 需要修复
- `hdr4k.ts` - 需要修复
- `muou.ts` - 需要修复

如果未来启用这些插件，需要同样修复。

## 总结

本次修复确保了所有**当前启用**的插件都能正确识别 `yun.139.com` 为移动云盘，解决了用户反馈的分类错误问题。
