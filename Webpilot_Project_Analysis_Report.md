# Webpilot 项目分析报告

## 项目概述

**项目名称**: Webpilot - Copilot for All, Free & Open  
**版本**: v0.10.0204  
**开发团队**: webpilot.ai (dev@webpilot.ai)  
**项目类型**: Chrome 浏览器扩展  
**开源状态**: 免费且开源  

Webpilot 是一个免费的开源"Web Copilot"，允许用户与网页进行自由形式的对话或与其他用户进行自动论证。与 ChatGPT 不同，无需聊天或切换网页，也无需不断地复制粘贴。

## 技术架构

### 核心技术栈
- **框架**: Vue 3.2.47
- **构建工具**: Plasmo 0.76.3 (专为浏览器扩展设计)
- **状态管理**: Pinia 2.0.36
- **包管理器**: pnpm 8.2.0
- **TypeScript**: 5.0.4

### 主要依赖库
- **VueUse**: `@vueuse/core@10.1.2` - Vue 组合式工具库
- **Plasmo Storage**: `@plasmohq/storage@1.6.1` - 扩展存储管理
- **Plasmo Messaging**: `@plasmohq/messaging@0.4.0` - 扩展消息传递
- **Mozilla Readability**: `@mozilla/readability@0.4.4` - 网页内容提取
- **js-tiktoken**: `@js-tiktoken@1.0.7` - Token 计算工具
- **Vue3-markdown-it**: `@vue3-markdown-it@1.0.10` - Markdown 渲染
- **Pangu**: `@pangu@4.0.7` - 中英文混排优化

### 开发工具链
- **代码质量**: ESLint + Prettier + Stylelint
- **Git Hooks**: Husky + lint-staged + Commitlint
- **国际化**: Vue3-gettext 2.5.0

## 核心功能特性

### 1. AI 对话集成
- **多 API 支持**: 
  - Webpilot 官方服务器 (`api.webpilotai.com`)
  - OpenAI 官方 API
  - OpenAI 代理服务
  - Azure OpenAI 服务
- **默认模型**: `gpt-4o-mini`
- **流式响应**: 实时显示 AI 回复
- **Token 管理**: 自动截断长内容以适应模型限制

