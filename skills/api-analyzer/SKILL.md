---
name: api-analyzer
description: PHP/Phalcon 接口（fbi/bi-common/ard-*）逆向分析：根据路由/URL 定位代码，产出参数语义、SQL 链路、字段翻译完整文档，自动写入飞书知识库并更新 FAQ 索引。当用户给出 API 路由要求"分析接口/接口文档化/接口逻辑梳理/这个接口干了什么"时使用。
---

## 本 skill 的工作方式

强**顺序门控**流程，输入是一个路由（如 `GET /bi/order/list` 或 `OrderController@detailAction`），输出是一篇飞书 docx + 一行 FAQ 索引更新。

**绝对不要**跳过 Step 1 直接读代码就开始写文档。每一步的产物喂给下一步，缺一不可。

---

## 激活后第一步（必须按顺序执行）

1. **读规则**：用 Read 读 `reference/checklist.md`，确认强制规则与各 Step 必读文件表。
2. **读配置**：用 Read 读 `workspace/config.md`
   - 如果文件不存在 → 从 `reference/config-template.md` 复制创建，**先问用户**：飞书目标 wiki node token、FAQ docx token、待分析的代码仓库本地路径，写入 config 后再继续
   - 如果存在 → 直接使用其中的 token 与 repo 路径
3. **建任务文件**：在 `workspace/docs/` 下建 `{接口路径kebab}.md`（如 `bi-order-list.md`），作为分析过程的草稿落地。
4. **按 Step 顺序执行**，每步完成必须等用户明确"通过"才能进入下一步。

⛔ 严禁行为：
- 未读 config 就开始动飞书 API
- **未跑 Step 0（FAQ 反查）就直接进入分析** —— 会重复新建文档
- 未定位到入口文件就臆造参数表
- SQL 只看一层 Model 调用就当成全部链路
- 把用户对单个疑问的回应当作整个 Step 的通过确认

---

## 流程概览

| Step | 动作 | 必读文件 |
|------|------|----------|
| 0 | **已存在文档检查**：FAQ 反查路由，决定本次走"新建"还是"更新" | `reference/preexist-check.md` |
| 1 | 入口定位：路由 → Controller@action | `reference/locate-entry.md` |
| 2 | 参数分析：入参清单 + 校验 + SQL 条件映射 | `reference/param-analysis.md` |
| 3 | 主 SQL 链路：核心查询 SQL 拆解（表/JOIN/WHERE/字段） | `reference/sql-extraction.md` |
| 4 | 后处理链路：数据加工、二次查询、外部调用 | `reference/post-processing.md` |
| 5 | 字段翻译：locale key + 含义 | `reference/translation-extraction.md` |
| 6 | 输出飞书：按 Step 0 的 mode 分支 — create 走新建 docx + FAQ 追加；update 走覆盖已有 docx + FAQ 行更新时间戳 | `reference/feishu-output.md` |

---

## 产物位置

| 类型 | 路径 |
|------|------|
| 本地分析草稿 | `workspace/docs/{接口路径kebab}.md` |
| 配置（token/repo） | `workspace/config.md` |
| 飞书文档 | 配置中 wiki node 下新建的 docx |
| FAQ 更新 | 配置中 FAQ docx 末尾追加一行 |

---

## 输出文档结构（飞书 docx 与本地草稿一致）

```
# 接口分析：{METHOD} {PATH}

## 1. 基本信息
- 路由 / Controller / Action / 文件:行号
- 鉴权方式 / 调用方

## 2. 入参清单
| 字段 | 类型 | 必填 | 含义 | 默认值 | SQL 条件映射 |

## 3. 主查询 SQL
- SQL 全文（参数化形式）
- 涉及表与 JOIN 关系
- WHERE 条件来源（哪个入参 → 哪个条件）
- 返回字段列表

## 4. 后处理逻辑
- 数据加工步骤（按顺序）
- 二次 SQL 查询（每条都列出来源代码位置）
- 外部服务调用（RPC / HTTP）

## 5. 字段翻译表
| 翻译 key | 中文含义 | 使用场景 |

## 6. 返回结构
最终响应 JSON 示例

## 7. 已知坑 / FAQ
本次分析中发现的非显然点
```

---

## 飞书相关依赖

- 写 docx：依赖 `lark-doc` skill 的 `docs +create`（v2 + DocxXML 或 Markdown）
- 挂到 wiki 节点：依赖 `lark-wiki` skill
- 更新 FAQ：依赖 `lark-doc` 的 patch/append 能力

具体调用方式见 `reference/feishu-output.md`。
