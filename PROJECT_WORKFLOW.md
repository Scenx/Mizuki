# Mizuki 项目工作流程文档

> 📝 本文档记录了 Mizuki Pages 项目的工作流程，用于快速恢复项目上下文。

---

## 📁 项目结构

```
Mizuki/
├── src/
│   └── content/
│       └── posts/          # 📝 文章 Markdown 文件目录
│           ├── cocos_h5_reverse.md
│           ├── openwrt_unexpected_poweroff.md
│           ├── mysql_unexpected_poweroff.md
│           └── nginx_301_redirect.md
│
├── public/
│   └── images/             # 🖼️ 图片资源目录
│       ├── albums/         # 相册目录
│       ├── device/         # 设备图片
│       ├── diary/          # 日记图片
│       ├── cocos_h5_reverse.png
│       ├── openwrt_unexpected_poweroff.png
│       ├── openwrt_system_mount.png
│       ├── mysql_unexpected_poweroff.png
│       └── nginx_301_redirect.png
│
└── PROJECT_WORKFLOW.md     # 本文档
```

---

## ✍️ 文章格式规范

### Frontmatter 格式

每篇文章必须包含以下 frontmatter：

```yaml
---
title: 🔧 文章标题（支持 emoji）
published: 2025-12-06
updated: 2025-12-06
description: '文章描述'
image: '/images/封面图.png'
tags: [标签1, 标签2]
category: '分类'
draft: false
---
```

### 文章风格特点

- ✅ **标题使用 emoji**：增强视觉效果（如 🔧 💻 🚀）
- ✅ **引用块突出关键信息**：使用 `>` 引用块标注问题描述和核心解决方案
- ✅ **清晰的步骤结构**：使用 `## 步骤一`、`## 步骤二` 等分段
- ✅ **代码块有详细注释**：代码块包含清晰的命令说明和注释
- ✅ **专业技术文档风格**：简洁、直接、聚焦核心解决方案
- ✅ **避免冗余内容**：不添加不必要的"其他方案"、"常见问题"等内容

### 图片引用格式

```markdown
![Alt text](/images/文件名.png)
```

**注意事项：**
- 图片路径必须以 `/images/` 开头
- 图片文件名使用小写英文和下划线，如 `mysql_unexpected_poweroff.png`
- 封面图在 frontmatter 中使用相同格式：`image: '/images/文件名.png'`

---

## 🎯 创建新文章的标准流程

### 1. 准备配图

用户会提前将配图放到 `public/images/` 目录，文件名可能是随机生成的（如 `Gemini_Generated_Image_xxx.png`）

### 2. 重命名配图

```bash
mv "public/images/原文件名.png" "public/images/新文件名.png"
```

**命名规范：**
- 使用小写英文单词 + 下划线
- 命名与文章主题相关
- 例如：`mysql_unexpected_poweroff.png`、`nginx_301_redirect.png`

### 3. 创建文章文件

**文件路径：** `src/content/posts/文章名.md`

**文件名规范：**
- 使用小写英文单词 + 下划线
- 与配图文件名保持一致
- 例如：`mysql_unexpected_poweroff.md`、`nginx_301_redirect.md`

### 4. 编写文章内容

遵循上述"文章格式规范"和"文章风格特点"编写内容。

**核心原则：**
- **简洁直接**：聚焦核心解决方案，避免冗余
- **步骤清晰**：问题描述 → 原因分析 → 解决步骤 → 总结
- **代码实用**：提供可直接使用的命令和配置
- **风格一致**：与现有文章保持一致的风格

---

## 📤 提交代码规范

### Git 提交流程

```bash
# 1. 查看状态
git status

# 2. 添加文件并提交
git add public/images/新图片.png src/content/posts/新文章.md
git commit -m "$(cat <<'EOF'
简短的提交信息

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"

# 3. 推送到远程仓库
git push
```

### 提交信息规范

- **简洁明了**：一句话概括本次提交内容
- **中文描述**：使用中文描述提交内容
- **格式固定**：必须包含 Claude Code 生成标识和 Co-Authored-By 信息

**示例：**
- `新增 MySQL 断电修复文章`
- `新增 Nginx 301 重定向问题解决文章`

---

## 📋 常用分类和标签

### 分类（category）

- `os` - 操作系统相关
- `逆向` - 逆向工程
- `数据库` - 数据库相关
- `web` - Web 服务器/开发
- `开发` - 软件开发

### 常用标签（tags）

- `[openwrt]` - OpenWrt 相关
- `[mysql, 数据库]` - MySQL 数据库
- `[nginx, 运维]` - Nginx 运维
- `[cocos, 逆向]` - Cocos 引擎逆向

---

## ⚠️ 重要注意事项

1. **图片引用格式正确**：所有图片路径必须以 `/images/` 开头
2. **文件命名规范**：使用小写英文 + 下划线，避免空格和特殊字符
3. **风格保持一致**：参考现有文章风格，保持专业简洁
4. **不添加冗余内容**：除非用户明确要求，否则不添加"其他方案"、"常见问题"等章节
5. **代码可直接使用**：提供的命令和配置必须是可直接复制使用的

---

## 📚 现有文章参考

### 1. OpenWrt 断电修复
- **文件：** `openwrt_unexpected_poweroff.md`
- **特点：** 步骤清晰，图文并茂，代码注释详细

### 2. MySQL 断电修复
- **文件：** `mysql_unexpected_poweroff.md`
- **特点：** 聚焦核心解决方案，步骤简洁直接

### 3. Nginx 301 重定向
- **文件：** `nginx_301_redirect.md`
- **特点：** 问题场景清晰，配置示例实用

### 4. Cocos H5 逆向
- **文件：** `cocos_h5_reverse.md`
- **特点：** 技术细节丰富，代码示例完整

---

## 🔄 快速恢复工作流程

1. **阅读本文档**：快速了解项目结构和规范
2. **查看现有文章**：参考 `src/content/posts/` 目录下的现有文章
3. **确认图片位置**：检查 `public/images/` 目录中的最新图片
4. **按流程创建文章**：遵循"创建新文章的标准流程"
5. **提交并推送**：按"提交代码规范"完成提交

---

**最后更新：** 2025-12-06