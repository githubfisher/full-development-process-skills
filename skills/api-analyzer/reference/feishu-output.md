# Step 6：飞书输出

## 前置门（R5）

⚠️ **不允许直接动飞书 API。** 用户在确认点 1 回"发飞书"后，进入此步前先做 self-check，再展示上传计划：

**create 模式：**
```
本次走【新建】流程：
1. 创建 docx 到文件夹 {wiki_node_token}
2. 在 FAQ docx 末尾追加索引行
```

**update 模式：**
```
本次走【更新】流程：
1. 覆盖已有文档 {existing_url}
2. 刷新 FAQ 第 {faq_line_no} 行的更新时间
（不新建 docx，不新增 FAQ 行）
```

展示后**立即执行**，无需再等用户回复。

## 分支：mode = create

走 6.2c → 6.3c → 6.4c → 6.5

## 分支：mode = update

走 6.2u → 6.4u → 6.5（跳过 wiki 挂载，文档已经在 wiki 树里了）

---

## 6.1 读取 config 与 mode

```
Read workspace/config.md
Read 草稿 frontmatter，取出 mode / existing_docx_token / faq_line_no
```

---

## 6.2c 创建 docx 并写入内容（仅 create 模式）

调用 `lark-doc` skill，用 `--folder-token` 直接建到目标云盘文件夹：

```bash
# config 里的 wiki_node_token 实为云盘文件夹 token，用 --folder-token 指定
lark-cli docs +create --api-version v2 \
  --doc-format markdown \
  --folder-token {wiki_node_token} \
  --content "$(cat workspace/docs/{kebab}-upload.md)"
```

得到返回的 `document_id`（docx_token）与 `url`。

> ⚠️ 不要用 `--wiki-node`（那是 wiki 知识库节点专用），也不要用 `--markdown`（v2 要用 `--content`）。

## 6.3c（已合并至 6.2c）

`--folder-token` 已在创建时直接指定父目录，无需单独 move 步骤。

## 6.4c FAQ 追加索引行（仅 create 模式）

在 FAQ docx 末尾追加一行：

```
| {METHOD} {PATH} | {新建 wiki 链接} | {关键坑点摘要} | 创建 {分析日期} |
```

调用 `lark-doc` 的 update（v2 + DocxXML）追加到 FAQ docx 末尾的表格。具体接口形态见 lark-doc skill 内的 `docs +update` 说明。

---

## 6.2u 覆盖已有 docx（仅 update 模式）

⚠️ **绝对不调用 docs +create，不调用 wiki node move。**

1. 用 `existing_docx_token`（来自 Step 0 frontmatter）作为目标
2. 调用 `lark-doc` 的 update 把当前草稿 markdown 内容**全量替换**到该 docx：

```bash
lark-cli docs +update --api-version v2 --format markdown \
  --doc-token {existing_docx_token} \
  --strategy replace-all \
  --from-file workspace/docs/{kebab}.md
```

如果 `lark-doc` 没有"全量替换"参数，则：
- 先 fetch 拿到旧内容做 backup → 存到 `workspace/docs/.backup-{kebab}-{timestamp}.md`
- 然后用 patch / block 重建方式逐段替换（具体 API 形态见 lark-doc skill 文档）

3. 在草稿正文顶部加一段**更新历史**（如果之前没有，建一个；有则追加一行）：

```markdown
## 更新历史
- 2026-05-25：本次更新（修改点摘要：xxx）
- 上次更新日期（从 .existing-{kebab}.md 解析）：原内容
```

## 6.4u 刷新 FAQ 行（仅 update 模式）

定位 FAQ docx 第 `faq_line_no` 行，更新两列：

- "更新时间" → 今天的日期
- "关键坑点摘要" → 本次新增/变化的坑（如无变化保留原文）

**不允许新增 FAQ 行**，不允许把旧行删掉再加新行（会破坏行号引用）。

---

## 6.5 在本地草稿底部写回飞书链接（两种 mode 都执行）

```markdown
---
**飞书文档**: {wiki url}
**生成时间**: 2026-05-25 14:30
```

便于以后通过本地草稿反查飞书位置。

## auto_create_subdoc=false 的简化模式

如果 config 里把 `auto_create_subdoc` 设为 false：
- create 模式：跳过 6.2c/6.3c/6.4c，提示用户手动用 lark-drive 导入并加 FAQ 行
- update 模式：跳过 6.2u/6.4u，提示用户手动覆盖 docx 与刷新 FAQ 行
- 两种模式都给出待执行操作清单 + 草稿路径

## 失败回滚

**create 模式：**
- `docs +create` 失败：报告错误，不挂 wiki、不更 FAQ
- `wiki node move` 失败：docx 已建，提示用户手动 move
- FAQ 追加失败：docx 与 wiki 都已就绪，提示用户手动加索引行
- **任何失败都不要重试创建**（会建出重复 docx）

**update 模式：**
- `docs +update` 失败：旧 backup 已在 `.backup-*.md`，提示用户手动恢复
- FAQ 刷新失败：docx 已更新，提示用户手动改 FAQ 那一行
- **绝对不要因为 update 失败就 fallback 到 create**（会产生重复 docx）

## 完成条件

- 飞书文档已建好（create）或已覆盖（update）
- FAQ 索引已追加（create）或已刷新时间戳（update）
- 本地草稿底部已回写飞书链接
- 把飞书链接给用户作为最终交付，并明确告知本次走的是 create 还是 update
