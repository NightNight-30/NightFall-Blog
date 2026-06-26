---
title: JavaScript 小技巧合集
date: 2024-01-01 10:00:00
updated: 2024-01-15 15:30:00
notebook: tech-notes
menu_id: tech
tags: [JavaScript, 前端]
excerpt: 记录日常开发中积累的 JavaScript 实用技巧
---

## 数组去重的几种方法

### 方法 1：Set
```javascript
const arr = [1, 2, 2, 3, 3, 3]
const unique = [...new Set(arr)]
```

### 方法 2：filter + indexOf
```javascript
const arr = [1, 2, 2, 3, 3, 3]
const unique = arr.filter((item, index) => arr.indexOf(item) === index)
```

---

## 可选链操作符 ?.

安全访问嵌套对象属性，避免报错：

```javascript
// ❌ 不安全
const name = user.address.city.name

// ✅ 安全
const name = user?.address?.city?.name
```
