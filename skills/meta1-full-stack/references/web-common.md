# @meta-1/web-common

Meta-1 前端通用库，提供 React 组件、Hooks、工具函数和配置，用于构建 Next.js 应用。

## ✨ 特性

- 🎨 **React 组件** - 通用 UI 组件库
- 🪝 **自定义 Hooks** - 数据查询、表单验证、国际化等 Hooks
- 🌍 **国际化支持** - 基于 i18next 的多语言支持
- 🔧 **工具函数** - REST API、加密、Token 管理等工具
- ⚙️ **配置预设** - Next.js 和 i18next 配置
- 🎯 **类型安全** - 完整的 TypeScript 支持

## 📦 安装

此包为 Meta-1 项目内部包，通过 monorepo 工作区使用。

```bash
# 在项目根目录安装依赖
pnpm install
```

## 🚀 使用

### 组件

#### RootLayout - 根布局组件

用于 Next.js App Router 的根布局，提供全局 Provider 配置。

```typescript
import { RootLayout } from "@meta-1/web-common/components/layout/root";

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <RootLayout locale="zh-cn" theme="system">
      {children}
    </RootLayout>
  );
}
```

#### ThemeSwitcher - 主题切换组件

支持 light、dark、system 三种主题切换，带有平滑的 View Transition 动画。

```typescript
import { ThemeSwitcher } from "@meta-1/web-common/components/theme-switcher";

function Header() {
  return (
    <div>
      <ThemeSwitcher size="md" />
    </div>
  );
}
```

**Props:**
- `size?: "sm" | "md" | "lg"` - 组件尺寸，默认 `md`
- `activeClassName?: string` - 激活状态的样式类名
- `className?: string` - 自定义样式类名

#### OTPInput - OTP 验证码输入组件

6 位数字验证码输入组件。

```typescript
import { OTPInput } from "@meta-1/web-common/components/input/otp";

function VerifyCode() {
  const [code, setCode] = useState("");

  return (
    <OTPInput
      value={code}
      onChange={setCode}
    />
  );
}
```

**Props:**
- `value?: string` - 当前值
- `onChange?: (value: string) => void` - 值变化回调

#### Loading - 加载组件

全屏加载动画组件。

```typescript
import { Loading } from "@meta-1/web-common/components/loading";

function App() {
  if (isLoading) {
    return <Loading />;
  }
  return <div>内容</div>;
}
```

#### Visible - 可见性控制组件

根据条件显示/隐藏内容。

```typescript
import { Visible } from "@meta-1/web-common/components/visible";

function MyComponent() {
  const [show, setShow] = useState(true);

  return (
    <Visible visible={show}>
      <div>条件显示的内容</div>
    </Visible>
  );
}
```

**Props:**
- `visible?: boolean` - 是否显示，默认 `true`
- `children: React.ReactNode` - 子元素

#### Uploader - 文件上传组件

文件上传组件，基于 `@meta-1/design` 的 Uploader 封装。

```typescript
import { Uploader } from "@meta-1/web-common/components/uploader";

function FileUpload() {
  const [files, setFiles] = useState([]);

  return (
    <Uploader
      value={files}
      onChange={setFiles}
      action="/api/upload"
    />
  );
}
```

**Props:**
- `value?: UploadFile[]` - 已上传的文件列表
- `onChange?: (value: UploadFile[]) => void` - 文件变化回调
- 其他 props 继承自 `@meta-1/design` 的 Uploader

### Hooks

#### useQuery - REST API 查询 Hook

基于 TanStack Query 封装的查询 Hook，自动处理 REST API 响应格式和错误提示。

```typescript
import { useQuery } from "@meta-1/web-common/hooks";
import { get } from "@meta-1/web-common/utils/rest";

function UserList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ["users"],
    queryFn: async () => {
      return await get<User[]>("/api/users");
    },
    showError: true, // 是否自动显示错误消息，默认 false
  });

  if (isLoading) return <div>加载中...</div>;
  if (error) return <div>错误: {error.message}</div>;

  return (
    <ul>
      {data?.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

**特点:**
- 自动处理 `RestResult<T>` 格式的响应
- 自动解包 `data` 字段
- 支持错误自动提示（可选）
- 取消请求的错误（错误码 499）不会显示

#### useMutation - REST API 变更 Hook

基于 TanStack Query 封装的变更 Hook，用于 POST、PUT、DELETE 等操作。

```typescript
import { useMutation } from "@meta-1/web-common/hooks";
import { post } from "@meta-1/web-common/utils/rest";
import { useQueryClient } from "@tanstack/react-query";

