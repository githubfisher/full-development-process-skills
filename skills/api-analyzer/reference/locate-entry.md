# Step 1：入口定位

## 输入
- 用户给的路由，可能形态：
  - `GET /bi/order/list`
  - `POST /api/v2/user/detail`
  - `OrderController@detailAction`
  - 完整 URL：`https://bi.xxx.com/api/order/list`

## Phalcon 路由定位策略（按顺序尝试）

### 策略 A：路由表反查
Phalcon 路由通常注册在：
- `app/config/routes.php`
- `app/config/services.php` 中 `$router->add(...)` 调用
- 模块化路由：`app/modules/{Module}/Module.php` 的 `registerRoutes()`

用 grep 找路径中独特的段：
```bash
grep -rn "order/list" app/config/
grep -rn "->add\(" app/config/ | grep -i "order"
```

### 策略 B：Controller 命名约定
Phalcon 默认路由 `/{module}/{controller}/{action}` 映射到：
- `app/controllers/{Module}/{Controller}Controller.php::{action}Action`

`/bi/order/list` → 候选：
- `app/controllers/Bi/OrderController.php::listAction`
- `app/modules/bi/controllers/OrderController.php::listAction`

### 策略 C：注解路由
某些项目用注解：
```bash
grep -rn "@Route\|@RequestMapping" app/ src/
```

### 策略 D：全仓库直接搜方法名
```bash
grep -rn "function listAction\|function list_action" app/ src/
```

## 多仓库搜索顺序

依据 `workspace/config.md` 的仓库列表，按声明顺序逐仓库 grep，找到第一个命中即停。如果多个命中，报告所有命中位置让用户选。

## 入口确认输出格式

定位完成后，在草稿 markdown 的 `## 1. 基本信息` 段落写入：

```markdown
## 1. 基本信息
- **路由**: GET /bi/order/list
- **入口文件**: app/controllers/Bi/OrderController.php:128
- **类@方法**: \Bi\OrderController::listAction
- **路由注册**: app/config/routes.php:45（如有显式注册）
- **鉴权**: 见 beforeExecuteRoute / Middleware（具体写出位置）
```

## 完成条件

- 文件路径 + 行号已确认
- 用户回 "Step 1 通过" / "下一步" 才能进入 Step 2
