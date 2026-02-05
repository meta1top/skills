# Meta1 Skills

这是在 Meta1 相关的 App 开发工作中，总结出的 Skills 集合。这些 Skills 为 AI 助手提供了专业的知识和指导，帮助开发者更高效地使用 Meta1 相关的技术栈和工具。

## 可用 Skills

### meta1-design

基于 Shadcn + Tailwind CSS v4 构建的组件库使用指南。

**适用场景：**
- 创建新项目时，询问是否安装依赖
- 项目 package.json 中包含 `@meta-1/design` 依赖
- 构建交互性友好的 UI 界面时

**主要内容：**
- 包的引入和转译配置
- Tailwind CSS 配置和主题设置
- 组件清单和使用方法
- 与 Shadcn 的差异说明

### meta1-full-stack

Meta1 全栈开发技能，支持 Next.js + NestJS 的 TypeScript 项目。

**适用场景：**
- Node.js 项目 package.json 包含 `@meta-1/nest-*` 相关的包时
- 创建、迭代或者调用一个新接口
- 构建界面（通常要配合 meta1-design 技能）
- 声明前后端共享的类型、修改数据库字段
- 修改持久化层、业务层、Controller 层等各层的业务逻辑时
- 修改页面交互时

**主要内容：**
- **后端库（@meta-1/nest-*）**：
  - `@meta-1/nest-common` - 缓存、分布式锁、事务管理、国际化、统一响应格式、错误处理、分页、HTTP 客户端、配置加载等
  - `@meta-1/nest-security` - 用户认证、JWT Token 管理、会话管理、OTP 验证码、加密解密等
  - `@meta-1/nest-assets` - 文件上传、文件存储、预签名 URL、存储提供商（S3、OSS、MinIO）等
  - `@meta-1/nest-message` - 邮件服务、验证码管理、错误码定义等
  - `@meta-1/nest-types` - 前后端共享的类型定义、Zod Schema 验证等
- **前端库（@meta-1/web-common）**：
  - React Hooks（useQuery、useMutation、useTableLoader、useLocale、useSchema、useEncrypt 等）
  - React 组件（布局、输入、页面、UI、上传组件等）
  - 工具函数（REST API、Token 管理、加密工具、国际化等）
  - 状态管理（Jotai Atoms）
- **工作流指南**：完整的全栈开发工作流，从持久层到 Web 端的开发流程

## 安装

安装单个 skill：

```bash
# 安装 meta1-design
npx skills add https://github.com/meta1top/skills --skill meta1-design

# 安装 meta1-full-stack
npx skills add https://github.com/meta1top/skills --skill meta1-full-stack
```

安装所有 skills：

```bash
npx skills add https://github.com/meta1top/skills
```

## 使用

安装后，AI 助手会在相关场景下自动使用这些 Skills 来提供更准确的帮助和指导。

## 贡献

欢迎贡献新的 Skills！请参考 [Skill Creator Guide](https://github.com/meta1top/skills) 了解如何创建和提交新的 Skills。