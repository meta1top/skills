# @meta-1/nest-common

NestJS 应用的通用工具库，提供缓存、分布式锁、国际化、错误处理、事务管理等功能。

## ✨ 功能特性

- 🎯 **缓存装饰器** - 类似 Spring Boot 的 `@Cacheable` 和 `@CacheEvict` 装饰器，支持 Redis
- 🔒 **分布式锁** - `@WithLock` 装饰器，基于 Redis 实现分布式锁
- ❄️ **雪花ID生成器** - `@SnowflakeId` 装饰器，自动生成分布式唯一ID
- 🔄 **事务管理** - `@Transactional` 装饰器，自动管理数据库事务
- 🌍 **国际化支持** - `@I18n` 装饰器和 `I18nContext`，支持命名空间和自动采集
- ⚡ **响应拦截器** - 统一的 API 响应格式
- 🚨 **错误处理** - 全局错误过滤器，支持自定义错误码
- 📄 **分页DTO** - 标准化的分页请求和响应 DTO
- 🌐 **HTTP服务** - 增强的 HTTP 客户端，支持重试和文件下载
- ⚙️ **配置加载器** - 支持从本地 YAML 文件或 Nacos 配置中心加载配置
- ✅ **验证工具** - `createI18nZodDto` 自动采集验证错误消息
- 📚 **Swagger工具** - 分页响应的 Swagger Schema 生成工具

## 📦 安装

```bash
npm install @meta-1/nest-common
# 或
pnpm add @meta-1/nest-common
# 或
yarn add @meta-1/nest-common
```

###  peer 依赖

```bash
npm install @nestjs/common @nestjs/platform-express nestjs-i18n ioredis nacos yaml
```

## 🚀 使用指南

### 1. 模块导入

```typescript
import { CommonModule } from '@meta-1/nest-common';

@Module({
  imports: [CommonModule],
})
export class AppModule {}
```

`CommonModule` 会自动注册以下全局功能：
- `ResponseInterceptor` - 统一响应格式
- `ErrorsFilter` - 全局错误处理
- `ZodValidationPipe` - Zod 验证管道
- `HttpService` - HTTP 客户端服务

### 2. 缓存装饰器

#### 初始化

```typescript
import { CacheableInitializer } from '@meta-1/nest-common';
import { InjectRedis } from '@nestjs-modules/ioredis';
import { Redis } from 'ioredis';

@Module({
  providers: [
    CacheableInitializer,
    {
      provide: 'REDIS_CLIENT',
      useFactory: () => {
        // 返回 Redis 实例
        return new Redis({
          host: 'localhost',
          port: 6379,
        });
      },
    },
  ],
})
export class AppModule {}
```

#### 使用示例

```typescript
import { CacheableService, Cacheable, CacheEvict } from '@meta-1/nest-common';

@CacheableService()
@Injectable()
export class UserService {
  // 缓存结果，默认 TTL 300 秒
  @Cacheable({ key: 'user:#{0}', ttl: 300 })
  async getUserById(id: string) {
    return await this.userRepository.findOne({ where: { id } });
  }

  // 使用对象属性作为缓存键
  @Cacheable({ key: 'user:#{user.id}:profile', ttl: 600 })
  async getUserProfile(user: { id: string }) {
    return await this.userRepository.findProfile(user.id);
  }

  // 清除特定缓存
  @CacheEvict({ key: 'user:#{0}' })
  async updateUser(id: string, data: UpdateUserDto) {
    return await this.userRepository.update(id, data);
  }

  // 清除所有缓存
  @CacheEvict({ allEntries: true })
  async resetAllUsers() {
    return await this.userRepository.clear();
  }
}
```

**缓存键占位符：**
- `#{0}`, `#{1}`, `#{2}` - 使用参数位置索引
- `#{user.id}`, `#{name}` - 使用第一个参数的属性（等同于 `#{0.user.id}`）
- `#{1.book.title}` - 使用指定参数的路径属性

### 3. 分布式锁装饰器

#### 初始化

