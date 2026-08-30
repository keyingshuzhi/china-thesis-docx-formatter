# Academic Paper Formatter

面向中国国内本科、硕士、博士学位论文的 Word `.docx` 智能排版技能。它先从学校提供的 Word 模板中提取可审计的规则 JSON，再将规则应用于论文副本；不把任意学校的规则硬编码到项目中。

> 学校模板和书面规范是唯一的格式依据。学位层级仅提供“待人工确认”的结构清单，不能替代学校规定。

## 当前能力

- 从学校 `.docx` 模板提取页边距、段落、字体、标题、图表题注、表格与参考文献样式，生成规则 JSON。
- 按模板规则格式化论文副本，并审计格式一致性。
- 为本科、硕士、博士识别常见结构：中英文摘要与关键词、目录、声明、参考文献、致谢、附录和攻读学位期间成果页。
- 检查数字顺序编码引用与参考文献条目；支持半角/全角方括号；仅能插入用户提供的权威参考文献数据。
- 查找并替换用户已审核的内容占位符，不自动编造论文内容或学术来源。
- 规范浮动图片锚点，并请求 Word 在打开文档时更新目录、交叉引用和页码字段。
- 从学校 PDF 规范中提取带页码的文字证据，供人工确认；扫描件、图示和视觉排版仍需人工检查。

## 智能体接入与工具支持

| 对象 | 接入方式 | 当前支持范围 | 状态 |
| --- | --- | --- | --- |
| **Codex Desktop / Codex CLI** | 原生 Codex Skill（`academic-paper-formatter`） | 自动发现技能、运行全部脚本、生成与校验 DOCX 规则 | 原生支持 |
| **具备终端与 Python 环境的智能体** | 调用本仓库 `scripts/*.py` | 可执行相同的模板分析、格式化、引用审计及 PDF 证据提取流程 | CLI 兼容 |
| **Claude Code、Cursor Agent、OpenCode、Aider 等** | 作为上一项的具体示例：在其工作区中克隆仓库并调用 CLI | 不提供专属插件、MCP Server 或平台配置；以其终端权限和 Python 环境为准 | CLI 兼容，未做原生适配 |
| **Microsoft Word 桌面版** | 打开生成的 `.docx` | 更新 TOC、REF、PAGEREF 字段并进行最终人工版式检查 | 文档目标环境 |
| **LibreOffice（无头模式）** | 渲染 DOCX 为页面图片 | 用于视觉 QA；最终提交前仍建议在学校要求的 Word 版本中复核 | 辅助工具 |
| **学校 PDF 规范** | `parse_pdf_rules.py` + `pdfplumber` | 提取可复制文本中的页码证据；不做完整语义解析或 OCR | 有限支持 |

当前**未提供** ChatGPT App、Claude MCP Server、Cursor 扩展、Word 加载项、云端 API 或自动提交学校系统的专属集成。普通 ChatGPT 对话若不能访问本地文件和终端，也不能直接运行此技能。

## 快速开始

```bash
# 1. 从学校模板生成规则；degree 只启用结构复核提示
python scripts/analyze_docx.py school-template.docx --degree master --output school-rules.json

# 2. 检查规则 JSON
python scripts/validate_format.py school-rules.json

# 3. 格式化论文副本（不会改动原稿）
python scripts/format_document.py thesis-source.docx thesis-formatted.docx --rules school-rules.json

# 4. 审计版式并输出硕士论文结构复核提示
python scripts/validate_format.py school-rules.json --document thesis-formatted.docx --degree master
```

`--degree` 可选值为 `undergraduate`、`master`、`doctoral` 和 `auto`。只有在学校书面规范已明确要求全部项目时，才使用 `--strict-degree` 使缺失项成为失败条件。

## 常用工作流

```bash
# 内容占位符：先报告，再用用户审核过的映射写入副本
python scripts/complete_content.py thesis.docx --report placeholders.json
python scripts/complete_content.py thesis.docx --replacements approved-content.json --output thesis-completed.docx

# 引用与参考文献：先审计，绝不凭空生成条目
python scripts/manage_references.py thesis.docx --rules school-rules.json --report references-audit.json

# 学校 PDF 规范：提取带页码的候选证据，之后人工确认
python scripts/parse_pdf_rules.py school-guideline.pdf --output pdf-requirements.json

# 浮动图片与 Word 字段
python scripts/normalize_floating_figures.py thesis.docx thesis-figures.docx --horizontal center --vertical top
python scripts/update_fields.py thesis-figures.docx thesis-final.docx
```

## 环境要求

- Python 3.10+（建议使用虚拟环境）
- Microsoft Word：用于最终字段更新和学校要求版本的人工复核
- LibreOffice：可选，用于无头渲染和视觉 QA

安装依赖：

```bash
python -m pip install -r requirements.txt
```

## 限制与安全边界

- 不修改学校模板和论文原稿；所有输出应写入新文件。
- 不凭空补写摘要、结论、实验结果、引用条目或学校格式要求。
- PDF 文本提取不是 PDF 规范的完整语义解析，扫描页和图示页必须人工复核。
- Word 字段在脚本中仅被标记为“打开时更新”；目录和交叉引用的最终结果由 Word 更新。
- `templates/` 中的 JSON 仅是测试/起点配置，不能作为任何学校的提交标准。

## 项目结构

```text
SKILL.md                 # Codex 技能入口与安全工作流
scripts/                 # 模板分析、排版、审计与修复脚本
references/              # 中国学位论文、引文与 DOCX 操作说明
templates/               # 本科、硕士、博士通用起点 JSON
tests/                   # 回归测试
test_data/               # 隐私安全的演示输入与验证输出
```

详见 [SKILL.md](SKILL.md)、[中国学位论文结构核对](references/chinese-degree-thesis-workflow.md) 与 [引用规则](references/citation-rules.md)。
