# @local/dsh-session-archive-vault — 归档会话（把垃圾盖起来）
注意注意我搞错了，直接下载压缩包就行，其他文件包括readme都在压缩包里，第一次上传的时候忘了压缩导致两个文件夹没上传上

把「归档」真正变成**收起来**：归档后的会话不再出现在左侧的工作区 / 未分组列表里，
同时在**设置**里新增一个「归档会话」页面，按会话归档前所属工作区分组、标注最后对话时间，
可以**撤销归档**，也可以**查看它在磁盘上的真实存放目录**（用于手动删除）。

**本插件从不删除、移动或改写任何会话文件。** 它只读目录列表与文件大小。

---

## 1. 它做什么

| 行为 | 说明 |
|---|---|
| 归档即隐藏 | 侧栏「视图选项」里内置的三选一（隐藏已归档 / 全部对话 / 仅显示已归档）被强制纠正为「隐藏已归档」，归档会话因此不会出现在工作区树或未分组列表中 |
| 设置 → 归档会话 | 新页面，按归档前所属工作区分组（组头显示工作区名与其目录路径），未归属任何工作区的归入「未分组」 |
| 最后对话时间 | 每行显示会话摘要的最近活动时间（与侧栏「Last updated」同源），取不到时显示「未知」 |
| 撤销归档 | 调用平台自带的 `ctx.uiWorkspace.unarchiveSession()`，会话按原位置回到侧栏 |
| 查看文件地址 | 按需向宿主请求，返回**实际扫描到**的会话目录、文件数、占用大小，可一键复制路径 |

## 2. 它特意不做什么

- **不删文件。** 归档不是删除，聊天记录仍完整留在本机（`<DSH_HOME>/sessions/...`）。要彻底清除，请用页面给出的目录手动删除。
- **不做批量操作。** 撤销归档是逐行确认的动作。
- **不碰子代理会话。** 那些会话本来就由平台隐藏、也不可归档，不在归档集合里。

---

## 3. 安装（当前**尚未安装**，需要时再执行）

### 方式 A：插件管理器（推荐）

```
plugin_manager(action="install_bundle", target="D:\\path\\to\\dsh-session-archive-vault")
```

它会把这个目录登记进 profile 的 `node_modules` 与 `dsh.profile.bundles`，无需手工编辑。

### 方式 B：手工登记（等价，便于审查）

1. 在 `%DSH_HOME%\profiles\<profile>\package.json` 的 `dependencies` 里加一行：

   ```json
   "@local/dsh-session-archive-vault": "link:D:/path/to/dsh-session-archive-vault"
   ```

2. 在同一个文件的 `dsh.profile.bundles` 数组里加上：

   ```json
   "@local/dsh-session-archive-vault"
   ```

3. 在该 profile 目录执行一次安装（使用随包提供的 pnpm / node）：

   ```
   <DSH 安装目录>\resources\runtime\bin\node.cmd <DSH 安装目录>\resources\runtime\pnpm\bin\pnpm.cjs install
   ```

   （工作目录：`%DSH_HOME%\profiles\<profile>`）

4. 重启 DSH（新增依赖与新增 Loader 行通常需要重启；`patchReload: live` 只对已存在行的配置改动生效）。

本包**没有任何 npm 依赖**，`link:` 登记后不需要额外解析第三方包。

### 兼容性

- `dsh.manifestVersion = 1`；在 `0.1.7-rc.2` 上按源码逐项核对过所用扩展缝（见第 6 节）。
- Host 半只用 `node:fs/promises`、`node:os`、`node:path`；浏览器半只 require 基线模块 `react`（外加内置的 `@deepseek-ai/dsh-client-ui-primitives` 依赖策略，实际未使用）。

## 4. 卸载

方式 A 用插件管理器移除；方式 B 反向做两步即可：从 `dsh.profile.bundles` 与 `dependencies`
里删掉这一项，再跑一次 `pnpm install`（或直接删掉 `node_modules\@local\dsh-session-archive-vault` 链接），重启。

想**临时**关掉而不卸载：在 profile 的 `cordis.patch.yml` 里加一行

```yaml
- id: session-archive-vault
  disabled: true
```

## 5. 配置（`cordis.patch.yml` 里的 `config`）

| 字段 | 默认 | 含义 |
|---|---|---|
| `sessionsRoot` | 空 → `<DSH_HOME>/sessions` | 会话日志根目录。若 profile 把 `session-persistence-jsonl` 的 `root` 改到别处，必须在这里填同一个路径，否则页面只能显示「未找到本地文件」 |
| `enforceHideArchived` | `1` | `1` = 强制隐藏归档会话（纠正内置视图选项）；`0` = 只装页面，隐藏与否交还内置三选一 |

