# 版本发布规范

本文件定义本仓库的版本号语义、分支模型、标签规则与发布流程。所有正式发布必须遵守本规范。

---

## 一、版本号语义（SemVer 2.0.0）

版本号格式为 `MAJOR.MINOR.PATCH`，标签格式为 `vMAJOR.MINOR.PATCH`。

- **MAJOR**：不兼容的变更。例如配置文件结构变更、数据源路径变更、启动方式移除、缓存格式不再向后兼容。
- **MINOR**：向后兼容的功能新增。例如新增数据源、新增指标、新增告警类型、新增服务商模板。
- **PATCH**：向后兼容的修复与内部改进。例如修复 DPI 偏移、修复缓存失效、修复崩溃。

预发布版本使用 `vX.Y.Z-rc.N` 形式，仅用于发布候选验证，不得标记为 Latest。

---

## 二、分支模型

采用「主干发布 + 维护分支」的轻量模型，保证任一标签都能追溯到明确的分支。

- `main`：稳定发布线。任何时候 `main` 的 HEAD 都对应最近一次正式发布，且带有对应标签。
- `release/MAJOR.MINOR`：维护分支。每次 MINOR 及以上版本发布后，从该版本标签切出，用于后续 PATCH 级修复。例如 v3.7.0 发布后切出 `release/3.7`。
- `feature/<简述>`：功能开发分支，合入 `main` 前必须实测。
- `fix/<简述>`：缺陷修复分支，针对已发布版本的问题时以对应的 `release/X.Y` 为基线。

分支与标签的对应关系：

- 一条发布线 `release/X.Y` 上的最高标签为 `vX.Y.Z`，即该分支的当前发布点。
- 标签只打在 `main` 或 `release/X.Y` 上，禁止在 `feature/*`、`fix/*` 上打标签。
- 热点修复流程为：从 `release/X.Y` 切 `fix/*`，修复并验证后合回 `release/X.Y`，在该分支打 `vX.Y.(Z+1)`，再合并回 `main`。

---

## 三、提交信息规范（Conventional Commits）

格式为 `type(scope): 描述`，描述使用中文，动宾结构，不使用句号结尾。

允许的 `type`：

- `feat`：新增功能
- `fix`：缺陷修复
- `perf`：性能优化
- `refactor`：既不修复缺陷也不新增功能的重构
- `docs`：文档变更
- `style`：不影响逻辑的格式调整
- `test`：测试相关
- `build`：构建脚本或依赖变更
- `ci`：持续集成配置
- `chore`：其它杂项
- `release`：版本发布准备（版本号、CHANGELOG、发布说明）

常见的 `scope` 取值：`collector`、`overlay`、`server`、`dashboard`、`sources`、`providers`、`petsprite`、`win32ui`、`docs`。

破坏性变更必须在提交正文中以 `BREAKING CHANGE:` 开头单独成段说明，或在 `type` 后加 `!`。

示例：

- `feat(overlay): 增加宠物尺寸五档切换`
- `fix(overlay): 修复窗口过程内回调 Tk 导致的解释器崩溃`
- `docs: 补充 v3.7.0 发布说明`
- `release: v3.7.0`

---

## 四、发布流程

1. **冻结范围**：确认本次发布包含的提交，未验证的功能不得进入发布分支。
2. **验证**：在目标 Windows 环境实测启动、胶囊或桌宠显示、仪表盘、本次变更点，记录验证结论。
3. **更新版本信息**：在 `CHANGELOG.md` 顶部新增本次版本条目，日期取定稿日。
4. **撰写发布说明**：在 `docs/releases/vX.Y.Z.md` 建立文件，内容将作为 GitHub Release 正文，必须包含版本号、更新内容、变更类型、修复项、已知问题。
5. **提交**：分别以 `docs:` 与 `release:` 类型提交，保持提交粒度清晰。
6. **打标签**：使用注解标签，而非轻量标签。

```
git tag -a v3.7.0 -m "v3.7.0 — 桌宠形态与多应用用量监测"
```

7. **推送**：`git push origin main --follow-tags`，确保标签与提交同时到达远端。
8. **创建 Release**：以标签为目标创建 GitHub Release，标题为 `vX.Y.Z`，正文取自 `docs/releases/vX.Y.Z.md`，早期版本可勾选 Latest。
9. **切维护分支**：`git branch release/3.7 v3.7.0` 并推送，建立该发布线的维护入口。
10. **回填**：若发布过程产生额外修复，须以 PATCH 版本重新走 1 至 8 步，不得原地改写已推送的标签。

---

## 五、标签纪律

- 已推送的标签不得删除、移动或强制覆盖。需要修正时必须发新版本。
- 标签注释必须包含版本号与一句话主题，格式为 `vX.Y.Z — 主题`。
- 每个正式版本都必须存在对应的 GitHub Release，且正文与本仓库 `docs/releases/` 下的同名文件一致。

---

## 六、发布前置检查

发布前必须确认以下各项，任一项不通过则不得发布：

- 仓库中不存在 `petconfig.json`、`.cache/`、`exports/`、`dist/`、`*.log` 等本地运行产物。
- 仓库中不存在任何 API Key、access token、Cookie 或账号凭据。
- `assets/logos/` 仅包含各服务商商标资源，商标归各自权利人，不改变其归属。
- `assets/ATTRIBUTION.md` 中的素材许可与署名要求未被破坏，精灵表仅按 CC BY-NC-SA 4.0 使用。
- 本次变更涉及的数据源路径、端口、快捷键已在 README 中同步更新。
