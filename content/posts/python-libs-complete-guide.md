+++
title = "Python 网络请求、加密和二维码综合指南"
date = "2026-04-12T15:00:00+08:00"

[taxonomies]
tags = ["python", "aiohttp", "cryptography", "qrcode", "异步网络", "加密", "二维码"]
categories = ["编程"]

[extra]
summary = "详细介绍 aiohttp 异步 HTTP 框架、cryptography 加密库和 qrcode 二维码生成库的完整使用指南，包含综合实战案例。"
author = "博主"
+++

本文档详细介绍三个常用 Python 库的功能与最佳实践：

- **aiohttp**: 异步 HTTP 客户端/服务器框架
- **cryptography**: 加密算法和密码学工具
- **qrcode**: 二维码生成工具

最后将演示如何结合三者构建完整的实际应用。

## 目录
<nav id="toc">

- [aiohttp 异步 HTTP 框架](#aiohttp)
- [cryptography 加密和密码学](#cryptography)
- [qrcode 二维码生成](#qrcode)
- [综合实战：加密二维码分发系统](#practice)
- [最佳实践总结](#best-practices)
</nav>

---

<h2 id="aiohttp">aiohttp - 异步 HTTP 框架</h2>

### 简介

`aiohttp` 是基于 `asyncio` 的异步 HTTP 客户端和服务器框架，支持 HTTP/1.1 和 WebSocket，适合高并发 I/O 密集型应用。

### 安装

```bash
pip install aiohttp
```

### 客户端功能

#### 基本 GET 请求

```python
import aiohttp
import asyncio

async def get_request():
    async with aiohttp.ClientSession() as session:
        async with session.get('https://api.example.com/data') as response:
            print(f"状态码: {response.status}")
            data = await response.json()
            return data

asyncio.run(get_request())
```

#### 基本 POST 请求

```python
async def post_request():
    async with aiohttp.ClientSession() as session:
        payload = {'key': 'value'}
        async with session.post(
            'https://api.example.com/submit',
            json=payload  # 或 data=payload 用于表单
        ) as response:
            return await response.json()
```

#### 带参数的请求

```python
# 查询参数
params = {'q': 'search', 'page': 1}
async with session.get(url, params=params) as response:
    ...

# 自定义请求头
headers = {
    'Authorization': 'Bearer token',
    'User-Agent': 'my-app/1.0'
}
async with session.get(url, headers=headers) as response:
    ...
```

#### 并发请求

```python
async def concurrent_requests():
    async with aiohttp.ClientSession() as session:
        # 创建多个任务
        tasks = [
            session.get(f'https://api.example.com/item/{i}')
            for i in range(10)
        ]
        # 并发执行
        responses = await asyncio.gather(*tasks)
        return [await r.json() for r in responses]
```

#### 超时设置

```python
from aiohttp import ClientTimeout

# 总超时 10 秒，连接超时 5 秒
timeout = ClientTimeout(total=10, connect=5)
async with aiohttp.ClientSession(timeout=timeout) as session:
    async with session.get(url) as response:
        ...
```

#### 流式响应

```python
async def stream_download():
    async with aiohttp.ClientSession() as session:
        async with session.get('https://example.com/large-file') as response:
            with open('file.bin', 'wb') as f:
                async for chunk in response.content.iter_chunked(8192):
                    f.write(chunk)
```

#### 文件上传

```python
async def upload_file():
    data = aiohttp.FormData()
    data.add_field('file',
                   open('test.txt', 'rb'),
                   filename='test.txt',
                   content_type='text/plain')

    async with aiohttp.ClientSession() as session:
        async with session.post('https://api.example.com/upload',
                               data=data) as response:
            return await response.json()
```

#### 会话保持（Cookies）

```python
# 自动管理 cookies
async with aiohttp.ClientSession() as session:
    # 第一次请求，服务器设置 cookie
    await session.get('https://api.example.com/login')
    # 后续请求自动携带 cookie
    await session.get('https://api.example.com/protected')
```

### 服务器功能

#### 基本 Web 服务器

```python
from aiohttp import web

async def handle(request):
    name = request.match_info.get('name', "World")
    text = f"Hello, {name}!"
    return web.Response(text=text)

app = web.Application()
app.router.add_get('/{name}', handle)
app.router.add_get('/', handle)

web.run_app(app, host='localhost', port=8080)
```

#### JSON 响应

```python
async def json_handler(request):
    data = {'message': 'success', 'code': 200}
    return web.json_response(data)

app.router.add_get('/api/data', json_handler)
```

#### 处理 POST 请求

```python
async def post_handler(request):
    # JSON 数据
    data = await request.json()

    # 表单数据
    # post_data = await request.post()

    return web.json_response({'received': data})

app.router.add_post('/api/submit', post_handler)
```

#### 中间件

```python
@web.middleware
async def logging_middleware(request, handler):
    print(f"请求：{request.method} {request.path}")
    response = await handler(request)
    print(f"响应：{response.status}")
    return response

app = web.Application(middlewares=[logging_middleware])
```

### WebSocket

#### WebSocket 服务器

```python
async def websocket_handler(request):
    ws = web.WebSocketResponse()
    await ws.prepare(request)

    async for msg in ws:
        if msg.type == aiohttp.WSMsgType.TEXT:
            if msg.data == 'close':
                await ws.close()
            else:
                await ws.send_json({'echo': msg.data})
        elif msg.type == aiohttp.WSMsgType.ERROR:
            print(f'错误：{ws.exception()}')

    return ws

app.router.add_get('/ws', websocket_handler)
```

#### WebSocket 客户端

```python
async def ws_client():
    async with aiohttp.ClientSession() as session:
        async with session.ws_connect('ws://localhost:8080/ws') as ws:
            await ws.send_str('Hello')
            msg = await ws.receive_json()
            print(f"收到：{msg}")
```

### 错误处理

```python
async def safe_request():
    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(url) as response:
                if response.status == 200:
                    return await response.json()
                elif response.status == 404:
                    print("资源未找到")
                else:
                    print(f"错误：{response.status}")
    except aiohttp.ClientError as e:
        print(f"客户端错误：{e}")
    except asyncio.TimeoutError:
        print("请求超时")
```

---

<h2 id="cryptography">cryptography - 加密和密码学</h2>

### 简介

`cryptography` 是 Python 的密码学库，提供高级接口（Fernet、RSA）和底层原语（AES、SHA）。

### 安装

```bash
pip install cryptography
```

### 对称加密（Fernet）

#### 基本加密

```python
from cryptography.fernet import Fernet

# 生成密钥
key = Fernet.generate_key()
fernet = Fernet(key)

# 加密
message = "Secret message".encode()
encrypted = fernet.encrypt(message)

# 解密
decrypted = fernet.decrypt(encrypted).decode()
```

#### 从密码生成密钥

```python
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes
import base64
import os

password = b"my_password"
salt = os.urandom(16)

kdf = PBKDF2HMAC(
    algorithm=hashes.SHA256(),
    length=32,
    salt=salt,
    iterations=480000,  # OWASP 推荐
)
key = base64.urlsafe_b64encode(kdf.derive(password))
fernet = Fernet(key)
```

### AES 加密

#### AES-GCM（推荐）

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

key = os.urandom(32)  # 256 位密钥
nonce = os.urandom(12)

# 加密
encryptor = Cipher(algorithms.AES(key), modes.GCM(nonce)).encryptor()
ciphertext = encryptor.update(b"plaintext") + encryptor.finalize()
tag = encryptor.tag

# 解密
decryptor = Cipher(algorithms.AES(key), modes.GCM(nonce, tag)).decryptor()
plaintext = decryptor.update(ciphertext) + decryptor.finalize()
```

#### AES-CBC

```python
from cryptography.hazmat.primitives import padding

key = os.urandom(32)
iv = os.urandom(16)

# 加密
padder = padding.PKCS7(128).padder()
padded_data = padder.update(b"plaintext") + padder.finalize()

encryptor = Cipher(algorithms.AES(key), modes.CBC(iv)).encryptor()
ciphertext = encryptor.update(padded_data) + encryptor.finalize()

# 解密
decryptor = Cipher(algorithms.AES(key), modes.CBC(iv)).decryptor()
padded_data = decryptor.update(ciphertext) + decryptor.finalize()

unpadder = padding.PKCS7(128).unpadder()
plaintext = unpadder.update(padded_data) + unpadder.finalize()
```

### 非对称加密（RSA）

#### 生成密钥对

```python
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization

# 生成私钥
private_key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=2048,
)
public_key = private_key.public_key()
```

#### 序列化密钥

```python
# 私钥 -> PEM
private_pem = private_key.private_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PrivateFormat.PKCS8,
    encryption_algorithm=serialization.NoEncryption()
)

# 公钥 -> PEM
public_pem = public_key.public_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PublicFormat.SubjectPublicKeyInfo
)

# 从 PEM 加载
loaded_private = serialization.load_pem_private_key(private_pem, password=None)
loaded_public = serialization.load_pem_public_key(public_pem)
```

#### RSA 加密/解密

```python
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.primitives import hashes

# 公钥加密
message = b"secret"
ciphertext = public_key.encrypt(
    message,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None
    )
)

# 私钥解密
plaintext = private_key.decrypt(
    ciphertext,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None
    )
)
```

### 数字签名

```python
# 签名
signature = private_key.sign(
    message,
    padding.PSS(
        mgf=padding.MGF1(hashes.SHA256()),
        salt_length=padding.PSS.MAX_LENGTH
    ),
    hashes.SHA256()
)

