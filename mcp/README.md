# 论文格式化 MCP Server

本 MCP Server 将仓库现有的模板分析、DOCX 排版与审计脚本封装为通用 MCP 工具，优先适配柯影智航（KoyingPilot），也可接入其他支持 stdio MCP 的客户端。它运行于本地 stdio，不启动网络服务；每次工具调用优先读取 MCP Roots。客户端不提供 Roots 时，Skill 会将当前会话选定的工作区通过 `workspace_root` 参数逐次传入，因此切换工作区无需修改 MCP 配置。服务将输入和输出限制在本次调用确认的工作区内，不会把源码目录当作论文工作区。

## 运行环境

- Python：严格固定为 **3.13.15**，版本记录在本目录的 `.python-version` 与 `pyproject.toml` 中。
- 依赖：由 uv 管理，`uv.lock` 锁定依赖版本。
- MCP Server 和仓库 `scripts/` 中的 CLI 共用此 uv 项目环境。
- 本地环境目录：`mcp/.venv/`，由 `.gitignore` 排除。

在仓库根目录运行：

```bash
uv python install 3.13.15
uv sync --locked --project mcp
uv run --locked --project mcp academic-paper-formatter-mcp
```

服务通过标准输入/输出接受 MCP 消息。stdio 上的标准输出专用于协议消息；日志写入标准错误。

## 柯影智航与其他 MCP 客户端配置

柯影智航“添加 MCP 服务”表单按以下内容填写。此处路径对应本仓库当前部署位置；若仓库移到其他机器或目录，需同步修改绝对路径。

| 表单字段 | 填写值 |
| --- | --- |
| 连接方式 | **本地命令（stdio）** |
| 服务标识 | `academic-paper-formatter` |
| 显示名称 | `学位论文格式化` |
| 描述 | `依据学校 Word 模板分析、格式化和审计学位论文 DOCX；保留源文件，并将结果写入单独文件。` |
| 启动命令 | `/opt/homebrew/bin/uv` |
| 工作目录 | `/Volumes/柯影数智/研发部/mcp/mcp-academic-paper-formatter/mcp` |

**参数**（每行一个，保持以下顺序）：

```text
run
--locked
academic-paper-formatter-mcp
```

**环境变量**（`NAME=VALUE`，不是凭据引用）：

```text
THESIS_FORMATTER_REPO_ROOT=/Volumes/柯影数智/研发部/mcp/mcp-academic-paper-formatter
```

`THESIS_FORMATTER_REPO_ROOT` 只用于定位仓库中的 `scripts/`。客户端提供 Roots 时，工具自动使用 Roots。客户端没有提供 Roots 时，Skill 从当前 Agent 会话上下文取得选定工作区，并在 `get_mcp_version` 和每次文件工具调用中传入 `workspace_root`；该路径只对本次调用生效，不需要配置或缓存每个工作区。只有无法提供 Roots、也无法传入动态工作区参数的旧客户端，才可选填 `THESIS_FORMATTER_WORKSPACE_ROOT` 作为固定兼容目录；该目录必须已存在且与源码仓库分离。不要将兼容目录设为 MCP 启动工作目录、源码仓库目录、它们的父目录、整个磁盘或用户主目录。

安装后先调用 `get_mcp_version`，确认返回的服务版本为当前版本，并检查 `workspace.configured` 为 `true`、`workspace.roots` 对应当前 Agent 工作区。若 Roots 为空，在 `get_mcp_version` 中传入当前工作区的 `workspace_root`。后续每个文件工具也必须传入同一 `workspace_root`；若无法从当前会话可靠取得工作区路径，所有文件工具都会拒绝执行。

**凭据引用（`NAME=REFERENCE`）**：不要添加条目，移除表单默认生成的空白行，最终保持凭据映射为空。本服务不需要 API 密钥或其他凭据。不要把普通环境变量填在此栏。

若安装时报 `MCP credentialEnv must map non-empty names to credential references`，检查并删除凭据引用栏中的空白行、`NAME=` 或未绑定的占位值；只在“环境变量”栏填写 `THESIS_FORMATTER_REPO_ROOT=...`。`THESIS_FORMATTER_WORKSPACE_ROOT` 仅用于旧客户端固定目录兼容。此提示来自宿主对凭据映射的校验，不代表 MCP Server 需要凭据。

其他 MCP 客户端可使用相同命令、参数、工作目录和环境变量；将这些值映射到对应版本的 stdio 配置字段即可。

输入和输出路径可使用相对于唯一 Agent 工作区的路径，也可使用任一已选工作区内的绝对路径；若客户端提供多个工作区，相对路径会被拒绝以免路径含义不清。除 `get_mcp_version` 外的所有文件工具都接受可选的 `workspace_root` 参数：当 Roots 可用时可省略；Roots 为空时必须传入当前 Agent 会话选定工作区的绝对路径。传入的路径只用于本次调用，不会保存在服务进程中；若 Roots 可用，提示路径还必须位于其中一个 Root 内。无 Roots 时，服务端无法独立验证客户端是否选中了该目录，因此 Skill 必须从当前 Agent 会话上下文读取该路径，不能从 MCP 启动目录、源码目录或输入文件路径猜测。服务仍会拒绝源码仓库及其父目录，并将所有文件路径限制在传入工作区内。输入文件必须存在；输出的子目录会在工作区内自动创建。若目标文件已存在，工具会拒绝覆盖；只有在客户端明确传入 `overwrite=true` 时才覆盖。输入与输出不能指向同一个文件。工具通过同目录临时文件写入结果后再替换目标路径。

