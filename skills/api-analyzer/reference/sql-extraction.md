# Step 3：主 SQL 链路提取

## 识别"主 SQL"

主 SQL = 接口核心数据来源的第一条查询。判定标准：
- 通常在 action 方法体上半段
- 通常用于决定返回总数 / 主列表
- 之后的查询都是用主 SQL 结果做 IN / JOIN / 翻译

## Phalcon 中 SQL 的几种形态

### 形态 A：Phalcon Model find / findFirst
```php
Order::find([
    "conditions" => "user_id = :uid: AND status = :s:",
    "bind" => ["uid" => $userId, "s" => $status],
    "order" => "created_at DESC",
    "limit" => $limit,
]);
```
**翻译成 SQL**:
```sql
SELECT * FROM `orders`
WHERE user_id = :uid AND status = :s
ORDER BY created_at DESC
LIMIT :limit
```

### 形态 B：QueryBuilder
```php
$this->modelsManager->createBuilder()
    ->from('Order')
    ->columns(['id', 'amount'])
    ->join('User', 'u.id = Order.user_id', 'u')
    ->where(...)
```
逐方法翻译成 SQL 段。

### 形态 C：原生 SQL
```php
$db->fetchAll("SELECT ... ", Db::FETCH_ASSOC, [':uid' => $userId]);
```
直接照抄。

### 形态 D：Repository / DAO 封装
- 跟进调用，找到底层最终的 find / SQL
- 文档里同时给出"业务层调用 + 实际 SQL"两层

## 输出格式

```markdown
## 3. 主查询 SQL

### 调用位置
`app/services/OrderService.php:67` → `OrderRepo::queryList()` → `app/repos/OrderRepo.php:42`

### 实际 SQL（参数化）
\`\`\`sql
SELECT o.id, o.order_no, o.amount, o.status, o.created_at,
       u.nickname AS user_nickname
FROM orders o
LEFT JOIN users u ON u.id = o.user_id
WHERE o.user_id = :userId
  AND o.created_at >= :startDate
  AND (:status = 0 OR o.status = :status)
ORDER BY o.created_at DESC
LIMIT :offset, :pageSize
\`\`\`

### 涉及表

| 表 | 别名 | 角色 | JOIN 条件 |
|----|------|------|-----------|
| orders | o | 主表 | - |
| users | u | 取昵称 | LEFT JOIN ON u.id = o.user_id |

### 入参绑定来源

| 占位符 | 来源入参 | 处理 |
|--------|----------|------|
| :userId | session.user.id | 直接绑定 |
| :startDate | request.start_date | strtotime 校验后传入 |
| :status | request.status | 默认 0 |
| :offset | request.page | `($page-1) * pageSize` |
| :pageSize | 常量 | PAGE_SIZE = 20 |

### 返回字段

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 订单 ID |
| order_no | string | 订单号 |
| ... | | |
```

## 关键要求

- 主 SQL 必须给出"等价 SQL 全文"，不能只截图 ORM 链
- 每个占位符都必须在"入参绑定来源"表里
- JOIN 出来的字段要标清楚是来自哪个表

## 完成条件

- 草稿 `## 3. 主查询 SQL` 段完整
- 用户回 "Step 3 通过" 进入 Step 4
