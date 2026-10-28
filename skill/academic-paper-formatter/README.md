# 学位论文格式化 Skill

该 Skill 优先适配柯影智航（KoyingPilot），也可移植到其他支持 Agent Skill 的智能体；适用于中国大陆本科、硕士、博士学位论文的 Word `.docx` 模板分析、排版和校验。学校提供的 Word 模板与书面规范是唯一格式依据；通用经验和学位层级清单不能替代学校要求。

## 目录结构

```text
academic-paper-formatter/
├── references/
│   ├── guide.md                              # MCP、uv 环境与 CLI 使用指南
│   ├── docx-formatting.md                    # DOCX 与版式说明
│   ├── citation-rules.md                     # 引文与参考文献规则
│   ├── chinese-degree-thesis-workflow.md     # 学位论文结构复核流程
│   └── chinese-thesis-format.md               # 通用格式原则
├── ANALYSIS_REPORT.md                        # 设计决策、风险和验收记录
├── CHANGELOG.md                              # 版本变更记录
├── extension.json                            # 项目本地元数据
├── LICENSE                                   # MIT 许可
├── README.md                                 # 本文件
└── SKILL.md                                  # 技能入口与执行策略
```

## 安装与接入

将本目录安装到柯影智航或其他 Agent 的 Skill 目录，并确保宿主加载 `SKILL.md`。具体安装位置与发现机制按宿主适配。`extension.json` 是本项目的本地元数据文件，不是通用 Skill 标准。

在柯影智航或其他支持 MCP 的 Agent 中配置本项目的本地 MCP Server，以调用模板分析、格式化和审计工具。MCP 环境由 uv 管理，Python 固定为 3.13.15。若宿主未启用 MCP，可在具备终端权限时按 [使用指南](references/guide.md) 调用 CLI。

## v0.1.1 状态与已知限制

v0.1.1 已锁定。`load_generated_artifact` 会把最终文件作为 MCP 嵌入资源返回。当前柯影智航客户端显示 `embedded resource unsupported`，不会将 DOCX/PDF 显示为可打开的附件卡片；客户端也未把 `file://` Markdown 链接渲染为可点击链接。MCP 工具的资源返回和工作区安全边界已通过端到端调用验证，柯影智航宿主端的资源呈现工作留待后续开发。

## 适用任务

- 从学校 DOCX 模板提取可审查的规则 JSON。
- 按已确认的模板规则格式化论文副本并检查差异。
- 检查数字顺序编码引用、占位内容和常见学位论文结构。
- 从可复制文字的 PDF 指南提取带页码的候选证据，供人工确认。

## 安全边界

- 保留模板和论文原稿；结果写入新的输出文件。
- 不编造学校要求、论文内容、研究结果或参考文献。
- 不把 PDF 文本提取或结构提示当成自动判定；交付前需人工检查版面。
- 只有在用户明确要求时，才覆盖已有输出文件。

## 许可

本 Skill 的原始文档和元数据采用 MIT 许可。Python、MCP SDK、Microsoft Word、LibreOffice 及其他第三方项目各自遵循其许可协议。
