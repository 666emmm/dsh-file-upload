# dsh 版本兼容说明（dsh-file-upload v0.2.0）

> 依据：`T1-dsh版本差异报告.md`（2026-08-29，deepseek-ai/dsh 当前安装版 vs 最新版）。

## 当前环境

- 安装版：`@deepseek-ai/dsh@0.1.1-rc.2`（npm 全局，lib/ 编译产物）
- 最新发布：`dsh-v0.1.2-alpha.1`（2026-08-27 GitHub Release，**npm 未发布**）
- 结论：本插件 v0.2.0 在 **0.1.1-rc.2 上直接可用，无需改动**。

## 升级到 v0.1.2-alpha.1 时必须改的地方（唯一破坏性变更）

| 项 | rc.2（现状） | alpha.1（必改） |
|---|---|---|
| `package.json` → `dsh.client.inject` | `"@deepseek-ai/dsh-client-runtime"` | 替换为 `"@deepseek-ai/dsh-client-ui-renderer"` |

原因：alpha.1 移除了 `@deepseek-ai/dsh-client-runtime` 包；`ctx.slots`（SlotRegistry）改由 `@deepseek-ai/dsh-client-ui-renderer` 提供（ui-renderer/src/client/registry.ts）。不改则客户端模块（上传按钮、剪贴板按钮、设置卡片）无法组合进 boot 图。

## 已核对兼容、无需改动的插件 API（rc.2 = alpha.1）

- `dsh.bundle.patch: ./cordis.patch.yml` 机制 ✓
- cordis patch 语法 `- insert: [{id, name}]` ✓
- `ctx.inject(['webServer'], wctx => wctx.webServer.register(route))` ✓（签名不变，alpha.1 仅新增 gzip 配置项）
- `sctx.settings.register(ns, z.object(...))` + `scope.get()/watch()` ✓
- 客户端 `ctx.slots.inject/register`、`ctx.settingsScope.bind` ✓
- `window.__ModuleLoader__.load({id, factory})`（lazy-CJS）✓
- `dsh plugin --profile web add <package>` CLI ✓
- `@deepseek-ai/cordis@4.0.1`、`@deepseek-ai/schemastery@3.18.1` 不变 ✓

## 其他环境性注意（非插件代码）

- **一次性 token 鉴权**：alpha.1 起远程访问 Web 界面需要 launch URL 里的一次性 token；`localhost` 不受影响。
- **安装途径**：npm 暂无 alpha.1，升级需 `npm i -g github:deepseek-ai/deepseek-harness#dsh-v0.1.2-alpha.1` 或等 npm 发布；`dsh plugin` 命令面不变。
- **重命名台账**：仓库名 / npm 包名 / CLI 命令均无变化（`deepseek-ai/deepseek-harness`、`@deepseek-ai/dsh`、`dsh`）；npm 上存在同名冒名包 `deepseek-harness`、`dsh`（unscoped），安装务必使用 `@deepseek-ai/dsh`。

## alpha.1 实测补充（2026-08-29，源码版 D:\DeepSeek Harness @ cd5ef81481）

- `@deepseek-ai/dsh-settings` 在 alpha.1 为 **0.1.2-alpha.1**；本插件 peer 范围已加 `|| ^0.1.2-alpha.1`。运行时 host 导入由 dsh 加载器按 **anchor（dsh 安装目录）→ profile** 顺序解析——profile 里没有 dsh-settings 也能工作，走 anchor 的 workspace 副本。
- **未知 `/api/*` 路径返回 401 "unauthorized"**（alpha.1 的 API 鉴权门，非 404）。插件路由必须注册成功才可达；升级后如某路由 401，先确认插件 bundle 已进 boot 图并重启。
- 客户端 `dsh.client.inject` 的边是**信息性**约束（ui-workspace 注释原文），名字写错只告警不崩溃；但 alpha.1 里 `dsh-client-runtime` 节点已不存在，务必换 `dsh-client-ui-renderer`。

## 本插件 v0.2.0 新增 API（均为可选、向后兼容）

- `GET /api/file-upload/files?kind=all|image|file|folder&page=&pageSize=` → 附件库列表（分页、mtime 倒序）
- `DELETE /api/file-upload/files`（body `{path}`）→ 删除附件库内文件/空文件夹（路径含附件库校验）
- `POST /api/file-upload/clipboard-paths` → 读系统剪贴板文件路径列表（Windows FileDropList / macOS furl / Linux uri-list）
- 辅助导出：`parseOriginalName`、`listAttachmentFiles`、`readClipboardPaths`（供测试/复用）
