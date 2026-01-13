# Token.html 使用指南 - Python 服务器版本

本文档详细说明如何使用 `token.html` 工具生成 StandX API 访问凭证，并在 Python 代码中集成这些凭证进行自动化交易。

---

## 目录

1. [工具简介](#1-工具简介)
2. [环境准备](#2-环境准备)
3. [启动本地服务器](#3-启动本地服务器)
4. [使用 token.html 生成凭证](#4-使用-tokenhtml-生成凭证)
5. [Python 代码集成](#5-python-代码集成)
6. [完整示例代码](#6-完整示例代码)
7. [常见问题](#7-常见问题)

---

## 1. 工具简介

`token.html` 是一个本地网页工具，用于生成 StandX 交易所 API 访问所需的认证凭证。

### 1.1 核心功能

- **本地生成 Ed25519 密钥对**：私钥不触网，保障安全
- **MetaMask 身份验证**：通过链下签名验证身份（无 Gas 费用）
- **获取 API Token**：生成有效期 7 天的 JWT Token
- **导出签名密钥**：提供十六进制格式的 Ed25519 私钥用于 API 请求签名

### 1.2 安全机制

StandX 采用双重认证机制：
- **身份认证**：通过 MetaMask 钱包签名验证用户身份
- **请求签名**：每个 API 请求使用 Ed25519 临时密钥签名

这种设计确保即使 API 凭证泄露，攻击者也无法访问您的链上资产。

---

## 2. 环境准备

### 2.1 前置要求

- **浏览器**：Chrome、Firefox、Edge 等现代浏览器
- **MetaMask 插件**：已安装并配置好钱包
- **Python**：Python 3.7 或更高版本

### 2.2 检查 Python 版本

```bash
python --version
# 或
python3 --version
```

如果未安装 Python，请访问 [python.org](https://www.python.org/downloads/) 下载安装。

---

## 3. 启动本地服务器

由于浏览器的安全策略限制，直接双击打开 HTML 文件可能导致某些功能无法正常工作。我们需要使用本地 HTTP 服务器。

### 3.1 方法一：使用 Python 内置服务器（推荐）

#### Python 3.x

```bash
# 在 token.html 所在目录打开终端，运行：
python -m http.server 8000
```

#### Python 2.x

```bash
python -m SimpleHTTPServer 8000
```

### 3.2 方法二：使用 VS Code Live Server

如果您使用 VS Code：
1. 安装 "Live Server" 插件
2. 右键点击 `token.html`
3. 选择 "Open with Live Server"

### 3.3 访问工具

启动服务器后，在浏览器中访问：

```
http://localhost:8000/token.html
```

---

## 4. 使用 token.html 生成凭证

### 4.1 步骤 1：连接 MetaMask & 生成密钥

1. 点击页面上的 **"1. 连接 MetaMask & 生成密钥"** 按钮
2. MetaMask 会弹出连接请求，选择您要使用的账户
3. 工具会自动生成 Ed25519 密钥对
4. 页面会显示 `Request ID`（公钥的 Base58 编码）

**运行日志示例：**
```
[14:23:45] 正在生成 Ed25519 密钥对...
[14:23:45] Ed25519 密钥生成完毕。RequestId: 5J7Kq...
[14:23:45] 请求连接钱包...
[14:23:47] 钱包已连接: 0x1234...5678
[14:23:47] 正在请求服务器预签名数据 (prepare-signin)...
[14:23:48] 获取 signedData 成功。
[14:23:48] 解析签名消息成功。准备签名...
```

### 4.2 步骤 2：签名并登录

1. 点击 **"2. 签名并登录"** 按钮
2. MetaMask 会弹出签名请求（这是链下签名，不消耗 Gas）
3. 确认签名后，工具会向 StandX 服务器请求 Token
4. 登录成功后，页面会显示三个关键信息

**运行日志示例：**
```
[14:24:01] 请在 MetaMask 中签名消息...
[14:24:05] 签名完成。
[14:24:05] 正在提交登录请求 (login)...
[14:24:06] 登录成功！
[14:24:06] === 流程结束，请复制 Token 和 Private Key 使用 ===
```

### 4.3 保存关键信息

登录成功后，页面会显示以下信息（点击输入框可自动全选复制）：

1. **API Access Token (JWT)**
   ```
   eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIweDEyMzQiLCJpYXQiOjE3MDk...
   ```
   - 用途：API 请求的身份认证
   - 有效期：7 天

2. **Ed25519 Private Key (Hex)**
   ```
   a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456
   ```
   - 用途：为每个 API 请求的 Body 进行签名
   - 格式：64 字符的十六进制字符串

3. **Request ID (Public Key Base58)**
   ```
   5J7KqRt8NmP3vWxYz...
   ```
   - 用途：公钥标识符（通常不需要在代码中使用）

⚠️ **重要提示**：请将这些信息保存到安全的地方，不要分享给他人！

---

## 5. Python 代码集成

### 5.1 安装依赖

```bash
pip install requests pynacl
```

**依赖说明：**
- `requests`：用于发送 HTTP 请求
- `pynacl`：用于 Ed25519 签名（PyNaCl 是 libsodium 的 Python 绑定）

### 5.2 环境变量配置

创建 `.env` 文件（推荐）：

```bash
# .env
STANDX_ACCESS_TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
STANDX_SIGNING_KEY=a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456
```

安装 `python-dotenv` 来加载环境变量：

```bash
pip install python-dotenv
```

---

## 6. 完整示例代码

### 6.1 基础版本（硬编码凭证）

```python
import requests
import json
import time
import uuid
from nacl.signing import SigningKey
from nacl.encoding import RawEncoder
import base64

# ================= 配置区域 =================
# 从 token.html 复制的凭证
ACCESS_TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
ED25519_PRIV_HEX = "a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456"

# 将十六进制私钥转换为字节
private_key_bytes = bytes.fromhex(ED25519_PRIV_HEX)
signing_key = SigningKey(private_key_bytes)
# ===========================================


def get_auth_headers(payload_str: str) -> dict:
    """
    生成 StandX 请求所需的签名 Header
    
    Args:
        payload_str: JSON 字符串格式的请求体
        
    Returns:
        包含所有认证头的字典
    """
    timestamp = str(int(time.time() * 1000))  # 毫秒时间戳
    request_id = str(uuid.uuid4())
    version = "v1"
    
    # 签名规则: "{version},{id},{timestamp},{payload}"
    sign_msg = f"{version},{request_id},{timestamp},{payload_str}"
    
    # 使用 Ed25519 私钥进行签名
    signed = signing_key.sign(sign_msg.encode('utf-8'), encoder=RawEncoder)
    signature_bytes = signed.signature
    
    # 转换为 Base64
    signature_base64 = base64.b64encode(signature_bytes).decode('utf-8')
    
    return {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {ACCESS_TOKEN}",
        "x-request-sign-version": version,
        "x-request-id": request_id,
        "x-request-timestamp": timestamp,
        "x-request-signature": signature_base64
    }


def place_order(symbol: str, side: str, order_type: str, quantity: str, price: str = None):
    """
    下单示例
    
    Args:
        symbol: 交易对，如 "BTC-USDT"
        side: 买卖方向，"BUY" 或 "SELL"
        order_type: 订单类型，"LIMIT" 或 "MARKET"
        quantity: 数量
        price: 价格（市价单可不传）
    """
    url = "https://api.standx.com/v1/trade/order"
    
    # 构造请求体
    payload = {
        "symbol": symbol,
        "side": side,
        "type": order_type,
        "quantity": quantity
    }
    
    if price:
        payload["price"] = price
    
    # 必须对 JSON 字符串进行签名，且发送的 body 必须与签名的字符串完全一致
    payload_str = json.dumps(payload, separators=(',', ':'))
    
    try:
        response = requests.post(
            url,
            data=payload_str,
            headers=get_auth_headers(payload_str)
        )
        
        print(f"状态码: {response.status_code}")
        print(f"响应: {response.json()}")
        
        return response.json()
        
    except Exception as e:
        print(f"请求失败: {e}")
        if hasattr(e, 'response') and e.response:
            print(f"错误详情: {e.response.text}")
        return None


def get_account_info():
    """
    获取账户信息示例
    """
    url = "https://api.standx.com/v1/account/info"
    
    # GET 请求，payload 为空字符串
    payload_str = ""
    
    try:
        response = requests.get(
            url,
            headers=get_auth_headers(payload_str)
        )
        
        print(f"账户信息: {response.json()}")
        return response.json()
        
    except Exception as e:
        print(f"请求失败: {e}")
        return None


# 使用示例
if __name__ == "__main__":
    print("=== StandX API 测试 ===\n")
    
    # 1. 获取账户信息
    print("1. 获取账户信息:")
    get_account_info()
    print()
    
    # 2. 下限价单
    print("2. 下限价单:")
    place_order(
        symbol="BTC-USDT",
        side="BUY",
        order_type="LIMIT",
        quantity="0.001",
        price="60000"
    )
    print()
    
    # 3. 下市价单
    print("3. 下市价单:")
    place_order(
        symbol="ETH-USDT",
        side="SELL",
        order_type="MARKET",
        quantity="0.1"
    )
```

### 6.2 生产版本（使用环境变量）

```python
import os
import requests
import json
import time
import uuid
from nacl.signing import SigningKey
from nacl.encoding import RawEncoder
import base64
from dotenv import load_dotenv

# 加载环境变量
load_dotenv()

# 从环境变量读取凭证
ACCESS_TOKEN = os.getenv("STANDX_ACCESS_TOKEN")
ED25519_PRIV_HEX = os.getenv("STANDX_SIGNING_KEY")

if not ACCESS_TOKEN or not ED25519_PRIV_HEX:
    raise ValueError("请在 .env 文件中配置 STANDX_ACCESS_TOKEN 和 STANDX_SIGNING_KEY")

# 初始化签名密钥
private_key_bytes = bytes.fromhex(ED25519_PRIV_HEX)
signing_key = SigningKey(private_key_bytes)


class StandXClient:
    """StandX API 客户端"""
    
    def __init__(self, base_url: str = "https://api.standx.com"):
        self.base_url = base_url
        self.access_token = ACCESS_TOKEN
        self.signing_key = signing_key
    
    def _get_auth_headers(self, payload_str: str) -> dict:
        """生成认证头"""
        timestamp = str(int(time.time() * 1000))
        request_id = str(uuid.uuid4())
        version = "v1"
        
        sign_msg = f"{version},{request_id},{timestamp},{payload_str}"
        signed = self.signing_key.sign(sign_msg.encode('utf-8'), encoder=RawEncoder)
        signature_base64 = base64.b64encode(signed.signature).decode('utf-8')
        
        return {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {self.access_token}",
            "x-request-sign-version": version,
            "x-request-id": request_id,
            "x-request-timestamp": timestamp,
            "x-request-signature": signature_base64
        }
    
    def _request(self, method: str, endpoint: str, payload: dict = None) -> dict:
        """统一请求方法"""
        url = f"{self.base_url}{endpoint}"
        
        if payload:
            payload_str = json.dumps(payload, separators=(',', ':'))
        else:
            payload_str = ""
        
        headers = self._get_auth_headers(payload_str)
        
        try:
            if method.upper() == "GET":
                response = requests.get(url, headers=headers)
            elif method.upper() == "POST":
                response = requests.post(url, data=payload_str, headers=headers)
            elif method.upper() == "DELETE":
                response = requests.delete(url, data=payload_str, headers=headers)
            else:
                raise ValueError(f"不支持的请求方法: {method}")
            
            response.raise_for_status()
            return response.json()
            
        except requests.exceptions.RequestException as e:
            print(f"请求失败: {e}")
            if hasattr(e, 'response') and e.response:
                print(f"错误详情: {e.response.text}")
            raise
    
    def get_account_info(self) -> dict:
        """获取账户信息"""
        return self._request("GET", "/v1/account/info")
    
    def place_order(self, symbol: str, side: str, order_type: str, 
                   quantity: str, price: str = None) -> dict:
        """下单"""
        payload = {
            "symbol": symbol,
            "side": side,
            "type": order_type,
            "quantity": quantity
        }
        if price:
            payload["price"] = price
        
        return self._request("POST", "/v1/trade/order", payload)
    
    def cancel_order(self, order_id: str) -> dict:
        """取消订单"""
        return self._request("DELETE", f"/v1/trade/order/{order_id}")
    
    def get_order_history(self, symbol: str = None, limit: int = 50) -> dict:
        """获取订单历史"""
        endpoint = f"/v1/trade/orders?limit={limit}"
        if symbol:
            endpoint += f"&symbol={symbol}"
        return self._request("GET", endpoint)


# 使用示例
if __name__ == "__main__":
    client = StandXClient()
    
    print("=== StandX API 客户端测试 ===\n")
    
    try:
        # 获取账户信息
        print("1. 账户信息:")
        account = client.get_account_info()
        print(json.dumps(account, indent=2, ensure_ascii=False))
        print()
        
        # 下单
        print("2. 下限价单:")
        order = client.place_order(
            symbol="BTC-USDT",
            side="BUY",
            order_type="LIMIT",
            quantity="0.001",
            price="60000"
        )
        print(json.dumps(order, indent=2, ensure_ascii=False))
        print()
        
        # 获取订单历史
        print("3. 订单历史:")
        history = client.get_order_history(symbol="BTC-USDT", limit=10)
        print(json.dumps(history, indent=2, ensure_ascii=False))
        
    except Exception as e:
        print(f"执行失败: {e}")
```

---

## 7. 常见问题

### 7.1 浏览器相关

**Q: 为什么直接打开 HTML 文件不工作？**

A: 由于浏览器的 CORS 安全策略，某些功能（如加载 ES 模块）需要通过 HTTP 服务器访问。请使用 Python 的 `http.server` 或其他本地服务器。

**Q: MetaMask 没有弹出连接请求？**

A: 
- 确保已安装 MetaMask 插件
- 检查浏览器控制台是否有错误信息
- 尝试刷新页面重新操作

### 7.2 认证相关

**Q: Token 过期了怎么办？**

A: Token 默认有效期为 7 天。过期后，API 请求会返回 `401` 错误。此时需要：
1. 重新打开 `token.html`
2. 重新执行生成凭证流程
3. 更新代码中的 `ACCESS_TOKEN` 和 `ED25519_PRIV_HEX`

**Q: 签名验证失败（403 错误）？**

A: 常见原因：
- **Body 不一致**：签名的字符串与实际发送的 Body 不完全相同
  - 解决方案：使用 `json.dumps(payload, separators=(',', ':'))` 确保格式一致
- **时间戳错误**：确保使用毫秒级时间戳
- **私钥错误**：检查是否正确复制了完整的 64 字符十六进制私钥

### 7.3 Python 相关

**Q: 如何安装 PyNaCl？**

A: 
```bash
pip install pynacl
```

如果遇到编译错误，可以尝试安装预编译版本：
```bash
pip install --only-binary :all: pynacl
```

**Q: 如何验证签名是否正确？**

A: 可以添加调试代码：
```python
def get_auth_headers(payload_str: str) -> dict:
    # ... 原有代码 ...
    
    # 调试输出
    print(f"签名消息: {sign_msg}")
    print(f"签名结果: {signature_base64}")
    
    return headers
```

### 7.4 安全相关

**Q: Ed25519 私钥泄露了怎么办？**

A: 
1. 立即停止使用该私钥
2. 重新生成新的凭证
3. 检查是否有异常交易
4. Ed25519 私钥只能在 Token 有效期内（7天）操作 API，无法转移链上资产

**Q: 如何安全存储凭证？**

A: 
- ✅ 使用 `.env` 文件（不要提交到 Git）
- ✅ 使用环境变量
- ✅ 使用密钥管理服务（如 AWS Secrets Manager）
- ❌ 不要硬编码在代码中
- ❌ 不要提交到版本控制系统

---

## 8. 最佳实践

### 8.1 开发环境

```python
# .env.development
STANDX_ACCESS_TOKEN=dev_token_here
STANDX_SIGNING_KEY=dev_key_here
STANDX_API_BASE_URL=https://testnet.standx.com
```

### 8.2 生产环境

```python
# .env.production
STANDX_ACCESS_TOKEN=prod_token_here
STANDX_SIGNING_KEY=prod_key_here
STANDX_API_BASE_URL=https://api.standx.com
```

### 8.3 错误处理

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def safe_request(func):
    """请求装饰器，统一错误处理"""
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except requests.exceptions.HTTPError as e:
            logger.error(f"HTTP 错误: {e.response.status_code} - {e.response.text}")
            raise
        except Exception as e:
            logger.error(f"请求失败: {str(e)}")
            raise
    return wrapper

@safe_request
def place_order_safe(*args, **kwargs):
    return client.place_order(*args, **kwargs)
```

---

## 9. 总结

通过本指南，您应该能够：

1. ✅ 使用 Python 启动本地服务器
2. ✅ 使用 `token.html` 生成 API 凭证
3. ✅ 在 Python 代码中集成 StandX API
4. ✅ 理解签名机制和安全最佳实践

如有问题，请参考 [StandX 官方文档](https://docs.standx.com) 或联系技术支持。
