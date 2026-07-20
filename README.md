# excode

统一的异常与错误码管理 Python 包，提供标准化的异常类、错误码枚举以及错误处理工具函数。

## 安装

```bash
pip install excode
```

## 快速开始

```python
from excode import ExCodeError, ErrorCode, raise_for_error

# 根据错误码自动抛出对应异常
try:
    raise_for_error(ErrorCode.AUTHENTICATION_ERROR)
except ExCodeError as e:
    print(e)  # [1002] 认证失败，API Key 无效或过期
```

## 错误码一览

| 错误码 | 枚举值 | 说明 |
|--------|--------|------|
| `0` | `ErrorCode.SUCCESS` | 成功 |
| `1000` | `ErrorCode.UNKNOWN_ERROR` | 未知错误 |
| `1001` | `ErrorCode.SERVICE_ERROR` | 服务错误 |
| `1002` | `ErrorCode.AUTHENTICATION_ERROR` | 认证失败 |
| `1003` | `ErrorCode.RATE_LIMIT` | 请求频率超限 |
| `1004` | `ErrorCode.QUOTA_EXCEEDED` | 配额/余额不足 |
| `1005` | `ErrorCode.SERVICE_TIMEOUT` | 服务超时 |
| `2001` | `ErrorCode.INVALID_PARAMS` | 请求参数无效 |
| `3001` | `ErrorCode.PROMPT_BANNED` | 提示词违禁 |
| `3002` | `ErrorCode.IMAGE_BANNED` | 图片违禁 |
| `4001` | `ErrorCode.INVALID_IMAGE` | 图片无效或损坏 |

## 异常类层级

所有异常均继承自 `ExCodeError`，可通过 `except ExCodeError` 统一捕获。

```
ExCodeError
├── ServiceError            # 通用服务错误
├── AuthenticationError     # 认证失败
├── RateLimitError          # 请求频率超限
├── QuotaExceededError      # 配额/余额不足
├── ServiceTimeoutError     # 服务超时
├── InvalidParamsError      # 请求参数无效
├── PromptBanError          # 提示词违禁
├── ImageBanError           # 图片违禁
└── InvalidImageError       # 图片无效或损坏
```

## 使用方式

### 1. 直接抛出异常

```python
from excode import AuthenticationError

raise AuthenticationError("API Key 已过期", extra_data={"key_id": "abc123"})
# 输出: [1002] API Key 已过期 [{'key_id': 'abc123'}]
```

### 2. 根据错误码抛出（工厂函数）

```python
from excode import ErrorCode, raise_for_error

# 使用默认描述
raise_for_error(ErrorCode.RATE_LIMIT)
# 输出: [1003] 请求频率超限，请稍后重试

# 使用自定义描述
raise_for_error(ErrorCode.RATE_LIMIT, error_msg="当前接口调用次数已达上限")
# 输出: [1003] 当前接口调用次数已达上限
```

### 3. 统一捕获并处理

```python
from excode import ExCodeError, ErrorCode, raise_for_error

try:
    raise_for_error(ErrorCode.QUOTA_EXCEEDED)
except ExCodeError as e:
    print(e.error_code)       # ErrorCode.QUOTA_EXCEEDED
    print(e.error_code.value) # 1004
    print(e.error_msg)        # 配额/余额不足
    print(e.extra_data)       # {}
```

### 4. 分类捕获

```python
from excode import AuthenticationError, RateLimitError, ExCodeError

try:
    # 业务逻辑
    ...
except AuthenticationError as e:
    # 处理认证失败
    refresh_api_key()
except RateLimitError as e:
    # 处理限流
    wait_and_retry()
except ExCodeError as e:
    # 兜底处理其他 excode 异常
    log_error(e)
```

### 5. 携带额外上下文

```python
from excode import InvalidParamsError

raise InvalidParamsError(
    "参数校验失败",
    extra_data={"field": "width", "value": -1, "reason": "必须为正整数"}
)
# 输出: [2001] 参数校验失败 [{'field': 'width', 'value': -1, 'reason': '必须为正整数'}]
```

### 6. 特殊异常的扩展属性

```python
from excode import PromptBanError, InvalidImageError

# PromptBanError 携带命中的违禁词列表
raise PromptBanError("提示词违禁", data=["暴力", "色情"])

# InvalidImageError 携带图片尺寸和原始异常
raise InvalidImageError("图片无法解码", image_size=2048, original_error=decode_err)
```

### 7. 获取错误码描述

```python
from excode import ErrorCode, get_error_message

msg = get_error_message(ErrorCode.SERVICE_TIMEOUT)
print(msg)  # 服务超时，请稍后重试
```

### 8. 注册自定义错误码映射

```python
from enum import IntEnum
from excode import ExCodeError, ErrorCode, raise_for_error, register_error_code

# 定义自定义异常
class NetworkError(ExCodeError):
    error_code = ErrorCode.UNKNOWN_ERROR  # 可复用或自行扩展

# 注册映射
register_error_code(ErrorCode.SERVICE_ERROR, NetworkError)

# 之后 raise_for_error 会使用你注册的异常类
raise_for_error(ErrorCode.SERVICE_ERROR, error_msg="网络连接失败")
```

## 技术架构

```
excode/
├── __init__.py       # 包入口，统一导出所有公开 API
├── codes.py          # ErrorCode 枚举、错误码描述映射
├── exceptions.py     # 异常类定义（继承体系）
└── handler.py        # raise_for_error 工厂函数、register_error_code 注册函数
```

