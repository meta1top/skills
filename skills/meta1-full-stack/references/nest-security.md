# @meta-1/nest-security

NestJS 安全和认证模块，提供会话管理、Token 管理、OTP 支持、加密解密等安全功能。

## ✨ 特性

- 🔐 **会话管理** - 基于 Redis 的会话存储和管理
- 🔑 **Token 管理** - JWT Token 生成、验证和解析
- 🔒 **OTP 支持** - 基于 TOTP 算法的一次性密码功能
- 🛡️ **认证拦截器** - 自动从请求头提取 token 并获取用户信息
- 🎯 **装饰器** - `@CurrentUser()` 和 `@Public()` 装饰器
- 🔄 **会话刷新** - 会话过期时间刷新和 payload 管理
- 🔐 **加密服务** - RSA 和 AES 加密解密支持
- 📝 **类型安全** - 完整的 TypeScript 类型支持

## 📦 安装

```bash
npm install @meta-1/nest-security
# 或
pnpm add @meta-1/nest-security
# 或
yarn add @meta-1/nest-security
```

### 依赖安装

```bash
npm install @nestjs/common @nestjs/platform-express @nestjs-modules/ioredis ioredis nestjs-i18n
```

## 🚀 快速开始

### 1. 模块配置

在 `AppModule` 中导入并配置 `SecurityModule`：

```typescript
import { Module } from '@nestjs/common';
import { SecurityModule } from '@meta-1/nest-security';

@Module({
  imports: [
    SecurityModule.forRoot({
      jwt: {
        secret: 'your-jwt-secret-key', // 必需：JWT 密钥
        expiresIn: '7d' // 可选：默认过期时间，如 '7d', '24h', '30m'
      },
      otp: {
        issuer: 'YourApp', // 必需：发行者名称
        debug: false, // 调试模式，开发环境可设为 true
        code: 123456, // 调试模式下的固定验证码
        expiresIn: '5m', // 密钥缓存过期时间
        secretSize: 32, // 可选：密钥大小，默认 32
        windowSize: 1, // 可选：时间窗口大小，默认 1
        secondPerSize: 30, // 可选：每个窗口的秒数，默认 30
        randomNumberAlgorithm: 'SHA1PRNG' // 可选：随机数算法，默认 SHA1PRNG
      }
    })
  ]
})
export class AppModule {}
```

### 2. Redis 配置

确保已配置 Redis 模块（使用 `@nestjs-modules/ioredis`）：

```typescript
import { Module } from '@nestjs/common';
import { RedisModule } from '@nestjs-modules/ioredis';

@Module({
  imports: [
    RedisModule.forRoot({
      type: 'single',
      url: 'redis://localhost:6379'
    }),
    SecurityModule.forRoot({
      // ... 配置
    })
  ]
})
export class AppModule {}
```

## 📖 使用指南

### TokenService - Token 管理

`TokenService` 提供 JWT Token 的创建、验证和解析功能。

```typescript
import { Injectable } from '@nestjs/common';
import { TokenService, CreateTokenData } from '@meta-1/nest-security';

@Injectable()
export class AuthService {
  constructor(private readonly tokenService: TokenService) {}

  // 创建 Token
  createToken(userId: string, username: string): string {
    const tokenData: CreateTokenData = {
      id: userId,
      username: username,
      expiresIn: '7d' // 可选，不传则使用配置中的默认值
    };
    return this.tokenService.create(tokenData);
  }

  // 验证 Token 是否有效
  validateToken(token: string): boolean {
    return this.tokenService.check(token);
  }

  // 解析 Token 获取 Payload
  parseToken(token: string) {
    try {
      const payload = this.tokenService.parse(token);
      console.log('User ID:', payload.jti);
      console.log('Username:', payload.sub);
      return payload;
    } catch (error) {
      // Token 已过期或无效
      throw error;
    }
  }

  // 刷新 Token（使用旧 Token 的数据创建新 Token）
  refreshToken(oldToken: string): string {
    return this.tokenService.refresh(oldToken, '7d'); // 可选：指定新的过期时间
  }

  // 快速提取用户 ID（不验证签名）
  extractUserId(token: string): string | null {
    return this.tokenService.extractUserId(token);
  }

  // 快速提取用户名（不验证签名）
  extractUsername(token: string): string | null {
    return this.tokenService.extractUsername(token);
  }
}
```

