# @meta-1/nest-types

共享类型定义和 Zod Schema，用于前后端类型共享和数据验证。

## ✨ 特性

- 🛡️ **类型安全** - 完整的 TypeScript 类型定义
- ✅ **数据验证** - 基于 Zod 的 Schema 验证
- 🔄 **共享复用** - 前后端共享类型定义
- 📝 **自动推导** - 从 Schema 自动推导 TypeScript 类型
- 🎯 **类型检查** - 编译时类型检查

## 📦 安装

```bash
npm install @meta-1/nest-types
# 或
pnpm add @meta-1/nest-types
# 或
yarn add @meta-1/nest-types
```

### 依赖安装

```bash
npm install zod
```

## 🚀 快速开始

### 基础导入

```typescript
import {
  // AI 相关
  InvokeAgentSchema,
  AiConfigSchema,
  AddDocumentSchema,
  SearchDocumentSchema,
  
  // Assets 相关
  StorageProvider,
  BucketType,
  PresignedUploadUrlRequestSchema,
  PresignedDownloadUrlRequestSchema,
  
  // Common 相关
  PageRequestSchema,
  RestResult,
  PageData,
  
  // Message 相关
  SendCodeSchema,
  
  // Zod 工具
  coerceNumber,
  coerceNumberOptional,
} from '@meta-1/nest-types';
```

## 📚 API 文档

### AI 相关类型

#### InvokeAgentSchema

Agent 调用请求 Schema。

```typescript
import { InvokeAgentSchema, InvokeAgent } from '@meta-1/nest-types';

// 验证数据
const data: InvokeAgent = InvokeAgentSchema.parse({
  message: '你好，请介绍一下自己',
});

// 或使用 safeParse
const result = InvokeAgentSchema.safeParse({
  message: '',
});

if (!result.success) {
  console.error(result.error.errors);
}
```

**类型定义：**
- `message`: string - 用户输入的消息内容（必填，最小长度 1）

#### AiConfigSchema

AI 配置 Schema，包含模型配置、向量存储配置、嵌入模型配置等。

```typescript
import { AiConfigSchema, AiConfig } from '@meta-1/nest-types';

const config: AiConfig = AiConfigSchema.parse({
  model: {
    name: 'gpt-4',
    apiKey: 'your-api-key',
    apiBaseUrl: 'https://api.openai.com',
    temperature: 0.7,
    maxTokens: 2000,
  },
  vectorStore: {
    name: 'qdrant',
    collectionName: 'documents',
    options: {
      url: 'http://localhost:6333',
      apiKey: 'optional-api-key',
    },
  },
  embeddings: {
    name: 'text-embedding-ada-002',
    apiKey: 'your-api-key',
    apiBaseUrl: 'https://api.openai.com',
  },
  textSplitter: {
    chunkSize: 1000,
    chunkOverlap: 100,
  },
  mcp: {
    name: 'mcp-server',
    version: '1.0.0',
  },
});
```

#### AddDocumentSchema

添加文档到向量存储的请求 Schema。

```typescript
import { AddDocumentSchema, AddDocument } from '@meta-1/nest-types';

const document: AddDocument = AddDocumentSchema.parse({
  id: 'doc-001',
  content: '这是文档内容...',
  metadata: {
    title: '文档标题',
    author: '作者名',
  },
});
```

#### DeleteDocumentSchema

删除文档请求 Schema。

```typescript
import { DeleteDocumentSchema, DeleteDocument } from '@meta-1/nest-types';

const request: DeleteDocument = DeleteDocumentSchema.parse({
  documentId: 'doc-001',
});
```

#### SearchDocumentSchema

搜索文档请求 Schema。

```typescript
import { SearchDocumentSchema, SearchDocument } from '@meta-1/nest-types';

const searchRequest: SearchDocument = SearchDocumentSchema.parse({
  message: '查询关键词',
  k: 5, // 可选，默认 4
});
```

