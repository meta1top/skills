---
name: meta1-full-stack
description: Meta1 全栈开发技能，支持 Next.js + NestJS 的 TypeScript 项目。提供 @meta-1/nest-* 后端库（缓存、安全、资源、消息、类型）和 @meta-1/web-common 前端库（React Hooks、组件、工具）的完整使用指南。帮助用户实现接口、类型定义、数据查询、表单处理、文件上传、认证授权等全栈功能。
language: typescript,tsx
framework: react,nextjs,tailwindcss,nestjs,typeorm
---

# Meta1 Full Stack 开发能力

拥有 Meta1 全栈 Monorepo 项目开发能力的专家指南。

## 何时使用

- Nodejs 项目 package.json 包含 @meta-1/nest-* 相关的包时
- 用户要创建、迭代或者调用一个新接口
- 用户构建界面（通常要配合 meta1-design 技能
- 声明前后端共享的类型、修改数据库字段
- 修改持久化层、业务层、Controller 层等各层的业务逻辑时
- 修改页面交互时候

## 如何使用

### 依赖

#### @meta-1/nest-common

**功能模块：**
- 🎯 **缓存装饰器** - `@Cacheable`, `@CacheEvict`, `CacheableService`，支持 Redis 缓存
- 🔒 **分布式锁** - `@WithLock` 装饰器，基于 Redis 实现分布式锁
- ❄️ **雪花ID生成器** - `@SnowflakeId` 装饰器，自动生成分布式唯一ID
- 🔄 **事务管理** - `@Transactional` 装饰器，自动管理数据库事务
- 🌍 **国际化支持** - `@I18n` 装饰器和 `I18nContext`，支持命名空间和自动采集
- ⚡ **响应拦截器** - `ResponseInterceptor`，统一的 API 响应格式 `RestResult<T>`
- 🚨 **错误处理** - `ErrorsFilter`，全局错误过滤器，支持自定义错误码
- 📄 **分页DTO** - `PageRequest`, `PageData`, `PageRequestSchema`，标准化的分页请求和响应
- 🌐 **HTTP服务** - `HttpService`，增强的 HTTP 客户端，支持重试和文件下载
- ⚙️ **配置加载器** - `ConfigLoader`，支持从本地 YAML 文件或 Nacos 配置中心加载配置
- ✅ **验证工具** - `createI18nZodDto`, `ZodValidationPipe`，自动采集验证错误消息
- 📚 **Swagger工具** - `PageResponseSchema`，分页响应的 Swagger Schema 生成工具

**使用场景：** 需要缓存、分布式锁、事务管理、国际化、统一响应格式、错误处理、分页、HTTP 客户端、配置加载等功能时。

详情查看：[@meta-1/nest-common](references/nest-common.md)

#### @meta-1/nest-security

**功能模块：**
- 🔐 **会话管理** - `SessionService`，基于 Redis 的会话存储和管理
- 🔑 **Token 管理** - `TokenService`，JWT Token 生成、验证和解析（`createToken`, `verifyToken`, `decodeToken`）
- 🔒 **OTP 支持** - `OtpService`，基于 TOTP 算法的一次性密码功能（`generateSecret`, `verifyCode`）
- 🛡️ **认证拦截器** - `AuthInterceptor`，自动从请求头提取 token 并获取用户信息
- 🎯 **装饰器** - `@CurrentUser()` 获取当前用户，`@Public()` 标记公开接口
- 🔄 **会话刷新** - `SessionService.refresh()`，会话过期时间刷新和 payload 管理
- 🔐 **加密服务** - `CryptoService`，RSA 和 AES 加密解密支持（`encryptRSA`, `decryptRSA`, `encryptAES`, `decryptAES`）

**使用场景：** 需要用户认证、JWT Token 管理、会话管理、OTP 验证码、加密解密等功能时。

详情查看：[@meta-1/nest-security](references/nest-security.md)

#### @meta-1/nest-assets

**功能模块：**
- 📦 **存储提供商** - 支持亚马逊 S3、阿里云 OSS 和 MinIO（通过 `StorageProvider` 枚举配置）
- 🔗 **预签名 URL** - `getPresignedUploadUrl()`, `getPresignedDownloadUrl()`，客户端直传模式
- 🔄 **统一接口** - `AssetsService`，内部自动切换存储提供商
- 🪣 **桶管理** - 支持私桶（`privateBucket`）和公桶（`publicBucket`）
- 🔐 **私桶授权** - 私桶访问自动签名授权
- ⏰ **签名有效期** - 可配置签名有效期（`expiresIn`）
- 🔑 **文件 Key** - 自动生成唯一文件 Key（带时间戳和随机字符串）

**使用场景：** 需要文件上传、文件存储、获取预签名 URL、管理文件资源等功能时。

详情查看：[@meta-1/nest-assets](references/nest-assets.md)

#### @meta-1/nest-message

