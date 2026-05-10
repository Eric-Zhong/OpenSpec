# 06-command-examples.md

## OpenSpec CLI 命令参数示例

### 常见命令使用案例

1. **初始化项目**
```bash
openspec init --tools all --force --profile custom
```
- `--tools all`：配置所有可用工具
- `--force`：强制初始化，覆盖现有文件
- `--profile custom`：使用自定义配置文件

2. **列出变更**
```bash
openspec list --type change --json
```
- `--type change`：指定列出变更类型
- `--json`：输出JSON格式结果

3. **查看变更详情**
```bash
openspec show change-123 --schema openspec --json
```
- `change-123`：变更ID
- `--schema openspec`：指定使用openspec schema
- `--json`：输出JSON格式结果

4. **验证变更**
```bash
openspec validate change-123 --strict --json
```
- `change-123`：变更ID
- `--strict`：启用严格验证模式
- `--json`：输出JSON格式结果

5. **归档变更**
```bash
openspec archive change-123 --yes --skip-specs
```
- `change-123`：变更ID
- `--yes`：跳过确认提示
- `--skip-specs`：跳过规格更新操作

6. **生成补全脚本**
```bash
openspec completion install --shell bash
```
- `--shell bash`：为bash shell生成补全脚本