---

## 6. 用到的扩展缝（已逐条核对源码）

| 用途 | 接口 | 出处 |
|---|---|---|
| 设置页 | `ctx.slots.inject('settings.section', () => ctx.slots.register({ name, id, order, label, locale }, Component))`；组件收到 `close` + `inject` 面 + `t` | `dsh-client-ui-agent-preset` / `dsh-client-ui-settings-plugins` 的实际注册代码 |
| 归档集合 | `ctx.uiWorkspace.workspaces.list.getSnapshot()` → `{ items, archivedSessionIds, pinnedSessionIds }` | `dsh-client-ui-workspace/lib/client.js` |
| 会话摘要 | `ctx.uiWorkspace.sessions.list.getSnapshot()` → `{ ids, byId }`，摘要含 `displayTitle` / `updatedAt` / `cwd` | 同上 |
| 撤销归档 | `ctx.uiWorkspace.unarchiveSession(sessionId)` | 同上（侧栏自身就是这么调用的） |
| 强制隐藏 | 内置视图 store 的写集 `ctx.uiWorkspace.view.setArchivedFilter('default')`；持久化键为 `dsh.workspace.view*`（值即 state 本体 JSON） | `WorkspaceViewStore` 定义 + `dsh-client-store` 的 `localStorage` 整值持久化 |
| 宿主只读接口 | `ctx.connection.fetch.register({ path, methods, requestBody, fetch })`，浏览器用**文档相对路径** `api/...` 调用 | `dsh-client-connection` 的包说明；`dsh-client-file-upload` 的同款用法 |
| 会话文件定位 | 直接扫描 `sessions/*/<sessionId>`，不重写存储内部的 cwd 编码 | `dsh-session-persistence-jsonl` 的磁盘布局说明 |

## 7. 离线验证（不需要安装、不需要浏览器）

```
node tools/check-manifest.mjs   # 2 项：package.json 结构 + cordis.patch.yml 真实 YAML 解析
node tools/check-host.mjs       # 7 项：路由、配置、输入校验、真实会话定位
node tools/check-browser.mjs    # 12 项：bundle 契约、页面注册、隐藏守卫、分组渲染、撤销归档、地址揭示
```

共 21 项，全部通过。清单检查是必要的：bundle patch 的 YAML 若解析失败会直接影响 profile 启动，
所以这里用真实 YAML 解析器校验，并核对被引用的每个文件存在、且本包保持零 npm 依赖（`link:` 安装的前提）。

`check-host.mjs` 会拿本机**真实存在**的会话做端到端定位校验（实测已通过，例如
`session-xxxxxxxx-... → %DSH_HOME%\sessions\--C-Users-you-Desktop-myproject--\session-xxxxxxxx-...`）。
`check-browser.mjs` 在 `node:vm` 里用最小 React/window/localStorage 宿主加载真实 bundle，并驱动
「渲染 → 点击 → 重渲染」，因此页面逻辑是被真实执行过的，而不是只做静态检查。

## 8. 已知限制与风险（诚实清单）

1. **强制隐藏会覆盖你自己的视图选项。** 选了「全部对话（显示已归档）」也会在 1.5 秒内被纠正回「隐藏已归档」；这就是「盖起来」的代价。不想要就设 `enforceHideArchived: 0`。
2. **依赖若干内部契约**（插槽键 `settings.section`、`ctx.uiWorkspace` 的字段、`dsh.workspace.view.v5` 持久化键）。DSH 升级若改名：最坏结果是页面不再出现或强制隐藏失效——**不会损坏数据**，因为插件完全不写会话数据。
3. **云会话后端不适用。** 若组合了 `dsh-session-log-deepseek` 之类的远端存储，会话可能不在本机，页面会显示「未找到本地文件」。
4. **手动删除有前提。** 删除目录前请确认 DSH 未打开该会话并已退出，否则文件可能被占用（Windows 上 JSONL 后端对每个会话持写锁）。
5. **未归档会话不在本页**，本页只反映归档集合。
6. **删除不可逆**：本插件不提供回收站；页面只负责告诉你路径。

## 9. 文件

```
dsh-session-archive-vault/
  package.json          # dsh.bundle.patch + dsh.client(platform web)
  cordis.patch.yml      # 一条 Loader 行 + 配置
  lib/index.js          # 宿主半（只读定位接口）
  lib/client.js         # 浏览器半（设置页 + 隐藏守卫）
  tools/check-manifest.mjs  # 清单与 bundle patch 校验
  tools/check-host.mjs      # 宿主半离线检查
  tools/check-browser.mjs   # 浏览器半离线检查
```
