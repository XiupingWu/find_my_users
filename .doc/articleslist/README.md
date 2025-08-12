# Articles Generator - articles.json 生成工具

这个工具用于根据 `data/Articles/` 目录中的 Markdown 文件重新生成 `articles.json`，确保数据一致性和时间准确性。

## 功能特点

- 🔄 **自动同步**：根据 `data/Articles/` 中的实际文件生成 `articles.json`
- 🗑️ **清理无效数据**：自动移除不存在的文章和无效条目
- ⏰ **时间准确性**：保留原有的创建时间，更新修改时间
- 💾 **安全备份**：自动创建原文件备份
- 🛡️ **数据验证**：验证条目有效性，过滤脏数据
- 📊 **详细报告**：提供处理结果和统计信息

## 使用方法

### 基本使用

```bash
# 在项目根目录下运行
node .doc/articleslist/generateArticles.js
```

或者给脚本添加执行权限后直接运行：

```bash
chmod +x .doc/articleslist/generateArticles.js
./.doc/articleslist/generateArticles.js
```

### 验证工具

验证生成的 `articles.json` 与实际文件的一致性：

```bash
node .doc/articleslist/validateArticles.js
```

## 处理逻辑

### 1. 文件扫描
- 扫描 `data/Articles/` 目录中的所有 `.md` 文件
- 分别处理 `zh` 和 `en` 目录
- 从文件名提取 `slug`

### 2. 数据读取
- 读取每个 Markdown 文件的内容
- 提取 frontmatter 数据（如果有）
- 提取第一个标题作为文章标题（如果没有 frontmatter）
- 提取第一段作为描述（如果没有 frontmatter）

### 3. 时间戳处理
- **新文章**：使用文件系统的创建和修改时间
- **现有文章**：保留原有的创建时间（`date`），更新修改时间（`lastModified`）

### 4. 数据验证
自动过滤以下无效条目：
- 缺少标题的条目
- 标题长度异常的条目
- 内容为空的条目

### 5. 输出格式
生成的每个条目包含：
```json
{
  "title_zh": "中文标题",
  "title_en": "English Title",
  "description_zh": "中文描述",
  "description_en": "English Description",
  "date": "2024-01-01T00:00:00.000Z",
  "lastModified": "2024-01-02T00:00:00.000Z",
  "slug": "article-slug"
}
```

## 输出示例

```
🚀 Starting articles.json generation...
📊 Loaded timestamps for 7 existing articles
📁 Found 12 Markdown files in zh directory
📁 Found 10 Markdown files in en directory
✅ Processed: article-1.md (zh)
✅ Processed: article-1.md (en)
...
💾 Created backup: articles.json.backup.1691234567890

🎉 Articles.json generation completed!
✅ Processed: 22 files
⏭️  Skipped: 0 files
📊 Total articles in articles.json: 12

📈 Statistics:
   📋 Bilingual articles: 10
   🇨🇳 Chinese only: 2
   🇺🇸 English only: 0
```

## 安全特性

### 自动备份
- 在覆盖原文件前自动创建备份
- 备份文件格式：`articles.json.backup.{timestamp}`
- 可用于恢复原数据

### 数据验证
- 验证必要字段的存在
- 过滤明显的无效数据
- 确保 slug 的有效性

### 错误处理
- 单个文件错误不影响整体处理
- 详细的错误日志和警告信息
- 优雅的错误恢复机制

## 注意事项

1. **备份重要**：虽然脚本会自动备份，但建议手动备份重要数据
2. **文件完整性**：确保 Markdown 文件格式正确
3. **权限检查**：确保脚本有读写相关目录和文件的权限
4. **定期运行**：建议在添加新文章或修改现有文章后运行此脚本

## 在 CI/CD 中使用

可以将此脚本集成到持续集成流程中，确保数据一致性：
```bash
# package.json scripts
{
  "scripts": {
    "update-articles": "node .doc/articleslist/generateArticles.js",
    "validate-articles": "node .doc/articleslist/validateArticles.js",
    "deploy": "npm run update-articles && npm run validate-articles && npm run build"
  }
}
```

## 故障排除

### 常见问题

**Q: 脚本运行后文章数量减少了**
A: 这是正常现象，脚本会自动清理无效数据。检查控制台输出了解哪些文章被跳过。

**Q: 时间戳信息丢失**
A: 脚本会尽量保留原有时间信息。如果文件系统不支持，会使用当前时间。

**Q: 某些文章没有出现在结果中**
A: 检查对应的 Markdown 文件是否存在于正确的目录，以及文件格式是否正确。