function CreateUser() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: async (data: CreateUserData) => {
      return await post<User>("/api/users", data);
    },
    showError: true, // 是否自动显示错误消息，默认 true
    noErrorCode: [400], // 不显示错误消息的错误码列表
    onSuccess: () => {
      // 成功后刷新列表
      queryClient.invalidateQueries({ queryKey: ["users"] });
    },
  });

  const handleSubmit = (data: CreateUserData) => {
    mutation.mutate(data);
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* 表单内容 */}
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? "提交中..." : "提交"}
      </button>
    </form>
  );
}
```

**特点:**
- 自动处理 `RestResult<T>` 格式的响应
- 自动解包 `data` 字段
- 支持错误自动提示（默认开启）
- 支持排除特定错误码
- 取消请求的错误（错误码 499）静默处理

#### useTableLoader - 表格数据加载 Hook

用于表格分页、筛选、排序等场景的数据加载 Hook，支持 URL 参数同步。

```typescript
import { useTableLoader } from "@meta-1/web-common/hooks";
import { get } from "@meta-1/web-common/utils/rest";
import type { PageResult } from "@meta-1/web-common/types/rest";

function UserTable() {
  const { loading, result, query, load, reset, refetch } = useTableLoader<User>({
    defaultValues: {
      keyword: "",
      status: "active",
    },
    action: async (params) => {
      return await get<PageResult<User>>("/api/users", params);
    },
    withURL: true, // 是否同步 URL 参数，默认 true
    encodeParams: ["keyword"], // 需要 URL 编码的参数
    keys: ["extraKey"], // 额外的查询参数键
  });

  const handleSearch = (keyword: string) => {
    load({ keyword, page: 1 });
  };

  const handleReset = () => {
    reset();
  };

  if (loading) return <div>加载中...</div>;

  return (
    <div>
      <input
        value={query.keyword}
        onChange={(e) => handleSearch(e.target.value)}
      />
      <button onClick={handleReset}>重置</button>
      <table>
        {/* 表格内容 */}
      </table>
      <div>
        当前页: {query.page}, 每页: {query.size}
        总数: {result?.data?.total || 0}
      </div>
    </div>
  );
}
```

**返回值:**
- `loading: boolean` - 是否正在加载
- `result: RestResult<PageData<T>> | null` - 查询结果
- `query: any` - 当前查询参数
- `load: (params?: any) => Promise<void>` - 加载数据（合并参数）
- `reset: () => void` - 重置到初始状态
- `refetch: () => Promise<void>` - 刷新当前数据

**特点:**
- 自动同步 URL 查询参数
- 自动处理分页参数（page, size）
- 自动矫正超出范围的页码
- 支持参数合并和重置
- 内置加载状态管理

#### useLocale - 国际化语言切换 Hook

用于切换应用语言。

```typescript
import { useLocale } from "@meta-1/web-common/hooks";

function LanguageSwitcher() {
  const [locale, setLocale] = useLocale();

  return (
    <select value={locale} onChange={(e) => setLocale(e.target.value)}>
      <option value="zh-cn">中文</option>
      <option value="en">English</option>
    </select>
  );
}
```

**返回值:**
- `[locale: string, setLocale: (value: string) => void]` - 当前语言和设置函数

#### useSchema - Zod Schema 国际化 Hook

用于将 Zod Schema 的错误消息进行国际化处理。

```typescript
import { useSchema } from "@meta-1/web-common/hooks";
import { z } from "zod";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

const LoginSchema = z.object({
  email: z.string().email("validation.email.invalid"),
  password: z.string().min(8, "validation.password.min"),
});

function LoginForm() {
  const schema = useSchema(LoginSchema);

  const form = useForm({
    resolver: zodResolver(schema),
  });

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      {/* 表单内容 */}
    </form>
  );
}
```

**特点:**
- 自动翻译 Schema 中的错误消息键
- 支持嵌套对象、数组、联合类型等复杂 Schema
- 支持 ZodEffects（refine、superRefine 等）

#### useEncrypt - RSA 加密 Hook

用于对敏感数据进行 RSA 加密。

```typescript
import { useEncrypt } from "@meta-1/web-common/hooks";

function LoginForm() {
  const encrypt = useEncrypt();

  const handleSubmit = async (password: string) => {
    const encryptedPassword = encrypt(password);
    // 发送加密后的密码
    await post("/api/login", { password: encryptedPassword });
  };

  return <form onSubmit={handleSubmit}>...</form>;
}
```

**返回值:**
- `encrypt: (text: string) => string` - 加密函数，如果未设置公钥则返回原文

### 工具函数

#### REST API 工具

提供 REST API 请求方法，自动处理响应格式和错误。

```typescript
import {
  get,
  post,
  put,
  patch,
  del,
  download,
  request,
  alias,
  config,
} from "@meta-1/web-common/utils/rest";
import type { RestResult } from "@meta-1/web-common/types/rest";

