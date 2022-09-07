https://github.com/lexguy/windnextblog

---

pm2 运行 build 版本问题:

https://github.com/Unitech/pm2/issues/4811

- Run one of the following
  yarn build or npm run build ("next build")

- Create a file with an extension .json for example "deploy.json":
  {
  "apps" : [{
  "name" : "Project_name",
  "script" : "./node_modules/next/dist/bin/next",
  "env":{
  "PORT": "5454"
  }
  }
  ]
  }

- Execute the following command

pm2 start deploy.json

按照这个方法之后，运行起来的仍然是 dev 版本，要运行 build 版本的话，需要修改 ./node_modules/next/dist/bin/next 文件

```js
const defaultCommand = 'start'
const commands = {
  build: () => Promise.resolve(require('../cli/next-build').nextBuild),
  start: () => Promise.resolve(require('../cli/next-start').nextStart),
  export: () => Promise.resolve(require('../cli/next-export').nextExport),
  dev: () => Promise.resolve(require('../cli/next-dev').nextDev),
  lint: () => Promise.resolve(require('../cli/next-lint').nextLint),
  telemetry: () => Promise.resolve(require('../cli/next-telemetry').nextTelemetry),
  info: () => Promise.resolve(require('../cli/next-info').nextInfo),
}
```

默认命令修改为

```js
const defaultCommand = 'start'
```

start 会从 .next 文件夹部署服务

---

远端 Linux 使用 Node 版本为 14.18.2 ，sharp 库依赖为 0.28.3 版本下载，其他 30+版本难以下载。。

---

远端 Linux 直接 screen 跑起来。

---

PM2 命令行运行 npm 命令

```shell
pm2 start npm --watch --name <taskname> -- run <scriptname>
```

> 注意 run 前后有空格

---

目前已知 linux 不挂断程序的方式

1. nohup
2. screen
3. pm2 (JS 程序)
