# Step 1：迁移方案设计模板

## 方案设计原则

1. **按文件拆分子步骤**：每个 webcommon 文件的迁移是一个独立子步骤
2. **依赖链决定顺序**：被依赖的文件先迁移，确保每步迁移后项目仍可运行
3. **三阶段优先级**：Phase 1（自有代码）→ Phase 2（SDK）→ Phase 3（移除加载）
4. **只规划复制和引用更新**：不规划任何逻辑修改

## 产出格式（必须包含 composer files 配置区块）

### 0. composer files 加载规则（固定规则）

以下文件必须加入 composer `files` 显式加载：

| 文件 | 目标路径 | 原因 |
|------|---------|------|
| `helpers.php` | `app/common/helpers.php` | 无命名空间，全局函数文件 |
| `Strtool.php` | `app/library/Strtool.php` | helpers.php 内部 `use Library\Strtool`，需确保在 helpers 之前加载 |

其他命名空间文件（如 `felog.php`、`jsonlog.php`）通过框架 loader 或目录注册加载，不需要加入 composer files。

**composer.json files 最终配置**（Phase 3 substep 直接应用）：
```json
"files": [
  "app/common/helpers.php",
  "app/library/Strtool.php"
]
```

---

### 1. 迁移路径映射表

#### Phase 1 - A 类：webcommon 自有代码

| 子步骤 | 源文件（webcommon） | 目标路径（项目本地） | 命名空间调整 | 依赖的其他子步骤 |
|--------|--------------------|--------------------|-------------|----------------|
| substep_1 | `{webcommon源路径}` | `{项目目标路径}` | `{旧namespace}` → `{新namespace}` 或 无需调整 | 无 |
| substep_2 | `{webcommon源路径}` | `{项目目标路径}` | ... | substep_1 |

#### Phase 2 - B 类：SDK 依赖

| 子步骤 | 源文件（webcommon） | 目标路径（项目本地） | 命名空间调整 | 依赖的其他子步骤 |
|--------|--------------------|--------------------|-------------|----------------|
| substep_N | `{SDK源路径}` | `{项目目标路径}` | ... | ... |

#### Phase 3 - C 类：移除加载入口

| 子步骤 | 配置文件 | 操作说明 |
|--------|---------|---------|
| substep_M | `{配置文件路径}` | 移除对 webcommon 的 autoload/bootstrap 加载 |

### 2. 引用更新映射表

记录每个子步骤需要更新的引用点：

| 子步骤 | 需更新的项目文件 | 原引用 | 新引用 |
|--------|----------------|--------|--------|
| substep_1 | `{文件路径}` | `use Webcommon\Xxx\Yyy` | `use App\Xxx\Yyy` |
| substep_1 | `{文件路径}` | `require 'webcommon/...'` | `require 'app/...'` |

### 3. 迁移顺序总览

```
Phase 1 - 自有代码（从底层到上层）：
  substep_1: {文件名} （无依赖，最先迁移）
  substep_2: {文件名} （依赖 substep_1）
  ...

Phase 2 - SDK 依赖：
  substep_N: {SDK文件名}
  ...

Phase 3 - 移除加载：
  substep_M: {配置文件名}
  ...

总计：{N} 个子步骤
```

### 4. 风险点

| 风险 | 影响 | 应对措施 |
|------|------|---------|
| {风险描述} | {影响范围} | {应对方式} |

### 5. 自检清单

```
□ 所有 A 类依赖文件都有对应的子步骤
□ 所有 B 类 SDK 文件都有对应的子步骤
□ 所有 C 类加载入口都有对应的子步骤
□ 依赖链顺序正确（被依赖的先迁移）
□ 每个子步骤的引用更新已列出
□ 目标路径符合项目现有目录结构
□ 命名空间调整仅限 namespace 声明行
□ composer files 策略已与用户确认（helpers 目录 + Strtool 加载方式）
□ composer.json 最终 files 配置已在方案中明确列出
```