文件工具的 JSON 参数表均包含可选的 `workspace_root`（未列入下表的业务参数栏），用于按调用动态指定当前 Agent 工作区。

## 工具清单

| 工具 | 主要参数 | 用途 / 文件影响 |
| --- | --- | --- |
| `get_mcp_version` | 可选 `workspace_root` | 返回 MCP 服务版本、Python 版本、Agent 工作区和工作区来源，不访问文件。 |
| `load_generated_artifact` | `artifact_path` | 将工作区内生成的 DOCX、PDF、JSON、Markdown 或 TXT 文件作为 MCP 嵌入资源返回。单个文件上限为 32 MiB；是否显示为可打开附件取决于客户端。 |
| `analyze_template` | `template_path`, `output_path`, `degree`, `overwrite` | 从学校 DOCX 模板提取规则到新 JSON，并返回提取警告与结构摘要。 |
| `validate_rules_file` | `rules_path` | 校验规则 JSON 的结构和必需字段，不修改文件。 |
| `format_document_copy` | `source_docx`, `rules_json`, `output_docx`, `overwrite` | 按已确认规则格式化论文副本。 |
| `validate_document_copy` | `rules_json`, `document_path`, `degree`, `strict_degree` | 审计文档格式，可附带学位结构复核提示，不修改文件。严格结构校验要求同时给出 `degree`。 |
| `parse_pdf_guide` | `guide_pdf`, `output_json`, `overwrite` | 从 PDF 指南提取带页码的候选文本证据到 JSON。 |
| `inspect_placeholders` | `document_path` | 查找论文中的内容占位符并返回位置与上下文，不修改文件。 |
| `apply_approved_replacements` | `source_docx`, `output_docx`, `replacements`, `overwrite` | 仅将用户审核过的精确映射写入论文副本。 |
| `audit_references` | `source_docx`, `rules_json` | 审计数字引用与参考文献条目，不修改文件。 |
| `update_references_copy` | `source_docx`, `rules_json`, `output_docx`, `approved_metadata`, `renumber_existing`, `overwrite` | 可插入用户提供的权威条目，或在缺失项与重复编号已解决后重新编号；写入副本。 |
| `normalize_floating_figures_copy` | `source_docx`, `output_docx`, `horizontal`, `vertical`, `relative_to`, `overwrite` | 规范现有浮动图片锚点，保留图像字节和题注，写入副本。 |
| `mark_word_fields_for_update_copy` | `source_docx`, `output_docx`, `overwrite` | 将 Word 的目录、交叉引用和页码字段标记为打开时更新，写入副本。 |

`degree` 可取 `auto`、`undergraduate`、`master`、`doctoral`。图片位置选项分别为：`horizontal` 的 `left` / `center` / `right`，`vertical` 的 `top` / `center` / `bottom`，以及 `relative_to` 的 `paragraph` / `margin`。

成功结果包含 `ok: true` 和工具结果；执行失败返回 `ok: false` 与错误信息。规则提取警告、格式差异及结构提示均需人工判断，不能视为学校要求的自动判定。

对于本轮需要交付的最终产物，完成文档校验后调用 `load_generated_artifact`，将文件内容作为 MCP `EmbeddedResource` 一并返回。工具接受工作区相对路径或工作区内绝对路径；Roots 不可用时传入本轮的 `workspace_root`。JSON、Markdown 和 TXT 按 UTF-8 文本加载，DOCX 与 PDF 以二进制资源加载。不要加载输入文件或仅供流程使用的中间产物。

**已知客户端限制（v0.1.1）**：端到端测试中，柯影智航收到 `load_generated_artifact` 返回的 DOCX 嵌入资源（MIME `application/vnd.openxmlformats-officedocument.wordprocessingml.document`，样例大小 39,842 bytes），但在会话中显示 `embedded resource unsupported`，没有可打开的附件卡片；`file://` Markdown 链接也未渲染为可点击控件。工作区边界负测通过，工作区外路径返回 `Path is outside the Agent workspace`。这是宿主端呈现能力限制，留待后续柯影智航开发处理，不改变本 MCP 工具的通用返回行为。

## 文件安全边界

- 优先处理 MCP 客户端提供的 Roots；Roots 为空时按调用的 `workspace_root` 动态限定本次调用，最后才使用可选的 `THESIS_FORMATTER_WORKSPACE_ROOT` 固定兼容目录。服务不会把 MCP 启动目录或源码目录作为默认值。
- 输入和输出均不能位于源码仓库内；相对路径以 Agent 工作区为基准。
- 输入路径规范化后先检查工作区边界，再检查文件存在性和扩展名；即使路径后缀无效，工作区外路径也优先返回边界拒绝。符号链接解析后的目标仍必须留在工作区内。
- 输出路径在创建子目录前检查工作区边界；不允许输出覆盖任一输入。
- 所有会写入 DOCX 的操作都生成副本；覆盖已有输出必须显式指定。
- 内容替换和参考文献插入只接受调用方提供的文本，不生成学术内容或书目数据。
- DOCX XML 解析禁用外部实体和网络访问；PDF 提取不执行 OCR，也不能验证图示中的版式要求。

## 与 CLI 的关系

MCP 工具复用仓库 `scripts/` 中的既有业务逻辑。客户端无法使用 MCP 时，可从仓库根目录使用 uv 环境调用原脚本，例如：

```bash
uv run --locked --project mcp python scripts/analyze_docx.py "/absolute/path/to/agent-workspace/school-template.docx" --degree master --output "/absolute/path/to/agent-workspace/school-rules.json"
```

面向智能体的工作顺序、安全要求和详细操作说明见 [`../skill/academic-paper-formatter/`](../skill/academic-paper-formatter/README.md)。
