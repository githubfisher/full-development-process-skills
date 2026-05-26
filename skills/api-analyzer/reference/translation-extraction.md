# Step 5：字段翻译提取

## 翻译 key 的出现形态

PHP/Phalcon 项目常见的翻译调用：

```php
$this->translate->_('order.status.paid')
$this->t('user.level.vip')
__('common.empty_data')
Translate::get('xxx.yyy')
trans('order.status.' . $row['status'])  // 动态拼接
```

枚举映射类内部也常调用翻译：
```php
class OrderStatus {
    public static function label(int $s): string {
        return self::LABELS[$s] ?? '';   // 直接中文
        // 或
        return __('order.status.' . self::CODE_MAP[$s]);
    }
}
```

## 提取流程

1. 在 action / service / 后处理代码中 grep 上述模式，列出所有静态 key
2. 动态拼接的 key（`'order.status.' . $row['status']`）→ 找到 `$row['status']` 的取值范围（看主 SQL / 枚举类），列出全部可能 key
3. 拿到 key 后，根据 `workspace/config.md` 中的 locale 路径，定位翻译文件，取实际译文
   - Phalcon 常用 `messages.php` 数组形式
   - 或 ini / json / yaml

## locale 文件常见组织

```
app/messages/zh_CN/order.php      → 数组 ['status' => ['paid' => '已支付', ...]]
app/messages/en_US/order.php
src/Translation/zh.json
```

读 `zh_CN` 即可（如有英文需求，再加 en 列）。

## 输出格式

```markdown
## 5. 字段翻译表

### 5.1 使用的翻译 key 列表（含义已取自 locale 文件）

| 翻译 key | 中文译文 | 使用位置 | 使用场景 |
|----------|----------|----------|----------|
| order.status.unpaid | 未支付 | OrderService.php:112 | status=0 → status_text |
| order.status.paid | 已支付 | OrderService.php:112 | status=1 → status_text |
| order.status.shipped | 已发货 | OrderService.php:112 | status=2 → status_text |
| order.status.completed | 已完成 | OrderService.php:112 | status=3 → status_text |
| order.empty_list | 暂无订单 | OrderController.php:140 | 返回空列表时 |

### 5.2 翻译文件来源

- `app/messages/zh_CN/order.php`
- `app/messages/zh_CN/common.php`

### 5.3 动态 key 展开说明

代码 `__('order.status.' . $row['status_code'])`，`$row['status_code']` 取值：
- `unpaid` / `paid` / `shipped` / `completed`（来源：orders.status_code 字段，DDL 枚举）
```

## 关键要求

- key 不能只列代码里出现的字面量；动态拼接的要展开
- 译文必须从 locale 文件实际取，缺失要标 `（locale 文件中未找到，疑似硬编码）`
- 使用位置要给文件:行号
- 如果项目用了多语言，至少列 zh_CN；其他语言看用户需要

## 完成条件

- `## 5. 字段翻译表` 段完整
- 用户回 "Step 5 通过" 进入 Step 6
