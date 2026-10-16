# AGENTS.md · AI 代理与协作者开发维护守则

> **最高铁律 · 禁止粗暴覆盖**：
> 当从原作者仓库（上游 `aimod-cc/agent2api`）合并更新、拉取新功能或同步补丁时，**严禁使用全量文件复制替换（Copy & Paste Overwrite）的形式！**
> 任何代码同步必须遵循**外科手术式增量合并（Surgical Incremental Merge）**原则，逐行审视变更差异，绝对禁止因盲目整文件替换而破坏、冲刷掉本仓库特有的自定义核心功能与工程改进。

---

## 1. 核心保护资产清单（严禁被上游更新覆盖清空）

在每次同步上游代码时，必须主动检查并 100% 保全以下 6 大核心模块：

### ① 国内专属服务商与 CodeBuddy 深度集成
- **保护文件**：
  - `desktop-tauri/src-tauri/server/src/core/registry.ts`
  - `desktop-tauri/src-tauri/server/src/providers/codebuddy.ts` 及相关凭证/短信登录逻辑
  - `desktop-tauri/ui-islands/src/islands/accounts-shared.ts`、`models-panel-state.ts`、`models-page.tsx`
- **保护目标**：保留 CodeBuddy 官方国内渠道、配额查询与多模型接入。

### ② 智能就地覆盖更新安装器（NSIS Smart In-Place Update）
- **保护文件**：`desktop-tauri/src-tauri/windows/installer.nsi`
- **保护目标**：
  - 必须保留 `RestorePreviousInstallLocation` 函数。
  - 用户安装新版本时，自动通过注册表（`Software\aimod-cc\Agent2API*` 及 Windows Uninstall 键）与标准目录识别本机已有的安装位置，并默认覆盖更新，免去用户手动选路径或产生多份残留。

### ③ 递增式软件标题版本规范（Title Versioning）
- **保护文件**：
  - `desktop-tauri/src-tauri/src/lib.rs`（`app_title()` 函数）
  - `desktop-tauri/src-tauri/tauri.conf.json`（`productName`）
  - `desktop-tauri/ui/index.html`（`<title>` 与品牌展示）
  - `desktop-tauri/src-tauri/windows/installer.nsi`
- **保护目标**：软件标题必须带 `v1`、`v2`、`v3` 等递增式标识（如 `Agent2API v3 · 多提供商本地网关`），方便用户一眼识别当前运行的是新版还是旧版。

### ④ 提示词文件导入与正文联动系统（Prompt File Import & Body Sync）
- **保护文件**：
  - `desktop-tauri/src-tauri/src/commands.rs`（`pick_prompt_file`）
  - `desktop-tauri/src-tauri/src/bridge.rs`（`pickPromptFile`）
  - `desktop-tauri/src-tauri/server/src/web_shim.rs`（`pickPromptFile`）
  - `desktop-tauri/ui-islands/src/islands/settings-page.tsx`、`settings-state.ts`、`settings-model.ts`
- **保护目标**：在设置页「提示词文件」旁边提供「导入文件…」按钮，既能记录文件绝对路径，又能直接将文件 UTF-8 文本导入为生效的「提示词正文」，效果等同于直接编辑正文。

### ⑤ 视觉安全拦截与清洗（Vision Gate & Image Strip）
- **保护文件**：
  - `desktop-tauri/src-tauri/server/src/core/provider_loop.rs` 中的图片剥离与清洗管道及单元测试。
- **保护目标**：针对上游不支持图片输入的场景提供稳健防报错降级。

### ⑥ CI/CD 自动构建与 Release 自动发布工作流
- **保护文件**：`.github/workflows/build.yml`
- **保护目标**：除 `README.md` 变更外，所有新代码推送均自动触发 GitHub Actions 编译 Windows `exe` 安装包与独立程序，并自动创建/更新对应版本的 GitHub Release。

---

## 2. 上游更新标准操作流程（SOP）

当上游原作者发布新版本（如 `3.0.4`、`3.0.5` 等）时，执行以下标准流程：

```bash
# 1. 抓取上游远端分支（确保已配置 git remote add upstream https://github.com/aimod-cc/agent2api.git）
git fetch upstream

# 2. 在隔离分支中进行比对与合并
git checkout -b merge-upstream-<version>
git merge upstream/main --no-commit

# 3. 逐个核对冲突文件与关键改动
# 严禁直接 git checkout --theirs <protected-files>
# 必须逐行分析 diff，将上游新功能合并进现有架构，保留受保护资产

# 4. 编译与双端验证
cd desktop-tauri/ui-islands && npm run build
cd ../src-tauri && cargo test --workspace

# 5. 软件标题递增
# 按照用户要求，将标题中的版本号递增（如 v2 -> v3）

# 6. 本地打包验证
cd .. && npm run build
```

---

## 3. 敏感数据与隐私零容忍原则
- 仓库代码内绝对严禁硬编码任何真实 API Key、Secret、个人 Token、密码或真实内网地址。
- 向 GitHub 推送前必须执行全量敏感数据排查。

---

## 4. 强制自动发布 GitHub Releases 铁律 (Mandatory Auto-Release Push)
- **编译完成必推 Release**：每次版本升级、合并新特性或本地/云端编译完成后，**必须确保产物自动推送到 GitHub Releases**，绝对严禁编译完成后仅留在本地、未挂载至 Releases 页面。
- **发布规范与闭环要求**：
  1. 打对应版本 Tag（如 `v4`、`v5` 等）并推送到远端仓库（`git tag vX && git push origin vX`）。
  2. 确保 GitHub Actions（`.github/workflows/build.yml`）及自动化发布通道能可靠创建/更新对应版本的 GitHub Release。
  3. Release 资产必须包含完整的 Windows 安装包（NSIS Setup）、免安装绿色版可执行文件（`agent2api.exe`）及全平台安装产物，并标记为 `make_latest: true`。
  4. 每次交付前，必须核验 GitHub Releases 页面已有对应版本（如 `v4`）的正式发布项与可下载文件。
