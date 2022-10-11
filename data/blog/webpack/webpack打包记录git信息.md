---
title: Webpack 打包记录 git commit 信息
date: '2021-06-19'
tags: ['Webpack']
summary: '打包的时候执行 git 命令, 获取到当前版本的 git commit hash, 将该 hash 写到文件名里，奇怪的 js 版本的查看方法又增加了'
---

## 场景

问题点：因为最近开始模块拆分的缘故，需要将子项目 build 出来的 JS 动态插入到主项目中加载，但是存在的一个问题就是测试环境子项目迭代较快，并没有打 tag 提发布的完整流程，每次生成的 JS 都没有记录版本信息，导致多次测试迭代后已不能快速找到当前测试使用的 JS 是在哪个代码版本上打包出来的。

> 在规范的 git 管理和发布流程下自然是没有这个问题的，对待线上环境自然本该如此。但问题出现点本身就是子项目测试环境，代码提交更新频繁，再加上子项目是本地打包，流程长的发布耗时不太有必要，这就导致了每个测试版本的 git 信息丢失。总的来说是为了快速开发和提测迭代产生一个版本管理的问题。

要做的就是需要将打包产物 JS 文件和当前的 git 版本信息关联起来，然后回查问题就能找到这个 JS 是在哪个版本上打包出来的，快速找到生效代码。

怎么关联？思路肯定就是在打包产物中留下 git 信息，比如：

- 将 git 信息写入到 JS (或者 html) 文件，然后页面上就能查找到
- 更直接的，将 git 提交写到生成的 JS 文件名上

## 方案

### 1. git-revision-webpack-plugin

webpack 插件： <a href="https://github.com/pirelenito/git-revision-webpack-plugin">git-revision-webpack-plugin</a>

按照文档配置尝试之后发现报错信息很频繁，`undefiend`， `not a constructor`...安装其他版本之后依然是没有解决。可能是哪里配置问题，或者更新暂慢的缘故。而且已经有其他备选思路，所以暂时不采用不稳定的第三方库。

### 2. node 读取 .git 文件中的信息

因为我们可以在 webpack 打包之前执行一段 node, 而且 git 管理的信息都在.git 文件夹中，其中的文件目录如下，那么就有了操作空间：使用 node 读取 .git 中的文件，这样自然就能获取到想要的内容。但是对其中的详细内容，git 信息记录的格式并没有进行深一层的研究，并不确定是否是易执行的方式，暂时搁置。

> 此处可深入点，.git 文件夹的结构和文件记录格式。
>
> git 文档链接：<a href="https://git-scm.com/book/zh/v2">https://git-scm.com/book/zh/v2</a>

### 3. node 执行 git 命令获取信息

这个方案应该是操作性最强最容易执行的了，还是在 webpack.config.js 中执行一段 node, 获取提交 commitId，然后写到生成文件名的配置。

```js
const { execSync } = require('child_process')
/*
 * ...
 */
let fileName = '[hash:8]' // 默认8位hash
if (env != 'development') {
  fileName = execSync('git rev-parse --short HEAD', { encoding: 'utf-8' }).replace('\n', '')
}

const config = {
  // ...
  output: {
    filename: `${fileName}.js`,
    //...
  },
}
module.exports = config
```

> `git rev-parse --short HEAD` 查看最近提交的短 commit id

如果是本地打包，还应该加上一个限制，只有提交之后才能`build JS`，否则很可能打包之后的内容包含了 y 一些未提交的临时修改，然后这些修改后来又被还原的话就无从查证了（但已经被 build 进去生效了）。未提交代码产生提交记录之前不能`build`:

```js
if (env != 'development') {
  let gitlocalDiff = execSync('git diff --name-only', { encoding: 'utf-8' }).replace('\n', '')
  if (gitlocalDiff) {
    console.error(new Error("you should commit your files before execuating 'npm run build'"))
    process.exit(-1)
  }
  commitID = execSync('git rev-parse --short HEAD', { encoding: 'utf-8' }).replace('\n', '')
}
```

> `git diff --name-only` 查看本地修改文件名

然后就可以进行正常的流程了。`commit` ->`build`->` 产生的 JS 以commitID为文件名`。

查找当前生效代码就是：`查看 JS 文件名（获取git提交id）` -> `切到对应的 commitId `就是当前的打包代码了。