// GET 请求
const result = await get<User[]>("/api/users", { page: 1, size: 10 });

// POST 请求
const result = await post<User>("/api/users", {
  name: "John",
  email: "john@example.com",
});

// PUT 请求
const result = await put<User>("/api/users/1", {
  name: "John Updated",
});

// PATCH 请求
const result = await patch<User>("/api/users/1", {
  name: "John Patched",
});

// DELETE 请求
const result = await del("/api/users/1");

// 文件下载
await download("/api/export", { format: "excel" }, {
  fileName: "users.xlsx",
});

// 自定义请求
const result = await request<Data>({
  url: "/api/custom",
  method: "POST",
  data: { ... },
});

// 配置 API 别名
alias({
  "@api": {
    url: "https://api.example.com",
    headers: async () => ({
      "X-Custom-Header": "value",
    }),
  },
});

// 使用别名
await get("@api/users");

// 全局配置
config({
  onResponse: (data, response) => {
    // 处理响应
  },
});
```

**响应格式:**
所有请求返回 `Promise<RestResult<T>>`，格式如下：

```typescript
type RestResult<T> = {
  success?: boolean;
  code?: number;
  message?: string;
  data?: T;
};
```

#### Token 工具

Token 的存储和读取工具。

```typescript
import {
  getToken,
  setToken,
  clearToken,
} from "@meta-1/web-common/utils/token";
import type { Token } from "@meta-1/web-common/types/token";

// 客户端获取 Token
const token = await getToken();

// 服务端获取 Token
import { cookies } from "next/headers";
const token = await getToken(cookies());

// 设置 Token
setToken({
  token: "eyJhbGciOiJIUzI1NiIs...",
  expiresIn: 3600,
});

// 清除 Token
clearToken();
```

#### 加密工具

RSA 加密工具。

```typescript
import { encrypt } from "@meta-1/web-common/utils/crypto";

const publicKey = "-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----";
const encrypted = encrypt("sensitive-data", publicKey);
```

#### 环境工具

环境检测和工具函数。

```typescript
import {
  isServer,
  isMobile,
  getRootDomain,
} from "@meta-1/web-common/utils/env";

// 判断是否在服务端
if (isServer) {
  // 服务端代码
}

// 判断是否为移动设备
if (isMobile()) {
  // 移动端代码
}

// 获取根域名（用于跨子域 Cookie）
const rootDomain = getRootDomain("app.example.com");
// => ".example.com"
```

#### 格式化工具

数据格式化工具。

```typescript
import { formatFileSize } from "@meta-1/web-common/utils/format";

const size = formatFileSize(1024 * 1024 * 5);
// => "5 MB"
```

#### 正则表达式常量

常用正则表达式。

```typescript
import {
  EMAIL,
  URL,
  USERNAME,
  PASSWORD,
  CODE,
  PHONE,
  NUMBER,
} from "@meta-1/web-common/utils/regular";

EMAIL.test("user@example.com"); // true
PHONE.test("13800138000"); // true
PASSWORD.test("Abc123!@#"); // true
```

#### URL 工具

URL 处理工具。

```typescript
import { getPathname } from "@meta-1/web-common/utils/url";

const pathname = getPathname("/api/users?page=1");
// => "/api/users"
```

#### React Query 工具

React Query 客户端和预取工具。

```typescript
import {
  getQueryClient,
  prefetchQuery,
  dehydrate,
  HydrationBoundary,
} from "@meta-1/web-common/utils/query";
import { get } from "@meta-1/web-common/utils/rest";

// 获取 QueryClient（服务端每次新建，客户端复用）
const queryClient = getQueryClient();

// 预取数据（服务端）
await prefetchQuery(queryClient, {
  queryKey: ["users"],
  queryFn: async () => {
    return await get<User[]>("/api/users");
  },
});

// 在组件中使用
export default function Page() {
  const queryClient = getQueryClient();
  const dehydratedState = dehydrate(queryClient);

  return (
    <HydrationBoundary state={dehydratedState}>
      <UserList />
    </HydrationBoundary>
  );
}
```

#### 请求头工具

客户端和服务端请求头生成工具。

```typescript
// 客户端
import { get } from "@meta-1/web-common/utils/headers.client";

const headers = await get();
// 包含: Accept-Language, CLIENT-PLATFORM-TYPE, Authorization 等