```typescript
import { LockInitializer } from '@meta-1/nest-common';
import { Redis } from 'ioredis';

@Module({
  providers: [
    LockInitializer,
    {
      provide: 'REDIS_CLIENT',
      useFactory: () => new Redis({ host: 'localhost', port: 6379 }),
    },
  ],
})
export class AppModule {}
```

#### 使用示例

```typescript
import { WithLock } from '@meta-1/nest-common';

@Injectable()
export class OrderService {
  // 防止同一用户重复创建订单
  @WithLock({
    key: 'order:create:#{0}',
    ttl: 10000,        // 锁过期时间：10秒
    waitTimeout: 3000, // 等待锁的超时时间：3秒
  })
  async createOrder(userId: string, items: OrderItem[]) {
    // 同一用户的订单创建操作会被加锁
    return await this.orderRepository.save({ userId, items });
  }

  // 防止重复支付
  @WithLock({
    key: 'payment:#{0}',
    ttl: 30000,
    waitTimeout: 0, // 不等待，立即失败
    errorMessage: '订单正在支付中，请勿重复提交',
  })
  async processPayment(orderId: string) {
    // 支付逻辑
  }

  // 使用对象属性作为锁键
  @WithLock({
    key: 'inventory:#{product.id}',
    ttl: 5000,
  })
  async reduceInventory(product: { id: string; quantity: number }) {
    // 库存扣减逻辑
  }
}
```

**配置选项：**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `key` | `string` | 必填 | 锁的键名，支持占位符 |
| `ttl` | `number` | `30000` | 锁的过期时间（毫秒） |
| `waitTimeout` | `number` | `5000` | 等待锁的超时时间（毫秒），0 表示不等待 |
| `retryInterval` | `number` | `100` | 重试获取锁的间隔（毫秒） |
| `errorMessage` | `string` | `'操作正在处理中，请稍后重试'` | 获取锁失败时的错误提示 |

### 4. 雪花ID生成器

```typescript
import { SnowflakeId } from '@meta-1/nest-common';
import { Entity, Column } from 'typeorm';

@Entity()
export class User {
  @SnowflakeId()
  id: string; // 自动生成分布式唯一ID

  @Column()
  name: string;
}

// 使用时无需手动设置 ID
const user = new User();
user.name = 'Alice';
await repository.save(user); // ID 自动生成
```

**环境变量配置：**

```bash
SNOWFLAKE_WORKER_ID=0      # 0-31
SNOWFLAKE_DATACENTER_ID=0 # 0-31
```

**特性：**
- 并发安全：通过 Promise 链确保多实例同时插入时 ID 不重复
- 分布式唯一：支持多机部署
- 时间有序：ID 按生成时间递增
- 高性能：单机每毫秒可生成 4096 个唯一 ID

### 5. 事务装饰器

```typescript
import { Transactional } from '@meta-1/nest-common';

@Injectable()
export class OrderService {
  constructor(
    @InjectRepository(Order) private orderRepo: Repository<Order>,
    @InjectRepository(OrderItem) private itemRepo: Repository<OrderItem>,
  ) {}

  @Transactional()
  async createOrder(orderData: CreateOrderDto) {
    // 所有数据库操作都在同一个事务中
    const order = await this.orderRepo.save(orderData);
    
    for (const item of orderData.items) {
      await this.itemRepo.save({ ...item, orderId: order.id });
    }
    
    // 如果任何操作失败，整个事务会自动回滚
    return order;
  }
}
```

**注意事项：**
- Service 中必须注入至少一个 Repository
- 方法内抛出的任何错误都会触发回滚
- 只有方法正常返回时事务才会提交

### 6. 国际化支持

#### 设置

```typescript
import { I18nModule } from 'nestjs-i18n';
import * as path from 'path';

@Module({
  imports: [
    I18nModule.forRoot({
      fallbackLanguage: 'en',
      loaderOptions: {
        path: path.join(__dirname, '/i18n/'),
        watch: true,
      },
    }),
  ],
})
export class AppModule {}
```

#### 使用示例

