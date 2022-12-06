---
title: 令人迷惑的 JavaScript
date: '2022-12-06'
tags: ['JavaScript']
summary: JavaScript 大多数时候靠谱，总有一些例外
---

```javascript
;(3.145).toFixed(2)
// '3.15'
;(3.155).toFixed(2)
// '3.15'
;(3.15).toFixed(1)
// '3.1'
;(3.25).toFixed(1)
// '3.3'
```
