# Python 开发规范

## 1. 工程结构

### 1.1 目录组织
```
project/
├── src/                     # 源代码根目录
│   ├── <package_name>/      # 主包
│   │   ├── __init__.py
│   │   ├── api/             # 接口层（FastAPI/Flask 路由）
│   │   ├── services/        # 业务逻辑层
│   │   ├── repositories/    # 数据访问层
│   │   ├── models/          # ORM 模型或领域模型
│   │   ├── schemas/         # Pydantic 序列化/校验模型
│   │   ├── core/            # 配置、异常、安全、工具
│   │   └── utils/           # 通用工具
│   └── tests/               # 测试，与源码镜像结构
├── scripts/                 # 辅助脚本
├── docs/                    # 文档
├── pyproject.toml           # 项目元数据与工具配置
├── .env.example             # 环境变量示例
└── README.md
```

### 1.2 模块与包
- 【强制】包使用 `__init__.py`（可空），模块名全小写、下划线分隔。
- 【强制】每个模块应职责单一，避免一个模块包含过多无关类或函数。
- 【推荐】使用相对导入或绝对导入，同一包内优先用相对导入（`from .module import X`）。

## 2. 命名规范

| 类型                 | 风格           | 示例                                |
| -------------------- | -------------- | ----------------------------------- |
| 包/模块              | 全小写、下划线 | `my_package`, `data_utils`          |
| 类/异常              | 大驼峰         | `UserService`, `ValidationError`    |
| 函数/方法            | 小写下划线     | `get_user_by_id`, `calculate_total` |
| 变量                 | 小写下划线     | `user_list`, `max_retry_count`      |
| 常量                 | 全大写、下划线 | `MAX_SIZE`, `DEFAULT_TIMEOUT`       |
| 私有成员             | 单下划线前缀   | `_private_method`, `_internal_data` |
| 双下划线（避免冲突） | 双下划线前缀   | `__hidden_attr`                     |
| 枚举值               | 全大写、下划线 | `OrderStatus.PENDING`               |

- 【强制】避免单字符变量名（循环中允许 `i`, `j`, `_` 等），禁止拼音或中英混合。
- 【推荐】布尔变量名以 `is_`、`has_`、`can_` 等开头。
- 【推荐】工厂函数、类方法名尽量体现构造方式，如 `from_json()`、`create_default()`。

## 3. 代码风格

### 3.1 缩进与长度
- 【强制】使用 4 个空格缩进，禁止 Tab。
- 【强制】单行长度不超过 120 个字符（文档字符串/注释不超过 72 字符）。
- 【强制】续行时，垂直对齐参数或增加悬挂缩进（4 空格）。

```python
# 垂直对齐
result = some_function(argument_one, argument_two,
                       argument_three)

# 悬挂缩进
result = some_function(
    argument_one,
    argument_two,
    argument_three,
)
```

### 3.2 空格与空行
- 【强制】二元运算符两侧加空格（`a + b`），逗号、冒号后加空格，之前不加。
- 【强制】函数定义之间空两行，类定义之间空两行，类内方法之间空一行。
- 【强制】函数体内逻辑分段用空行，但避免连续的多个空行。

### 3.3 导入
- 【强制】每个导入单独一行（`import os`, `import sys`），避免 `import os, sys`。
- 【强制】导入顺序：标准库 → 第三方库 → 本地模块，每组之间空行。
- 【强制】禁止使用通配符导入（`from module import *`）。
- 【推荐】使用绝对导入，包内可用相对导入。

### 3.4 引号
- 【强制】字符串使用双引号或单引号，保持项目一致性（推荐双引号，文档字符串用三重双引号）。
- 【推荐】优先使用 f-string 或 `.format()`，避免 `%` 格式化。

### 3.5 括号与换行
- 【推荐】如果 `if/for/while` 体只有一条语句，可不加大括号但缩进必须正确；但鼓励始终使用换行缩进明确控制流。
- 【强制】多行表达式在二元运算符之前换行。

```python
total = (first_variable
         + second_variable
         - third_variable)
```

## 4. 编程实践

### 4.1 类型注解
- 【强制】所有公开函数/方法必须添加类型注解，参数和返回值明确标注。
- 【强制】使用 `from __future__ import annotations`（Python 3.10+ 可选）延迟求值，支持前向引用。
- 【推荐】复杂类型使用 `typing` 模块：`List[str]`、`Optional[int]`、`Union[str, int]`，Python 3.10+ 可用 `str | None`。

```python
def get_user(user_id: int) -> User | None:
    ...
```

### 4.2 文档注释 (Docstring)
- 【强制】所有公开模块、类、方法/函数必须有 docstring，使用三重双引号。
- 【强制】采用 Google 风格或 NumPy 风格（项目统一选一种），描述功能、Args、Returns、Raises。

```python
def divide(a: float, b: float) -> float:
    """计算两数相除。

    Args:
        a: 被除数。
        b: 除数，不能为 0。

    Returns:
        a 除以 b 的结果。

    Raises:
        ZeroDivisionError: 如果 b 为 0。
    """
    if b == 0:
        raise ZeroDivisionError("除数不能为 0")
    return a / b
```

### 4.3 异常处理
- 【强制】异常不用于流程控制，捕获具体异常而非裸 `except:`。
- 【强制】捕获异常后必须记录日志或重新抛出，禁止空 `pass`。
- 【推荐】定义应用级异常基类 `AppException`，各层派生具体异常。
- 【强制】资源清理使用 `try...finally` 或上下文管理器（`with`）。

