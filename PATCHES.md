# PATCHES

本仓库是 ds-agent 冻结的 DirectorX fork，相对上游 `LaplaceYoung/dsh-directorx` 的本地改动清单。
每条补丁说明「改了什么 / 为什么 / 影响哪个 dsh 版本」，重装或升级时据此核对。

## 1. 非 ASCII 项目路径 header 编码

- 文件：`src/project.ts`、`src/client/stage/project.ts`
- 内容：`x-directorx-project` 请求头对项目路径做 `encodeURIComponent`（客户端）/ `decodeURIComponent`（服务端）
- 原因：项目/工作区路径含中文等非 Latin-1 字符时，浏览器 `fetch` 拒绝该 header，画布请求发不出去
- 状态：已提交 `c1e34b7`；上游 PR #4（待回应）

## 2. Windows `which` 探测修复

- 文件：`src/providers/ffmpeg.ts`
- 内容：`requireBinary()` 按平台选探测命令 `process.platform === 'win32' ? 'where' : 'which'`
- 原因：`which` 是 Unix 专用命令，Windows 原生环境没有；否则 ffmpeg/ffprobe 即使装了也被误判为缺失

## 3. `registerContinuableSetup` 守卫

- 文件：`src/subagents.ts`
- 内容：`registerContinuableSetup` 改为可选，`typeof !== 'function'` 时跳过（返回空 disposer）
- 原因：dsh 0.1.2+ 已把该扩展点改名为 `startContinuable`，0.1.5-rc.1 下旧名不存在；守卫让服务端能在 0.1.5-rc.1 上加载（放弃子代理编排注入，画布依赖的 `remote.*` 由客户端注入，不受影响）
- 关联：上游 issue #3（DirectorX 自身版本不一致）

## 4. 移除死依赖 tui-image-editor

- 文件：`package.json`（dependencies）
- 内容：删除 `tui-image-editor`（连带传递依赖 `fabric` → `canvas` 原生模块一并移除）
- 原因：源码中无任何 import（死依赖），但 `canvas` 原生模块在 Windows 上构建失败（无 node-v127 预编译 + 缺 MSVC），拖慢/污染 install；移除后彻底消除。Studio 编辑台实际用 three.js，与 tui-image-editor 无关。

## 5. `disableStudio` 开关：默认隐藏 Studio 编辑台

- 文件：`src/config.ts`（新增 `disableStudio` 配置，默认 true）、`src/tools.ts`（条件注册 directorx_studio）、`src/edit-plan.ts`（studio 路由重定向 image_edit/video_process）、`src/skill-route.ts`、`src/client/stage/SessionDock.tsx`、`src/client/stage/session-media.ts`、`src/canvas-parse.ts`、`src/tool-collect.ts`
- 内容：`disableStudio=true`（默认）时不注册 `directorx_studio` 工具；编辑路由把「调色/打开编辑台」重定向到 `directorx_image_edit` / `directorx_video_process`；客户端隐藏「打开编辑台」入口。改回 `false` 即重新注册工具。
- 原因：Studio 编辑台（three.js）非核心链路，默认隐藏；保留 three.js 依赖与工具代码，一键恢复。

## 6. `requireBinary` 改用 `-version` 探测（修复中文 PATH 漏检）

- 文件：`src/providers/ffmpeg.ts`
- 内容：`requireBinary()` 不再用 `where`/`which`（补丁 #2），改为 `spawnSync(command, ['-version'])` 直接探测
- 原因：`where` 是 ANSI 命令行工具，对非 ASCII（中文）PATH 条目会漏检——项目路径 `D:\AI大模型应用开发\...` 含中文，导致 vendor/ffmpeg 里的 ffprobe 被判为「不存在」；CreateProcess 走 Unicode PATH 搜索能正常命中，且跨平台。

## 版本 / tag

- 上游基点：`upstream-base` = `369955d`（最后纯上游 commit）
- 冻结 tag：`v0.2.0-ds-agent.1`（which/guard/header）、`v0.2.0-ds-agent.2`（移除 tui-image-editor + disableStudio 隐藏 Studio）、`v0.2.0-ds-agent.3`（requireBinary -version 探测修复中文 PATH）

## 许可证

Apache-2.0。保留 `LICENSE`、版权头与 attribution 即可，vendor/分发合规。
