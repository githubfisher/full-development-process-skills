# api-analyzer 强制规则与验收标准

## 强制规则（违反任意一条本步必须重做）

### R1 入口必须精确到文件:行号
不允许写"在 OrderController 里"这种模糊位置。必须给出 `app/controllers/OrderController.php:128` 这种粒度。

### R2 SQL 必须给出参数化原文
- 不允许写"按 user_id 过滤" 这种概括
- 必须给出实际 SQL（或 Phalcon ORM 调用片段），WHERE 条件中标明 `:userId` 这样的占位符来自哪个入参
- ORM 链式调用必须翻译成等价 SQL

### R3 后处理必须列全
- 主 SQL 之后的每个 `foreach`、每次再查库、每次 RPC 调用都要单独列出
- 顺序不能乱，按代码实际执行顺序写
- 二次 SQL 也按 R2 标准给出

### R4 翻译 key 必须含义匹配
- 不能只列 key 不列含义
- 含义要去 locale 文件里实际取值，不能编

### R5 飞书输出前必须先让用户审本地 markdown
- 飞书 API 调用之前，Step 6 的第一动作是把本地 `workspace/docs/*.md` 路径给用户，等用户回 "OK / 通过"
- 用户没确认不准动 docs +create / wiki move / FAQ patch

### R6 只有两个用户确认点
- Step 0-5 自动连续执行，**不在中间等待用户回复**
- 确认点 1：Step 5 完成后展示草稿摘要，等用户确认内容
- 确认点 2：用户回"发飞书"后才执行飞书 API
- 用户在确认点 1 指出问题 → 修改草稿后重新展示，不重新跑全部步骤

### R7 update 模式绝不调用 docs +create
- Step 0 标记 mode=update 后，Step 6 只允许走 6.2u/6.4u 分支
- 即使 docs +update 失败，也不允许 fallback 到新建（会产生重复 docx）
- 草稿 frontmatter 必须保留 `mode` 字段直到 Step 6 结束

---

## Step 必读文件表

| 当前 Step | 必读 |
|-----------|------|
| Step 0 | `reference/preexist-check.md` |
| Step 1 | `reference/locate-entry.md` |
| Step 2 | `reference/param-analysis.md` |
| Step 3 | `reference/sql-extraction.md` |
| Step 4 | `reference/post-processing.md` |
| Step 5 | `reference/translation-extraction.md` |
| Step 6 | `reference/feishu-output.md` + 复读 `reference/checklist.md` 的 R5 |

---

## 验收标准（Step 6 之前 self-check）

- [ ] 草稿 frontmatter 中 mode 字段已填（create / update）
- [ ] update 模式下 existing_docx_token 与 faq_line_no 都已记录
- [ ] 入口文件:行号已写
- [ ] 入参表每行都有 SQL 条件映射列（无映射的写"仅用于业务校验"）
- [ ] 主 SQL 是参数化形式，不是字符串拼接结果
- [ ] 主 SQL 涉及的所有表都在文档里说明（角色、JOIN 条件）
- [ ] 后处理每一步都有代码位置（文件:行号）
- [ ] 每条二次 SQL 都标明触发条件（什么情况下会执行）
- [ ] 翻译表的 key 已在 locale 文件中确认存在
- [ ] 返回 JSON 示例字段名与代码 return 一致
- [ ] FAQ 段落至少 1 条（如果没坑，写"无已知坑"）