# 验证
try:
    public_key.verify(signature, message, padding.PSS(...), hashes.SHA256())
    print("✓ 验证成功")
except Exception:
    print("✗ 验证失败")
```

### 哈希函数

#### SHA-256

```python
from cryptography.hazmat.primitives import hashes

digest = hashes.Hash(hashes.SHA256())
digest.update(b"data")
digest.update(b"more data")
hash_value = digest.finalize()
```

#### HMAC

```python
from cryptography.hazmat.primitives import hmac

key = os.urandom(32)
h = hmac.HMAC(key, hashes.SHA256())
h.update(b"message")
mac_tag = h.finalize()

# 验证
h2 = hmac.HMAC(key, hashes.SHA256())
h2.update(b"message")
h2.verify(mac_tag)  # 失败则抛出异常
```

### 密钥派生

```python
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC

kdf = PBKDF2HMAC(
    algorithm=hashes.SHA256(),
    length=32,
    salt=salt,
    iterations=480000,
)
key = kdf.derive(password)
```

---

<h2 id="qrcode">qrcode - 二维码生成</h2>

### 简介

`qrcode` 是 Python 的二维码生成库，支持自定义样式、颜色、logo 等。

### 安装

```bash
pip install "qrcode[pil]"  # 带 PIL 支持
```

### 基本使用

```python
import qrcode

