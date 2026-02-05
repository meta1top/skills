# 组件使用实例

以 Popover 为例，Shadcn 的使用实例：
```ts
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Label } from "@/components/ui/label"
import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from "@/components/ui/popover"

export function PopoverDemo() {
  return (
    <Popover>
      <PopoverTrigger asChild>
        <Button variant="outline">Open popover</Button>
      </PopoverTrigger>
      <PopoverContent className="w-80">
        <div className="grid gap-4">
          <div className="space-y-2">
            <h4 className="leading-none font-medium">Dimensions</h4>
            <p className="text-muted-foreground text-sm">
              Set the dimensions for the layer.
            </p>
          </div>
          <div className="grid gap-2">
            <div className="grid grid-cols-3 items-center gap-4">
              <Label htmlFor="width">Width</Label>
              <Input
                id="width"
                defaultValue="100%"
                className="col-span-2 h-8"
              />
            </div>
            <div className="grid grid-cols-3 items-center gap-4">
              <Label htmlFor="maxWidth">Max. width</Label>
              <Input
                id="maxWidth"
                defaultValue="300px"
                className="col-span-2 h-8"
              />
            </div>
            <div className="grid grid-cols-3 items-center gap-4">
              <Label htmlFor="height">Height</Label>
              <Input
                id="height"
                defaultValue="25px"
                className="col-span-2 h-8"
              />
            </div>
            <div className="grid grid-cols-3 items-center gap-4">
              <Label htmlFor="maxHeight">Max. height</Label>
              <Input
                id="maxHeight"
                defaultValue="none"
                className="col-span-2 h-8"
              />
            </div>
          </div>
        </div>
      </PopoverContent>
    </Popover>
  )
}
```

@meta-1/design 的使用示例：
```ts
...
import { ..., Image, Popover } from "@meta-1/design";

...
<Popover
  asChild={true}
  className="w-[300px]"
  content={
    <div className="space-y-2">
      <h4 className="font-medium leading-none">{t("支持主流的验证器")}</h4>
      <p className="text-muted-foreground text-sm">{t("请在应用市场下载适合您手机的应用使用")}</p>
      <div className="space-y-2">
        {apps.map((app) => {
          return (
            <div className="flex items-center space-x-2" key={app.name}>
              <Image
                alt={app.name}
                className="overflow-hidden rounded-md shadow"
                height={32}
                src={app.icon}
                width={32}
              />
              <span>{app.name}</span>
            </div>
          );
        })}
      </div>
    </div>
  }
>
  <InfoCircledIcon className="mr-2" />
</Popover>
...
```