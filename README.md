# FluxMobius

FluxMobius 是一款基于 Tauri 2、Vue 3、Vite、TypeScript、Milkdown 和 ProseMirror 构建的本地优先 Markdown 笔记工作站。它强调 Typora 式沉浸写作、本地文件可控、AI 辅助创作，以及面向长文档的稳定编辑体验。

当前版本：`2.0.0`

## 核心定位

FluxMobius 不是云端笔记服务，而是一个围绕本地 Markdown 文件工作的桌面应用：

- 文件直接保存在磁盘，可关联本地文件夹，也可使用虚拟工作区组织笔记。
- 编辑器默认使用 Milkdown/ProseMirror 提供所见即所得体验。
- 大文件会自动切换到 CodeMirror 源码模式，优先保证打开、搜索和编辑的流畅性。
- AI 能力通过专用服务层接入，支持 OpenAI 及兼容 OpenAI API 的提供商。
- 导入导出链路覆盖 Markdown、HTML、TXT、PDF、Word、思维导图等常见流转场景。

## 功能概览

### 编辑体验

- 所见即所得 Markdown 编辑，支持标题、列表、引用、代码块、任务列表、表格和分割线。
- Slash 菜单、块句柄、选区工具栏、表格工具栏、链接编辑浮窗等低干扰交互。
- KaTeX 数学公式、Mermaid 图表、目录块、图片节点和增强代码块。
- 文档内查找替换、跳转到文档开头/末尾、重复上一次编辑动作。
- 大文件源码模式：当文档超过约 1 MB、超过 30000 行，或存在超长单行时，自动使用 CodeMirror 编辑器打开。

### 工作区与检索

- 本地文件夹关联与虚拟分组并存。
- 文件新建、重命名、移动、删除、复制和打开所在位置。
- 全局全文搜索，支持大小写、全词和正则匹配。
- 文档大纲实时生成并支持快速跳转。

### 导入与导出

- 导入 Markdown、HTML、TXT 和 Word `.docx`。
- TXT 导入会自动识别 UTF-8、UTF-16 和 GB18030 等常见编码。
- Word 导入依赖 Pandoc，会将 `.docx` 转换为 Markdown，并把导出的图片复制到当前工作区资产目录。
- 导出 Markdown 捆绑包、单文件 HTML、PDF、Word `.docx`、纯文本 TXT、Markdown 源码 TXT、思维导图和思维导图 PDF。
- Word 导出会预渲染 Mermaid 图表为 SVG，并统一处理本地图片资源。

### 剪贴板与图片

- 内置“四阶段剪贴板架构”：影子剪贴板、智能匹配链、HTML/Markdown 解析和粘贴后归一化。
- 应用内复制会保留 ProseMirror Slice 结构，尽量还原自定义节点、公式、表格和图片。
- 外部粘贴会过滤伪富文本高亮污染，优先保留真正语义结构。
- 图片支持本地资产复制、引用修复、资源报告和导出内联。

### AI 辅助

- AI 对话、选区润色、翻译、续写、总结、扩写、改写和代码解释。
- 自定义提示词模板、流式生成、接受/拒绝生成内容、Token 用量统计。
- API Key 通过 Tauri Secret/系统安全能力封装，不在 UI 组件中直接处理复杂请求。

### 系统集成

- Tauri 单例模式：从外部打开 Markdown 文件时复用已有进程。
- Windows 文件关联、图标、打开方式和右键新建 Markdown 文件入口。
- 亮色/暗色主题适配，菜单栏和快捷键深度集成。

## 快速开始

### 环境依赖

- Node.js 18+
- Rust stable
- Pandoc 可选，仅在使用 Word 导入导出或纯文本语义导出时需要

### 安装依赖

```bash
npm install
```

### 本地开发

```bash
npm run tauri dev
```

`npm run dev` 只会启动前端 Vite 服务。完整桌面功能、文件系统访问、剪贴板、系统集成和 Pandoc 转换需要通过 Tauri 启动。

### 构建

```bash
npm run tauri build
```

构建产物通常位于 `src-tauri/target/release/bundle/`。

## Pandoc 配置

Word 导入导出和纯文本语义导出依赖 Pandoc。FluxMobius 会优先读取“首选项 > 转换”中配置的 `pandoc.exe` 路径；如果留空，会尝试从系统 `PATH` 自动检测。

支持的转换链路：

- `.docx` -> Markdown
- Markdown -> `.docx`
- Markdown -> plain TXT

## 关键开发红线

`src/components/MilkdownEditor.vue` 是当前编辑器容器，体积已经很大，后续修改必须遵守：

- 不在 `MilkdownEditor.vue` 中堆砌新的业务算法、文件处理或复杂 ProseMirror 事务。
- 纯逻辑放入 `src/utils/`，响应式复用逻辑放入 `src/composables/`，浮层和弹窗拆成独立 Vue 组件。
- Milkdown schema 扩展优先用 `ctx.update(existingSchema.key, ...)` 原位扩展，禁止随意重新注册同名节点污染 schema 顺序。
- 新增粘贴能力必须进入 `editorClipboard.ts` / `editorClipboardMatchers.ts` 的智能粘贴链路，禁止绕过影子剪贴板。

完整约束见 [AGENTS.md](./AGENTS.md)。

## 文档入口

- [架构与开发指南](./ARCHITECTURE.md)
- [组件架构说明](./docs/COMPONENT_ARCHITECTURE.md)
- [AI 功能开发指南](./AI_DEVELOPMENT_GUIDE.md)
- [AI 功能使用指南](./AI_USAGE.md)
- [主题系统说明](./THEME.md)
- [内置帮助文档](./src/assets/HELP.md)

## 技术栈

- Vue 3 + TypeScript + Vite
- Tauri 2 + Rust
- Pinia
- Milkdown + ProseMirror
- CodeMirror 6
- Tailwind CSS + CSS Variables
- Mermaid、KaTeX、PrismJS、Markmap
- Pandoc 外部转换器