# 最简单的方式
img = qrcode.make("https://example.com")
img.save("qrcode.png")
```

### 自定义参数

```python
qr = qrcode.QRCode(
    version=1,              # 控制大小（1-40）
    error_correction=qrcode.constants.ERROR_CORRECT_H,  # 高容错
    box_size=10,            # 每格像素
    border=4,               # 边框宽度
)

qr.add_data("https://example.com")
qr.make(fit=True)

img = qr.make_image(fill_color="black", back_color="white")
img.save("custom_qr.png")
```

### 容错级别

| 级别 | 常量 | 容错率 | 说明 |
|------|------|--------|------|
| L | `ERROR_CORRECT_L` | 7% | 最低容错 |
| M | `ERROR_CORRECT_M` | 15% | 默认级别 |
| Q | `ERROR_CORRECT_Q` | 25% | 较高容错 |
| H | `ERROR_CORRECT_H` | 30% | 最高容错（适合加 logo） |

### 彩色渐变二维码

```python
from qrcode.image.styledpil import StyledPilImage
from qrcode.image.styles.moduledrawers import RoundedModuleDrawer
from qrcode.image.styles.colormasks import RadialGradiantColorMask

qr = qrcode.QRCode(error_correction=qrcode.constants.ERROR_CORRECT_H)
qr.add_data("https://example.com")

img = qr.make_image(
    image_style=StyledPilImage,
    module_drawer=RoundedModuleDrawer(),  # 圆角
    color_mask=RadialGradiantColorMask(
        center_color=(0, 100, 255),
        edge_color=(0, 200, 255)
    )
)
img.save("colorful_qr.png")
```

### 添加 Logo

```python
from PIL import Image

qr = qrcode.QRCode(error_correction=qrcode.constants.ERROR_CORRECT_H)
qr.add_data("https://example.com")
qr_img = qr.make_image().convert('RGBA')

# 创建/加载 logo
logo = Image.open("logo.png").resize((80, 80))