#### SearchResultSchema

搜索结果 Schema。

```typescript
import { SearchResultSchema, SearchResult } from '@meta-1/nest-types';

const result: SearchResult = SearchResultSchema.parse({
  id: 'doc-001',
  chunkId: 'chunk-001',
  content: '匹配的文档内容',
  metadata: {
    title: '文档标题',
  },
});
```

### Assets 相关类型

#### StorageProvider

存储提供商枚举。

```typescript
import { StorageProvider } from '@meta-1/nest-types';

const provider = StorageProvider.S3; // 's3' | 'oss' | 'minio'
```

**可用值：**
- `StorageProvider.S3` - AWS S3
- `StorageProvider.OSS` - 阿里云 OSS
- `StorageProvider.MINIO` - MinIO

#### BucketType

桶类型枚举。

```typescript
import { BucketType } from '@meta-1/nest-types';

const bucketType = BucketType.PRIVATE; // 'private' | 'public'
```

**可用值：**
- `BucketType.PRIVATE` - 私有桶，需要签名访问
- `BucketType.PUBLIC` - 公共桶，可直接访问

#### PresignedUploadUrlRequestSchema

预签名上传 URL 请求 Schema。

```typescript
import {
  PresignedUploadUrlRequestSchema,
  PresignedUploadUrlRequestData,
  BucketType,
} from '@meta-1/nest-types';

const request: PresignedUploadUrlRequestData =
  PresignedUploadUrlRequestSchema.parse({
    fileName: 'example.jpg',
    bucketType: BucketType.PRIVATE,
    prefix: 'uploads/2024/',
    headers: {
      'Content-Type': 'image/jpeg',
      'Content-Length': '1024',
    },
  });
```

#### PresignedUploadUrlResponseSchema

预签名上传 URL 响应 Schema。

```typescript
import {
  PresignedUploadUrlResponseSchema,
  PresignedUploadUrlResponseData,
} from '@meta-1/nest-types';

const response: PresignedUploadUrlResponseData =
  PresignedUploadUrlResponseSchema.parse({
    fileName: 'example.jpg',
    signedUrl: 'https://s3.amazonaws.com/bucket/example.jpg?signature=...',
    url: 'https://s3.amazonaws.com/bucket/example.jpg',
    fileKey: 'uploads/2024/example.jpg',
    expiresAt: 1704067200000,
  });
```

#### PresignedDownloadUrlRequestSchema

预签名下载 URL 请求 Schema。

```typescript
import {
  PresignedDownloadUrlRequestSchema,
  PresignedDownloadUrlRequestData,
} from '@meta-1/nest-types';

const request: PresignedDownloadUrlRequestData =
  PresignedDownloadUrlRequestSchema.parse({
    url: 'https://s3.amazonaws.com/bucket/example.jpg',
  });
```

#### PresignedDownloadUrlResponseSchema

预签名下载 URL 响应 Schema。

```typescript
import {
  PresignedDownloadUrlResponseSchema,
  PresignedDownloadUrlResponseData,
} from '@meta-1/nest-types';

const response: PresignedDownloadUrlResponseData =
  PresignedDownloadUrlResponseSchema.parse({
    downloadUrl: 'https://s3.amazonaws.com/bucket/example.jpg?signature=...',
    expiresAt: 1704067200000, // Unix 时间戳（毫秒），公桶为 0
  });
```

### Common 相关类型

#### PageRequestSchema

分页请求 Schema，自动将查询参数字符串转换为数字。

```typescript
import { PageRequestSchema, PageRequestData } from '@meta-1/nest-types';

// 从查询参数解析（自动转换字符串为数字）
const pageRequest: PageRequestData = PageRequestSchema.parse({
  page: '1', // 字符串会自动转换为数字
  size: '20',
  keyword: '搜索关键词',
});

// 或直接使用数字
const pageRequest2: PageRequestData = PageRequestSchema.parse({
  page: 1,
  size: 20,
});
```