// 服务端
import { get } from "@meta-1/web-common/utils/headers.server";

const headers = await get();
// 包含: Accept-Language, CLIENT-PLATFORM-TYPE, Authorization 等
```

#### i18next 工具

i18next 初始化工具。

```typescript
import { initI18next } from "@meta-1/web-common/utils/i18next";

await initI18next({
  "zh-cn": {
    translation: {
      welcome: "欢迎",
    },
  },
  "en": {
    translation: {
      welcome: "Welcome",
    },
  },
});
```

#### 主题服务端工具

服务端获取主题配置。

```typescript
import { getTheme } from "@meta-1/web-common/utils/theme.server";

const theme = await getTheme();
// => "light" | "dark" | "system"
```

#### 格式化工具

数据格式化工具。

```typescript
import { join } from "@meta-1/web-common/formatters";

const result = join(["a", "b", "c"], ", ");
// => "a, b, c"
```

### 状态管理

#### 公共状态 Atoms

使用 Jotai 管理的全局状态。

```typescript
import {
  localeState,
  publicKeyState,
} from "@meta-1/web-common/state/public";
import { useAtom, useAtomValue } from "jotai";

function MyComponent() {
  const [locale, setLocale] = useAtom(localeState);
  const publicKey = useAtomValue(publicKeyState);

  return <div>当前语言: {locale}</div>;
}
```

### 类型定义

#### REST API 类型

```typescript
import type {
  RestResult,
  PageData,
  PageResult,
  PageRequest,
} from "@meta-1/web-common/types/rest";

type User = {
  id: number;
  name: string;
};

const result: RestResult<User> = {
  success: true,
  code: 200,
  message: "成功",
  data: {
    id: 1,
    name: "John",
  },
};

const pageResult: PageResult<User> = {
  success: true,
  code: 200,
  data: {
    total: 100,
    data: [/* users */],
  },
};
```

#### Token 类型

```typescript
import type { Token } from "@meta-1/web-common/types/token";

const token: Token = {
  token: "eyJhbGciOiJIUzI1NiIs...",
  expiresIn: 3600,
};
```

### 事件总线

全局事件总线，基于 EventEmitter3。

```typescript
import eventBus from "@meta-1/web-common/events";

// 监听事件
eventBus.on("user-login", (data) => {
  console.log("用户登录", data);
});

// 触发事件
eventBus.emit("user-login", { userId: 123 });

// 移除监听
eventBus.off("user-login", handler);
```

### 配置

#### i18next 配置

```typescript
import {
  getBaseI18nextConfig,
  FALLBACK,
  MISSING_KEY_PREFIX,
} from "@meta-1/web-common/config/i18next";

const config = getBaseI18nextConfig(resources);
```

## 📦 包含内容

### 组件

- **布局组件**: RootLayout, MainLayout, HtmlLayout, LoginLayout
- **输入组件**: OTPInput, CodeInput, EmailCodeInput, SliderInput
- **页面组件**: PageHeader, PageTitleBar, TabsTitle
- **UI 组件**: Loading, Coming, Visible, ThemeSwitcher, LangSelect
- **特效组件**: IridescenceBackground
- **上传组件**: Uploader
- **工具组件**: AtomsHydrate, ThemeSyncProvider, InfoTooltip

### Hooks

- `useQuery` - REST API 查询 Hook
- `useMutation` - REST API 变更 Hook
- `useTableLoader` - 表格数据加载 Hook
- `useLocale` - 国际化语言切换 Hook
- `useSchema` - Zod Schema 国际化 Hook
- `useEncrypt` - RSA 加密 Hook

### 工具函数

- **rest.ts** - REST API 请求工具
- **token.ts** - Token 管理工具
- **crypto.ts** - RSA 加密工具
- **query.ts** - React Query 客户端工具
- **env.ts** - 环境检测工具
- **format.ts** - 数据格式化工具
- **regular.ts** - 正则表达式常量
- **url.ts** - URL 处理工具
- **i18next.ts** - i18next 初始化工具
- **headers.client.ts** - 客户端请求头工具
- **headers.server.ts** - 服务端请求头工具
- **theme.server.ts** - 主题服务端工具
- **locale.server.ts** - 语言服务端工具

### 状态管理

- `localeState` - 语言状态 Atom
- `publicKeyState` - 公钥状态 Atom

### 类型定义

- `RestResult`, `PageResult`, `PageRequest` - REST API 类型
- `Token` - Token 类型

### 事件

- `eventBus` - 全局事件总线

### 格式化工具

- `join` - 数组连接工具

## 📄 许可证

MIT