### 4.4 上下文管理与资源
- 【强制】文件、网络连接、锁等资源必须使用 `with` 语句管理。
- 【推荐】自定义资源管理类实现 `__enter__` 和 `__exit__`。

### 4.5 集合与迭代
- 【强制】使用推导式而非 `map/filter`（可读性更强），复杂逻辑使用传统循环。
- 【强制】字典遍历使用 `items()`，需要索引用 `enumerate`，同时遍历多个序列用 `zip`。
- 【强制】判断集合是否为空用 `if not collection:` 而非 `len(collection) == 0`。
- 【推荐】使用生成器表达式处理大数据流，节省内存。

### 4.6 比较与空值
- 【强制】与 `None` 比较使用 `is`/`is not`，不使用 `==`。
- 【强制】检查变量是否为 `None` 使用 `if x is None`。
- 【强制】使用 `isinstance()` 检查类型，避免 `type() ==`。

### 4.7 字符串与编码
- 【强制】源文件头部声明 `# -*- coding: utf-8 -*-`（Python 3 默认为 UTF-8，可省略但建议保留以明确）。
- 【推荐】文件操作显式指定 `encoding="utf-8"`。

### 4.8 数据类与模型
- 【推荐】优先使用 `dataclasses` 或 `Pydantic` 定义数据结构，避免裸字典传递数据。
- 【强制】面向外部 API 的输入输出使用 Pydantic 模型进行校验和序列化。

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int
```

### 4.9 并发与异步
- 【推荐】IO 密集型业务使用 `async/await`，CPU 密集型任务使用进程池或任务队列。
- 【强制】异步函数不调用阻塞 IO 方法，需用 `run_in_executor` 或异步库。
- 【推荐】使用 `asyncio.gather()` 并发运行多个协程，注意异常处理。

## 5. 异常与日志

### 5.1 日志
- 【强制】使用标准库 `logging`，不直接使用 `print()` 输出日志。
- 【强制】日志记录采用模块级 logger：`logger = logging.getLogger(__name__)`。
- 【强制】敏感信息（密码、token）禁止写入日志。
- 【推荐】根据环境配置日志级别，生产环境使用 INFO 或 WARNING。
- 【推荐】异常日志使用 `logger.exception()` 自动附加堆栈。

```python
import logging

logger = logging.getLogger(__name__)

def process_order(order_id: str) -> None:
    logger.info("开始处理订单 order_id=%s", order_id)
    try:
        ...
    except Exception as e:
        logger.exception("处理订单失败 order_id=%s", order_id)
        raise
```

### 5.2 异常设计
- 【强制】自定义异常继承自 `Exception`，提供清晰的错误信息。
- 【推荐】错误码体系可参考 HTTP 状态码分段定义业务错误码，存储在 `core/exceptions.py`。

## 6. 配置管理

- 【强制】配置分离，使用环境变量或配置文件（如 YAML/TOML），不可硬编码到代码中。
- 【推荐】使用 Pydantic Settings 或 `python-dotenv` 加载环境变量，并提供默认值。
- 【强制】敏感配置（密钥、数据库密码）严禁提交到版本控制，通过 `.env` 文件排除在 Git 外。

## 7. 测试规范

- 【强制】使用 `pytest` 框架，测试文件以 `test_` 开头，测试函数以 `test_` 开头。
- 【强制】测试目录结构与源码镜像，如 `tests/services/test_user.py`。
- 【推荐】单元测试覆盖核心业务逻辑，API 测试可使用 `TestClient` (FastAPI) 或类似工具。
- 【推荐】使用 `fixture` 管理测试前置条件和资源，避免重复代码。
- 【推荐】运行测试时加入 `--cov` 检查覆盖率，目标 > 80%。

## 8. 依赖与工具

### 8.1 依赖管理
- 【强制】使用 `pyproject.toml` 管理项目元数据与依赖，使用 `pip` 或 `poetry` 锁定版本。
- 【强制】生产依赖与开发依赖分离（`[project.optional-dependencies]` dev 组）。

### 8.2 代码质量工具
- 【强制】使用 `black` 或 `ruff` 进行自动格式化，配置在 `pyproject.toml`。
- 【强制】使用 `ruff` 或 `flake8` 进行代码风格检查。
- 【强制】使用 `mypy` 进行静态类型检查（至少对核心模块）。
- 【推荐】使用 `pre-commit` 在提交前自动运行检查。

## 9. 安全实践

- 【强制】禁止使用 `eval()`、`exec()` 动态执行代码。
- 【强制】SQL 查询使用参数化查询或 ORM，严禁拼接字符串防止注入。
- 【强制】用户输入进行校验与清洗，输出到 HTML 需转义（XSS 防护）。
- 【强制】密码使用 `bcrypt`/`hashlib` 加盐哈希存储。

## 10. 快速检查清单

- 包/模块名小写下划线，类名大驼峰，函数/变量小写下划线。
- 所有公开 API 有类型注解和 docstring。
- 缩进 4 空格，行长 ≤120，使用 f-string 格式化。
- 导入按标准库、第三方、本地分组，无通配符导入。
- 异常具体捕获并记录日志，资源用 `with` 管理。
- 配置分离，敏感信息不提交仓库。
- 测试覆盖核心逻辑，使用 `pytest`。
- 使用 `black`/`ruff`/`mypy` 检查代码质量。