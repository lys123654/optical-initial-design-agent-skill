# Optical Initial Design Agent Skill

一个面向光学初始结构生成的 Codex/Claude skill：从性能指标出发，检索处方库和专利，审查候选处方，再通过 Zemax ZOS-API 生成可打开、可审查的初始结构，并输出初步对焦和审查报告。

它的定位很明确：**生成和审查 starting point，不承诺直接给出最终优化设计。**

## 这个 Skill 解决什么问题

典型输入：

```text
我要一个 74 mm、F2.8、16 mm 像圆、总长 <55.5 mm、后焦 >6 mm 的短总长长焦镜头，
允许 5-15 片，允许少量玻璃非球面，要求加 0.3 mm 保护玻璃。
请先从处方库找，没有再搜专利，审查后写入 Zemax。
```

典型输出：

```text
specs.json
candidate_search_report.md
candidate_audits/*.md
zemax/*.zmx
analysis_outputs/*
initial_review.md
run_project_name.m
```

## 能做到什么

- 把用户指标整理成可执行的 `specs.json`。
- 自动或半自动生成检索关键词。
- 优先查公开处方库，再查专利。
- 对候选结构做数值审查。
- 检查专利表格中的曲率、厚度、材料、非球面项、视场、TTL/BFL。
- 用 MATLAB/Python/C# 的 ZOS-API 写入 Zemax LDE。
- 加入保护玻璃、波长、视场、光阑、自动净口径。
- 执行 Quick Focus。
- 输出 spot、MTF、first-order 等初步分析。
- 生成 Markdown 审查报告，明确哪些满足、哪些不满足、哪些只是建模假设。

## 不能直接保证什么

- 不能保证公开专利就是最终产品设计。
- 不能保证第一次搭出的结构满足最终 MTF、CRA、相对照度、公差、热稳定性。
- 不能替代专业光学优化。
- 不能把折叠系统简单展开后就当成完整真实结构。
- 不能在没有 Zemax 许可证和 ZOS-API 环境时生成真实 `.zmx`。

## 和 `zemax-optical-designer` 的关系

公开可见的 `zemax-optical-designer` skill 更像一个通用 Zemax 设计顾问，覆盖顺序光线追迹、MTF/PSF、优化、容差和杂散光分析。它对后续优化阶段可能有参考价值，尤其是：

- Zemax 分析类型选择；
- merit function 设计；
- 公差分析；
- stray light 或非序列分析；
- Zemax 操作经验提醒。

但它不是本 skill 的替代品。当前 skill 的核心价值是：

```text
指标输入 -> 处方库/专利检索 -> 候选审查 -> Zemax 初始结构生成 -> Quick Focus -> 初步报告
```

也就是说，`zemax-optical-designer` 可以作为后续“优化和分析顾问”，而这个 skill 是“从零找到并搭出初始结构”的工作流。

## 安装到 Codex

如果你把本目录上传到了 GitHub，可以在支持 skill 安装的环境中安装：

```powershell
npx add-skill https://github.com/<your-name>/<your-repo>
```

如果仓库根目录不是 skill 根目录，需要指定子目录：

```powershell
npx add-skill https://github.com/<your-name>/<your-repo> --path optical-initial-design-agent-skill
```

也可以直接把本目录复制到：

```text
C:\Users\<you>\.codex\skills\optical-initial-design-agent
```

## 推荐仓库结构

```text
optical-initial-design-agent-skill/
  SKILL.md
  README.md
  templates/
  scripts/
  docs/
  examples/
  .gitignore
```

## 使用流程

1. 收集指标，填写 `templates/specs.template.json`。
2. 生成检索关键词和候选搜索报告。
3. 对每个候选填写 `templates/candidate_audit.template.md`。
4. 对专利候选额外填写 `templates/patent_prescription_audit.template.md`。
5. 用 ZOS-API 脚本生成 Zemax 文件。
6. Quick Focus。
7. 输出 spot/MTF/first-order 分析。
8. 填写 `templates/initial_review.template.md`。

## 当前 MVP 经验

这个 skill 来自一次 DJI 风格长焦大底短总长镜头 starting-point MVP。已验证的关键经验包括：

- Optical Bench 处方中的口径列可能是直径，不一定是 Zemax semi-diameter。
- AS/FS 行通常要切分前一个空气间隔，而不是无脑插入额外距离。
- 保护玻璃必须用前后两个面建模，否则 Quick Focus 可能改掉玻璃厚度。
- 非 stop、非 image 面建议使用自动净口径求解。
- 专利中的折叠结构不能简单展开后就宣称忠实复现。
- F/3 专利打开到 F/2.8 后边缘光线异常并不意外。
- ZMX 能打开不等于结构正确，必须检查 2D layout、first-order、spot/MTF 和约束表。

## 后续软件 Agent 形态

这个 skill 可以演进成一个完整 agent 软件：

```text
需求解析器
  -> 检索 Agent
  -> 候选抽取器
  -> 处方审查器
  -> Zemax 写入器
  -> 自动对焦/分析器
  -> 报告生成器
  -> 人工确认节点
```

建议先保持“半自动 + 人工审查节点”，等候选审查和 Zemax 写入稳定后，再逐步提高自动化程度。

