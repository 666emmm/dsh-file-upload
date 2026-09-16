# Changelog

## v0.2.1 — 适配 DSH 0.1.5-rc.2

### 修复（不兼容变更：v0.2.0 在 0.1.5-rc.2 上完全无法加载）

- **host 导入失败**：`@deepseek-ai/dsh-settings` 自 0.1.5 起不再导出 `settingsNamespace`，v0.2.0 的 `import { settingsNamespace }` 直接抛
  `SyntaxError: The requested module ... does not provide an export named 'settingsNamespace'`，插件整个装不起来。
  现在命名空间就是字面量字符串（`'file-upload'`），注册/读写继续走 provider 级 API（`settings.register(NS, schema)` /
  `settings.get(NS)` / `settings.update(NS, patch)`，在 0.1.5-rc.2 中仍然是公开契约）。
- **官方附件条 API 改名**：0.1.5 起 `conversation.createDraftImages()` + `inputActions.addImages()` 变为
  `conversation.createDrafts(sessionId, files)` + `inputActions.addAttachments(ids)`（回滚方法相应变为
  `releaseDraftAttachments()`）。客户端现在**两代都兼容**，按存在性自动选择，图片仍能进官方附件条。
- **loader 条目 id 冲突**：DSH 自带 `@deepseek-ai/dsh-client-file-upload` 已占用 loader 条目 id `file-upload`，
  重复 id 会让整个 profile 启动失败。本插件条目 id 改为 **`file-upup`**（settings 命名空间与 HTTP 路由故意保持
  `file-upload` 不变，避免孤立已有用户设置）。此改动见 v0.2.0 之后的提交。

### 其他

- `peerDependencies` 移除 `@deepseek-ai/dsh-settings`：代码已不再导入该包（只使用 dsh 提供的 `settings` 服务），
  且其预发布版本号无法用常规 semver 范围覆盖 0.1.5-rc.2。

## v0.2.0（2026-08-29）— fork 首发

基于 [a903067276-rgb/dsh-file-upload](https://github.com/a903067276-rgb/dsh-file-upload) v0.1.7（MIT）扩展。

### 新增

- **已上传文件管理**：设置卡片「文件上传」底部新增「已上传文件」区块——按 全部/图片/文件/文件夹 分类分页浏览附件库（`images/` 与 `files/` 子树，按修改时间倒序），显示原始名/大小/时间/相对路径；一键复制 `@路径` 或删除（二次确认 + host 端路径包含校验，防目录穿越）。
- **剪贴板读真实路径（零拷贝）**：文件管理器复制文件/文件夹后，点输入框上传按钮旁的剪贴板按钮 → host 读系统剪贴板真实路径列表（Windows `Get-Clipboard -Format FileDropList` / macOS osascript furl / Linux xclip|wl-paste）→ 以 `@路径`/`@"路径"` 插入草稿。不传内容、不落盘、无大小限制，模型按地址现读。
- **粘贴纯路径文本 → `@引用`**：「监听剪贴板 = 全部文件」时，粘贴的纯绝对路径文本自动转成 `@引用`。

### 修复

- Windows「打开文件夹」误报失败：`explorer.exe` 成功打开时退出码也常为 1（单实例委派），已容忍该退出码。

### 兼容性

- dsh **0.1.1-rc.2**：直接可用，无需改动。
- dsh **0.1.2-alpha.1+**：`dsh.client.inject` 已适配（`@deepseek-ai/dsh-client-runtime` → `@deepseek-ai/dsh-client-ui-renderer`），详见 `COMPAT-dsh版本.md`。

## v0.1.7 及更早

原仓库历史版本，见 https://github.com/a903067276-rgb/dsh-file-upload 。
