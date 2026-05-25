# Step 0：已存在文档检查（前置）

⚠️ 这是流程的第一步，**必须在 Step 1 入口定位之前执行**。

## 目的

判断本次分析是**首次分析（新建）**还是**已有文档更新**。两者在 Step 6 走不同分支，不能搞错——重复新建会让 wiki 出现两篇同接口文档，FAQ 索引也会有重复行。

## 检查流程

### 0.1 规范化路由

把用户输入统一成查询键：

| 用户输入 | 规范化 key |
|----------|-----------|
| `GET /bi/order/list` | `GET /bi/order/list` |
| `https://bi.xxx.com/api/v2/order/list?x=1` | `/api/v2/order/list` |
| `OrderController@detailAction` | `OrderController::detailAction` |
| `/bi/order/list/` (带尾斜杠) | `/bi/order/list` |

规则：去 host、去 query string、去尾部 `/`、HTTP 动词大写、`@` 转 `::`。

### 0.2 读取 FAQ docx

调用 lark-doc 把 `workspace/config.md` 里 `faq_doc_token` 对应的 FAQ 文档拉下来：

```bash
lark-cli docs +fetch --api-version v2 --format markdown \
  --doc-token {faq_doc_token}
```

把内容保存到 `workspace/docs/.faq-cache.md`（每次都覆盖，不缓存过久）。

### 0.3 在 FAQ 中搜索路由

用 grep 在 FAQ 缓存里搜规范化 key（注意大小写、转义 `/`）：

```bash
grep -nF "{规范化key}" workspace/docs/.faq-cache.md
```

也搜一下"宽松匹配"——例如用户给的是 `/bi/order/list` 而 FAQ 里写的是 `GET /bi/order/list`，反之亦然。

### 0.4 判定与产出

**情况 A：未找到**
- 标记 `mode = create`
- 在草稿 markdown 顶部 frontmatter 加 `mode: create`
- 继续 Step 1

**情况 B：找到唯一一行**
- 从 FAQ 行里提取飞书链接（wiki url 或 docx url），解析出 `obj_token`
- 调用 `lark-doc` 把已有 docx 拉下来到 `workspace/docs/.existing-{kebab}.md`
- 标记 `mode = update`
- 在草稿 markdown 顶部 frontmatter 写：
  ```yaml
  ---
  mode: update
  existing_docx_token: {token}
  existing_url: {url}
  faq_line_no: {grep 命中的行号}
  ---
  ```
- 把已有文档摘要（小节标题列表 + 上次更新日期）给用户看，问一句：
  > "已在 FAQ 找到 {METHOD} {PATH} 的历史文档：{url}。本次将走**更新**流程。请确认。"
- 等用户回 "继续" / "走更新" 才进入 Step 1

**情况 C：找到多行**
- 全部列出，让用户选要更新哪一条，或确认是新建（同路由不同语义版本，例如 v1/v2）
- 用户选定后再标 mode

### 0.5 mode 一旦定下不再改

- 后续 Steps 1-5 不受 mode 影响（都要重新分析）
- 但 Step 4/5 在标"已知坑"时，**可对比 existing 文档**，标出"本次相比上次新增/移除"的点（对 update 模式特别有价值）

## 完成条件

- frontmatter 中 `mode` 已确定（create / update）
- update 模式下 existing_docx_token 已抓取并缓存
- 用户回 "Step 0 通过" / "继续" 后进入 Step 1
