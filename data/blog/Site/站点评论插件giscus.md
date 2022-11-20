---
title: 站点评论插件giscus
date: '2022-11-20'
tags: ['Site']
summary: 'giscus是一个由 github discussion 驱动的站点评论插件, 不需要额外管理数据，支持表情回应，简单易用~'
---

由 github Discussion 驱动的站点评论插件 giscus, 不用部署服务管理数据, 支持评论和表情回应, 简单易用，记录一下接入过程~

- github 地址：https://github.com/giscus/giscus
- 配置生成： https://giscus.app/zh-CN

---

### 配置仓库

评论数据不需要额外管理，但是也是记录在一个仓库的 discussion 中的，所以还是需要一个仓库来“存储”这些数据，可以是博客站点项目或者其他任意项目，甚至是空项目，需要满足三点：

- 仓库权限为公开
- 需要为仓库安装 <a href="https://github.com/apps/giscus">giscus ( github app )</a>.
- 需要为仓库开启 Discussion 功能

<img src="https://christop.oss-cn-guangzhou.aliyuncs.com/mainsite/site/github_setting.png" />

### 创建页面元素

到配置生成的网址(https://giscus.app/zh-CN) 上填入自己仓库以及 Discussion 相关的信息，就会生成获得一段 `script`代码。然后启用`giscus`只需要在需要出现的位置下添加获取的`script`代码，其中包含了仓库 id 和 discussion 的分类 id, 然后评论就会被放置在该位置。

```html
<script
  src="https://giscus.app/client.js"
  data-repo="[在此输入仓库]"
  data-repo-id="[在此输入仓库 ID]"
  data-category="[在此输入分类名]"
  data-category-id="[在此输入分类 ID]"
  data-mapping="pathname"
  data-strict="0"
  data-reactions-enabled="1"
  data-emit-metadata="0"
  data-input-position="bottom"
  data-theme="preferred_color_scheme"
  data-lang="zh-CN"
  crossorigin="anonymous"
  async
></script>
```

如果是使用其他框架，则一般需要将上面的仓库名称、id 等信息填入配置文件或者 giscus 组件中以使其生效。比如 next.js, 可以放在`.env`文件中读取，填入上面仓库名和 id，分类名和 id。

```
NEXT_PUBLIC_GISCUS_REPO=
NEXT_PUBLIC_GISCUS_REPOSITORY_ID=
NEXT_PUBLIC_GISCUS_CATEGORY=
NEXT_PUBLIC_GISCUS_CATEGORY_ID=
```
