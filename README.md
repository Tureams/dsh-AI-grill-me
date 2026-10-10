# 需求侦探（dsh-detective）

DSH（DeepSeek Harness）的一个**可选 Agent 模式**：在模式选择器里与「标准模式 / 创造模式」并列。
它把「先问清楚，再动手」做成一个可以随时切进去的工作方式。

```
追问清楚  →  出《推理总结 + 功能大纲 + 交付物清单 + 验收标准 + 未决清单》  →  等你一句「确认」
          →  自己把项目完整落地（全部文件写盘、自检、能跑、附交付说明）  →  追问你「满意吗」
          →  你说满意才算成功；每满十轮必须强制交付一版初版
```

## 它解决什么问题

一句话说出一个模糊的念头（「我将会制作一个 AI Skill」）时，普通模式容易直接开写，
结果做出来的东西不是你要的。需求侦探反过来：**默认动作是提问**，先亮出它对你意图的猜测，
让你纠正而不是从零解释；把需求逼成一份你能点头、且照着就能交付的大纲；你不点头，它一个字都不写。

## 五个固定产出

| # | 产出 | 内容 |
|---|---|---|
| 1 | 《推理总结》 | 我认为你要什么，按置信度排序，附依据 |
| 2 | 《功能大纲》 | 每项功能标 `核心` / `可选` / `建议砍掉` |
| 3 | 《交付物清单》 | 文件类型 × 数量 × 命名 × 目录，逐项可勾选 |
| 4 | 《验收标准》 | 从你嘴里抠出来的「怎样算满意」 |
| 5 | 《未决问题清单》 | 没问、没答、含糊的地方 |

《需求纪要》只出现在对话里，不落盘成项目文件。

## 两条铁律

1. **确认前不写入**：你明确说「确认／定稿／可以开始」之前，它不创建、不修改、不删除任何文件，
   每轮回复末尾自报一行「本轮未写入任何文件」。
2. **不替用户决定**：你没明确确认的判断一律标「我的推理（未确认）」并保留在《未确认清单》里；
   你没指出错误的项它不反复追问，但也绝不替你填成既定事实。

## 节奏与验收

- **问题数量自适应**：一轮问几个由它判断，你在赶时间它就少问、拆轮。
- **轮数不设上限，但每满十轮强制交付初版**（一问一答算一轮）：到点先给你一版能看的东西，再继续问或继续改。
- **验收＝你说「满意」**：交付后它会追问「这样你满意吗」；你说满意之前任务不算成功。
- **v1 是一轮的终点**：先交付一版，出 bug 继续修，不追求一次完美。

## 安装

```bash
# 从 GitHub 安装（推荐）
dsh plugin --profile web add github:Tureams/dsh-AI-grill-me

# 本地开发安装（本目录）
dsh plugin --profile web add link:<本目录绝对路径>
```

> 安装规格里用的是**仓库名** `Tureams/dsh-AI-grill-me`；装进 `node_modules` 与 `dsh.profile.bundles`
> 的名字由包内 `package.json` 的 `name` 决定，即 `dsh-detective`。

安装后**重启 DSH**（或重新打开客户端），新建会话时即可在模式选择器里看到「需求侦探」。
桌面端（Electron 客户端）请在桌面端会话里让 DSH 自己装（桌面端读 `desktop` profile，CLI 不接受 `--profile desktop`）。

## 卸载

```bash
dsh plugin --profile web remove dsh-detective
```

## 怎么用

1. 新建会话，模式选「需求侦探」。
2. 用一句话说出你想做的东西，越随口越好。
3. 回答它的提问（可以只纠正、不重述）。
4. 拿到五件套大纲，逐条确认或改。
5. 说「确认」，等它把项目建完并给你交付说明。
6. 回答它那句「满意吗」——这是它判定成功的唯一标准。

## 它是怎么工作的（技术）

本包是一个标准 DSH bundle 插件，**不含任何 JS 代码**，只有一个挂载声明：

- `package.json` 的 `dsh.bundle.patch` 指向 `cordis.patch.yml`；
- `cordis.patch.yml` 用一条 `- insert:` 插入一个 `@deepseek-ai/dsh-agent-preset` 行
  （`config.id: detective`，显示名「需求侦探」），并用 `@deepseek-ai/dsh-persona`
  写入常驻系统提示（模式的行为底色：两条铁律、问题量自适应、十轮强制交付、满意才算成功）；
- 该 preset 里的 `@deepseek-ai/dsh-skill-filesystem` 通过 `customSkillDirs`
  **只挂载本包自带的 `skills/` 目录**，所以这个模式不读取你工作区里任何东西就能工作；
- 完整流程与模板在 `skills/detective/SKILL.md`（按需由 `skill` 工具加载），
  细化问题库在 `skills/detective/references/必问清单.md`，输出模板在 `skills/detective/templates/需求纪要.md`。

```
dsh-detective/
├─ package.json                              ← dsh.bundle.patch 声明
├─ cordis.patch.yml                          ← 一条 insert：新模式 detective
├─ skills/detective/
│  ├─ SKILL.md                               ← 执行手册（追问→大纲→确认→施工→交付→迭代）
│  ├─ references/必问清单.md                  ← 九格细化问题库
│  └─ templates/需求纪要.md                   ← 五件套输出模板
├─ README.md
├─ LICENSE                                   ← MIT
└─ 设计决定.md                                ← 已确认决定与技术选择记录
```

## 兼容性

- 开发与验证环境：DSH `0.2.0-rc.2`（npm 包 `@deepseek-ai/dsh`）。
- 依赖的都是 DSH 内置包（`dsh-agent-preset` / `dsh-persona` / `dsh-skill-filesystem` / `dsh-tool-*`），
  本包自身零运行时依赖。
- 有意不声明 `peerDependencies`：避免版本不匹配时整包被 DSH 静默跳过（见 `设计决定.md`）。

## 已知限制

- 门禁是**方案乙**：确认前不许写，靠系统提示与 `SKILL.md` 规则约束，不是物理隔离。
  它不挂 `plan-mode`（那是物理禁写的方案甲）。唯一例外是「满十轮的强制初版」，见 `SKILL.md` 第 4 节。
- 模式在**新建会话**时选择；已开会话的行为准则不随模式切换而改变。
- 没有挂 `subagent` / `workflow`：需要并行大工程的场景请切到标准模式或创造模式。

## 许可证

MIT，版权人 Tureams。