```typescript
import { I18n, I18nContext } from '@meta-1/nest-common';

@Controller('users')
export class UserController {
  @Get()
  async getUsers(@I18n() i18n: I18nContext) {
    const users = await this.userService.findAll();
    
    return {
      message: i18n.t('users.list.success'), // 自动添加 'common.' 前缀
      data: users,
    };
  }

  @Post()
  async createUser(
    @Body() dto: CreateUserDto,
    @I18n() i18n: I18nContext,
  ) {
    const user = await this.userService.create(dto);
    
    return {
      message: i18n.t('users.create.success', {
        args: { name: user.name },
      }),
      data: user,
    };
  }
}
```

**自定义命名空间：**

```typescript
import { createI18nContext } from '@meta-1/nest-common';
import { I18n as NestI18n } from 'nestjs-i18n';

@Controller('products')
export class ProductController {
  @Get()
  async getProducts(@NestI18n() rawI18n: RawI18nContext) {
    const i18n = createI18nContext(rawI18n, 'products');
    
    return {
      message: i18n.t('list.success'), // 翻译为 'products.list.success'
      data: await this.productService.findAll(),
    };
  }
}
```

### 7. 错误处理

#### 定义错误码

```typescript
import { defineErrorCode, AppError } from '@meta-1/nest-common';

// 定义模块错误码
export const UserErrorCode = defineErrorCode({
  USER_NOT_FOUND: { code: 2000, message: 'User not found' },
  USER_ALREADY_EXISTS: { code: 2001, message: 'User already exists' },
});

// 使用
@Injectable()
export class UserService {
  async getUserById(id: string) {
    const user = await this.userRepository.findOne({ where: { id } });
    
    if (!user) {
      throw new AppError(UserErrorCode.USER_NOT_FOUND, { userId: id });
    }
    
    return user;
  }
}
```

**错误码范围约定：**
- `0-999`: 通用错误（`@meta-1/nest-common`）
- `1000-1999`: Message 模块错误
- `2000-2999`: User 模块错误
- `3000-3999`: Auth 模块错误
- `100-199`: 分布式锁错误

**错误响应格式：**

```json
{
  "code": 2000,
  "success": false,
  "message": "User not found",
  "data": { "userId": "123" },
  "timestamp": "2024-01-01T00:00:00.000Z",
  "path": "/api/users/123"
}
```

### 8. 分页DTO

```typescript
import { PageRequestDto, PageDataDto } from '@meta-1/nest-common';
import { ApiOkResponse } from '@nestjs/swagger';
import { createPageSchema, createPageModels } from '@meta-1/nest-common';

@Controller('users')
export class UserController {
  @Get()
  @ApiOkResponse({
    schema: createPageSchema(UserDto),
  })
  @ApiExtraModels(...createPageModels(UserDto))
  async getUsers(@Query() query: PageRequestDto) {
    const [data, total] = await this.userService.findAndCount(query);
    
    return PageDataDto.of(total, data);
  }
}
```

### 9. HTTP服务

```typescript
import { HttpService } from '@meta-1/nest-common';

@Injectable()
export class ExternalApiService {
  constructor(private readonly httpService: HttpService) {}

  async fetchData() {
    // GET 请求
    const response = await this.httpService.get<Data>('https://api.example.com/data');
    return response.data;
  }

  async postData(data: any) {
    // POST 请求（支持重试）
    const response = await this.httpService.post<Result>('https://api.example.com/data', data, {
      retries: 3,
      retryDelay: 1000,
    });
    return response.data;
  }

  async downloadFile(url: string, filePath: string) {
    // 下载文件（支持进度回调）
    await this.httpService.download({
      url,
      filePath,
      onProgress: (progress) => {
        console.log(`下载进度: ${progress}%`);
      },
    });
  }
}
```

### 10. 配置加载器

```typescript
import { ConfigLoader, ConfigSourceType } from '@meta-1/nest-common';

// 从本地 YAML 文件加载
const loader = new ConfigLoader<AppConfig>({
  type: ConfigSourceType.LOCAL_YAML,
  filePath: './config/app.yaml',
});
const config = await loader.load();

// 从 Nacos 加载
const nacosLoader = new ConfigLoader<AppConfig>({
  type: ConfigSourceType.NACOS,
  server: '127.0.0.1:8848',
  dataId: 'app-config',
  group: 'DEFAULT_GROUP',
  namespace: 'public',
  username: 'nacos',
  password: 'nacos',
});
const nacosConfig = await nacosLoader.load();
```

