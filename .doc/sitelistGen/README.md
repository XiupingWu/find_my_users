# Sitelist Generator - sitelists.json 生成工具

这个工具用于根据 `data/Site/` 目录中的 JSON 文件重新生成 `sitelists.json`，确保数据一致性和时间准确性。

## 功能特点

- 🔄 **自动同步**：根据 `data/Site/` 中的实际文件生成 `sitelists.json`
- 🗑️ **清理无效数据**：自动移除不存在的站点和无效条目
- ⏰ **时间准确性**：保留原有的创建时间，更新修改时间
- 💾 **安全备份**：自动创建原文件备份
- 🛡️ **数据验证**：验证条目有效性，过滤脏数据
- 📊 **详细报告**：提供处理结果和统计信息

## 使用方法

### 基本使用

```bash
# 在项目根目录下运行
node .doc/sitelistGen/generateSitelists.js
```

或者给脚本添加执行权限后直接运行：

```bash
chmod +x .doc/sitelistGen/generateSitelists.js
./.doc/sitelistGen/generateSitelists.js
```

## 处理逻辑

### 1. 文件扫描
- 扫描 `data/Site/` 目录中的所有 `.json` 文件
- 从文件名提取 `slug`（移除 `.json` 扩展名）

### 2. 数据读取
- 读取每个 JSON 文件的 `name_zh` 和 `name_en` 字段
- 获取文件的创建和修改时间戳

### 3. 时间戳处理
- **新文件**：使用文件系统的创建和修改时间
- **现有文件**：保留原有的创建时间（`date`），更新修改时间（`lastModified`）

### 4. 数据验证
自动过滤以下无效条目：
- 缺少 `name_zh` 和 `name_en` 的条目
- 包含无效 slug 模式的条目：
  - `-`（单个连字符）
  - `unknown-site`
  - 描述性文本片段
- 名称字段包含描述性文本而非真实站点名称的条目

### 5. 输出格式
生成的每个条目包含：
```json
{
  "name_zh": "站点中文名",
  "name_en": "Site English Name",
  "slug": "site-slug",
  "date": "2024-01-01T00:00:00.000Z",
  "lastModified": "2024-01-02T00:00:00.000Z"
}
```

## 输出示例

```
🚀 Starting sitelists.json generation...
📊 Loaded timestamps for 7 existing sites
📁 Found 12 JSON files in Site directory
✅ Processed: ruan-yifengs-blog-weekly.json (阮一峰的网络日志)
✅ Processed: phoenix-frontier.json (不死鸟)
✅ Processed: dig-internet.json (挖互联网)
...
💾 Created backup: sitelists.json.backup.1691234567890

🎉 Sitelists generation completed!
✅ Processed: 12 sites
⏭️  Skipped: 0 files
📊 Total entries in sitelists.json: 19

📈 Statistics:
   📋 Bilingual sites: 17
   🇨🇳 Chinese only: 1
   🇺🇸 English only: 1
```

## 安全特性

### 自动备份
- 在覆盖原文件前自动创建备份
- 备份文件格式：`sitelists.json.backup.{timestamp}`
- 可用于恢复原数据

### 数据验证
- 验证必要字段的存在
- 过滤明显的脏数据和描述性文本
- 确保 slug 的有效性

### 错误处理
- 单个文件错误不影响整体处理
- 详细的错误日志和警告信息
- 优雅的错误恢复机制

## 高级功能

### 批量清理
脚本会自动识别并移除以下类型的无效数据：
- 不完整的站点信息
- 重复或冲突的条目
- CSV 解析错误产生的脏数据
- 描述性文本片段

### 时间戳管理
- **创建时间保持不变**：确保站点的原始添加时间准确
- **修改时间实时更新**：反映文件的最新修改状态
- **时间格式标准化**：统一使用 ISO 8601 格式

## 注意事项

1. **备份重要**：虽然脚本会自动备份，但建议手动备份重要数据
2. **文件完整性**：确保 `data/Site/` 中的 JSON 文件格式正确
3. **权限检查**：确保脚本有读写相关目录和文件的权限
4. **定期运行**：建议在添加新站点或修改现有站点后运行此脚本

## 额外工具

### 验证脚本
验证生成的 `sitelists.json` 与实际文件的一致性：

```bash
node .doc/sitelistGen/validateSitelists.js
```

输出示例：
```
🔍 Starting sitelists.json validation...
📁 Found 12 JSON files in Site directory
📋 Found 12 entries in sitelists.json

✅ Perfect sync! All files are properly listed.

📊 Validation Summary:
   ✅ Synced files: 12
   ❌ Missing from sitelists: 0
   ❌ Orphaned entries: 0
```

## 与其他工具的集成

### 与 CSV 转换工具配合使用
```bash
# 1. 先转换 CSV 到 JSON
node .doc/siteGen/csvToJson.js

# 2. 清理生成的 sitelists.json
node .doc/sitelistGen/generateSitelists.js

# 3. 验证结果
node .doc/sitelistGen/validateSitelists.js
```

### 在 CI/CD 中使用
可以将此脚本集成到持续集成流程中，确保数据一致性：
```bash
# package.json scripts
{
  "scripts": {
    "update-sitelists": "node .doc/sitelistGen/generateSitelists.js",
    "deploy": "npm run update-sitelists && npm run build"
  }
}
```

## 故障排除

### 常见问题

**Q: 脚本运行后条目数量减少了**
A: 这是正常现象，脚本会自动清理无效数据。检查控制台输出了解哪些条目被跳过。

**Q: 时间戳信息丢失**
A: 脚本会尽量保留原有时间信息。如果文件系统不支持，会使用当前时间。

**Q: 某些站点没有出现在结果中**
A: 检查对应的 JSON 文件是否存在于 `data/Site/` 目录，以及文件格式是否正确。
