# 07-error-examples.md

## OpenSpec CLI 常见错误示例

### 错误示例与解决方案

1. **错误：命令不存在**
```bash
$ openspec unknown-command
error: unknown command "unknown-command"
```
- **原因**：输入了不存在的命令
- **解决方案**：检查命令拼写或查看可用命令列表

2. **错误：参数缺失**
```bash
$ openspec init
error: missing required argument "path"
```
- **原因**：缺少必需的参数
- **解决方案**：提供完整的参数，如 `openspec init my-project`

3. **错误：无效的选项**
```bash
$ openspec init --invalid-option
error: invalid option "--invalid-option"
```
- **原因**：使用了不支持的选项
- **解决方案**：查看命令帮助，如 `openspec init --help`

4. **错误：权限不足**
```bash
$ openspec archive change-123
error: permission denied
```
- **原因**：没有足够的权限执行操作
- **解决方案**：以管理员身份运行命令或检查权限设置

5. **错误：文件无法访问**
```bash
$ openspec list --json
error: cannot access path "./": No such file or directory
```
- **原因**：指定的路径不存在
- **解决方案**：确认路径正确性或创建所需目录

6. **错误：配置文件无效**
```bash
$ openspec init --profile invalid-profile
error: invalid profile "invalid-profile"
```
- **原因**：指定的配置文件无效
- **解决方案**：使用有效的配置文件名或检查配置文件格式