# @meta-1/nest-message

NestJS 消息服务模块，支持邮件发送和验证码管理。

## ✨ 功能特性

- 📧 **邮件服务** - 支持 AWS SES 和阿里云 DirectMail
- 🔐 **验证码管理** - 基于 Redis 的验证码发送和验证
- 🎯 **错误码定义** - 类型安全的错误码
- 🐛 **调试模式** - 开发环境支持固定验证码，无需真实发送邮件
- ⏱️ **频率限制** - 自动限制验证码发送频率（1分钟内不可重复发送）
- ⏰ **自动过期** - 验证码自动过期（默认 5 分钟）

## 📦 安装

```bash
npm install @meta-1/nest-message
# 或
pnpm add @meta-1/nest-message
# 或
yarn add @meta-1/nest-message
```

### peer 依赖

```bash
npm install @nestjs/common @nestjs-modules/ioredis ioredis
```

## 🚀 使用方法

### 1. 导入并配置模块

在 `AppModule` 中导入 `MessageModule`：

```typescript
import { Module } from '@nestjs/common';
import { RedisModule } from '@nestjs-modules/ioredis';
import { MessageModule } from '@meta-1/nest-message';

@Module({
  imports: [
    // 配置 Redis（MailCodeService 需要）
    RedisModule.forRoot({
      type: 'single',
      url: 'redis://localhost:6379',
    }),
    
    // 使用 AWS SES
    MessageModule.forRoot({
      debug: false,
      mail: {
        type: 'aws-ses',
        ses: {
          accessKeyId: 'your-access-key-id',
          accessKeySecret: 'your-secret-access-key',
          region: 'us-east-1',
          fromEmail: 'noreply@example.com'
        }
      }
    }),
    
    // 或使用阿里云 DirectMail
    MessageModule.forRoot({
      debug: false,
      mail: {
        type: 'alc-dm',
        dm: {
          accessKeyId: 'your-access-key-id',
          accessKeySecret: 'your-secret-access-key',
          region: 'cn-hangzhou', // 区域代码，如: cn-hangzhou, ap-southeast-1
          fromEmail: 'noreply@example.com',
          fromAlias: 'My Application' // 可选，发件人别名
        }
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
import { MessageModule, MessageConfig } from '@meta-1/nest-message';

interface AppConfig {
  message: MessageConfig;
}

async function bootstrap() {
  const configLoader = new ConfigLoader<AppConfig>({
    source: ConfigSourceType.FILE,
    filePath: './config/app.yaml',
  });
  
  const config = await configLoader.load();
  
  const app = await NestFactory.create(AppModule);
  
  // 动态配置 MessageModule
  app.get(MessageModule).forRoot(config.message);
  
  await app.listen(3000);
}
```

### 3. 邮件服务

#### 发送普通邮件

```typescript
import { Injectable } from '@nestjs/common';
import { MailService } from '@meta-1/nest-message';

@Injectable()
export class UserService {
  constructor(private readonly mailService: MailService) {}

  async sendWelcomeEmail(email: string) {
    const result = await this.mailService.sendEmail({
      to: email,
      subject: '欢迎加入我们！',
      html: '<h1>欢迎加入我们的平台！</h1><p>感谢您的注册。</p>',
      text: '欢迎加入我们的平台！感谢您的注册。', // 可选，纯文本版本
    });

    if (!result.success) {
      console.error('邮件发送失败:', result.error);
    } else {
      console.log('邮件发送成功，MessageId:', result.messageId);
    }
  }

  async sendEmailWithOptions(email: string) {
    await this.mailService.sendEmail({
      to: ['user1@example.com', 'user2@example.com'], // 支持多个收件人
      cc: 'cc@example.com', // 抄送
      bcc: 'bcc@example.com', // 密送
      replyTo: 'reply@example.com', // 回复地址
      subject: '重要通知',
      html: '<p>这是一封重要邮件</p>',
    });
  }
}
```

#### 发送验证码邮件

```typescript
import { Injectable } from '@nestjs/common';
import { MailService } from '@meta-1/nest-message';

@Injectable()
export class AuthService {
  constructor(private readonly mailService: MailService) {}

  async sendVerificationCodeEmail(email: string, code: string) {
    const result = await this.mailService.sendVerificationCode(
      email,
      code,
      10 // 过期时间（分钟），默认 10 分钟
    );

    if (!result.success) {
      throw new Error(`验证码邮件发送失败: ${result.error}`);
    }
  }
}
```

