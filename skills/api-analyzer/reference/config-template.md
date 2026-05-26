# api-analyzer 配置（复制为 workspace/config.md 后填写）

## 飞书目标

- **wiki_node_token**: ``
  - 接口分析文档会作为子节点挂在这个 wiki 节点下
  - 获取方式：打开飞书知识库目录页 → 右键节点 → 复制链接，链接末段或调用 `wiki node search` 获取 node_token

- **faq_doc_token**: ``
  - 统一 FAQ 索引 docx 的 obj_token
  - 每分析完一个接口，在该 docx 末尾追加一行：`{METHOD} {PATH} → 文档链接 → 关键坑点`
  - 获取方式：打开 FAQ docx → URL 形如 `https://xxx.feishu.cn/docx/{token}` → token 即为此值

## 代码仓库

每个仓库一行 `name=local_path`：

```
fbi=/Users/liuwenchao/sites/fbi
bi-common=/Users/liuwenchao/sites/bi-common
ard-api=/Users/liuwenchao/sites/ard-api
ard-etl=/Users/liuwenchao/sites/ard-etl
```

用户给路由时如未指定仓库，skill 应按顺序在所有仓库中 grep 入口。

## locale 路径（字段翻译来源）

每个仓库可能有自己的 locale 目录，写出来便于 Step 5 直接查找：

```
fbi.locale=app/messages
bi-common.locale=src/Translation
```

## 默认行为

- `auto_create_subdoc`: true / false
  - true：Step 6 直接调用 `docs +create` + `wiki node move`
  - false：仅生成本地 markdown，等用户手动导入
