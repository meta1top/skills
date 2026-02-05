# @meta-1/nest-assets

NestJS 资源管理模块，支持亚马逊 S3、阿里云 OSS 和 MinIO 对象存储。

## 功能特性

- ✅ 支持亚马逊 S3、阿里云 OSS 和 MinIO
- ✅ 预签名 URL 模式，客户端直传
- ✅ 统一接口，内部自动切换存储提供商
- ✅ 支持私桶和公桶
- ✅ 私桶访问自动签名授权
- ✅ 可配置签名有效期
- ✅ 自动生成唯一文件 Key（带时间戳和随机字符串）

## 安装

```bash
npm install @meta-1/nest-assets
```

所有必需的依赖（包括 AWS S3 SDK、阿里云 OSS SDK 和 MinIO SDK）会自动安装。

## 使用方法

### 1. 导入并配置模块

在 `AppModule` 中导入 `AssetsModule`：

```typescript
import { Module } from '@nestjs/common';
import { AssetsModule } from '@meta-1/nest-assets';
import { StorageProvider } from '@meta-1/nest-types';

@Module({
  imports: [
    // 使用 S3
    AssetsModule.forRoot({
      storage: {
        provider: StorageProvider.S3,
        publicBucket: 'my-public-bucket',
        privateBucket: 'my-private-bucket',
        expiresIn: '30m'  // 支持字符串（如 '30m', '1h', '2d'）或数字（毫秒）
      },
      s3: {
        region: 'us-east-1',
        accessKeyId: 'your-access-key-id',
        secretAccessKey: 'your-secret-access-key',
        endpoint: 'https://s3.amazonaws.com'  // 可选，用于兼容 S3 的服务
      }
    }),
    
    // 或使用阿里云 OSS
    AssetsModule.forRoot({
      storage: {
        provider: StorageProvider.OSS,
        publicBucket: 'my-public-bucket',
        privateBucket: 'my-private-bucket',
        expiresIn: '30m'
      },
      oss: {
        region: 'oss-cn-hangzhou',
        accessKeyId: 'your-access-key-id',
        accessKeySecret: 'your-secret-access-key'
      }
    }),
    
    // 或使用 MinIO
    AssetsModule.forRoot({
      storage: {
        provider: StorageProvider.MINIO,
        publicBucket: 'my-public-bucket',
        privateBucket: 'my-private-bucket',
        expiresIn: '30m'
      },
      minio: {
        endpoint: 'http://localhost:9000',  // 支持 http://host:port 或 https://host:port 格式
        accessKeyId: 'your-access-key-id',
        secretAccessKey: 'your-secret-access-key',
        useSSL: false,  // 可选，默认 false。也可以通过 endpoint 的协议自动推断
        region: 'us-east-1'  // 可选，默认 'us-east-1'
      }
    })
  ]
})
export class AppModule {}
```

### 2. 从配置文件加载

使用 `ConfigLoader` 从配置文件加载配置：

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { ConfigLoader, ConfigSourceType } from '@meta-1/nest-common';
import { AssetsModule, AssetsConfig } from '@meta-1/nest-assets';

interface AppConfig {
  assets: AssetsConfig;
}