**配置特性：**
- 自动将 kebab-case 键名转换为 camelCase
- 支持 YAML 格式
- 支持 Nacos 配置中心

### 11. 验证工具

```typescript
import { createI18nZodDto } from '@meta-1/nest-common';
import { z } from 'zod';

const CreateUserSchema = z.object({
  email: z.string().email('Invalid email format'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

export class CreateUserDto extends createI18nZodDto(CreateUserSchema) {}
```

**特性：**
- 自动采集 Schema 中的所有验证错误消息到 i18n collector
- 支持开发环境自动收集翻译 key

### 12. Swagger工具

```typescript
import { createPageSchema, createPageModels } from '@meta-1/nest-common';
import { ApiOkResponse, ApiExtraModels } from '@nestjs/swagger';

@Controller('users')
export class UserController {
  @Get()
  @ApiOkResponse({
    schema: createPageSchema(UserDto),
  })
  @ApiExtraModels(...createPageModels(UserDto))
  async getUsers() {
    // ...
  }
}
```

### 13. 工具函数

```typescript
import { generateKey, md5, PlainTextLogger } from '@meta-1/nest-common';

// 生成动态键名
const key = generateKey('user:#{0}:profile:#{1.name}', ['123', { name: 'Alice' }]);
// 结果: 'user:123:profile:Alice'

// MD5 哈希
const hash = md5('hello world');

// TypeORM 纯文本日志输出器
const logger = new PlainTextLogger();
```

## 📝 API 参考

### 装饰器

- `@CacheableService()` - 标记服务类支持缓存
- `@Cacheable(options)` - 缓存方法结果
- `@CacheEvict(options)` - 清除缓存
- `@WithLock(options)` - 分布式锁
- `@SnowflakeId()` - 自动生成雪花ID
- `@Transactional()` - 自动事务管理
- `@I18n()` - 注入 I18nContext

### 类

- `CommonModule` - 通用模块
- `AppError` - 自定义错误类
- `I18nContext` - 国际化上下文
- `ErrorsFilter` - 全局异常过滤器
- `ResponseInterceptor` - 响应拦截器
- `HttpService` - HTTP 客户端服务
- `PageRequestDto` - 分页请求 DTO
- `PageDataDto<T>` - 分页响应 DTO
- `ConfigLoader<T>` - 配置加载器
- `PlainTextLogger` - TypeORM 纯文本日志输出器

### 函数

- `defineErrorCode(definition)` - 定义错误码
- `createI18nZodDto(schema)` - 创建支持 i18n 的 Zod DTO
- `createI18nContext(context, namespace)` - 创建自定义命名空间上下文
- `createPageSchema(itemDto)` - 创建分页 Swagger Schema
- `createPageModels(itemDto)` - 创建分页 Swagger Models
- `generateKey(pattern, args)` - 生成动态键名
- `md5(text)` - MD5 哈希

### 错误码

**通用错误码 (0-999):**
- `SERVER_ERROR` (500) - 服务器错误
- `VALIDATION_FAILED` (400) - 验证失败
- `UNAUTHORIZED` (401) - 未授权
- `FORBIDDEN` (403) - 禁止访问
- `NOT_FOUND` (404) - 未找到
- `I18N_CONTEXT_NOT_FOUND` (500) - I18n 上下文未找到
- `CONFIG_NOT_FOUND` (3000) - 配置未找到
- `CONFIG_INVALID` (3001) - 配置无效

**分布式锁错误码 (100-199):**
- `REDIS_NOT_INJECTED` (100) - Redis 未注入
- `LOCK_ACQUIRE_FAILED` (110) - 获取锁失败
- `LOCK_ACQUIRE_ERROR` (111) - 获取锁时发生错误
- `LOCK_RELEASE_ERROR` (112) - 释放锁时发生错误

## 📄 License

MIT
