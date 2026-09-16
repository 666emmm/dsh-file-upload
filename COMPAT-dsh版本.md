# dsh 版本兼容说明（dsh-file-upload v0.2.1）

> 依据：`T1-dsh版本差异报告.md` + 对源码版 checkout 的实测核对。

## 当前目标环境

- dsh：**`dsh-v0.1.5-rc.2`**（源码版 `D:\DeepSeek Harness 0.1.5-rc.2` @ `fb2c4b9e69`）
- 本插件：**v0.2.1 已适配**。⚠️ **v0.2.0 在 0.1.5-rc.2 上完全无法加载**（见下），请用 v0.2.1+

## DSH 0.1.5-rc.2 的三处破坏性变更（v0.2.1 已修）

| 项 | 0.1.2-alpha.1 及更早 | 0.1.5-rc.2 | 本插件处理 |
|---|---|---|---|
| settings 注册辅助函数 | `import { settingsNamespace } from '@deepseek-ai/dsh-settings'` | **该导出已移除**（包只导出 `SettingsConflictError` / `SettingsProvider`(default) / `redactSecrets`） | 命名空间改为字面量 `'file-upload'`；注册/读写继续用 provider 级 `settings.register(NS, schema)` / `settings.get(NS)` / `settings.update(NS, patch)`（0.1.5-rc.2 中仍是公开契约） |
| 官方草稿附件 API | `conversation.createDraftImages(files)` + `inputActions.addImages(ids)`（回滚 `releaseDraftImages`） | `conversation.createDrafts(sessionId, files)` + `inputActions.addAttachments(ids)`（回滚 `releaseDraftAttachments`） | 客户端**两代自动兼容**（存在性检查选路），图片仍进官方附件条 |
| loader 条目 id | `file-upload` 可用 | DSH 自带 `@deepseek-ai/dsh-client-file-upload` **已占用 `file-upload`** | 本插件条目 id 改为 **`file-upup`**（settings 命名空间与 HTTP 路由保持 `file-upload` 不变，避免孤立已有用户设置） |

### 已实测确认在 0.1.5-rc.2 中未变

- `webServer.register({ kind: 'exact', path, handler })` ✓（重复 path 抛错，语义不变）
- 服务名 `webServer` / `sessions` / `settings` / `attachments` / `llm` ✓
- 客户端槽位 `conversation.input.left`（list/session）、`settings.section`、`settings.plugin.item` ✓
- 客户端 inject 名 `@deepseek-ai/dsh-api-remotes` / `dsh-client-ui-renderer` / `dsh-client-ui-conversation` ✓
- `@deepseek-ai/schemastery@3.18.2`（满足 peer `^3.18.1`）✓
- 官方 `@deepseek-ai/dsh-client-file-upload` 是**上传基础设施服务**（Blob/流式接收 + staged receipt），不注册同名 settings 命名空间、也不占 `conversation.input.left`，与本插件无功能冲突 ✓

### 本次验证方式

`lib/index.js` 冒烟测试（链接真实 0.1.5-rc.2 的 schemastery 3.18.2）：模块导入 ✓ → `apply()` 注册 6 条路由 ✓ → `settings.register('file-upload', schema)` 收到合法 schema ✓ → `GET /api/file-upload/config` 200 且读到 settings 值 ✓ → `GET /api/file-upload/files` 200 列表正确 ✓

---

## 历史版本兼容（0.1.1-rc.2 / 0.1.2-alpha.1）

v0.2.0 在 **0.1.1-rc.2** 上直接可用、在 **0.1.2-alpha.1** 上需下述 inject 改动（v0.2.0 已内含）。

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
