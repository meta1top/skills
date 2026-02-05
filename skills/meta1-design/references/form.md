# 表单使用

## 示例

以下以登录为例，展示如果使用 @meta-1/design 完成表单构建。

```ts
import { z } from "zod";
import { Button, Form, FormItem, Input } from "@meta-1/design";

export const CODE = /^\d{6}$/;

export const LoginSchema = z.object({
  email: z.string().email({ message: "邮箱格式不正确" }).describe("邮箱"),
  password: z.string().min(1, "请输入密码").describe("密码"),
  otpCode: z.string().regex(CODE, "请输入6位数字验证码").optional().describe("OTP验证码"),
});

export type LoginData = z.infer<typeof LoginSchema>;

export const Demo = () => {
  // 如果需要 form 实例的话
  const form = Form.useForm<LoginData>();

  return <Form<LoginData> form={form} onSubmit={(data) => onSubmit(data)} schema={LoginSchema}>
    <FormItem label={t("邮箱")} name="email">
      <Input placeholder={t("请输入邮箱")} />
    </FormItem>
    <FormItem label={t("密码")} name="password">
      <Input placeholder={t("请输入密码")} type="password" />
    </FormItem>
    <Button className="mt-sm" loading={loading && isPending} long type="submit">
      {t("提交")}
    </Button>
  </Form>
}
```