### SessionService - 会话管理

`SessionService` 提供基于 Redis 的会话管理功能。

```typescript
import { Injectable } from '@nestjs/common';
import { SessionService, SessionUser, TokenService } from '@meta-1/nest-security';

@Injectable()
export class AuthService {
  constructor(
    private readonly sessionService: SessionService,
    private readonly tokenService: TokenService
  ) {}

  // 用户登录
  async login(userId: string, username: string): Promise<string> {
    // 1. 创建 JWT Token
    const jwtToken = this.tokenService.create({
      id: userId,
      username: username,
      expiresIn: '7d'
    });

    // 2. 构建会话数据
    const sessionUser: SessionUser = {
      id: userId,
      username: username,
      jwtToken: jwtToken,
      expiresIn: '7d', // 使用 ms 格式字符串，如 '7d', '24h', '30m'
      authorities: ['ROLE_USER'], // 可选：用户权限列表
      apis: [ // 可选：用户可访问的 API 列表
        { path: '/api/users', method: 'GET' },
        { path: '/api/users/:id', method: 'PUT' }
      ],
      payload: { // 可选：自定义负载数据
        email: 'user@example.com',
        role: 'admin'
      }
    };

    // 3. 存储会话，返回 MD5 后的 token（客户端使用此 token）
    const tokenHash = await this.sessionService.login(sessionUser);
    return tokenHash;
  }

  // 用户登出
  async logout(tokenHash: string): Promise<void> {
    await this.sessionService.logout(tokenHash);
  }

  // 获取会话信息
  async getSession(tokenHash: string): Promise<SessionUser | null> {
    return await this.sessionService.get(tokenHash);
  }

  // 检查会话是否存在
  async checkSession(tokenHash: string): Promise<boolean> {
    return await this.sessionService.exists(tokenHash);
  }

  // 刷新会话过期时间
  async refreshSession(tokenHash: string): Promise<boolean> {
    const expiresInMs = 7 * 24 * 60 * 60 * 1000; // 7 天（毫秒）
    return await this.sessionService.refresh(tokenHash, expiresInMs);
  }

  // 获取会话的 payload 数据
  async getPayload<T>(tokenHash: string): Promise<T | null> {
    return await this.sessionService.getPayload<T>(tokenHash);
  }

  // 设置会话的 payload 数据
  async setPayload<T>(tokenHash: string, payload: T): Promise<boolean> {
    return await this.sessionService.setPayload(tokenHash, payload);
  }
}
```

### OTPService - 一次性密码

`OTPService` 提供基于 TOTP 算法的一次性密码功能，支持生成密钥、二维码和验证码验证。

```typescript
import { Injectable } from '@nestjs/common';
import { OTPService } from '@meta-1/nest-security';

@Injectable()
export class AuthService {
  constructor(private readonly otpService: OTPService) {}

  // 启用 MFA：生成密钥并获取二维码
  async enableMFA(username: string) {
    // 1. 生成密钥并缓存到 Redis（默认 5 分钟过期）
    const secret = await this.otpService.getSecret(username);

    // 2. 生成二维码字符串（用于前端生成二维码）
    const qrCode = this.otpService.getQRCode(username, secret);

    return {
      secret, // 密钥（Base32 编码）
      qrCode // 二维码字符串，格式：otpauth://totp/...
    };
  }

  // 验证用户在启用 MFA 时输入的验证码
  async verifyMFASetup(username: string, code: string): Promise<boolean> {
    // 1. 从 Redis 获取缓存的密钥
    const secret = await this.otpService.getCachedSecret(username);
    
    if (!secret) {
      throw new Error('密钥已过期，请重新生成');
    }

    // 2. 验证验证码
    const isValid = this.otpService.check(secret, code);

    if (isValid) {
      // 验证成功，保存密钥到用户表，并删除缓存
      await this.saveSecretToUser(username, secret);
      await this.otpService.deleteCachedSecret(username);
    }

    return isValid;
  }

  // 验证登录时的 OTP 验证码
  async verifyOTP(username: string, code: string): Promise<boolean> {
    // 从用户表获取保存的密钥
    const secret = await this.getSecretFromUser(username);
    
    if (!secret) {
      throw new Error('用户未启用 MFA');
    }

    // 验证验证码
    return this.otpService.check(secret, code);
  }

  // 取消启用 MFA：删除缓存的密钥
  async cancelMFA(username: string): Promise<void> {
    await this.otpService.deleteCachedSecret(username);
  }

  private async saveSecretToUser(username: string, secret: string) {
    // 保存密钥到用户表的逻辑
  }

  private async getSecretFromUser(username: string): Promise<string | null> {
    // 从用户表获取密钥的逻辑
    return null;
  }
}
```