**类型定义：**
- `page`: number - 页码（最小 1，默认 1）
- `size`: number - 每页数量（最小 1，最大 100，默认 20）
- `keyword`: string - 关键词搜索（可选）

#### PageDataSchema

分页响应 Schema 工厂函数，用于创建特定数据类型的分页响应 Schema。

```typescript
import { PageDataSchema } from '@meta-1/nest-types';
import { z } from 'zod';

// 定义单个数据项的 Schema
const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
});

// 创建分页响应 Schema
const UserPageDataSchema = PageDataSchema(UserSchema);

// 使用
const pageData = UserPageDataSchema.parse({
  total: 100,
  data: [
    { id: '1', name: '用户1', email: 'user1@example.com' },
    { id: '2', name: '用户2', email: 'user2@example.com' },
  ],
});
```

#### RestResult

REST API 通用响应结果类型。

```typescript
import { RestResult } from '@meta-1/nest-types';

const successResponse: RestResult<{ id: string; name: string }> = {
  success: true,
  code: 200,
  message: '操作成功',
  data: {
    id: '1',
    name: '示例数据',
  },
};

const errorResponse: RestResult<null> = {
  success: false,
  code: 400,
  message: '请求参数错误',
};
```

#### PageData

分页数据类型。

```typescript
import { PageData } from '@meta-1/nest-types';

const pageData: PageData<{ id: string; name: string }> = {
  total: 100,
  data: [
    { id: '1', name: '项目1' },
    { id: '2', name: '项目2' },
  ],
};
```

#### PageResult

分页响应结果类型。

```typescript
import { PageResult } from '@meta-1/nest-types';

const pageResult: PageResult<{ id: string; name: string }> = {
  success: true,
  code: 200,
  message: '查询成功',
  data: {
    total: 100,
    data: [
      { id: '1', name: '项目1' },
      { id: '2', name: '项目2' },
    ],
  },
};
```

#### PageRequest

分页请求参数类型。

```typescript
import { PageRequest } from '@meta-1/nest-types';

const pageRequest: PageRequest = {
  page: 1,
  size: 20,
  keyword: '搜索关键词',
};

// 扩展额外的查询参数
const extendedPageRequest: PageRequest<{ status: string }> = {
  page: 1,
  size: 20,
  keyword: '搜索关键词',
  status: 'active',
};
```

### Message 相关类型

#### SendCodeSchema

发送验证码请求 Schema。

```typescript
import { SendCodeSchema, SendCodeData } from '@meta-1/nest-types';

const request: SendCodeData = SendCodeSchema.parse({
  email: 'user@example.com',
  action: 'register',
});
```

**类型定义：**
- `email`: string - 邮箱地址（必填，需符合邮箱格式）
- `action`: string - 操作类型（必填）

### Zod 工具函数

#### coerceNumber

自定义的数字转换函数，将空字符串、null、undefined 转换为 undefined。

```typescript
import { coerceNumber } from '@meta-1/nest-types';
import { z } from 'zod';

const NumberSchema = z.object({
  value: coerceNumber(),
});

// 空字符串、null、undefined 都会转换为 undefined
NumberSchema.parse({ value: '' }); // { value: undefined }
NumberSchema.parse({ value: null }); // { value: undefined }
NumberSchema.parse({ value: '123' }); // { value: 123 }
```

#### coerceNumberOptional

可选的数字转换函数。

```typescript
import { coerceNumberOptional } from '@meta-1/nest-types';
import { z } from 'zod';

const OptionalNumberSchema = z.object({
  value: coerceNumberOptional(),
});

OptionalNumberSchema.parse({ value: '' }); // { value: undefined }
OptionalNumberSchema.parse({ value: '123' }); // { value: 123 }
OptionalNumberSchema.parse({}); // { value: undefined }
```