### 4. 验证码服务

#### 发送验证码

```typescript
import { Injectable } from '@nestjs/common';
import { MailCodeService, SendCodeDto } from '@meta-1/nest-message';
import { AppError } from '@meta-1/nest-common';
import { ErrorCode } from '@meta-1/nest-message';

@Injectable()
export class AuthService {
  constructor(private readonly mailCodeService: MailCodeService) {}

  async sendVerificationCode(email: string, action: string) {
    try {
      await this.mailCodeService.send({
        email,
        action, // 操作类型，如: 'register', 'login', 'reset-password'
      });
    } catch (error) {
      if (error instanceof AppError) {
        // 处理特定错误
        if (error.code === ErrorCode.VERIFICATION_CODE_SEND_TOO_FREQUENTLY.code) {
          throw new Error('验证码发送过于频繁，请稍后再试');
        }
      }
      throw error;
    }
  }
}
```

#### 验证验证码

```typescript
import { Injectable } from '@nestjs/common';
import { MailCodeService } from '@meta-1/nest-message';

@Injectable()
export class AuthService {
  constructor(private readonly mailCodeService: MailCodeService) {}

  async verifyCode(email: string, action: string, code: string): Promise<boolean> {
    const isValid = await this.mailCodeService.verify(email, action, code);
    
    if (!isValid) {
      throw new Error('验证码无效或已过期');
    }
    
    return true;
  }
}
```

#### 在 Controller 中使用

```typescript
import { Body, Controller, Post } from '@nestjs/common';
import { ApiOperation } from '@nestjs/swagger';
import { MailCodeService, SendCodeDto } from '@meta-1/nest-message';

@Controller('/api/mail/code')
export class MailCodeController {
  constructor(private readonly mailCodeService: MailCodeService) {}

  @Post('/send')
  @ApiOperation({ summary: '发送验证码' })
  async sendCode(@Body() body: SendCodeDto) {
    await this.mailCodeService.send(body);
    return { message: '验证码已发送' };
  }

  @Post('/verify')
  @ApiOperation({ summary: '验证验证码' })
  async verifyCode(
    @Body() body: { email: string; action: string; code: string }
  ) {
    const isValid = await this.mailCodeService.verify(
      body.email,
      body.action,
      body.code
    );
    
    if (!isValid) {
      throw new Error('验证码无效或已过期');
    }
    
    return { message: '验证成功' };
  }
}
```

### 5. 调试模式

在开发环境中，可以启用调试模式，避免真实发送邮件：

```typescript
MessageModule.forRoot({
  debug: true, // 启用调试模式
  code: '123456', // 固定验证码（可选）
  mail: {
    type: 'aws-ses',
    ses: {
      // ... 配置可以留空或使用测试配置
      accessKeyId: 'test',
      accessKeySecret: 'test',
      region: 'us-east-1',
      fromEmail: 'test@example.com'
    }
  }
})
```

**调试模式特性：**
- 不会真实发送邮件
- 验证码会输出到日志中
- 如果设置了 `code`，验证码固定为该值
- 验证码验证逻辑仍然正常工作

## 🚨 错误码

Message 模块定义了以下错误码（范围：1000-1199）：

### 验证码相关错误（1000-1099）

| 错误码常量 | Code | Message |
|-----------|------|---------|
| `VERIFICATION_CODE_STORAGE_FAILED` | 1000 | Verification code storage failed |
| `EMAIL_SENDING_FAILED` | 1001 | Email sending failed |
| `VERIFICATION_CODE_SEND_FAILED` | 1002 | Failed to send verification code |
| `VERIFICATION_CODE_EXPIRED` | 1003 | Verification code expired or does not exist |
| `VERIFICATION_CODE_INCORRECT` | 1004 | Verification code incorrect |
| `VERIFICATION_CODE_SEND_TOO_FREQUENTLY` | 1005 | Verification code sent too frequently, please try again later |

### 邮件服务错误（1100-1199）

| 错误码常量 | Code | Message |
|-----------|------|---------|
| `MAIL_SERVICE_NOT_CONFIGURED` | 1100 | Mail service not configured correctly |
| `MAIL_CONTENT_EMPTY` | 1101 | Email content cannot be empty |

