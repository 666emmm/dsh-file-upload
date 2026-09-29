# Changelog

## v0.3.0 — 按光标插入 + 适配 DSH 0.2.0-rc.1

### 修复（会丢用户输入的老 bug）

- **点「读取剪贴板路径」不再清空输入框**。根因：会话插槽的标准 props 只有
  `{ useConversation, useInput, inputActions }`（`ui-conversation/apply.ts` 的
  `uiSession.provide({ hooks: ['conversation','input'], props: ['inputActions'] })`），
  **没有 `input` 对象**；旧代码读 `props.input?.draft` 恒得空串，于是"追加"退化成
  `setDraft(路径)` —— 而 `setDraft` 是**整篇替换**（Lexical `root.clear()` + 逐行重建 +
  `selectEnd()`），已输入内容被清掉、光标跳到末尾。
- 改用官方 `useInput` 钩子读实时草稿；上传按钮、文件夹上传、粘贴接管三处一并修正
  （此前它们有同样的缺陷，只是常在空输入框下使用而不易察觉）。

### 新增

- **按光标插入**：读 composer（`[data-composer-input]`）内的真实选区，把路径文本插到
  光标处（有选区则替换），并把光标还原到插入内容之后；读不到光标时退化为"追加到末尾"，
  两条路径都**不会丢文本**。
- 两个按钮 `mousedown` 阻止默认行为，避免点击抢走 composer 的焦点与选区。
- 检测到草稿里已有 `@引用` chip（Lexical decorator）时自动跳过按光标插入
  （DOM 字符偏移与 draft 投影不一致），改为追加——宁可退化也不误插。

### 兼容 DSH 0.2.0-rc.1

- host 侧 `settings` 服务在 0.2.0-rc.1 已换成 **`SettingsForms`**（配置文档编辑器），
  **不再有 `register(ns, schema)` / `get(ns)`**。现改为能力探测：
  - 有命名空间注册表（≤0.1.x）→ 照旧 `register/get/update`，行为完全不变；
  - 没有（≥0.2.0-rc.1）→ 跳过注册 + 导出 `export const Config`，读取走 loader 注入的
    条目 config，写入落到 `$DSH_HOME/file-upload-settings.json`（条目 config 打底、
    自管文件覆盖它显式保存过的键）。
  这同时消除了"升级到 0.2.0-rc.1 后 `apply` 抛 TypeError、插件甚至整个 profile 起不来"的风险。
- 客户端依赖的 `useInput` / `inputActions.setDraft` / `[data-composer-input]` 在 0.2.0-rc.1 实测未变。

### 验证

- 客户端：Node + 最小 DOM/React 桩实测 4 用例 —— 光标中间插入（光标还原到 2+插入串长度）、
  末尾插入、选区替换、无选区降级追加；原文本零丢失。
- host：两种 settings 形态各跑一遍 —— 注册表形态（注册/读取/写入照旧）、SettingsForms 形态
  （apply 不抛错、读条目 config、写自管文件并回读）。

## v0.2.2 — 设置入口去重

### 变更

- **不再注册 `settings.plugin.item`**：官方「插件」设置页的卡片列表是**两个账本的交集**——Host 提供的 settings 命名空间 ∩ 注册进该槽位的卡片（`ui-settings-plugins/src/client/tab-store.ts` 原注释：*"A served namespace no card claims renders nothing"*）。之前本插件两边都注册，于是「插件」页里出现了一张与独立分区完全重复的卡片。
  现在**只保留独立分区** `settings.section`（label「文件上传」，order 40），「插件」页不再显示本插件的配置卡片。
- settings 命名空间 `file-upload`、HTTP 路由 `/api/file-upload/*` 与已保存的用户设置**均不变**（卡片少了，配置与数据一个没动）。

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