# 居中粘贴
qr_width, qr_height = qr_img.size
logo_x = (qr_width - 80) // 2
logo_y = (qr_height - 80) // 2
qr_img.paste(logo, (logo_x, logo_y), logo)

qr_img.save("qr_with_logo.png")
```

### 数据类型支持

```python
# WiFi 配置
wifi_data = "WIFI:T:WPA;S:MyNetwork;P:MyPassword;;"

# 邮箱
email_data = "mailto:user@example.com?subject=Hello"

# 电话
phone_data = "tel:+86-138-0000-0000"

# 纯文本
text_data = "这是一段文本"

# URL
url_data = "https://example.com"
```

---

<h2 id="practice">综合实战：加密二维码分发系统</h2>

本示例演示如何结合三个库构建完整应用：

1. **生成 API 密钥** 并使用 cryptography 加密
2. **创建二维码** 包含加密后的访问令牌
3. **搭建服务器** 验证二维码并处理请求
4. **客户端** 扫描二维码并发出请求

### 完整项目结构

```
secure_qr_system/
├── server.py          # aiohttp 服务器
├── qr_generator.py    # 二维码生成与加密
├── client.py          # 客户端
└── requirements.txt
```

### 核心代码：qr_generator.py

```python
"""二维码生成与加密模块"""

import qrcode
import os
import base64
import time
import json
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC


class SecureQRGenerator:
    """安全的二维码生成器"""

    def __init__(self, master_password: str = None):
        self.master_password = master_password or os.urandom(32).hex()
        self.key = self._derive_key(self.master_password)
        self.fernet = Fernet(self.key)

    def _derive_key(self, password: str) -> bytes:
        """从密码派生加密密钥"""
        salt = b'secure_qr_system_salt'  # 生产环境应随机生成
        kdf = PBKDF2HMAC(
            algorithm=hashes.SHA256(),
            length=32,
            salt=salt,
            iterations=480000,
        )
        key = base64.urlsafe_b64encode(kdf.derive(password.encode()))
        return key

    def generate_token(self, user_id: str, permissions: list) -> str:
        """生成加密令牌"""
        token_data = {
            'user_id': user_id,
            'permissions': permissions,
            'timestamp': time.time(),
            'nonce': os.urandom(16).hex()
        }

        # 加密令牌
        token_json = json.dumps(token_data)
        encrypted = self.fernet.encrypt(token_json.encode())

        return base64.urlsafe_b64encode(encrypted).decode()

    def create_qr(self, token: str, filename: str,
                  fill_color: str = "black",
                  back_color: str = "white"):
        """创建二维码图片"""
        # 构建 URL（包含加密令牌）
        url = f"https://your-app.com/auth?token={token}"

        # 生成二维码
        qr = qrcode.QRCode(
            version=5,
            error_correction=qrcode.constants.ERROR_CORRECT_H,
            box_size=10,
            border=4,
        )
        qr.add_data(url)
        qr.make(fit=True)

        img = qr.make_image(
            fill_color=fill_color,
            back_color=back_color
        )
        img.save(filename)
        print(f"✓ 二维码已保存：{filename}")

        return filename

    def decrypt_token(self, encrypted_token: str) -> dict:
        """解密令牌"""
        encrypted_bytes = base64.urlsafe_b64decode(encrypted_token)
        decrypted = self.fernet.decrypt(encrypted_bytes)

        return json.loads(decrypted)


# 使用示例
if __name__ == "__main__":
    generator = SecureQRGenerator(master_password="my_secret_key")

    # 生成令牌
    token = generator.generate_token(
        user_id="user_123",
        permissions=["read", "write"]
    )
    print(f"加密令牌：{token[:50]}...")

    # 创建二维码
    generator.create_qr(token, "access_qr.png")

    # 解密验证
    token_data = generator.decrypt_token(token)
    print(f"解密数据：{token_data}")
```

### 核心代码：server.py

```python
"""aiohttp 服务器 - 验证二维码令牌"""

import json
import time
from aiohttp import web
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
import base64


