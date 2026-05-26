# Step 2：参数分析

## 提取范围

从入口 action 起，提取**所有外部输入来源**：

1. `$this->request->get('xxx')` / `getPost` / `getQuery` / `getJsonRawBody`
2. `$this->dispatcher->getParam('xxx')`（URL 路径参数）
3. `$_GET / $_POST / $_REQUEST`（老代码）
4. 通过 Form / Validator 类校验的字段
5. 隐式：从 session / token 中取出的当前 user_id、role 等

## 输出表格式

```markdown
## 2. 入参清单

| 字段 | 类型 | 必填 | 含义 | 默认值 | 校验规则 | SQL 条件映射 |
|------|------|------|------|--------|----------|--------------|
| start_date | string | 是 | 查询起始日期 | - | Y-m-d 格式 | `WHERE created_at >= :start_date` |
| status | int | 否 | 订单状态 | 0=全部 | 0/1/2/3 枚举 | `status != 0` 时追加 `AND status = :status` |
| page | int | 否 | 分页页码 | 1 | >=1 | `LIMIT :offset, :pageSize` |
| user_id | int | 自动 | 当前登录用户（session 取） | - | - | `WHERE user_id = :userId`（强制） |
```

## 关键要求

### Q1 含义不许编
- 如果代码里没注释，就去问 ORM Model 的字段注释、数据库表 DDL 注释
- 实在没有，标 `（含义待补充）`，不要编

### Q2 SQL 条件映射列
- 每个入参必须填这一列
- 不参与 SQL 的入参填"仅用于业务校验"或"用于后处理（见 Step 4）"
- 条件来自代码哪一行也要标，复杂情况在表格下方加脚注

### Q3 默认值不许漏
- `?:` / `??` / `Request::get($key, $default)` 第二个参数都是默认值来源
- 默认值会影响 SQL 行为（如 status=0 跳过 WHERE）

### Q4 隐式参数
- 当前用户 ID、租户 ID、角色等从 session/token 取的字段也要列入表
- 标"自动"列代替"必填"，并说明来源（`$this->session->get('user')`）

## 完成条件

- 入参表已写入草稿 `## 2. 入参清单`
- 每行都有"SQL 条件映射"列
- 用户回 "Step 2 通过" 才进入 Step 3