## 💡 使用示例

### 后端使用（NestJS）

#### DTO 验证

```typescript
import { Controller, Post, Body, Query } from '@nestjs/common';
import {
  SendCodeSchema,
  SendCodeData,
  PageRequestSchema,
  PageRequestData,
} from '@meta-1/nest-types';

@Controller('mail')
export class MailController {
  @Post('send-code')
  async sendCode(@Body() dto: SendCodeData) {
    // 验证数据
    const validatedData = SendCodeSchema.parse(dto);
    return await this.mailService.sendCode(validatedData);
  }

  @Get('list')
  async list(@Query() query: PageRequestData) {
    // 自动转换查询参数
    const pageRequest = PageRequestSchema.parse(query);
    return await this.mailService.list(pageRequest);
  }
}
```

#### 使用 nestjs-zod 集成

```typescript
import { createZodDto } from 'nestjs-zod';
import { SendCodeSchema } from '@meta-1/nest-types';

export class SendCodeDto extends createZodDto(SendCodeSchema) {}

@Controller('mail')
export class MailController {
  @Post('send-code')
  async sendCode(@Body() dto: SendCodeDto) {
    // 自动验证
    return await this.mailService.sendCode(dto);
  }
}
```

### 前端使用（Next.js/React）

#### 表单验证

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { SendCodeSchema, SendCodeData } from '@meta-1/nest-types';

export function SendCodeForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<SendCodeData>({
    resolver: zodResolver(SendCodeSchema),
  });

  const onSubmit = async (data: SendCodeData) => {
    await fetch('/api/mail/send-code', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} type="email" />
      {errors.email && <span>{errors.email.message}</span>}

      <input {...register('action')} />
      {errors.action && <span>{errors.action.message}</span>}

      <button type="submit">发送验证码</button>
    </form>
  );
}
```

#### API 调用

```typescript
import {
  PresignedUploadUrlRequestSchema,
  PresignedUploadUrlRequestData,
  BucketType,
} from '@meta-1/nest-types';

async function uploadFile(file: File) {
  // 验证请求数据
  const requestData: PresignedUploadUrlRequestData =
    PresignedUploadUrlRequestSchema.parse({
      fileName: file.name,
      bucketType: BucketType.PRIVATE,
      headers: {
        'Content-Type': file.type,
        'Content-Length': file.size.toString(),
      },
    });

  // 获取预签名 URL
  const response = await fetch('/api/assets/presigned-upload-url', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(requestData),
  });

  const { signedUrl, url } = await response.json();

  // 使用预签名 URL 上传文件
  await fetch(signedUrl, {
    method: 'PUT',
    body: file,
    headers: requestData.headers,
  });

  return url;
}
```

#### 分页查询

```typescript
import { PageRequestSchema, PageRequestData } from '@meta-1/nest-types';

async function fetchUsers(page: number, size: number, keyword?: string) {
  const params: PageRequestData = PageRequestSchema.parse({
    page: page.toString(), // 从 URL 查询参数来的字符串
    size: size.toString(),
    keyword,
  });

  const queryString = new URLSearchParams({
    page: params.page.toString(),
    size: params.size.toString(),
    ...(params.keyword && { keyword: params.keyword }),
  });

  const response = await fetch(`/api/users?${queryString}`);
  return response.json();
}
```

## 📖 最佳实践

1. **类型安全** - 始终使用 Schema 验证数据，确保类型安全
2. **前后端共享** - 确保前后端使用相同的 Schema 定义
3. **错误处理** - 使用 `safeParse()` 进行验证，优雅处理错误
4. **边界验证** - 在数据进入系统的边界进行验证（API 入口、表单提交等）
5. **类型推导** - 使用 `z.infer<typeof Schema>` 自动推导类型，避免重复定义

## 📄 许可证

MIT

## 🤝 贡献

欢迎贡献！请随时提交 Pull Request。
