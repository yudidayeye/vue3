<!--
 * @Description:
 * @Author: lijin
 * @Date: 2023-07-27 13:58:10
 * @LastEditTime: 2023-08-07 11:16:32
 * @LastEditors:
-->

# vue3 源码调试

> 参考:
>
> https://zhuanlan.zhihu.com/p/460681229;
>
> https://juejin.cn/book/7070324244772716556/section/7071922360592367627

## 操作步骤

- 下载源码：https://github.com/vuejs/core
  源码采用 monorepo 的形式组织，各个模块源码和编译后代码放在 /package 对应目录下，其中运行时模块为 runtime-core
- 构建源码: 新建 build 脚本 -- 执行 vue 构建 `pnpm` run build

  ```json
  {
    "script": {
      "build:sourcemap": "node scripts/build.js --sourcemap -t"
    }
  }
  ```

  > --sourcemap 是输出 sourcemap，方便调试
  >
  > -t 是输出 typescript 类型声明文件，也是方便调试

- 从脚手架创建项目：根目录新建文件夹 `demos` -- `demos` 目录下创建 vite 项目 `pnpm create vite` -- 修改 monorepo 配置 `pnpm-workspace.yaml` 如下 -- 进入 vite 项目安装依赖 `pnpm install` (由于是 monorepo 项目，pnpm 会将本地 vue 安装到 vite 项目的 node_modules 中)
  ```yaml
  # pnpm-workspace.yaml
  packages:
    - 'packages/*'
    - 'demos/*'
  ```
- 项目运行调试：运行项目 `pnpm run dev` -- 创建 vscode 调试 `.vscode/launch.json` 如下 -- 开始调试（断点，查看调用堆栈） "launch Chrome"
  ```json
  // .vscode/launch.json
  {
    "version": "0.2.0",
    "configurations": [
      {
        "name": "Launch Chrome",
        "request": "launch",
        "type": "chrome",
        "url": "http://localhost:5173",
        "webRoot": "${workspaceFolder}",
        "runtimeArgs": ["--auto-open-devtools-for-tabs"]
      },
  }
  ```
- 修改 sourcemap 路径：发现调用堆栈的文件不可修改，定位错误 -- `rollup.config.js` 中修改 sourcemap 路径 -- 重新打包 -- 项目运行，断点调试 -- 查看调用堆栈路径

  ```js
  // rollup.config.js

  // sourcemap 配置
  output.sourcemap = !!process.env.SOURCE_MAP
  // 修改sourcemap的路径
  output.sourcemapPathTransform = (relativeSourcePath, sourcemapPath) => {
    const newSourcePath = path.join(
      path.dirname(sourcemapPath),
      relativeSourcePath
    )
    return newSourcePath
  }
  ```
