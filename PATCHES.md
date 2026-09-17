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

## 版本 / tag

- 上游基点：`upstream-base` = `369955d`（最后纯上游 commit）
- 冻结 tag：`v0.2.0-ds-agent.1`

## 许可证

Apache-2.0。保留 `LICENSE`、版权头与 attribution 即可，vendor/分发合规。
