# 手动挂载（备选安装方式）

> 推荐方式是一行命令：`dsh plugin --profile web add "github:666emmm/dsh-file-upload#main"`
> （需要 pnpm 在 PATH 上；`dsh plugin` 会转发给 pnpm）。
> 以下手动方式仅在你无法使用 `dsh plugin` 时使用。

## 步骤

1. 把插件放入 profile 的 `node_modules`（Windows 可用目录联接，避免复制）：

   ```sh
   # Windows（在 ~/.dsh/profiles/web 下）
   cmd /c mklink /J "C:\Users\<你>\.dsh\profiles\web\node_modules\dsh-file-upload" "D:\path\to\dsh-file-upload"
   ```

2. 在 `~/.dsh/cordis.patch.yml` 加入**一条**插入（多写一条会导致插件 apply 两次、路由重复注册崩溃）：

   ```yaml
   - insert:
       - id: file-upup
         name: dsh-file-upload
   ```

3. 重启 `dsh web`（bundle 层在启动时合成）。

## 注意

- **条目 id 必须用 `file-upup`，不要用 `file-upload`**：DSH 自带的 `@deepseek-ai/dsh-web-app`
  已经注册了 id `file-upload`（`@deepseek-ai/dsh-client-file-upload`）。加载器的条目 id 是扁平的、
  无命名空间，同 id 会让加载器在挂载时报 `duplicate loader entry id: "file-upload"` 并让整个
  profile 起不来；市场的“试启动校验”也会因此回滚任何更新。
- 保持插件目录可访问（不要移动/删除）；
- 升级代码时把新文件覆盖进插件目录，重启即可（client 改动刷新页面即可生效）。