### 2. 文本选择与交互
- **智能文本选择**: 支持鼠标和键盘选择文本
- **即时反馈**: 选择文本后自动显示操作选项
- **位置计算**: 精确计算弹出位置，支持textarea和普通文本
- **快捷键支持**: 可自定义快捷键 (默认 Ctrl + `)

### 3. 预设提示词系统
**页面问答提示词**:
- 总结 (Summarize)
- SEO 关键词生成
- 问题解决分析

**文本选择提示词**:
- 解释说明 (Explain)
- 文本润色 (Refine) 
- 图像描述生成 (Draw)

### 4. 多种显示模式
- **弹窗模式** (popUp): 浮动弹窗显示
- **侧边栏模式** (sideBar): 侧边栏固定显示
- **拖拽功能**: 支持拖拽移动界面元素

### 5. 多语言支持
- **支持语言**: 英语 (en) 和简体中文 (zh_CN)
- **国际化框架**: Vue3-gettext
- **动态语言切换**: 实时切换界面语言

## 配置选项详解

### API 配置 (`src/config.ts`)
```typescript
{
  apiOrigin: 'general' | 'openAI' | 'OpenAIProxy' | 'azure', // API 来源
  authKey: string,                    // API 密钥
  selfHostUrl: string,               // 自托管 URL
  azureApiVersion: string,           // Azure API 版本
  azureDeploymentID: string,         // Azure 部署 ID
  model: {
    model: 'gpt-4o-mini',           // 模型名称
    temperature: 1,                 // 温度参数
    top_p: 0.9,                    // Top-p 参数
    frequency_penalty: 0,          // 频率惩罚
    presence_penalty: 0,           // 存在惩罚
    stop: '<|endoftext|>'          // 停止标记
  }
}
```

### 用户界面配置
```typescript
{
  autoPopup: boolean,                // 选中文本时自动弹出
  displayMode: 'popUp' | 'sideBar', // 显示模式
  customShortcut: string[],          // 自定义快捷键
  showShortcutTips: boolean,         // 显示快捷键提示
  isFinishSetup: boolean,           // 是否完成初始设置
  latestAskedQuestionPromptIndex: number,    // 最近使用的问答提示词索引
  latestTextSelectionPromptIndex: number     // 最近使用的文本选择提示词索引
}
```

## 项目结构分析

### 目录结构
```
Webpilot/
├── src/                           # 主要源代码
│   ├── components/               # Vue 组件库
│   │   ├── InteractiveIcon/     # 交互式图标组件
│   │   ├── SuperButton/         # 超级按钮组件
│   │   └── SendButton/          # 发送按钮组件
│   ├── contents/                # 内容脚本
│   ├── csui/                    # Content Script UI 组件
│   ├── background/              # 后台脚本
│   ├── options/                 # 选项页面
│   ├── popup/                   # 弹窗页面
│   ├── tabs/                    # 新标签页面
│   ├── hooks/                   # Vue Composition API 钩子
│   ├── stores/                  # Pinia 状态管理
│   └── utils/                   # 工具函数
├── assets/                      # 静态资源
│   ├── locales/                # 国际化文件
│   ├── styles/                 # 样式文件
│   └── icon/                   # 图标资源
└── scripts/                     # 构建脚本
```

### 核心模块功能

#### 1. 内容脚本 (`src/contents/`)
- **Index.vue**: 主入口，注入到所有网页
- **特定站点支持**: Discord, GitHub, Slack, Twitter
- **Shadow DOM**: 使用 Shadow DOM 隔离样式

#### 2. 后台脚本 (`src/background/`)
- **消息处理**: 处理扩展内部消息传递
- **设置管理**: 打开设置页面
- **弹窗检查**: 检查弹窗状态

#### 3. Hooks 系统 (`src/hooks/`)
- **useAskAi**: AI 对话核心逻辑
- **useSelectedText**: 文本选择处理
- **useDraggable**: 拖拽功能实现
- **useMessage**: 消息传递封装

#### 4. 状态管理 (`src/stores/`)
- **store.ts**: 主要配置存储
- **user.ts**: 用户信息管理
- **api.ts**: API 相关状态

## 浏览器扩展权限

### Manifest 权限配置
```json
{
  "host_permissions": ["<all_urls>"],
  "permissions": [
    "tabs",           // 标签页访问
    "storage",        // 本地存储
    "clipboardWrite"  // 剪贴板写入
  ]
}
```

## 开发和构建

### 开发命令
- **开发模式**: `pnpm dev`
- **构建**: `pnpm build`
- **打包**: `pnpm package`
- **代码检查**: `pnpm lint`
- **国际化处理**: `pnpm gettext`

### 代码质量控制
- **ESLint**: JavaScript/TypeScript 代码检查
- **Stylelint**: CSS/SCSS 样式检查
- **Prettier**: 代码格式化
- **Husky**: Git hooks 管理
- **Commitlint**: 提交消息规范检查

### 开发环境设置
1. 运行 `pnpm dev` 启动开发服务器
2. 打开 Chrome 扩展管理页面 (`chrome://extensions`)
3. 启用开发者模式
4. 加载 `build/chrome-mv3-dev` 目录
5. 将扩展固定到 Chrome 工具栏

## 安全和隐私

### 数据处理
- **本地存储**: 配置数据存储在浏览器本地
- **API 密钥**: 用户自定义 API 密钥本地加密存储
- **网页内容**: 使用 Mozilla Readability 提取网页内容
- **隐私政策**: 项目包含详细隐私政策文档

### API 安全
- **多重验证**: 支持多种 API 认证方式
- **错误处理**: 完善的 API 错误处理机制
- **请求限制**: 自动处理 API 请求限制和配额

## 特色亮点

1. **零配置使用**: 默认使用官方免费服务，无需配置即可使用
2. **完全开源**: 所有代码开源，支持自由修改和分发
3. **多平台兼容**: 专门优化支持主流网站 (GitHub, Discord, Slack 等)
4. **响应式设计**: 支持不同屏幕尺寸和设备
5. **无缝集成**: 直接在网页上操作，无需切换标签页
6. **智能优化**: 自动处理长文本截断和 Token 优化
7. **可扩展架构**: 模块化设计，易于扩展新功能

## 使用场景

- **网页内容总结**: 快速获取网页核心信息
- **文本润色修改**: 选中文本进行语法和表达优化
- **多语言翻译**: 文本翻译和解释
- **SEO 优化**: 生成相关关键词和优化建议
- **学习辅助**: 解释复杂概念和术语
- **创意写作**: 基于选中文本生成创意描述

## 技术创新点

1. **Shadow DOM 隔离**: 避免与宿主页面样式冲突
2. **流式响应处理**: 实时显示 AI 生成内容
3. **智能位置计算**: 精确定位弹窗显示位置
4. **多 API 统一接口**: 统一处理不同 AI 服务 API
5. **Token 智能管理**: 自动处理模型上下文长度限制
6. **国际化最佳实践**: 使用专业的 gettext 国际化方案

---

**报告生成时间**: 2024年  
**项目当前版本**: v0.10.0204  
**分析覆盖度**: 完整项目结构和核心功能分析