**功能模块：**
- 📧 **邮件服务** - `MailService`，支持 AWS SES 和阿里云 DirectMail（`sendEmail`）
- 🔐 **验证码管理** - `MailCodeService`，基于 Redis 的验证码发送和验证（`sendCode`, `verifyCode`）
- 🎯 **错误码定义** - `MessageErrorCode`，类型安全的错误码枚举
- 🐛 **调试模式** - 开发环境支持固定验证码，无需真实发送邮件
- ⏱️ **频率限制** - 自动限制验证码发送频率（1分钟内不可重复发送）
- ⏰ **自动过期** - 验证码自动过期（默认 5 分钟，可配置）

**使用场景：** 需要发送邮件、发送验证码、验证验证码等功能时。

详情查看：[@meta-1/nest-message](references/nest-message.md)

#### @meta-1/nest-types

**功能模块：**
- 🛡️ **类型定义** - 完整的 TypeScript 类型定义，前后端共享
- ✅ **Zod Schema** - 基于 Zod 的 Schema 验证，包含 AI、Assets、Common、Message 等模块的 Schema
- 📝 **类型推导** - 从 Schema 自动推导 TypeScript 类型（`z.infer<typeof Schema>`）
- 🔧 **工具函数** - `coerceNumber`, `coerceNumberOptional` 等 Zod 工具函数

**主要导出：**
- **AI 相关** - `InvokeAgentSchema`, `AiConfigSchema`, `AddDocumentSchema`, `SearchDocumentSchema`
- **Assets 相关** - `StorageProvider`, `BucketType`, `PresignedUploadUrlRequestSchema`, `PresignedDownloadUrlRequestSchema`
- **Common 相关** - `PageRequestSchema`, `RestResult`, `PageData`, `PageRequest`
- **Message 相关** - `SendCodeSchema`

**使用场景：** 需要定义前后端共享的类型、数据验证 Schema、分页类型、API 响应类型等功能时。

详情查看：[@meta-1/nest-types](references/nest-types.md)

#### @meta-1/web-common

**功能模块：**

**React Hooks（核心功能）：**
- 🪝 **useQuery** - REST API 查询 Hook，基于 TanStack Query，自动处理 `RestResult<T>` 格式和错误提示
- 🪝 **useMutation** - REST API 变更 Hook，用于 POST、PUT、DELETE 等操作，支持错误自动提示
- 🪝 **useTableLoader** - 表格数据加载 Hook，支持分页、筛选、排序，自动同步 URL 参数
- 🪝 **useLocale** - 国际化语言切换 Hook
- 🪝 **useSchema** - Zod Schema 国际化 Hook，自动翻译错误消息
- 🪝 **useEncrypt** - RSA 加密 Hook，用于敏感数据加密

**React 组件：**
- 🎨 **布局组件** - `RootLayout`, `MainLayout`, `HtmlLayout`, `LoginLayout`
- 🎨 **输入组件** - `OTPInput`, `CodeInput`, `EmailCodeInput`, `SliderInput`
- 🎨 **页面组件** - `PageHeader`, `PageTitleBar`, `TabsTitle`
- 🎨 **UI 组件** - `Loading`, `Coming`, `Visible`, `ThemeSwitcher`, `LangSelect`
- 🎨 **上传组件** - `Uploader`（基于 `@meta-1/design` 封装）

**工具函数：**
- 🔧 **REST API** - `get`, `post`, `put`, `patch`, `del`, `download`, `request`，自动处理响应格式
- 🔧 **Token 管理** - `getToken`, `setToken`, `clearToken`（支持客户端和服务端）
- 🔧 **加密工具** - `encrypt`（RSA 加密）
- 🔧 **React Query** - `getQueryClient`, `prefetchQuery`, `dehydrate`（服务端预取）
- 🔧 **请求头** - `headers.client`, `headers.server`（客户端和服务端请求头生成）
- 🔧 **国际化** - `initI18next`（i18next 初始化）
- 🔧 **环境工具** - `isServer`, `isMobile`, `getRootDomain`
- 🔧 **格式化** - `formatFileSize`, `join`

**状态管理：**
- 📦 **Jotai Atoms** - `localeState`, `publicKeyState`（全局状态）

**类型定义：**
- 🛡️ **REST API 类型** - `RestResult<T>`, `PageData<T>`, `PageResult<T>`, `PageRequest`
- 🛡️ **Token 类型** - `Token`

**使用场景：** 
- 前端需要调用 REST API 时使用 `useQuery`, `useMutation`
- 需要表格分页功能时使用 `useTableLoader`
- 需要文件上传时使用 `Uploader` 组件
- 需要国际化时使用 `useLocale`, `useSchema`
- 需要加密敏感数据时使用 `useEncrypt`
- 需要 REST API 请求时使用 `get`, `post` 等工具函数

详情查看：[@meta-1/web-common](references/web-common.md)

#### @meta-1/design

请安装并加载 meta1-design 技能，以确定 `@meta-1/design` 如何使用。

```bash
npx skills add https://github.com/meta1top/skills --skill meta1-design
```

### 工作流

这里整理了一份完整的工作流：[workflow](references/workflow.md)

工作流展示了：
- 持久层声明
- 共享类型声明
- 业务层编写
- Controller 层编写
- Web 端编写