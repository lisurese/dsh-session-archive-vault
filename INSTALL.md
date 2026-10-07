# 安装说明 — dsh-session-archive-vault（归档会话）

一个 DSH 插件：让「归档」真正变成**收起来**。

- 归档后的会话**不再出现在左侧的工作区 / 未分组列表**里；
- 左下角**设置**里新增「**归档会话**」页：按归档前所属工作区分组、标注最后对话时间、可**撤销归档**、可**查看会话在磁盘上的真实目录**（用于手动删除）。
- **本插件从不删除、移动或改写任何会话文件**，只读目录列表与文件大小。

---

## 一、安装（三选一）

### 方式 1：tarball（最省事）

1. 打开 DSH → 左下角**设置** → 侧栏 **Plugins（插件）** 页；
2. 点 **Add plugin**；
3. 把 `dsh-session-archive-vault-0.1.0.tgz` **的绝对路径**粘进去，例如：

   ```
   C:\Users\你的用户名\Downloads\dsh-session-archive-vault-0.1.0.tgz
   ```

   （也可以直接把文件拖进对话框，若界面支持。）
4. 等它完成 → **刷新页面**；若设置页没出现，**重启 DSH**。

### 方式 2：解压后按本地目录安装

1. 解压 `dsh-session-archive-vault-0.1.0.zip`，得到 `dsh-session-archive-vault\` 目录；
2. **Add plugin** 里粘贴该目录的**绝对路径**，例如：

   ```
   C:\Users\你的用户名\Downloads\dsh-session-archive-vault
   ```

3. 同样：刷新页面，必要时重启。

> 本地路径必须是绝对路径；相对路径会被拒绝。`file:` / `link:` 前缀可选。

### 方式 3：手工登记（可审查）

1. 把解压后的目录放到任意位置；
2. 在 `%DSH_HOME%\profiles\<profile>\package.json` 的 `dependencies` 加：

   ```json
   "@local/dsh-session-archive-vault": "link:C:/路径/dsh-session-archive-vault"
   ```

3. 同一个文件的 `dsh.profile.bundles` 数组里加：

   ```json
   "@local/dsh-session-archive-vault"
   ```

4. 在该 profile 目录跑一次 `pnpm install`，然后重启 DSH。

---

## 二、怎么用

1. 随便哪个会话行 →「...」菜单 → **归档**（内置功能）。
2. 该会话立刻从工作区 / 未分组列表消失（本插件会把「视图选项」纠正回「隐藏已归档」）。
3. 需要找回或清理时：**设置 → 归档会话**：
   - 按原工作区分组，组头显示工作区名与其目录；
   - 每行显示**最后对话时间**；
   - **撤销归档** → 会话回到侧栏原位；
   - **查看文件地址** → 展开真实目录、占用大小，可复制路径去手动删除。

## 三、卸载

**Plugins** 页找到该 bundle → 详情页 → 卸载。临时关闭而不卸载：在 profile 的 `cordis.patch.yml` 里加

```yaml
- id: session-archive-vault
  disabled: true
```

## 四、配置（可选）

在 profile 的 `cordis.patch.yml` 里覆盖：

```yaml
- id: session-archive-vault
  config:
    sessionsRoot: ''          # 留空 = <DSH_HOME>/sessions
    enforceHideArchived: 1    # 1 = 强制隐藏归档会话；0 = 只装页面，隐藏权交还内置视图选项
```

## 五、已知限制（诚实清单）

1. **强制隐藏会覆盖你自己的视图选项**：选「全部对话（显示已归档）」也会在约 1.5 秒内被纠正回「隐藏已归档」。不想要就设 `enforceHideArchived: 0`。
2. **依赖若干 DSH 内部契约**（设置页插槽 `settings.section`、客户端 `ctx.uiWorkspace` 的字段、侧栏视图的 `localStorage` 键 `dsh.workspace.view*`）。DSH 升级若改名，最坏结果是**页面不再出现或强制隐藏失效**——不会损坏数据，因为插件不写任何会话数据。
3. **归档 ≠ 删除**：聊天记录仍完整留在本机，本插件只帮你找到目录，删除要你自己动手。
4. **手动删除有前提**：删除目录前请确认 DSH 未打开该会话并已退出（Windows 上会话日志可能有写锁）。
5. **云会话后端不适用**：若会话保存在远端（如 deepseek 云日志），本地找不到文件，页面会显示「未找到本地文件」。
6. **本页只显示归档集合中的会话**，未归档会话不在其中。

## 六、验证与来源

- 已在 `DSH 0.1.7-rc.2` 上按源码逐条核对所用扩展缝；包内 `tools/` 有三个**离线检查**（不需要安装、不需要浏览器）：

  ```
  node tools/check-manifest.mjs   # 清单与 bundle patch 的 YAML 解析
  node tools/check-host.mjs       # 宿主半：路由、输入校验、真实会话定位
  node tools/check-browser.mjs    # 浏览器半：页面注册、隐藏守卫、分组渲染、撤销归档、地址揭示
  ```

  共 21 项，全部通过。`check-host.mjs` 会用本机真实存在的会话做端到端定位校验。

- 包内文件：`package.json`、`cordis.patch.yml`、`lib/index.js`（宿主半）、`lib/client.js`（浏览器半）、`README.md`、`tools/`。

---

### 打包信息

| 项 | 值 |
|---|---|
| 包名 | `@local/dsh-session-archive-vault` |
| 版本 | `0.1.0` |
| 依赖 | 无（零 npm 依赖，`link:` / tarball 安装均不需要解析第三方包） |
| 兼容 | 按 `DSH 0.1.7-rc.2` 核对；声明 `dsh.manifestVersion = 1` |
