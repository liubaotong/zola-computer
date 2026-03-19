+++
title = "飞书机器人事件回调 WebHook 开发指南"
date = "2026-03-19T02:30:00+08:00"

[taxonomies]
tags = ["飞书", "WebHook", "Python", "API 集成"]
categories = ["网络"]

[extra]
summary = "本文详细介绍如何使用 Python 开发飞书机器人事件回调服务器，涵盖配置流程、加密解密、签名验证等核心内容。"
author = "博主"
+++

本文档总结使用 Python 开发飞书机器人事件回调服务器的完整流程和注意事项。

## 一、配置流程

### 1.1 飞书后台配置

1. 登录 [飞书开发者后台](https://open.feishu.cn/)
2. 创建或选择企业自建应用
3. 进入 **事件与回调** 页面

### 1.2 加密策略配置

在 **事件与回调 -> 加密策略** 页签：

| 参数 | 说明 | 是否必须 |
|------|------|----------|
| **Encrypt Key** | 用于加密/解密事件数据，提高传输安全性 | 可选（建议配置） |
| **Verification Token** | 应用验证标识，用于验证事件来源 | 可选（建议配置） |

> **注意**：
> - `Verification Token` 是系统自动生成的，无需手动设置
> - `Encrypt Key` 需要手动设置，建议使用 16 位以上的随机字符串
> - 如果配置了 `Encrypt Key`，所有推送的事件都会被加密

### 1.3 事件订阅配置

1. 进入 **事件配置** 页签
2. 点击 **订阅方式** 旁边的编辑图标
3. 选择 **将事件发送至开发者服务器**
4. 填写请求地址（必须是公网可访问的 URL）
5. 点击 **保存**

保存后，飞书会向配置的 URL 发送验证请求。

### 1.4 添加事件

1. 在 **已订阅的回调** 区域，点击 **添加事件**
2. 搜索并添加需要订阅的事件，如：
   - `im.message.receive_v1` - 接收消息 v2.0
   - `bot.add` - 机器人加入群组
   - `bot.remove` - 机器人离开群组

---

## 二、服务器开发

### 2.1 核心流程

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  飞书服务器   │ ──► │  你的服务器   │ ──► │  业务逻辑    │
└─────────────┘     └─────────────┘     └─────────────┘
       │                   │
       │  1. URL 验证       │
       │  2. 事件推送       │
       │  (加密/明文)       │
       ▼                   ▼
```

### 2.2 URL 验证处理

飞书保存回调地址时，会发送验证请求：

**请求格式（加密）：**
```json
{
  "encrypt": "Base64 编码的加密数据"
}
```

**解密后格式：**
```json
{
  "challenge": "848d55f5-922e-4a29-a41c-36d4c9ab618b",
  "token": "XlVzQQd7peXa0b2P21kDee4rCAcy1m3D",
  "type": "url_verification"
}
```

**响应要求：**
- 必须在 **1 秒内** 返回
- 响应格式：`{"challenge": "原样返回挑战码"}`
- HTTP 状态码：200

### 2.3 事件处理

**事件推送格式（解密后）：**
```json
{
  "challenge": "",
  "token": "XlVzQQd7peXa0b2P21kDee4rCAcy1m3D",
  "type": "event_callback",
  "event": {
    "type": "im.message.receive_v1",
    "message": {...},
    "sender": {...}
  },
  "header": {
    "event_id": "...",
    "event_type": "...",
    "create_time": "..."
  }
}
```

---

## 三、加密解密实现

### 3.1 加密原理

飞书使用 **AES-256-CBC** 加密算法：

1. 使用 SHA256 对 `Encrypt Key` 进行哈希得到密钥 `key`（32 字节）
2. 生成 16 字节的随机 IV
3. 使用 `key` 和 `IV` 对事件内容加密
4. 将 `IV` 和加密数据拼接后 Base64 编码

### 3.2 Python 解密代码

```python
import base64
import hashlib
import json
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

def decrypt_message(encrypt_key: str, encrypted_data: str) -> dict:
    """解密飞书加密消息"""
    # Base64 解码
    encrypted_bytes = base64.b64decode(encrypted_data)
    
    # IV 是前 16 字节
    iv = encrypted_bytes[:16]
    ciphertext = encrypted_bytes[16:]
    
    # 密钥是 Encrypt Key 的 SHA256 哈希（32 字节）
    key = hashlib.sha256(encrypt_key.encode("utf-8")).digest()
    
    # AES-256-CBC 解密
    cipher = AES.new(key, AES.MODE_CBC, iv)
    decrypted_padded = cipher.decrypt(ciphertext)
    
    # 去除 PKCS7 填充
    padding_len = decrypted_padded[-1]
    decrypted = decrypted_padded[:-padding_len]
    
    return json.loads(decrypted.decode("utf-8"))
```

### 3.3 依赖安装

```bash
pip install pycryptodome
```

---

## 四、签名验证（可选）

如果需要进一步验证请求来源，可以进行签名校验：

### 4.1 签名生成规则

飞书使用以下公式生成签名：

```
signature = SHA256(timestamp + nonce + encryptKey + body)
```

### 4.2 请求头参数

| 参数 | 说明 |
|------|------|
| `X-Lark-Request-Timestamp` | 请求时间戳 |
| `X-Lark-Request-Nonce` | 随机 nonce |
| `X-Lark-Signature` | 签名值 |

### 4.3 验证代码

```python
import hashlib
import hmac
import time

def validate_signature(timestamp: str, nonce: str, body: str, 
                       header_signature: str, encrypt_key: str) -> bool:
    # 时间戳验证（容错 60 秒）
    if abs(int(time.time()) - int(timestamp)) > 60:
        return False
    
    # 计算签名
    sign_string = f"{timestamp}{nonce}{encrypt_key}{body}"
    signature = hashlib.sha256(sign_string.encode("utf-8")).hexdigest()
    
    # 比较签名（常量时间比较）
    return hmac.compare_digest(signature.lower(), header_signature.lower())
```

---

## 五、注意事项

### 5.1 配置相关

| 问题 | 解决方案 |
|------|----------|
| Challenge code 没有返回 | 确保 1 秒内返回 challenge，检查解密逻辑 |
| 解密失败 | 检查 Encrypt Key 是否正确，使用 AES-256-CBC |
| 签名验证失败 | 检查密钥、时间戳、请求体是否被修改 |

### 5.2 安全相关

1. **防重放攻击**：缓存并检查 nonce，拒绝重复请求
2. **时间戳验证**：拒绝超过 60 秒的请求
3. **签名校验**：验证请求来源合法性
4. **HTTPS**：生产环境建议使用 HTTPS

### 5.3 性能相关

1. **响应时间**：URL 验证必须在 1 秒内响应
2. **异步处理**：事件处理建议异步执行，避免阻塞
3. **幂等性**：事件可能重复推送，业务逻辑需支持幂等

### 5.4 调试技巧

1. **打印原始请求**：将飞书的 POST 请求完整打印出来
2. **本地测试**：使用 ngrok 等工具将本地服务暴露到公网
3. **日志记录**：记录解密前后的数据，便于排查问题

---

## 六、完整示例

### 6.1 项目结构

```
/mnt/d/SourceCodes/Python/
├── main.py              # 主服务器代码
├── config.example.py    # 配置示例
└── FeiShu-WebHook.md    # 本文档
```

### 6.2 启动服务器

```bash
# 安装依赖
pip install pycryptodome

# 编辑配置（修改 CONFIG 中的 encrypt_key 和 verification_token）
vim main.py

# 启动服务器
python main.py
```

### 6.3 测试验证

1. 确保服务器在公网可访问
2. 在飞书后台填写回调 URL
3. 点击 **保存**，查看验证结果
4. 验证通过后，添加需要订阅的事件

---

## 七、完整示例代码

### 7.1 自动回复示例

在 `handle_message_event` 中添加自动回复逻辑：

```python
def handle_message_event(event: dict) -> dict:
    """处理消息事件"""
    message = event.get("message", {})
    sender = event.get("sender", {})
    
    # 解析消息内容
    content = message.get("content", {})
    if isinstance(content, str):
        content = json.loads(content)
    
    # 获取聊天 ID 和用户信息
    chat_id = message.get("chat_id", "")
    user_text = content.get("text", "")
    open_id = sender.get("sender_id", {}).get("open_id", "")
    
    logger.info(f"收到来自 {open_id} 的消息：{user_text}")
    
    # 自动回复
    if user_text:
        reply_message(chat_id, f"收到你的消息：{user_text}")
    
    return {"status": "success"}
```

### 7.2 智能回复示例（集成 AI）

```python
import requests

def call_ai_api(user_text: str) -> str:
    """调用 AI 接口生成回复"""
    # 示例：调用某个 AI API
    url = "https://api.example.com/chat"
    response = requests.post(url, json={"message": user_text})
    result = response.json()
    return result.get("reply", "抱歉，我暂时无法回答")

def handle_message_event(event: dict) -> dict:
    """处理消息事件 - 智能回复版"""
    message = event.get("message", {})
    content = message.get("content", {})
    
    if isinstance(content, str):
        content = json.loads(content)
    
    chat_id = message.get("chat_id", "")
    user_text = content.get("text", "")
    
    # 调用 AI 接口生成回复
    ai_reply = call_ai_api(user_text)
    
    # 发送回复
    reply_message(chat_id, ai_reply)
    
    return {"status": "success"}
```

---

## 八、常见问题

### Q1: 验证时返回 401 错误

**原因**：Encrypt Key 或 Verification Token 配置不匹配

**解决**：检查飞书后台和代码中的配置是否一致

### Q2: 解密后是乱码

**原因**：密钥派生方式错误

**解决**：使用完整的 SHA256 哈希（32 字节）作为 AES-256 密钥

### Q3: 事件重复推送

**原因**：网络问题或响应超时导致飞书重试

**解决**：业务逻辑实现幂等性，使用 event_id 去重

### Q4: 如何回复消息

回复消息需要调用飞书的发送消息 API，并且 `receive_id_type` 参数必须放在 URL 查询参数中：

```python
import requests

# 1. 获取 app_access_token
def get_app_access_token(app_id: str, app_secret: str) -> str:
    url = "https://open.feishu.cn/open-apis/auth/v3/app_access_token/internal"
    data = {"app_id": app_id, "app_secret": app_secret}
    response = requests.post(url, json=data)
    result = response.json()
    return result.get("app_access_token", "")

# 2. 发送消息（注意 receive_id_type 在 URL 中）
def reply_message(chat_id: str, content: str, token: str):
    # receive_id_type 必须放在 URL 查询参数中！
    url = "https://open.feishu.cn/open-apis/im/v1/messages?receive_id_type=chat_id"
    headers = {
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json"
    }
    data = {
        "receive_id": chat_id,  # p2p 聊天传 chat_id，群聊也传 chat_id
        "msg_type": "text",
        "content": json.dumps({"text": content})
    }
    response = requests.post(url, headers=headers, json=data)
    return response.json()

# 使用示例
token = get_app_access_token("your_app_id", "your_app_secret")
reply_message("oc_xxx", "收到你的消息", token)
```

**关键点：**
- `receive_id_type` 必须放在 URL 查询参数中，不是请求体
- p2p 聊天和群聊都使用 `chat_id`（以 `oc_` 开头）
- 需要配置应用的 `app_id` 和 `app_secret`

---

## 九、参考文档

- [飞书开放平台 - 事件概述](https://open.feishu.cn/document/ukTMukTMukTM/uUTNz4SN1MjL1UzM?lang=zh-CN)
- [飞书开放平台 - 将事件发送至开发者服务器](https://feishu.apifox.cn/doc-7518435)
- [飞书开放平台 - 接收事件](https://feishu.apifox.cn/doc-7518444)
- [飞书开放平台 - 发送消息](https://open.feishu.cn/document/server-docs/im-v1/message/create)
- [博客：深入理解飞书 Webhook 签名验证](https://www.cnblogs.com/mudtools/p/19492945)