### EncryptService - 加密解密

`EncryptService` 提供 RSA 和 AES 加密解密功能。

```typescript
import { Injectable } from '@nestjs/common';
import { EncryptService } from '@meta-1/nest-security';

@Injectable()
export class AuthService {
  constructor(private readonly encryptService: EncryptService) {}

  // RSA 解密（兼容前端 JSEncrypt）
  decryptPassword(encryptedPassword: string, privateKey: string): string {
    // encryptedPassword: 前端使用公钥加密的 Base64 字符串
    // privateKey: RSA 私钥（PEM 格式）
    return this.encryptService.decryptWithPrivateKey(encryptedPassword, privateKey);
  }

  // AES 加密
  encryptWithAES(text: string, aesKey: string): string {
    // aesKey: AES 密钥（32 字节）
    // 返回格式: iv:encryptedData（均为 Base64 编码）
    return this.encryptService.encryptWithAES(text, aesKey);
  }

  // AES 解密
  decryptWithAES(encryptedText: string, aesKey: string): string {
    // encryptedText: 格式为 iv:encryptedData（均为 Base64 编码）
    // aesKey: AES 密钥（32 字节）
    return this.encryptService.decryptWithAES(encryptedText, aesKey);
  }
}
```

### AuthInterceptor - 认证拦截器

`AuthInterceptor` 会自动从请求头中提取 token 并获取用户信息，存储到 `request.user` 中。

**注意**：`SecurityModule.forRoot()` 会自动注册 `AuthInterceptor`，无需手动配置。

拦截器会：
1. 从 `Authorization` header 中提取 Bearer token（MD5 hash）
2. 通过 `SessionService` 获取会话信息
3. 将用户信息存储到 `request.user` 中

### 装饰器使用

#### @CurrentUser() - 获取当前用户

```typescript
import { Controller, Get, UnauthorizedException } from '@nestjs/common';
import { CurrentUser, SessionUser } from '@meta-1/nest-security';

@Controller('users')
export class UserController {
  @Get('profile')
  getProfile(@CurrentUser() user: SessionUser | undefined) {
    if (!user) {
      throw new UnauthorizedException('Please login first');
    }
    
    return {
      id: user.id,
      username: user.username,
      authorities: user.authorities,
      payload: user.payload
    };
  }

  @Get('info')
  getUserInfo(@CurrentUser() user: SessionUser) {
    // 如果确定用户已登录，可以直接使用
    return user;
  }
}
```

#### @Public() - 标记公开路由

使用 `@Public()` 装饰器标记不需要认证的接口：

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { Public } from '@meta-1/nest-security';

@Controller('auth')
export class AuthController {
  @Public()
  @Post('login')
  async login(@Body() loginDto: LoginDto) {
    // 不需要鉴权，任何人都可以访问
    return await this.authService.login(loginDto);
  }

  @Public()
  @Post('register')
  async register(@Body() registerDto: RegisterDto) {
    return await this.authService.register(registerDto);
  }

