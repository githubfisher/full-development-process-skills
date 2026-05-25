---
name: api-analyzer
description: PHP/Phalcon 接口（fbi/bi-common/ard-*）逆向分析：根据路由/URL 定位代码，产出参数语义、SQL 链路、字段翻译完整文档，自动写入飞书知识库并更新 FAQ 索引。当用户给出 API 路由要求"分析接口/接口文档化/接口逻辑梳理/这个接口干了什么"时使用。
---

## 本 skill 的工作方式

输入是一个路由（如 `/api/storeabnormalapi/store_abnormal_list`），自动完成代码分析，产出草稿后由用户确认，确认后上传飞书。

**全程只有两个用户确认点：**

1. **草稿生成后**：展示本地 markdown 路径 + 关键摘要，用户确认内容正确
2. **发飞书前**：用户回"发飞书"后才调用飞书 API

---

## 激活后立即执行（无需等待确认）

1. 读 `reference/checklist.md`（强制规则）
2. 读 `workspace/config.md`（token / repo 路径）
   - 不存在 → 按 `reference/config-template.md` 创建并询问用户填写
3. 创建草稿文件 `workspace/docs/{接口路径kebab}.md`
4. **连续执行 Step 0 → 1 → 2 → 3 → 4 → 5**，每步读对应 reference 文件，结果写入草稿

⛔ 严禁行为：
- 未读 config 就开始动飞书 API
- 未跑 Step 0（FAQ 反查）就直接分析（会重复新建文档）
- 未定位到入口文件就臆造参数表
- SQL 只看一层 Model 调用就当成全部链路

---

## 流程

| 阶段 | 步骤 | 说明 | 必读文件 |
|------|------|------|----------|
| **自动执行** | Step 0 | FAQ 反查路由，决定 create / update | `reference/preexist-check.md` |
| | Step 1 | 路由 → Controller@action 入口定位 | `reference/locate-entry.md` |
| | Step 2 | 入参清单 + SQL 条件映射 | `reference/param-analysis.md` |
| | Step 3 | 主 SQL 链路拆解 | `reference/sql-extraction.md` |
| | Step 4 | 后处理 + 二次查询 + 外部调用 | `reference/post-processing.md` |
| | Step 5 | 字段翻译 key + locale 含义 | `reference/translation-extraction.md` |
| **⏸ 确认点 1** | — | 展示草稿路径 + 摘要，等用户确认内容 | — |
| **自动执行** | Step 6 self-check | 按 checklist 验收草稿完整性 | `reference/checklist.md` |
| **⏸ 确认点 2** | — | 展示飞书上传计划，等用户回"发飞书" | `reference/feishu-output.md` |
| **自动执行** | Step 6 上传 | create / update 分支执行 | `reference/feishu-output.md` |

---

## 确认点 1 的提示格式

Step 0-5 全部完成后，输出：

```
草稿已生成：workspace/docs/{kebab}.md

【摘要】
- 入口：{文件:行号}
- 主表：{表名}
- 入参数量：{n} 个
- 后处理步骤：{n} 个
- 已知坑：{n} 条

如有需要修改的内容请告诉我，没有问题回复"发飞书"直接上传。
```

> 用户可以在这里指出任何错误或补充，我修改草稿后重新展示。
> 用户回"发飞书"即代表确认，进入 Step 6 self-check 和上传。

---

## 产物位置

| 类型 | 路径 |
|------|------|
| 本地分析草稿 | `workspace/docs/{接口路径kebab}.md` |
| 配置 | `workspace/config.md` |
| 飞书文档 | config 中 `wiki_node_token` 文件夹下新建的 docx |
| FAQ 更新 | config 中 FAQ docx 末尾追加一行 |

---

## 输出文档结构

> 面向受众：产品 / 测试 / 数据。不含代码路径、继承关系、鉴权实现等开发内部信息。

```
# 接口分析：{METHOD} {PATH}

## 1. 基本信息
| 路由 | 支持国家 | 功能说明 |
（如有国家间行为差异，在表格下方单独说明）

## 2. 请求参数
| 字段 | 类型 | 必填 | 说明 | 默认值 | 可选值/范围 |

## 3. 查询 SQL
- SQL 全文（参数化形式，供数据同学直接参考）
- 涉及表说明（表名 + 业务含义）

## 4. 数据处理逻辑
- 按步骤列出业务处理（不含文件:行号等代码细节）
- 二次查询说明（触发条件 + SQL）

## 5. 枚举值说明
| 字段 | 值 | 含义 |
（合并所有枚举、翻译 key 的含义，面向业务理解）

## 6. 返回示例
最终响应 JSON 示例 + 关键字段说明

## 7. 注意事项
已知坑、边界条件、容易踩的问题
```

---

## 飞书相关依赖

- 创建 docx：`lark-cli docs +create --api-version v2 --doc-format markdown --folder-token {wiki_node_token} --content "$(cat ...)"`
- 更新 FAQ：`lark-cli docs +update --api-version v2 --command append`

具体调用方式见 `reference/feishu-output.md`。
