# killbug

> 6 轮 + 自定义工业级代码审查与修复流水线（移植 / 大改后专用）

为 [Claude Code](https://github.com/anthropics/claude-code) 设计的 Skill。一次启动，把项目拉成上帝视角扫一圈，从根因解决问题，而不是补丁式叠加。

## 这是什么

固定 6 轮，每轮一个维度，互不重叠：

| 轮次 | 维度 | 覆盖范围 |
|---|---|---|
| 1 | 后端 | 路由 / DB / 数据模型 / 启动入口；框架用法、SQL 安全、移植残留、契约对齐 |
| 2 | 前端（逻辑契约） | 接口对齐、状态管理、前端交互框架行为、模板片段契约、状态信号触发 |
| 3 | 样式与体验 | 视觉一致性、布局 / 动画 / 移动端、状态四态、控件四态、无障碍 |
| 4 | 冗余与债务 | 死代码、重复逻辑、过时垫片、未用依赖、错位注释、过期 TODO |
| 5 | 安全（攻击者视角） | 注入 / 越权 / 认证 / CSRF / 信息泄露 / XSS / 依赖 CVE / 限流 |
| 6 | 收官 · 全局交叉 | 读所有报告做收官统筹，扫描隐式断链 + 数据流闭环 + 架构自洽 |

加上**自定义轮次**：在启动器里直接说要查什么（"做一下性能审查"、"扫一遍 i18n"），AI 加工成 prompt 给你确认后开跑。

每轮的执行链路：找问题 → 出方案 → 跨轮影响判断 → 排批次 → 红线披露 → 单次决策 → 一路执行不打断 → 精简报告 → 进入下一轮。

## 适用场景

- ✅ **跨框架 / 跨语言移植后** —— 大量隐性残留要清理
- ✅ **大版本重构后** —— 架构变了，链路要重对齐
- ✅ **接手他人代码 / 老项目** —— 用 6 轮维度系统性扫
- ❌ 小修小补 —— 杀鸡用牛刀
- ❌ 新建项目早期 —— 还没成型，不值得审

## 安装

把仓库 clone 到 Claude Code 的 skill 目录：

```bash
# Mac / Linux
git clone https://github.com/<your-username>/KillBug ~/.claude/skills/killbug

# Windows (PowerShell)
git clone https://github.com/<your-username>/KillBug "$env:USERPROFILE\.claude\skills\killbug"
```

> 注：仓库名 `KillBug` 大小写不影响 GitHub 访问;但**本地目标路径必须小写** `killbug`,与 SKILL.md 里 `name: killbug` 对齐。

或下载 zip 解压到 `~/.claude/skills/killbug/`，确保目录结构为：

```
~/.claude/skills/killbug/
├── SKILL.md
└── prompts/
    ├── bug-repair.md
    ├── round-1-backend.md
    ├── round-2-frontend.md
    ├── round-3-style.md
    ├── round-4-debt.md
    ├── round-5-security.md
    └── round-6-global.md
```

重启 Claude Code 即可识别。

## 调用

在 Claude Code 里：

- `/killbug` —— 弹启动器，4 选 1（全自动 / 单轮 / 查进度 / 自然语言自定义）
- `/killbug N` —— N=1-6，直接启动第 N 轮单轮审查（高级用户跳过启动器）

## 二次加工

默认模板里以 FastAPI / HTMX / Jinja2 / Argon2id 等技术栈作为示例锚点。如果你的栈差异很大（React / Vue / NestJS / Django / Spring 等），建议改 `prompts/round-*.md` 里的"重点关注"列表，把示例换成你的栈关键词。Skill 本体（`SKILL.md`）的执行流程不需要改。

## 设计原则

- **本轮闭环优先** —— 问题在哪轮发现就尽量在哪轮修。只有"同轮冲突 / 后续轮冲突 / 改动量+风险都极高"三种情况才能跨轮交接
- **单次决策、一路到底** —— 用户只在 Step 3 后做一次决策，不再因预披露红线动作中途打断
- **报告精简强约束** —— 给第 6 轮收官提供干净上下文，任何废话都是带成本的
- **红线两类协同** —— 一类·可预披露（删项目内文件、改 schema）一次性披露后就批准；二类·真·红线（升权限、扩范围、危险数据动作）即使披露也不该出现在方案中

完整流程见 [SKILL.md](SKILL.md)，修复哲学见 [prompts/bug-repair.md](prompts/bug-repair.md)。

## License

[MIT](LICENSE)
