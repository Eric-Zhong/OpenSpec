# 01-command-reference.md

## OpenSpec CLI 命令参数说明

### 常用命令参数

| 参数 | 中文说明 | 英文说明 | 示例 |
|------|----------|----------|------|
| `--tools` | 配置AI工具 | Configure AI tools non-interactively | `--tools all` |
| `--force` | 强制操作 | Force operation without confirmation | `--force` |
| `--profile` | 指定配置文件 | Override global config profile (core or custom) | `--profile custom` |
| `--json` | JSON格式输出 | Output as JSON | `--json` |
| `--no-color` | 禁用颜色输出 | Disable color output | `--no-color` |
| `--yes` | 跳过确认提示 | Skip confirmation prompts | `--yes` |
| `--long` | 显示详细信息 | Show detailed information | `--long` |
| `--schema` | 指定schema | Specify schema override | `--schema openspec` |
| `--type` | 指定类型 | Specify item type when ambiguous | `--type change` |

### 特殊参数说明

- `--tools` 参数支持的工具列表: `all`, `none`, `tool1`, `tool2`, ...
- `--type` 参数支持的类型: `change`, `spec`
- `--schema` 参数支持的schema: `default`, `openspec`, `custom`
- `--profile` 参数支持的配置文件: `core`, `custom`