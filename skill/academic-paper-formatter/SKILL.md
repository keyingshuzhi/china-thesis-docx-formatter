---
name: academic-paper-formatter
description: 面向柯影智航并兼容其他支持 Agent Skill 的智能体。依据中国高校提供的 Word 模板分析、格式化和校验本科、硕士及博士学位论文 DOCX，适用于论文排版、模板规则提取、引用审查和文档结构复核任务。
metadata:
  short-description: 基于学校模板的中文学位论文排版
---

# 学位论文格式化

始终以学校的 DOCX 模板和书面规范为格式依据。保留原始模板和论文文件，所有修改输出到单独文件。不要把任何通用样式或学位层级清单当作学校要求。

## 工作流程

1. 从当前 Agent 会话上下文识别选定的工作区绝对路径（若可用），并用该值调用 `get_mcp_version(workspace_root=...)`；若当前上下文没有路径，则省略参数并依赖客户端 Roots。只有 `workspace.configured=true` 且返回的根目录包含或覆盖当前工作区时才开始文件操作。
2. 若 `get_mcp_version` 返回 `workspace.source="workspace_hint"`，每次后续 MCP 文件工具调用都传入同一个 `workspace_root`。该参数只作用于当前调用，不会写入 MCP 配置或缓存到后续会话。若 Roots 与动态路径都不可用，停止文件操作并请用户提供或选择工作区；不要从 MCP 启动目录、源码目录或输入文件路径猜测。
3. 所有模板、论文、规则文件和结果都必须位于已确认工作区内。只有一个有效工作区时优先使用相对路径；不要把论文或生成文件放进源码仓库。
4. 优先使用本地 MCP 工具处理文件。MCP 不可用且有终端时，按 [使用指南](references/guide.md) 使用 uv 管理的 CLI 环境。
5. 从学校模板提取规则，检查提取警告和模糊的样式角色，再校验规则 JSON。
6. 将规则应用于论文副本，并按规则审计结果。学位层级清单只是人工复核提示；只有学校书面规范明确要求全部项目时才使用严格结构校验。
7. 将输出 DOCX 渲染并逐页检查版面。通过校验后，对本轮需要交付的最终产物调用 `load_generated_artifact`，将文件作为 MCP 嵌入资源返回。若客户端无法呈现嵌入资源，明确说明该客户端当前不支持会话内打开，并给出工作区文件路径；不要声称已出现附件卡片。重点检查页面尺寸、中文字体、分页、表格裁切、题注、页眉页脚与参考文献换行。

## MCP 调用纪律

- 依赖上一步产物的文件工具必须串行调用；只有收到 MCP 的结构化 `ok=true` 结果后，才使用返回的输出路径继续下一步。
- 界面中仅出现函数调用请求、没有 MCP 工具结果时，不要重复发送同一写入调用；先确认工具实际执行状态。
- 写入工具返回 `Output already exists` 时，保留现有文件，不要自动设置 `overwrite=true` 或重复写同一路径。需要继续时，先用只读工具验证已有文件；确需重跑时选择新的唯一输出路径。
- 有依赖关系的格式化、图片规范和字段更新必须按前一环节的输出逐步串联，不能从同一旧源并行生成最终候选文件。
- 用户需要在当前会话交付的最终产物，在完成校验后调用 `load_generated_artifact(artifact_path=...)`；Roots 不可用时传入与本轮其他文件调用相同的 `workspace_root`。每个用户可见的最终文件分别加载，不要加载源文件或临时中间产物。会话能否显示为可打开附件由客户端决定；柯影智航当前版本不支持呈现该嵌入资源，此宿主能力留待后续开发。

## 内容与引用变更

- 不编写或静默插入摘要、结论、实验结果、学校要求、引用或参考文献条目。
- 先检查内容占位符并报告；只有在用户审核精确替换映射后，才将替换写入新副本。
- 修改参考文献前先审计数字引用。只从用户提供的权威元数据插入条目；只有缺失条目与重复编号已解决后，才重新编号。
- 未经用户明确要求，不覆盖已有输出文件。
- 工作区为空时，先让用户把模板和论文放入工作区；MCP 不会从源码目录或工作区外读取输入。

## 工具选择

- 模板与版式：`analyze_template`、`validate_rules_file`、`format_document_copy`、`validate_document_copy`。
- 引用与内容：`audit_references`、`inspect_placeholders`、`apply_approved_replacements`、`update_references_copy`。
- 辅助操作：`parse_pdf_guide`、`normalize_floating_figures_copy`、`mark_word_fields_for_update_copy`。
- 会话交付：`load_generated_artifact`，将工作区内的 DOCX、PDF、JSON、Markdown 或 TXT 文件作为 MCP 嵌入资源返回；具体呈现由客户端决定。
- 版本和工作区：`get_mcp_version`。

读取 [references/guide.md](references/guide.md) 了解 MCP 配置、工具参数和 CLI 备用流程。开始排版前阅读 [references/docx-formatting.md](references/docx-formatting.md)；处理书目时阅读 [references/citation-rules.md](references/citation-rules.md)；复核学位论文结构时阅读 [references/chinese-degree-thesis-workflow.md](references/chinese-degree-thesis-workflow.md)。