  @Post('logout')
  async logout(@CurrentUser() user: SessionUser) {
    // 需要鉴权（没有 @Public 装饰器）
    return await this.authService.logout(user);
  }
}
```

在 Guard 中使用 `@Public()` 装饰器：

```typescript
import { Injectable, CanActivate, ExecutionContext, UnauthorizedException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from '@meta-1/nest-security';

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // 检查是否有 @Public 装饰器
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (isPublic) {
      return true; // 跳过鉴权
    }

    const request = context.switchToHttp().getRequest();
    const user = request.user;

    if (!user) {
      throw new UnauthorizedException('Please login first');
    }

    return true;
  }
}
```

## 📝 API 参考

### TokenService

- `create(data: CreateTokenData): string` - 创建 JWT Token
- `check(token: string): boolean` - 验证 Token 是否有效
- `parse(token: string): TokenPayload` - 解析 Token 获取 Payload（会验证签名和过期时间）
- `refresh(token: string, expiresIn?: string): string` - 刷新 Token
- `extractUserId(token: string): string | null` - 提取用户 ID（不验证签名）
- `extractUsername(token: string): string | null` - 提取用户名（不验证签名）

### SessionService

- `login(user: SessionUser): Promise<string>` - 用户登录，返回 MD5 后的 token
- `logout(tokenHash: string): Promise<void>` - 用户登出
- `get(tokenHash: string): Promise<SessionUser | null>` - 获取会话信息
- `exists(tokenHash: string): Promise<boolean>` - 检查会话是否存在
- `refresh(tokenHash: string, expiresIn: number): Promise<boolean>` - 刷新会话过期时间（毫秒）
- `getPayload<T>(tokenHash: string): Promise<T | null>` - 获取会话的 payload 数据
- `setPayload<T>(tokenHash: string, payload: T): Promise<boolean>` - 设置会话的 payload 数据

### OTPService

- `generateSecret(): string` - 生成 OTP 密钥（Base32 编码）
- `getSecret(username: string): Promise<string>` - 生成密钥并缓存到 Redis
- `getCachedSecret(username: string): Promise<string | null>` - 从 Redis 获取缓存的密钥
- `deleteCachedSecret(username: string): Promise<void>` - 删除 Redis 中缓存的密钥
- `getQRCode(user: string, secret: string): string` - 生成二维码字符串
- `check(secret: string, code: string): boolean` - 验证 OTP 验证码

### EncryptService

- `decryptWithPrivateKey(encryptedText: string, privateKey: string): string` - RSA 解密（兼容 JSEncrypt）
- `encryptWithAES(text: string, aesKey: string): string` - AES-256-CBC 加密
- `decryptWithAES(encryptedText: string, aesKey: string): string` - AES-256-CBC 解密

### 类型定义

```typescript
// Token 相关
interface CreateTokenData {
  id: string; // 用户 ID
  username: string; // 用户名
  expiresIn?: ms.StringValue; // 可选：过期时间
}

interface TokenPayload {
  jti: string; // JWT ID（用户 ID）
  sub: string; // 用户名
  iat: number; // 签发时间（秒级时间戳）
  exp: number; // 过期时间（秒级时间戳）
  [key: string]: unknown; // 额外的自定义数据
}

// 会话相关
interface SessionUser<T = unknown> {
  id: string; // 用户 ID
  username: string; // 用户名
  jwtToken: string; // JWT Token
  expiresIn: ms.StringValue; // 过期时间
  authorities?: string[]; // 可选：用户权限列表
  apis?: SessionApi[]; // 可选：用户可访问的 API 列表
  payload?: T; // 可选：自定义负载数据
}

interface SessionApi {
  path: string; // API 路径
  method: string; // HTTP 方法
}
```

## 🔧 配置选项

### SecurityConfig

```typescript
interface SecurityConfig {
  jwt: {
    secret: string; // 必需：JWT 密钥
    expiresIn?: ms.StringValue; // 可选：默认过期时间，如 '7d', '24h', '30m'
  };
  otp: {
    issuer: string; // 必需：发行者名称
    debug: boolean; // 调试模式
    code: number; // 调试模式下的固定验证码
    expiresIn: ms.StringValue; // 密钥缓存过期时间
    secretSize?: number; // 可选：密钥大小，默认 32
    windowSize?: number; // 可选：时间窗口大小，默认 1
    secondPerSize?: number; // 可选：每个窗口的秒数，默认 30
    randomNumberAlgorithm?: string; // 可选：随机数算法，默认 SHA1PRNG
  };
}
```

## 🔐 安全最佳实践

1. **使用强密钥** - JWT secret 应该足够复杂且定期更换
2. **合理设置过期时间** - 根据业务需求设置合适的会话和 token 过期时间
3. **保护敏感信息** - 不要在 token 中存储敏感信息
4. **使用 HTTPS** - 生产环境必须使用 HTTPS
5. **限制 OTP 尝试次数** - 防止暴力破解
6. **记录安全事件** - 记录登录、登出、token 刷新等安全事件
7. **定期清理过期会话** - Redis 会自动清理过期 key，但建议监控内存使用
8. **保护私钥** - RSA 私钥和 AES 密钥应该安全存储，不要硬编码在代码中

## 📄 许可证

MIT

## 🤝 贡献

欢迎贡献！请随时提交 Pull Request。