class TokenValidator:
    """令牌验证器"""

    def __init__(self, master_password: str):
        salt = b'secure_qr_system_salt'
        kdf = PBKDF2HMAC(
            algorithm=hashes.SHA256(),
            length=32,
            salt=salt,
            iterations=480000,
        )
        key = base64.urlsafe_b64encode(kdf.derive(master_password.encode()))
        self.fernet = Fernet(key)

    def validate_token(self, encrypted_token: str) -> dict:
        """验证并解密令牌"""
        try:
            encrypted_bytes = base64.urlsafe_b64decode(encrypted_token)
            decrypted = self.fernet.decrypt(encrypted_bytes)
            token_data = json.loads(decrypted)

            # 检查令牌是否过期（24 小时）
            if time.time() - token_data['timestamp'] > 86400:
                raise ValueError("令牌已过期")

            return token_data
        except Exception as e:
            raise ValueError(f"令牌无效：{e}")


validator = TokenValidator(master_password="my_secret_key")


async def auth_handler(request):
    """验证二维码令牌"""
    token = request.query.get('token')

    if not token:
        return web.json_response(
            {'error': '缺少令牌'},
            status=400
        )

    try:
        token_data = validator.validate_token(token)
        return web.json_response({
            'status': 'success',
            'message': '认证成功',
            'user_id': token_data['user_id'],
            'permissions': token_data['permissions']
        })
    except ValueError as e:
        return web.json_response({
            'status': 'error',
            'message': str(e)
        }, status=401)


async def api_data_handler(request):
    """受保护的 API 端点"""
    # 验证请求头中的令牌
    auth_header = request.headers.get('Authorization')
    if not auth_header or not auth_header.startswith('Bearer '):
        return web.json_response(
            {'error': '未授权'},
            status=401
        )

    token = auth_header.split(' ', 1)[1]

    try:
        token_data = validator.validate_token(token)
        return web.json_response({
            'data': '敏感数据内容',
            'user': token_data['user_id']
        })
    except ValueError:
        return web.json_response(
            {'error': '令牌无效'},
            status=401
        )


async def websocket_handler(request):
    """WebSocket 实时通信"""
    ws = web.WebSocketResponse()
    await ws.prepare(request)

    token = request.query.get('token')
    if not token:
        await ws.close(code=4001, message=b'缺少令牌')
        return ws

    try:
        token_data = validator.validate_token(token)
        await ws.send_json({
            'type': 'welcome',
            'user_id': token_data['user_id']
        })

        async for msg in ws:
            if msg.type == web.WSMsgType.TEXT:
                await ws.send_json({
                    'type': 'response',
                    'data': f"收到：{msg.data}"
                })

    except ValueError as e:
        await ws.close(code=4001, message=str(e).encode())

    return ws


# 创建应用
app = web.Application()
app.router.add_get('/auth', auth_handler)
app.router.add_get('/api/data', api_data_handler)
app.router.add_get('/ws', websocket_handler)

if __name__ == '__main__':
    print("服务器启动：http://localhost:8080")
    web.run_app(app, host='localhost', port=8080)
```

### 核心代码：client.py

```python
"""客户端 - 扫描二维码并请求服务器"""

import asyncio
import aiohttp


async def authenticate_with_qr(qr_file: str):
    """使用二维码认证"""
    # 读取二维码内容（需要使用 qrcode 读取库）
    # url = read_qr_code(qr_file)

    # 模拟解析 token
    token = "example_token"

    # 发送认证请求
    async with aiohttp.ClientSession() as session:
        async with session.get(
            f'http://localhost:8080/auth?token={token}'
        ) as response:
            result = await response.json()
            print(f"认证结果：{result}")

            if result['status'] == 'success':
                return token

    return None


async def access_protected_api(token: str):
    """访问受保护的 API"""
    async with aiohttp.ClientSession() as session:
        headers = {'Authorization': f'Bearer {token}'}

        async with session.get(
            'http://localhost:8080/api/data',
            headers=headers
        ) as response:
            result = await response.json()
            print(f"API 数据：{result}")


async def websocket_demo(token: str):
    """WebSocket 实时通信"""
    async with aiohttp.ClientSession() as session:
        async with session.ws_connect(
            f'ws://localhost:8080/ws?token={token}'
        ) as ws:
            # 接收欢迎消息
            msg = await ws.receive_json()
            print(f"服务器：{msg}")

            # 发送消息
            await ws.send_str("Hello from client!")
            msg = await ws.receive_json()
            print(f"服务器：{msg}")


async def main():
    """主流程"""
    # 1. 使用二维码认证
    token = await authenticate_with_qr('access_qr.png')

    if token:
        # 2. 访问受保护 API
        await access_protected_api(token)

        # 3. WebSocket 通信
        await websocket_demo(token)


