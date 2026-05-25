# Step 4：后处理链路

主 SQL 拿到数据后，到 `return $this->jsonResponse(...)` 之前发生的所有事情。

## 提取范围

按代码执行顺序梳理：

1. **数据形变**：`foreach`、`array_map`、`array_column` 等
2. **二次 SQL**：通常在 foreach 里按 id 再查一次
3. **N+1 模式**：识别出来要标记（这是常见坑）
4. **外部调用**：HTTP / RPC / Redis / 文件
5. **字段加工**：金额单位换算、时间格式化、状态枚举映射
6. **聚合统计**：sum / count / 分组

## 输出格式

```markdown
## 4. 后处理逻辑

### 4.1 数据加工步骤（按执行顺序）

#### Step 4.1 收集 user_ids
- 位置: `OrderService.php:80`
- 代码: `array_column($rows, 'user_id')`
- 用途: 为下一步批量查用户做准备

#### Step 4.2 批量查用户详情
- 位置: `OrderService.php:82`
- 触发条件: `count($userIds) > 0`
- 二次 SQL:
  \`\`\`sql
  SELECT id, avatar, level FROM users WHERE id IN (:ids)
  \`\`\`
- 用途: 给每条订单补 avatar、level

#### Step 4.3 调用商品中心 RPC
- 位置: `OrderService.php:95`
- 服务: `\Rpc\ProductService::batchGet($productIds)`
- 触发条件: 总是触发
- 用途: 补商品名称、缩略图

#### Step 4.4 金额单位换算
- 位置: `OrderService.php:108`
- 代码: `$row['amount'] = bcdiv($row['amount'], 100, 2)`
- 说明: 数据库存分，输出元

#### Step 4.5 状态枚举翻译
- 位置: `OrderService.php:112`
- 代码: `$row['status_text'] = OrderStatus::label($row['status'])`
- 说明: 跳到 Step 5 的翻译表

### 4.2 N+1 风险点

| 位置 | 描述 | 建议 |
|------|------|------|
| OrderService.php:82 | 已批量化，无 N+1 | - |
| OrderService.php:120 | 在 foreach 里调 getUserCoupon()，每行查 1 次 | 改批量查询 |

### 4.3 二次 SQL 汇总

| # | 触发条件 | SQL | 位置 |
|---|----------|-----|------|
| 1 | 总是 | `SELECT ... FROM users WHERE id IN (...)` | OrderService.php:82 |
| 2 | row.has_coupon = 1 | `SELECT ... FROM coupons WHERE order_id = :oid` | OrderService.php:120 |
```

## 关键要求

- 每一步都标行号
- 二次 SQL 同样按 Step 3 标准给出参数化形式
- 触发条件不能漏（"总是触发"也要明写，避免被认为是漏标）
- N+1 / 慢查询 / 缺索引 等已知坑，单独列入 4.2 节，会同步进 Step 7 的 FAQ

## 完成条件

- `## 4. 后处理逻辑` 段完整
- 用户回 "Step 4 通过" 进入 Step 5
