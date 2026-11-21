# Z-Library AI Workbench 集成约定

## 目标

配套项目 `coconut9826/zlibrary-ai-workbench` 负责搜索下载、书架、命名、PDF 转 EPUB、Ollama 总结和任务编排；AudioSmith 负责本地语音模型、有声书项目、章节与角色音色、渲染、响度和 M4B 导出。

两个项目保持独立仓库和独立发布周期，通过 AudioSmith 现有的 MCP 或 Local API 连接。

## 安全边界

- 默认只连接 `127.0.0.1:3901`。
- 工作台不得自动开启 AudioSmith、修改 AudioSmith 设置、接受模型许可或下载模型。
- 项目创建、渲染和导出都记录 `project_id`、`job_id`、输出文件和错误代码。
- API 密钥、声音参考文件、书籍正文和生成音频不得提交到 Git。
- 若启用 AudioSmith 的局域网分享，须由用户在应用里明确配置；工作台不自动开放端口或建立隧道。

## 首版流程

1. 工作台下载并按规则命名书籍。
2. EPUB、TXT、Markdown 或 DOCX 直接交给 `create_audiobook`。
3. PDF 先由工作台转换：
   - `reflow` 生成可调字号 EPUB并保留可提取图片；
   - `visual` 逐页渲染，优先保证版面和图片不丢失；
   - `auto` 根据文字密度和版面复杂度选择模式。
4. 工作台调用 `update_longform` 设置作者、语言、封面、旁白、M4B 和 ACX 响度。
5. `render_longform` 返回后台任务，工作台通过 `get_job` 跟踪进度。
6. 完成后由 `export_longform` 导出到工作台配置的有声书目录。

## 现有接口覆盖

| 工作台需求 | AudioSmith 能力 |
|---|---|
| 创建有声书 | `create_audiobook` |
| 作者、语言、封面、旁白、格式和响度 | `update_longform` |
| 后台渲染 | `render_longform` |
| 任务进度和错误 | `get_job` |
| M4B/WAV/章节导出和自定义目录 | `export_longform` |
| 主流 AI 会话调用 | MCP HTTP/stdio |
| 本地大模型优先 | AudioSmith 本地写作模型及可选 Ollama |
| PDF 直接导入 | 当前不支持；首版由工作台先转 EPUB |

## 验收清单

- [ ] 使用模拟 MCP 响应验证创建、更新、渲染、查询和导出的参数契约。
- [ ] 使用短 EPUB 在真实 AudioSmith 中完成一次 M4B 渲染。
- [ ] 封面、书名、作者、语言、旁白和章节元数据写入 M4B。
- [ ] 输出目录来自工作台配置，不落入未授权目录。
- [ ] `busy`、`model_not_downloaded`、`licence_required`、`not_ready` 和任务失败被原样返回。
- [ ] 工作台重启后能用保存的 `project_id` / `job_id` 继续查看进度。
- [ ] 相同书籍不会在未确认时重复创建和重复渲染。

## 可选增强：AudioSmith 直接导入 PDF

只有用户确实需要在 AudioSmith 界面直接拖入 PDF 时才实现。推荐复用工作台的转换组件或稳定的本地转换接口，不在 AudioSmith 内维护第二套 PDF 解析逻辑。实现前必须定义：

- 扫描件、复杂表格、双栏和图文混排的降级策略；
- 原始图片与页面渲染图的取舍；
- 页数、文件大小、像素和解压限制；
- 转换质量报告与用户提示；
- 中间 EPUB 的清理和失败恢复规则。

## 许可证

AudioSmith 当前采用 AGPL-3.0。此分叉及网络提供的修改版本必须继续提供对应源码，并保留 LICENSE、NOTICE 与第三方模型的独立许可说明。
