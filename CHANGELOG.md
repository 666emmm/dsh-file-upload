# Changelog

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
