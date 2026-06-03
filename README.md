# Optical Initial Design Agent Skill

一个面向光学初始结构生成的 Codex/Claude skill：从性能指标出发，检索处方库和专利，审查候选处方，再通过 Zemax ZOS-API 生成可打开、可审查的初始结构，并输出初步对焦和审查报告。

它的定位很明确：**生成和审查 starting point，不承诺直接给出最终优化设计。**

## 这个 Skill 解决什么问题

典型输入：

```text
我要一个中长焦、大像面、短总长的成像镜头初始结构，
光圈接近快速成像镜头水平，总长和后焦有明确包装约束，
允许若干片玻璃/塑料镜片和少量非球面，并需要加入指定厚度的保护玻璃。
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

## 安装到 Codex

推荐安装到 Codex 全局 skill 目录，这样所有项目都能使用：

```powershell
npx skills add lys123654/optical-initial-design-agent-skill -g -y
```

如果仓库是私有仓库，先确认本机已经登录 GitHub：

```powershell
gh auth status
```

如果 CLI 安装因为私有仓库权限或网络代理失败，可以手动安装：

```powershell
git clone https://github.com/lys123654/optical-initial-design-agent-skill.git `
  "$env:USERPROFILE\.codex\skills\optical-initial-design-agent"
```

也可以直接把本目录复制到：

```text
C:\Users\<you>\.codex\skills\optical-initial-design-agent
```

安装后确认这个文件存在：

```text
C:\Users\<you>\.codex\skills\optical-initial-design-agent\SKILL.md
```

重新打开 Codex 后，就可以在相关任务中触发这个 skill。

## 推荐仓库结构

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

这个 skill 来自一次中长焦大像面短总长镜头 starting-point MVP。已验证的关键经验包括：

- Optical Bench 处方中的口径列可能是直径，不一定是 Zemax semi-diameter。
- AS/FS 行通常要切分前一个空气间隔，而不是无脑插入额外距离。
- 保护玻璃必须用前后两个面建模，否则 Quick Focus 可能改掉玻璃厚度。
- 非 stop、非 image 面建议使用自动净口径求解。
- 专利中的折叠结构不能简单展开后就宣称忠实复现。
- 将公开处方强行打开到更快光圈后，边缘光线异常并不意外。
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