async function bootstrap() {
  // 加载配置
  const loader = new ConfigLoader<AppConfig>({
    type: ConfigSourceType.LOCAL_YAML,
    filePath: './config/app.yaml'
  });
  
  const config = await loader.load();
  
  // 创建模块
  @Module({
    imports: [AssetsModule.forRoot(config.assets)]
  })
  class AppModule {}
  
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

配置文件示例 (`config/app.yaml`):

```yaml
assets:
  storage:
    provider: s3  # 's3' | 'oss' | 'minio'
    publicBucket: my-public-bucket
    privateBucket: my-private-bucket
    expiresIn: 30m
  s3:
    region: us-east-1
    accessKeyId: ${AWS_ACCESS_KEY_ID}
    secretAccessKey: ${AWS_SECRET_ACCESS_KEY}
    endpoint: ${AWS_S3_ENDPOINT}  # 可选
  oss:
    region: oss-cn-hangzhou
    accessKeyId: ${ALIYUN_ACCESS_KEY_ID}
    accessKeySecret: ${ALIYUN_ACCESS_KEY_SECRET}
  minio:
    endpoint: ${MINIO_ENDPOINT}  # 支持 http://host:port 或 https://host:port 格式
    accessKeyId: ${MINIO_ACCESS_KEY_ID}
    secretAccessKey: ${MINIO_SECRET_ACCESS_KEY}
    useSSL: false  # 可选，默认 false
    region: us-east-1  # 可选，默认 'us-east-1'
```

### 3. 创建 Controller

**重要：** `AssetsModule` 只提供 Service，Controller 需要由业务方实现。这样可以添加权限控制、日志等业务逻辑。

```typescript
// src/controllers/assets.controller.ts
import { Body, Controller, Post, UseGuards } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiBearerAuth } from '@nestjs/swagger';
import { JwtAuthGuard } from '@/guards/jwt-auth.guard';
import {
  AssetsService,
  PresignedUploadUrlRequestDto,
  PresignedUploadUrlResponseDto,
  PresignedDownloadUrlRequestDto,
  PresignedDownloadUrlResponseDto,
} from '@meta-1/nest-assets';
import { BucketType } from '@meta-1/nest-types';

@ApiTags('Assets')
@Controller('api/assets')
@UseGuards(JwtAuthGuard)  // ✅ 添加权限验证
@ApiBearerAuth()
export class AssetsController {
  constructor(private readonly assetsService: AssetsService) {}

  @Post('presigned-upload-url')
  @ApiOperation({ summary: '生成预签名上传 URL' })
  async generatePresignedUploadUrl(
    @Body() request: PresignedUploadUrlRequestDto
  ): Promise<PresignedUploadUrlResponseDto> {
    return this.assetsService.generatePresignedUploadUrl(request);
  }

  @Post('presigned-download-url')
  @ApiOperation({ summary: '生成预签名下载 URL' })
  async generatePresignedDownloadUrl(
    @Body() request: PresignedDownloadUrlRequestDto
  ): Promise<PresignedDownloadUrlResponseDto> {
    return this.assetsService.generatePresignedDownloadUrl(request);
  }
}
```

然后在 `AppModule` 中注册：

```typescript
@Module({
  imports: [AssetsModule.forRoot(config)],
  controllers: [AssetsController],  // ✅ 注册业务方的 Controller
})
export class AppModule {}
```

### 4. 在业务代码中使用服务

```typescript
import { Injectable } from '@nestjs/common';
import { AssetsService } from '@meta-1/nest-assets';
import { BucketType } from '@meta-1/nest-types';

@Injectable()
export class MyService {
  constructor(private readonly assetsService: AssetsService) {}

  async uploadFile() {
    // 生成预签名上传 URL
    const result = await this.assetsService.generatePresignedUploadUrl({
      fileName: 'example.jpg',
      bucketType: BucketType.PUBLIC,  // 或 BucketType.PRIVATE
      prefix: 'images',  // 可选，文件路径前缀
      headers: {
        'Content-Type': 'image/jpeg',  // 可选，指定文件类型
      }
    });

    // 返回给客户端
    return {
      signedUrl: result.signedUrl,  // 客户端用此 URL 上传文件
      url: result.url,               // 上传成功后的访问地址
      fileKey: result.fileKey,       // 文件唯一标识
      expiresAt: result.expiresAt,   // 过期时间（Unix 时间戳，毫秒）
    };
  }

  async downloadFile(fileUrl: string) {
    // 生成预签名下载 URL
    // 如果是公桶，直接返回原 URL
    // 如果是私桶，返回带签名的临时 URL
    const result = await this.assetsService.generatePresignedDownloadUrl({
      url: fileUrl,
    });

    return {
      downloadUrl: result.downloadUrl,  // 下载 URL
      expiresAt: result.expiresAt,       // 过期时间（0 表示不过期，仅公桶）
    };
  }
}
```

### 5. API 请求示例

```bash
# 生成上传 URL
POST /api/assets/presigned-upload-url
Content-Type: application/json

{
  "fileName": "example.jpg",
  "bucketType": "public",
  "prefix": "images",
  "headers": {
    "Content-Type": "image/jpeg"
  }
}

# 响应示例
{
  "fileName": "example.jpg",
  "signedUrl": "https://my-public-bucket.s3.us-east-1.amazonaws.com/images/1234567890_abc_example.jpg?X-Amz-Algorithm=...",
  "url": "https://my-public-bucket.s3.us-east-1.amazonaws.com/images/1234567890_abc_example.jpg",
  "fileKey": "images/1234567890_abc_example.jpg",
  "expiresAt": 1234567890000
}

# 生成下载 URL
POST /api/assets/presigned-download-url
Content-Type: application/json

{
  "url": "https://my-private-bucket.s3.us-east-1.amazonaws.com/images/1234567890_abc_example.jpg"
}

# 响应示例（私桶）
{
  "downloadUrl": "https://my-private-bucket.s3.us-east-1.amazonaws.com/images/1234567890_abc_example.jpg?X-Amz-Algorithm=...",
  "expiresAt": 1234567890000
}

# 响应示例（公桶）
{
  "downloadUrl": "https://my-public-bucket.s3.us-east-1.amazonaws.com/images/1234567890_abc_example.jpg",
  "expiresAt": 0
}
```

**注意：** 所有请求参数都会自动进行类型验证，Swagger UI 会显示完整的 Schema 信息。

## 客户端上传示例

```typescript
// 1. 获取预签名上传 URL
const response = await fetch('/api/assets/presigned-upload-url', {
  method: 'POST',
  headers: { 
    'Content-Type': 'application/json',
    'Authorization': 'Bearer your-token'
  },
  body: JSON.stringify({
    fileName: 'example.jpg',
    bucketType: 'public',
    prefix: 'images',
    headers: {
      'Content-Type': 'image/jpeg'
    }
  }),
});

const { signedUrl, url, fileKey, expiresAt } = await response.json();

// 2. 使用预签名 URL 上传文件
const file = document.getElementById('fileInput').files[0];
await fetch(signedUrl, {
  method: 'PUT',
  headers: { 
    'Content-Type': 'image/jpeg'  // 必须与请求时指定的 Content-Type 一致
  },
  body: file,
});

// 3. 上传成功后，使用 url 访问文件（公桶可直接访问）
console.log('文件地址:', url);

// 4. 如果是私桶，需要先获取预签名下载 URL
const downloadResponse = await fetch('/api/assets/presigned-download-url', {
  method: 'POST',
  headers: { 
    'Content-Type': 'application/json',
    'Authorization': 'Bearer your-token'
  },
  body: JSON.stringify({
    url: url
  }),
});

const { downloadUrl } = await downloadResponse.json();
console.log('下载地址:', downloadUrl);
```

## 桶类型说明

模块支持两种桶类型，通过 `BucketType` 枚举指定，系统会自动将文件存储到对应的桶中：

- **公桶 (BucketType.PUBLIC)**: 
  - 文件存储到 `storage.publicBucket` 配置的桶中
  - 上传需要签名
  - 访问不需要签名，可直接访问
  - 适用于公开资源（如网站图片、文档等）

- **私桶 (BucketType.PRIVATE)**:
  - 文件存储到 `storage.privateBucket` 配置的桶中
  - 上传需要签名
  - 访问需要签名，有时效性
  - 适用于私密资源（如用户私人文件、敏感数据等）

### 配置示例

```yaml
assets:
  storage:
    provider: s3
    publicBucket: my-public-bucket    # 公共文件存储桶
    privateBucket: my-private-bucket  # 私密文件存储桶
    expiresIn: 30m                    # 私桶签名有效期
```

当调用 API 时指定 `bucketType: "public"` 或 `bucketType: "private"`，文件会自动存储到对应的桶中。

## API 参考

### AssetsService

#### `generatePresignedUploadUrl(request: PresignedUploadUrlRequestDto)`

生成预签名上传 URL。

**请求参数:**
- `fileName` (string, 必填): 文件名
- `bucketType` (BucketType, 必填): 桶类型 (`BucketType.PUBLIC` | `BucketType.PRIVATE`)
- `prefix` (string, 可选): 文件路径前缀
- `headers` (Record<string, string>, 可选): HTTP 请求头，如 `{ "Content-Type": "image/jpeg" }`

**返回:**
- `fileName` (string): 文件名
- `signedUrl` (string): 预签名上传 URL
- `url` (string): 文件访问 URL（上传成功后的访问地址）
- `fileKey` (string): 文件唯一标识（自动生成，格式：`{prefix}/{timestamp}_{random}_{fileName}`）
- `expiresAt` (number): 过期时间（Unix 时间戳，毫秒）

#### `generatePresignedDownloadUrl(request: PresignedDownloadUrlRequestDto)`

生成预签名下载 URL。会自动判断是公桶还是私桶：
- **公桶**：直接返回原 URL（不过期）
- **私桶**：返回带签名的临时 URL（有过期时间）

**请求参数:**
- `url` (string, 必填): 文件完整 URL（从上传接口返回的 `url` 字段）

**返回:**
- `downloadUrl` (string): 下载 URL（公桶返回原 URL，私桶返回预签名 URL）
- `expiresAt` (number): 过期时间（Unix 时间戳，毫秒）。公桶为 0（不过期），私桶为实际过期时间

#### `isConfigured(): boolean`

检查服务是否已配置。

**返回:**
- `boolean`: 如果已配置返回 `true`，否则返回 `false`

## 文件 Key 生成规则

文件 Key 会自动生成，格式为：`{prefix}/{timestamp}_{random}_{fileName}`

- `prefix`: 可选的路径前缀
- `timestamp`: 当前时间戳（毫秒）
- `random`: 随机字符串（13 位）
- `fileName`: 原始文件名

例如：
- 无前缀：`1234567890_abc123def456_example.jpg`
- 有前缀：`images/1234567890_abc123def456_example.jpg`

这样可以确保文件名的唯一性，避免文件名冲突。

## 错误处理

模块使用统一的错误码系统，常见错误码：

- `3100`: 存储提供商不支持
- `3101`: S3 服务未配置
- `3102`: OSS 服务未配置
- `3103`: MinIO 服务未配置
- `3104`: S3 服务初始化失败
- `3105`: OSS 服务初始化失败
- `3106`: MinIO 服务初始化失败
- `3107`: S3 URL 格式无效
- `3108`: OSS URL 格式无效
- `3109`: MinIO URL 格式无效
- `3110`: S3 桶未找到
- `3111`: OSS 桶未找到
- `3112`: MinIO 桶未找到
- `3200`: 生成上传 URL 失败
- `3201`: 生成下载 URL 失败

## 许可证

MIT