if __name__ == '__main__':
    asyncio.run(main())
```

---

<h2 id="best-practices">最佳实践总结</h2>

### aiohttp 最佳实践

| 场景 | 推荐做法 | 说明 |
|------|----------|------|
| **会话复用** | 复用 `ClientSession` | 避免频繁创建，使用连接池 |
| **并发请求** | 使用 `asyncio.gather()` | 提高吞吐量 |
| **超时控制** | 始终设置 `ClientTimeout` | 防止请求挂起 |
| **错误处理** | 捕获 `aiohttp.ClientError` | 处理网络异常 |
| **资源释放** | 使用 `async with` | 自动关闭连接 |
| **大数据** | 使用流式响应 | 避免内存溢出 |
| **生产环境** | 使用 `gunicorn` + `aiohttp` | 更好的性能和稳定性 |

```python
# ✅ 好的做法
async with aiohttp.ClientSession() as session:
    tasks = [session.get(url) for url in urls]
    responses = await asyncio.gather(*tasks, return_exceptions=True)

# ❌ 不好的做法
for url in urls:
    async with aiohttp.ClientSession() as session:
        await session.get(url)  # 每次创建会话，性能差
```

### cryptography 最佳实践

| 场景 | 推荐做法 | 说明 |
|------|----------|------|
| **对称加密** | 优先使用 Fernet | 简单易用，自动处理 nonce |
| **大数据** | 使用 AES-GCM | 性能好，提供认证 |
| **密钥存储** | 使用环境变量/密钥管理 | 不要硬编码在代码中 |
| **密码派生** | 使用 PBKDF2/Argon2 | 迭代次数至少 480000 |
| **非对称加密** | RSA 至少 2048 位 | 推荐使用 OAEP 填充 |
| **随机数** | 使用 `os.urandom()` | 不要用 `random` 模块 |
| **哈希** | 使用 SHA-256+ | 不要用 MD5/SHA-1 |

```python
# ✅ 好的做法
from cryptography.fernet import Fernet
key = Fernet.generate_key()
fernet = Fernet(key)

# ❌ 不好的做法
import base64
key = base64.urlsafe_b64encode(b'weak_key')  # 弱密钥
```

### qrcode 最佳实践

| 场景 | 推荐做法 | 说明 |
|------|----------|------|
| **容错率** | 添加 logo 用 ERROR_CORRECT_H | 保证二维码可识别 |
| **数据量** | 数据多时增加 version | 自动调整大小 |
| **样式** | 使用 StyledPilImage | 丰富的自定义选项 |
| **清晰度** | 增加 box_size | 打印需要高分辨率 |
| **安全** | 不要在二维码中存敏感信息 | 可被任何人扫描 |
| **验证** | 生成后验证可扫描 | 使用库验证二维码 |

```python
# ✅ 好的做法
qr = qrcode.QRCode(
    error_correction=qrcode.constants.ERROR_CORRECT_H,
    box_size=10,
)
qr.add_data("公开信息")

# ❌ 不好的做法
qr.add_data("password=123456")  # 敏感信息不应放在二维码中
```

### 综合最佳实践

1. **不要在二维码中存储敏感信息**
   - 使用加密令牌代替直接存储密码或密钥
   - 二维码应包含可撤销的临时令牌

2. **令牌应具备时效性**
   - 设置合理的过期时间（如 24 小时）
   - 使用 nonce 防止重放攻击

3. **分层安全策略**
   - 二维码仅作为入口验证
   - 关键操作需要额外验证（如二次认证）

4. **日志与监控**
   - 记录所有认证尝试
   - 监控异常模式（如暴力破解）

5. **错误信息脱敏**
   - 不向客户端暴露详细的错误原因
   - 统一返回 "认证失败" 而非 "令牌过期"/"签名无效"

---

## 附录：常用命令

### 安装依赖

```bash
pip install aiohttp cryptography "qrcode[pil]"
```

### 生成 requirements.txt

```bash
pip freeze > requirements.txt
```

### 快速启动服务器

```bash
python server.py
```

### 运行客户端

```bash
python client.py
```

---

## 参考资料

- [aiohttp 官方文档](https://docs.aiohttp.org/)
- [cryptography 官方文档](https://cryptography.io/)
- [qrcode 官方文档](https://github.com/lincolnloop/python-qrcode)
