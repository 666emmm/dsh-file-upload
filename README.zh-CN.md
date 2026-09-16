# dsh-file-upload ⬆️

[English](README.md) | [简体中文](README.zh-CN.md)

![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)

[![Awesome DSH Plugin](https://awesome-dsh-plugin.com/badge.svg)](https://awesome-dsh-plugin.com)

**统一上传按钮 + 拖拽文件直进对话** —— DeepSeek Harness（dsh）web 插件。

*非官方项目：社区成员独立开发维护，非 DeepSeek 官方产品。*

> **Fork 自 [a903067276-rgb/dsh-file-upload](https://github.com/a903067276-rgb/dsh-file-upload)**（MIT）。v0.2.0 新增**已上传文件管理**与**剪贴板读真实路径（零拷贝）**——见下方 [v0.2.0 新增](#v020-新增本-fork)。

## 截图

输入框工具行里的上传图标按钮（官方 dsw 风格，跟随深浅色主题）；支持的图片进入官方附件条（自动 file_id 复用），其他文件把路径文本插入输入框。

## 功能

| 操作 | 效果 |
|---|---|
| 点上传图标 | 系统文件选择器（可多选）→ 智能分流 → 附件条和/或路径文本 |
| 把文件拖进窗口 | 图片/任意文件都接管（不再弹"不支持"）→ 智能分流 |
| 把文件夹拖进窗口 | 递归读取整个文件夹 → 在附件库中按原结构重建 → 只插文件夹路径一行（不展开内部文件） |
| **图片**（PNG/JPEG/WebP/GIF） | ① 留档到**附件目录**（按天分文件夹）② 进**官方草稿附件条** → 发送后自动 DeepSeek Files API `file_id`（同图复用，7 天过期自动重传） |
| **其他文件** | 存附件库 `files/` 子目录，`<前缀> 路径` 文本进草稿 |
| 模型不支持图片 | 图片降级为"留档 + 路径文本"（绝不发送图片块，不报 400） |
| **已上传文件管理** | 设置卡片内直接浏览附件库（图片/文件/文件夹分类、分页），显示原始名/大小/时间，一键复制 `@路径` 或删除 |
| **复制文件 → 取真实路径（零拷贝）** | 文件管理器里复制文件后，点输入框旁的剪贴板按钮 → 读系统剪贴板真实路径 → 以 `@引用` 插入，不传内容、不落盘、无大小限制；粘贴纯路径文本也自动转 `@引用`（监听剪贴板=全部文件档） |

- 文件上限：单文件 **64MB**（DeepSeek Files API 硬限；主程序附件库默认 20MB，见 *大图（20~64MB）*）
- 所有上传统一进**附件库**：图片 `~/Documents/DSH/Attachments/images/<YYYY-MM-DD>/`，其他文件 `.../files/<YYYY-MM-DD>/`（设置里可改根目录）
- 上传中按钮变灰，失败有中文提示

## v0.2.0 新增（本 fork）

- **已上传文件管理**：设置卡片「文件上传」底部新增「已上传文件」区块——按 全部/图片/文件/文件夹 分类分页浏览附件库，显示原始名/大小/时间/相对路径，一键复制 `@路径` 或删除（二次确认 + host 端路径包含校验）。
- **剪贴板读真实路径（零拷贝）**：文件管理器复制文件/文件夹后，点输入框上传按钮旁的剪贴板按钮——host 读系统剪贴板真实路径列表（Windows FileDropList / macOS furl / Linux uri-list），以 `@路径`/`@"路径"` 插入草稿。不传内容、不落盘、无大小限制，模型按地址现读。
- **粘贴路径文本 → `@引用`**：「监听剪贴板 = 全部文件」时，粘贴的纯绝对路径文本自动转成 `@引用`。
- **修复**：Windows「打开文件夹」不再误报失败（explorer.exe 成功时退出码也是 1）。

## 设置卡片

- **附件目录**（默认 `~/Documents/DSH/Attachments`，支持 `~` 前缀）—— 只给图片留档用
- **路径前缀**（默认 `[上传文件]`）—— 插入输入框时加在路径前的文本，清空 = 裸路径
- **图片走官方附件**（默认开）—— 关 = 图片走老路径文本逻辑
- **留档图片到附件目录**（默认开）—— 关 = 只走官方附件（省磁盘；官方通道不可用时仍强制留档）
- **允许公网上传**（默认关）—— 关 = 同源校验保持仅放行本机（防 CSRF）；开 = 放行任意来源，供公网/内网穿透域名访问（如 ddnsto）。仅在信任所有能访问到你 DSH 的人时开启。
- **监听剪贴板**（默认「关」，保持官方内建粘贴；可选 关 / 只监听图片 / 全部文件）—— 粘贴接管档位。关 = 官方内建粘贴；只监听图片 = 截图/复制图片时由本插件接管（模型不支持识图也能贴图，自动降级为路径文本）；全部文件 = 粘贴任何文件都接管（落盘附件库 + 路径文本进草稿）。改档无需刷新页面，保存即生效。
- 只读显示：当前图片来源上限（来自宿主配置）

## 大图（20~64MB）

DeepSeek 单图上限 **64 MB**；主程序本地附件库默认 **20 MB**——超过且 ≤64 MB 的图片走"留档 + 路径文本"（模型仍可经 `read_image` 读，上限同源跟随）。

想让大图也走官方附件路径，在 `~/.dsh/profiles/web/cordis.patch.yml` 追加以下配置并重启 `dsh web`：

```yaml
- id: attachment-local
  config:
    maxImageBytes: 67108864   # 20 MiB → 64 MiB（DeepSeek 官方硬限）
```

注意：该行整行替换配置，需要的键都要写明；主程序升级后随版本核对。无论原图多大，模型看到的始终是主程序规范化版本（≤2048px / ≤4 MiB，每张图 ≤384 token）。

## 安装

官方 bundle 一行安装：

```sh
dsh plugin --profile web add "github:666emmm/dsh-file-upload#main"
```

装完重启 `dsh web`（bundle 层在启动时合成）。需要 pnpm（`dsh plugin` 是 pnpm 转发器）。

手动挂载（兜底）：见 [docs/install.md](docs/install.md) —— 软链到 `~/.dsh/profiles/web/node_modules/` + 在 `~/.dsh/cordis.patch.yml` 里加**单条** entry（双条会让插件 apply 两次、路由重复注册崩溃），然后重启。

> **条目 id 说明**：本项目注册的 loader 条目 id 是 **`file-upup`**，不是 `file-upload`。DSH 自带的
> `@deepseek-ai/dsh-web-app` 已经占用了 `file-upload`（指向 `@deepseek-ai/dsh-client-file-upload`），
> 而加载器的条目 id 是扁平、无命名空间的：两行同 id 会让加载器在挂载时报
> `duplicate loader entry id: "file-upload"` 并让整个 profile 起不来，市场的更新前试启动校验也会因此回滚。
> 手动挂载时请照抄 `cordis.patch.yml` 里的 `id: file-upup`。设置命名空间与 HTTP 路由仍保留 `file-upload`
> 字样（与条目 id 无关，改名会丢掉用户已有设置）。

## 使用

1. 点上传图标选文件（可多选），或把文件/文件夹拖进窗口任意位置。
2. **图片**（当前模型支持看图时）：留档附件目录 + 进入官方附件条——发送即模型看图（DeepSeek Files API `file_id`，自动复用）。
3. **图片但模型不支持**（或你关了官方路径）：留档到 `images/` 后写入 `<前缀> <绝对路径>` 行——如 `[上传文件] /path/to/Attachments/images/xxx.png`——保留已有草稿。
4. **其他文件**：存附件库 `files/`，路径文本进草稿；发送后模型按路径读取。
5. **文件夹**：拖入后在附件库按原结构重建，只把文件夹路径（一行）写进草稿。

## 已上传文件管理

设置卡片底部新增「已上传文件」区块：

- 按 **全部 / 图片 / 文件 / 文件夹** 分类分页浏览附件库（`images/` 与 `files/` 子树），列表按修改时间倒序；
- 每行显示**原始文件名**（自动从 `<时间戳>-<uuid8>-<原名>` 解析）、大小、修改时间、相对路径；
- 「复制@」把 `@路径`（含空格自动 `@"路径"`）拷到系统剪贴板，可直接粘进输入框让模型按地址读取；
- 「删除」直接删除附件库中的文件（空文件夹也可删；非空文件夹需先清空）。删除有二次确认；宿主端校验路径必须位于附件库内（防目录穿越）。

## 复制文件 → 读取真实路径（零拷贝）

浏览器拖拽只能拿到文件**内容**，永远拿不到原路径——所以本插件新增「剪贴板路径」按钮（输入框工具行、上传按钮旁边）：

1. 在文件管理器/桌面**复制**文件或文件夹（Windows 资源管理器「复制」）；
2. 在 dsh 里点剪贴板按钮 → host 读系统剪贴板 **FileDropList**（真实路径列表，不是字节）；
3. 以 `@路径` / `@"路径"` 插入输入框——**不传内容、不落盘、无大小限制**，模型用 `read`/`glob` 按地址现读。

配套：设置里「监听剪贴板 = 全部文件」时，粘贴的**纯路径文本**（如复制的路径）也会自动转成 `@引用`。

平台支持：Windows 用 PowerShell `Get-Clipboard -Format FileDropList`（完整支持，含中文路径）；macOS 用 osascript furl（尽力支持）；Linux 用 xclip / wl-paste 的 text/uri-list。

## 平台支持

| 平台 | 状态 |
|---|---|
| macOS | ✅ 完整测试（开发环境） |
| Linux | ✅ 预期可用（纯 Node 实现），未测 |
| Windows | ⚠️ 预期可用（纯 Node 实现、Windows 安全文件名清洗、平台分隔符路径），未测 |

## 版本兼容（dsh 升级注意）

对 dsh **v0.1.2-alpha.1 及更高版本**：`@deepseek-ai/dsh-client-runtime` 包已被移除，`ctx.slots` 改由 `@deepseek-ai/dsh-client-ui-renderer` 提供——**升级后必须把 `package.json` 里 `dsh.client.inject` 列表中的 `@deepseek-ai/dsh-client-runtime` 替换为 `@deepseek-ai/dsh-client-ui-renderer`**，否则客户端模块（按钮/设置卡片）无法加载。

其余插件 API（`dsh.bundle.patch`、cordis patch 语法、`webServer.register`、`settings.register`、`slots`、`settingsScope`、`__ModuleLoader__.load`）在 alpha.1 均保持兼容；当前 **0.1.1-rc.2 无需任何改动**。详细核对清单见 `COMPAT-dsh版本.md`。

## 依赖要求

- DSH web >= 0.1.0-rc.7（`dsh web` 运行）
- **版本对照**（尽力兼容——新功能已在本地 0.1.1-rc.2 与 0.1.2-alpha.1（源码版）实测；0.1.0-rc.7/rc.8 上的官方附件条无法完整验证，**不保证**）：
- **维护策略**：本插件将持续跟随 DSH 最新版本演进；对旧版 DSH 的兼容仅是尽力而为、不保证长期有效。

| 你的 DSH 版本 | 装这个 | 说明 |
|---|---|---|
| 0.1.2-alpha.1+ | `main`（v0.2.0+） | 本 fork 版本——client inject 已用 `@deepseek-ai/dsh-client-ui-renderer`（见 `COMPAT-dsh版本.md`） |
| 0.1.1-rc.1 – 0.1.1-rc.2 | `main`（v0.2.0+） | 全功能（含官方附件条） |
| 0.1.0-rc.7 – 0.1.0-rc.8 | `main`（v0.1.5+） | 正常；官方附件条自动降级为路径文本（除非会话模型收图）。保守回退：`v0.1.4` — `dsh plugin add github:a903067276-rgb/dsh-file-upload#v0.1.4` |
| 0.1.0-rc.6 及更早 | `v0.1.2` — `dsh plugin add github:a903067276-rgb/dsh-file-upload#v0.1.2` | 最后一个无设置卡片的版本（设置卡片用 rc.7+ keyed slot 契约） |

- 除剪贴板外纯 Node：保存/文件夹重建/列表/删除/设置全走 `node:fs`；剪贴板读路径会调用平台剪贴板命令（PowerShell `Get-Clipboard -Format FileDropList` / osascript / xclip|wl-paste）——这是已知的平台限制。

## 工作原理

- **Host**（`lib/index.js`）：`POST /api/file-upload/save`——校验会话与大小，用**纯 Node** 写 base64 到 `<附件库>/images/<YYYY-MM-DD>/`（`mode=image`）或 `<附件库>/files/<YYYY-MM-DD>/`（`mode=file`）；`POST /api/file-upload/save-folder`——接收相对路径 + base64 列表，在附件库 `files/<日期>/<时间戳>-<文件夹名>/` 下按原结构重建（逐段 sanitize + 拒绝 `..` 防目录穿越）；`GET/POST /api/file-upload/config`——读写设置（官方 settings 服务）+ 暴露宿主图片上限 + 当前会话模型是否收图（`llm.resolveModel` 的 `inputModalities`，与适配器同源）。
- **文件管理与剪贴板（v0.2.0）**：`GET/DELETE /api/file-upload/files` 列出（分页、分类）并删除（含路径校验）附件库；`POST /api/file-upload/clipboard-paths` 读系统剪贴板真实路径（Windows FileDropList / macOS furl / Linux uri-list）→ 以 `@引用` 插入，零拷贝。
- **Client**（`lib/client.js`）：上传图标挂 `conversation.input.left`；捕获阶段接管拖拽，`webkitGetAsEntry` 递归读入文件夹目录树；分流规则：支持图片 + 开关开 + 模型收图 + 不超宿主上限 → 留档 + `conversation.createDraftImages` + `inputActions.addImages`（官方 InputBar 同款机制）→ 官方附件条（不写路径文本）；其余降级"留档 + 路径文本"；>64MB 拒绝并提示。
- **错误边界**：渲染崩溃降级为"⚠ 上传组件异常"小图标，不卸载整个输入框。

## 备注

- 附件库只增不减，**从不自动清理**（我们不删你的文件）——需要时手动清理。
- 改插件后重启 `dsh web` 生效（client 改动刷新页面即生效；host 改动需重启）。

## 为什么有这个插件

DSH 原生在模型不支持图片时会直接拒绝拖入的图片。本插件在模型能看图时把图片走**官方附件路径**（搭 DeepSeek Files API 的 `file_id` 快车），同时留一份**你能自己找到的附件目录**副本，其余情况降级为纯**路径文本**——纯文本消息能过模型的图片检查，任何模型/视觉插件都能用（降级路径上从不提交图片块，绕开 DSH 原生拒绝）。

## License

[MIT](LICENSE)
