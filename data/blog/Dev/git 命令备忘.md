---
title: git 命令备忘
date: '2022-09-07'
tags: ['dev', 'git']
summary: git 操作记录, tag、remote...
---

## 标签

1. 创建标签：

```shell
git tag v1.0.1
git tag -a v1.0.1
```

-a 需要添加额外注解信息。

2. 同步新标签到远端：

```shell
git push origin v1.0.1
// 推送全部 tag
git push origin --tags
git push --tags
```

3. 删除标签

```
git tag -d v1.0.1
```

4. 同步删除标签到远端：

```shell
git push origin :v1.0.0
```

## 远端

更新 origin url:

```shell
git remote set-url origin git@github.com:name/project.git
```