### 使用示例

```typescript
import { AppError } from '@meta-1/nest-common';
import { ErrorCode } from '@meta-1/nest-message';

try {
  await mailCodeService.send({ email: 'test@example.com', action: 'register' });
} catch (error) {
  if (error instanceof AppError) {
    switch (error.code) {
      case ErrorCode.VERIFICATION_CODE_SEND_TOO_FREQUENTLY.code:
        // 处理发送频率限制错误
        break;
      case ErrorCode.EMAIL_SENDING_FAILED.code:
        // 处理邮件发送失败错误
        break;
    }
  }
}
```

## 📝 API 参考

### MailService

#### `sendEmail(options: SendEmailOptions)`

发送普通邮件。

**参数：**
- `to: string | string[]` - 收件人邮箱地址（支持多个）
- `subject: string` - 邮件主题
- `html?: string` - HTML 格式的邮件内容
- `text?: string` - 纯文本格式的邮件内容（至少需要提供 html 或 text 之一）
- `cc?: string | string[]` - 抄送地址（可选）
- `bcc?: string | string[]` - 密送地址（可选）
- `replyTo?: string | string[]` - 回复地址（可选）

**返回值：**
```typescript
Promise<{
  success: boolean;
  messageId?: string;
  error?: string;
}>
```

#### `sendVerificationCode(to: string, code: string, expiryMinutes?: number)`

发送验证码邮件（使用内置模板）。

**参数：**
- `to: string` - 收件人邮箱地址
- `code: string` - 验证码
- `expiryMinutes?: number` - 过期时间（分钟），默认 10 分钟

**返回值：**
```typescript
Promise<{
  success: boolean;
  messageId?: string;
  error?: string;
}>
```

### MailCodeService

#### `send(options: SendCodeDto)`

发送验证码。

**参数：**
- `options.email: string` - 收件人邮箱地址
- `options.action: string` - 操作类型（如: 'register', 'login', 'reset-password'）

**特性：**
- 自动生成 6 位数字验证码
- 验证码存储在 Redis 中，有效期 5 分钟
- 1 分钟内不可重复发送
- 验证码使用后自动删除（一次性使用）

**抛出：**
- `AppError` - 发送失败时抛出，错误码见上方错误码表

#### `verify(to: string, action: string, code: string)`

验证验证码。

**参数：**
- `to: string` - 邮箱地址
- `action: string` - 操作类型（必须与发送时一致）
- `code: string` - 验证码

**返回值：**
```typescript
Promise<boolean>
```

**特性：**
- 验证成功后自动删除验证码（一次性使用）
- 验证失败或验证码不存在返回 `false`

### SendCodeDto

验证码发送 DTO，基于 Zod Schema 验证。

**字段：**
- `email: string` - 邮箱地址（必须符合邮箱格式）
- `action: string` - 操作类型

## 🔧 配置说明

### MessageConfig

```typescript
interface MessageConfig {
  debug?: boolean; // 是否开启调试模式，默认 false
  code?: string; // 固定验证码（仅在 debug=true 时生效）
  mail: {
    type: 'aws-ses';
    ses: {
      accessKeyId: string;
      accessKeySecret: string;
      region: string;
      fromEmail: string;
    };
  } | {
    type: 'alc-dm';
    dm: {
      accessKeyId: string;
      accessKeySecret: string;
      region: string; // 区域代码，如: cn-hangzhou, ap-southeast-1
      fromEmail: string;
      fromAlias?: string; // 可选，发件人别名
    };
  };
}
```

### AWS SES 配置

- `accessKeyId` - AWS Access Key ID
- `accessKeySecret` - AWS Secret Access Key
- `region` - AWS 区域，如: `us-east-1`, `ap-southeast-1`
- `fromEmail` - 发件人邮箱地址（必须在 SES 中已验证）

### 阿里云 DirectMail 配置

- `accessKeyId` - 阿里云 AccessKey ID
- `accessKeySecret` - 阿里云 AccessKey Secret
- `region` - 区域代码，如: `cn-hangzhou`（华东1），`ap-southeast-1`（新加坡）
- `fromEmail` - 发件人邮箱地址（必须在阿里云邮件推送中配置）
- `fromAlias` - 发件人别名（可选）

## 📄 License

MIT
