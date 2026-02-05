# 主题使用

## 默认主题
```css
@import "tw-animate-css";
@import "tailwindcss";
@custom-variant dark (&:is(.dark *));

@theme design {
  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-popover: var(--popover);
  --color-popover-foreground: var(--popover-foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-destructive-foreground: var(--destructive-foreground);
  --color-success: var(--success);
  --color-success-foreground: var(--success-foreground);
  --color-error: var(--error);
  --color-error-foreground: var(--error-foreground);
  --color-warning: var(--warning);
  --color-warning-foreground: var(--warning-foreground);
  --color-info: var(--info);
  --color-info-foreground: var(--info-foreground);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);
  --color-chart-1: var(--chart-1);
  --color-chart-2: var(--chart-2);
  --color-chart-3: var(--chart-3);
  --color-chart-4: var(--chart-4);
  --color-chart-5: var(--chart-5);
  --color-sidebar: var(--sidebar);
  --color-sidebar-foreground: var(--sidebar-foreground);
  --color-sidebar-primary: var(--sidebar-primary);
  --color-sidebar-primary-foreground: var(--sidebar-primary-foreground);
  --color-sidebar-accent: var(--sidebar-accent);
  --color-sidebar-accent-foreground: var(--sidebar-accent-foreground);
  --color-sidebar-border: var(--sidebar-border);
  --color-sidebar-ring: var(--sidebar-ring);
  --color-action-hover: var(--action-hover);

  --radius-default: 4px;
  --gap-default: 0.5rem;

  /* Button */
  --radius-button: var(--radius-default);
  --padding-button: 0.75rem;
  --spacing-button: 2rem;

  /* Alert */
  --radius-alert: var(--radius-default);
  --padding-alert: 0.75rem;
  --gap-alert: var(--gap-default);

  /* Dialog */
  --radius-dialog: var(--radius-default);
  --padding-dialog: 1rem;
  --gap-dialog: 1rem;

  /* Badge */
  --radius-badge: var(--radius-default);
  --padding-badge: var(--padding-badge-y) var(--padding-badge-x);
  --padding-badge-x: 0.3rem;
  --padding-badge-y: 0.1rem;

  /* Card */
  --radius-card: var(--radius-default);
  --padding-card: 1rem;
  --gap-card: var(--gap-default);

  /* Select */
  --radius-select: var(--radius-default);
  --padding-select: 0.5rem;
  --spacing-select: 2rem;

  /* Popover */
  --radius-popover: var(--radius-default);
  --padding-popover: 0.25rem;

  /* Command */
  --radius-command: var(--radius-default);
  --padding-command: 0.375rem;

  /* Calendar */
  --radius-calendar: var(--radius-default);
  --padding-calendar: 0.5rem;
  --gap-calendar: var(--gap-default);

  /* Button */
  --radius-input: var(--radius-default);
  --padding-input: 0.5rem;
  --spacing-input: 2rem;

  /* Tabs */
  --radius-tabs-title: var(--radius-default);
  --spacing-tabs-title: 2rem;

  /* Message */
  --radius-message: var(--radius-default);
  --padding-message: var(--padding-message-y) var(--padding-message-x);
  --padding-message-x: 0.75rem;
  --padding-message-y: 0.5rem;

  /* Sheet */
  --radius-sheet: var(--radius-default);
  --padding-sheet: 1rem;
  --gap-sheet: 1rem;

  /* Tooltip */
  --radius-tooltip: var(--radius-default);
  --padding-tooltip: 0.5rem;
}

@layer base {
  :root {
    --background: hsl(0 0% 100%);
    --foreground: hsl(0 0% 3.9%);
    --card: hsl(0 0% 100%);
    --card-foreground: hsl(0 0% 3.9%);
    --popover: hsl(0 0% 100%);
    --popover-foreground: hsl(0 0% 3.9%);
    --primary: rgba(53, 104, 82, 1);
    --primary-foreground: hsl(0 0% 98%);
    --secondary: hsl(0 0% 96.1%);
    --secondary-foreground: hsl(0 0% 9%);
    --success: rgba(232, 255, 234, 1);
    --success-foreground: rgba(0, 180, 42, 1);
    --error: rgba(255, 236, 232, 1);
    --error-foreground: rgba(245, 63, 63, 1);
    --warning: rgba(255, 247, 232, 1);
    --warning-foreground: rgba(255, 125, 0, 1);
    --info: rgba(232, 243, 255, 1);
    --info-foreground: rgba(37, 99, 235, 1);
    --muted: hsl(0 0% 96.1%);
    --muted-foreground: hsl(0 0% 45.1%);
    --accent: hsl(0 0% 96.1%);
    --accent-foreground: hsl(0 0% 9%);
    --destructive: rgba(245, 63, 63, 1);
    --destructive-foreground: rgba(250, 250, 250, 1);
    --radius: 0.5rem;
    --border: oklch(0.922 0 0);
    --input: oklch(0.922 0 0);
    --ring: oklch(0.708 0 0);
    --chart-1: oklch(0.646 0.222 41.116);
    --chart-2: oklch(0.6 0.118 184.704);
    --chart-3: oklch(0.398 0.07 227.392);
    --chart-4: oklch(0.828 0.189 84.429);
    --chart-5: oklch(0.769 0.188 70.08);
    --sidebar: oklch(0.985 0 0);
    --sidebar-foreground: oklch(0.145 0 0);
    --sidebar-primary: oklch(0.205 0 0);
    --sidebar-primary-foreground: oklch(0.985 0 0);
    --sidebar-accent: oklch(0.97 0 0);
    --sidebar-accent-foreground: oklch(0.205 0 0);
    --sidebar-border: oklch(0.922 0 0);
    --sidebar-ring: oklch(0.708 0 0);
    --action-hover: rgba(0, 0, 0, 0.08);
  }

  .dark {
    --background: hsl(0 0% 7%);
    --foreground: hsl(0 0% 98%);
    --card: hsl(0 0% 3.9%);
    --card-foreground: hsl(0 0% 98%);
    --popover: hsl(0 0% 3.9%);
    --popover-foreground: hsl(0 0% 98%);
    --primary: rgba(68, 133, 105, 1);
    --primary-foreground: rgba(255, 255, 255, 1);
    --success: rgba(20, 83, 45, 0.2);
    --success-foreground: rgba(74, 222, 128, 1);
    --error: rgba(127, 29, 29, 0.2);
    --error-foreground: rgba(252, 165, 165, 1);
    --warning: rgba(120, 53, 15, 0.2);
    --warning-foreground: rgba(251, 191, 36, 1);
    --info: rgba(30, 58, 138, 0.2);
    --info-foreground: rgba(147, 197, 253, 1);
    --secondary: hsl(0 0% 14.9%);
    --secondary-foreground: hsl(0 0% 98%);
    --muted: hsl(0 0% 14.9%);
    --muted-foreground: hsl(0 0% 63.9%);
    --accent: hsl(0 0% 14.9%);
    --accent-foreground: hsl(0 0% 98%);
    --destructive: rgba(245, 63, 63, 1);
    --destructive-foreground: rgba(250, 250, 250, 1);
    --border: oklch(1 0 0 / 10%);
    --input: oklch(1 0 0 / 15%);
    --ring: oklch(0.556 0 0);
    --chart-1: oklch(0.488 0.243 264.376);
    --chart-2: oklch(0.696 0.17 162.48);
    --chart-3: oklch(0.769 0.188 70.08);
    --chart-4: oklch(0.627 0.265 303.9);
    --chart-5: oklch(0.645 0.246 16.439);
    --sidebar: oklch(0.205 0 0);
    --sidebar-foreground: oklch(0.985 0 0);
    --sidebar-primary: oklch(0.488 0.243 264.376);
    --sidebar-primary-foreground: oklch(0.985 0 0);
    --sidebar-accent: oklch(0.269 0 0);
    --sidebar-accent-foreground: oklch(0.985 0 0);
    --sidebar-border: oklch(1 0 0 / 10%);
    --sidebar-ring: oklch(0.556 0 0);
    --action-hover: rgba(255, 255, 255, 0.08);
  }

  html {
    @apply bg-background text-foreground;

    *,
    ::after,
    ::before,
    ::backdrop,
    ::file-selector-button {
      border-color: var(--border);
    }

    svg,
    img {
      display: inline-block;
    }
  }
}

@utility action-effect {
  @apply hover:bg-[var(--action-hover)] hover:text-primary;
  @apply focus:bg-[var(--action-hover)] focus:border-[var(--border)] focus:shadow-[0_0_0_1px_var(--card),0_0_0_3px_var(--primary)];
  @apply dark:focus:border-[var(--sidebar-border)] dark:focus:shadow-[0_0_0_1px_var(--secondary),0_0_0_3px_var(--primary)];
}

@utility action-effect-active {
  @apply active:bg-[var(--action-hover)] active:border-[var(--border)] active:shadow-[0_0_0_1px_var(--card),0_0_0_3px_var(--primary)];
  @apply dark:active:border-[var(--sidebar-border)] dark:active:shadow-[0_0_0_1px_var(--secondary),0_0_0_3px_var(--primary)];
}

@utility action-active {
  @apply text-primary bg-[var(--action-hover)];
}

@utility action-effect-disabled {
  @apply opacity-50 cursor-not-allowed focus:bg-transparent active:bg-transparent focus:shadow-none active:shadow-none focus:border-transparent active:border-transparent;
  @apply hover:bg-transparent;
}
```