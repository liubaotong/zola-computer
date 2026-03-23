+++
title = "密码学算法完全指南 2026：Golang 和 Python 实现详解"
date = "2026-03-19T10:00:00+08:00"

[taxonomies]
tags = ["密码学", "Golang", "Python", "加密", "哈希", "数字签名", "AES", "RSA"]
categories = ["编程"]

[extra]
summary = "全面介绍现代密码学算法，包含哈希函数、对称加密、非对称加密、数字签名等核心概念，提供 Golang 和 Python 两种语言的完整实现代码，涵盖最新后量子密码学和 FIPS 140-3 合规实践。"
author = "博主"
+++

# 密码学算法完全指南 2026：Golang 和 Python 实现详解

密码学是现代信息安全的基石，从保护用户密码到确保网络通信安全，从数字签名到区块链技术，密码学应用无处不在。本文将深入讲解现代密码学的核心算法，并提供 Golang 和 Python 两种语言的完整实现。

---

## 一、密码学基础概念

### 1.1 密码学三大支柱

```
┌─────────────────────────────────────────────────────────┐
│                    现代密码学                             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   哈希函数    │  │   对称加密    │  │   非对称加密  │  │
│  │  (Hash)      │  │  (Symmetric) │  │ (Asymmetric) │  │
│  │              │  │              │  │              │  │
│  │ • SHA-256    │  │ • AES        │  │ • RSA        │  │
│  │ • SHA-3      │  │ • ChaCha20   │  │ • ECC        │  │
│  │ • BLAKE3     │  │ • DES/3DES   │  │ • Ed25519    │  │
│  │ • Argon2     │  │ • SM4        │  │ • ML-KEM     │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                         │
│  用途：完整性验证    用途：数据保密      用途：密钥交换    │
│        密码存储          文件加密          数字签名      │
└─────────────────────────────────────────────────────────┘
```

### 1.2 加密 vs 哈希 vs 签名

| 特性 | 哈希 (Hashing) | 加密 (Encryption) | 签名 (Signing) |
|------|---------------|------------------|---------------|
| **可逆性** | 不可逆（单向） | 可逆（需要密钥） | 可验证（公钥验证） |
| **密钥** | 无需密钥 | 需要密钥 | 需要密钥对 |
| **用途** | 完整性验证、密码存储 | 数据保密 | 身份认证、不可否认 |
| **算法** | SHA-256、BLAKE3 | AES、ChaCha20 | RSA、Ed25519 |
| **输出** | 固定长度摘要 | 密文 | 签名 |

---

## 二、哈希函数（Hash Functions）

### 2.1 哈希函数原理

哈希函数将任意长度的输入转换为固定长度的输出（哈希值），具有以下特性：

1. **确定性**：相同输入总是产生相同输出
2. **快速计算**：给定输入可快速计算哈希值
3. **不可逆性**：无法从哈希值反推原始输入
4. **抗碰撞**：难以找到两个不同输入产生相同哈希值
5. **雪崩效应**：输入微小变化导致输出巨大差异

### 2.2 Golang 实现

#### SHA-256 哈希

```go
package main

import (
    "crypto/sha256"
    "encoding/hex"
    "fmt"
)

func main() {
    data := []byte("Hello, 密码学！")
    
    // 计算 SHA-256 哈希
    hash := sha256.Sum256(data)
    
    // 转换为十六进制字符串
    hashHex := hex.EncodeToString(hash[:])
    
    fmt.Printf("原始数据：%s\n", string(data))
    fmt.Printf("SHA-256: %s\n", hashHex)
    fmt.Printf("哈希长度：%d 字节 (%d 位)\n", len(hash), len(hash)*8)
}
```

#### 多种哈希算法对比

```go
package main

import (
    "crypto/md5"
    "crypto/sha1"
    "crypto/sha256"
    "crypto/sha512"
    "encoding/hex"
    "fmt"
    
    "golang.org/x/crypto/blake2b"
    "golang.org/x/crypto/blake2s"
)

func main() {
    data := []byte("密码学测试数据")
    
    // MD5 (128 位，已不安全，仅用于兼容性)
    md5Hash := md5.Sum(data)
    fmt.Printf("MD5:        %s\n", hex.EncodeToString(md5Hash[:]))
    
    // SHA-1 (160 位，已不推荐用于安全场景)
    sha1Hash := sha1.Sum(data)
    fmt.Printf("SHA-1:      %s\n", hex.EncodeToString(sha1Hash[:]))
    
    // SHA-256 (256 位，推荐)
    sha256Hash := sha256.Sum256(data)
    fmt.Printf("SHA-256:    %s\n", hex.EncodeToString(sha256Hash[:]))
    
    // SHA-512 (512 位，更安全)
    sha512Hash := sha512.Sum512(data)
    fmt.Printf("SHA-512:    %s\n", hex.EncodeToString(sha512Hash[:]))
    
    // BLAKE2b (512 位，性能优于 SHA-2)
    blake2bHash, _ := blake2b.New512(nil)
    blake2bHash.Write(data)
    fmt.Printf("BLAKE2b:    %s\n", hex.EncodeToString(blake2bHash.Sum(nil)))
    
    // BLAKE2s (256 位，适用于 32 位平台)
    blake2sHash, _ := blake2s.New256(nil)
    blake2sHash.Write(data)
    fmt.Printf("BLAKE2s:    %s\n", hex.EncodeToString(blake2sHash.Sum(nil)))
}
```

#### 文件完整性校验

```go
package main

import (
    "crypto/sha256"
    "encoding/hex"
    "fmt"
    "io"
    "os"
)

// 计算文件 SHA-256 哈希
func hashFile(filename string) (string, error) {
    file, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer file.Close()
    
    hasher := sha256.New()
    
    // 分块读取大文件
    buf := make([]byte, 32*1024)
    for {
        n, err := file.Read(buf)
        if n > 0 {
            hasher.Write(buf[:n])
        }
        if err == io.EOF {
            break
        }
        if err != nil {
            return "", err
        }
    }
    
    return hex.EncodeToString(hasher.Sum(nil)), nil
}

// 验证文件完整性
func verifyFile(filename, expectedHash string) (bool, error) {
    actualHash, err := hashFile(filename)
    if err != nil {
        return false, err
    }
    
    return actualHash == expectedHash, nil
}

func main() {
    // 计算文件哈希
    hash, err := hashFile("example.zip")
    if err != nil {
        fmt.Printf("错误：%v\n", err)
        return
    }
    
    fmt.Printf("文件 SHA-256: %s\n", hash)
    
    // 验证文件
    expectedHash := "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    valid, err := verifyFile("example.zip", expectedHash)
    if err != nil {
        fmt.Printf("验证错误：%v\n", err)
        return
    }
    
    if valid {
        fmt.Println("✓ 文件完整性验证通过")
    } else {
        fmt.Println("✗ 文件已被修改或损坏")
    }
}
```

### 2.3 Python 实现

#### 使用 hashlib

```python
import hashlib
from pathlib import Path

def hash_string(text: str, algorithm: str = 'sha256') -> str:
    """计算字符串哈希"""
    hasher = hashlib.new(algorithm)
    hasher.update(text.encode('utf-8'))
    return hasher.hexdigest()

def hash_file(filepath: str, algorithm: str = 'sha256', chunk_size: int = 8192) -> str:
    """计算文件哈希（支持大文件）"""
    hasher = hashlib.new(algorithm)
    
    with open(filepath, 'rb') as f:
        while chunk := f.read(chunk_size):
            hasher.update(chunk)
    
    return hasher.hexdigest()

def verify_file(filepath: str, expected_hash: str, algorithm: str = 'sha256') -> bool:
    """验证文件完整性"""
    actual_hash = hash_file(filepath, algorithm)
    return actual_hash == expected_hash

# 使用示例
if __name__ == "__main__":
    text = "Hello, 密码学！"
    
    # 多种哈希算法
    print(f"原始数据：{text}")
    print(f"MD5:      {hash_string(text, 'md5')}")
    print(f"SHA-1:    {hash_string(text, 'sha1')}")
    print(f"SHA-256:  {hash_string(text, 'sha256')}")
    print(f"SHA-512:  {hash_string(text, 'sha512')}")
    
    # 文件哈希
    file_hash = hash_file('example.zip')
    print(f"\n文件 SHA-256: {file_hash}")
    
    # 验证文件
    expected = "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    if verify_file('example.zip', expected):
        print("✓ 文件完整性验证通过")
    else:
        print("✗ 文件已被修改或损坏")
```

#### 使用 cryptography 库

```python
from cryptography.hazmat.primitives import hashes, hmac
from cryptography.hazmat.backends import default_backend

def hash_with_cryptography(data: bytes) -> str:
    """使用 cryptography 库计算哈希"""
    hasher = hashes.Hash(hashes.SHA256(), backend=default_backend())
    hasher.update(data)
    return hasher.finalize().hex()

def generate_hmac(key: bytes, data: bytes) -> str:
    """生成 HMAC（密钥散列消息认证码）"""
    h = hmac.HMAC(key, hashes.SHA256(), backend=default_backend())
    h.update(data)
    return h.finalize().hex()

def verify_hmac(key: bytes, data: bytes, expected_mac: str) -> bool:
    """验证 HMAC"""
    try:
        h = hmac.HMAC(key, hashes.SHA256(), backend=default_backend())
        h.update(data)
        h.verify(bytes.fromhex(expected_mac))
        return True
    except Exception:
        return False

# 使用示例
if __name__ == "__main__":
    data = b"敏感数据"
    key = b"密钥（应安全存储）"
    
    # 计算哈希
    hash_value = hash_with_cryptography(data)
    print(f"SHA-256: {hash_value}")
    
    # 生成 HMAC
    mac = generate_hmac(key, data)
    print(f"HMAC:    {mac}")
    
    # 验证 HMAC
    if verify_hmac(key, data, mac):
        print("✓ HMAC 验证通过")
    else:
        print("✗ HMAC 验证失败")
```

### 2.4 密码哈希（Password Hashing）

#### Golang - 使用 bcrypt

```go
package main

import (
    "fmt"
    "golang.org/x/crypto/bcrypt"
)

// 哈希密码
func hashPassword(password string) (string, error) {
    // bcrypt 会自动生成盐值
    hashedBytes, err := bcrypt.GenerateFromPassword(
        []byte(password),
        bcrypt.DefaultCost, // 成本因子（10-31，越高越安全但越慢）
    )
    if err != nil {
        return "", err
    }
    
    return string(hashedBytes), nil
}

// 验证密码
func checkPassword(password, hash string) bool {
    err := bcrypt.CompareHashAndPassword(
        []byte(hash),
        []byte(password),
    )
    return err == nil
}

func main() {
    password := "MySecurePassword123!"
    
    // 哈希密码
    hash, err := hashPassword(password)
    if err != nil {
        fmt.Printf("错误：%v\n", err)
        return
    }
    
    fmt.Printf("原始密码：%s\n", password)
    fmt.Printf("哈希后：  %s\n", hash)
    
    // 验证密码
    if checkPassword(password, hash) {
        fmt.Println("✓ 密码验证通过")
    } else {
        fmt.Println("✗ 密码验证失败")
    }
    
    // 错误密码测试
    if checkPassword("WrongPassword", hash) {
        fmt.Println("✗ 错误密码通过了验证（不应该）")
    } else {
        fmt.Println("✓ 错误密码被正确拒绝")
    }
}
```

#### Python - 使用 bcrypt 和 argon2

```python
import bcrypt
import argon2
from argon2 import PasswordHasher, Type

# ========== bcrypt ==========

def hash_password_bcrypt(password: str, salt_rounds: int = 12) -> str:
    """使用 bcrypt 哈希密码"""
    salt = bcrypt.gensalt(rounds=salt_rounds)
    hashed = bcrypt.hashpw(password.encode('utf-8'), salt)
    return hashed.decode('utf-8')

def verify_password_bcrypt(password: str, hash: str) -> bool:
    """验证 bcrypt 密码"""
    return bcrypt.checkpw(
        password.encode('utf-8'),
        hash.encode('utf-8')
    )

# ========== Argon2 (推荐) ==========

def hash_password_argon2(password: str) -> str:
    """使用 Argon2 哈希密码（2015 年密码哈希竞赛获胜者）"""
    ph = PasswordHasher(
        time_cost=3,      # 迭代次数
        memory_cost=65536, # 内存使用量 (KB)
        parallelism=4,     # 并行度
        hash_len=32,       # 哈希长度
        salt_len=16,       # 盐长度
        type=Type.ID       # Argon2id（抗侧信道攻击）
    )
    return ph.hash(password)

def verify_password_argon2(password: str, hash: str) -> bool:
    """验证 Argon2 密码"""
    ph = PasswordHasher()
    try:
        ph.verify(hash, password)
        return True
    except argon2.exceptions.VerifyMismatchError:
        return False

# 使用示例
if __name__ == "__main__":
    password = "MySecurePassword123!"
    
    # bcrypt
    print("=== bcrypt ===")
    bcrypt_hash = hash_password_bcrypt(password)
    print(f"哈希：{bcrypt_hash}")
    print(f"验证：{verify_password_bcrypt(password, bcrypt_hash)}")
    
    # Argon2（推荐用于新系统）
    print("\n=== Argon2 ===")
    argon2_hash = hash_password_argon2(password)
    print(f"哈希：{argon2_hash}")
    print(f"验证：{verify_password_argon2(password, argon2_hash)}")
```

---

## 三、对称加密（Symmetric Encryption）

### 3.1 对称加密原理

对称加密使用**同一个密钥**进行加密和解密：

```
┌─────────────────────────────────────────────────────────┐
│                   对称加密流程                           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  明文 ──────→ [加密算法 + 密钥 K] ──────→ 密文          │
│                                                         │
│  密文 ──────→ [解密算法 + 密钥 K] ──────→ 明文          │
│                                                         │
│  常见算法：AES、ChaCha20、SM4                            │
│  密钥长度：128 位、192 位、256 位                         │
│  工作模式：CBC、GCM、CTR、CFB                           │
└─────────────────────────────────────────────────────────┘
```

### 3.2 AES 加密详解

AES（Advanced Encryption Standard）是最常用的对称加密算法，支持 128、192、256 位密钥。

#### Golang 实现 - AES-GCM（推荐）

```go
package main

import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "encoding/base64"
    "errors"
    "fmt"
    "io"
)

// AESGCM 封装 AES-GCM 加密
type AESGCM struct {
    key []byte
}

// NewAESGCM 创建新的 AES-GCM 实例
func NewAESGCM(key []byte) (*AESGCM, error) {
    if len(key) != 32 { // AES-256 需要 32 字节密钥
        return nil, errors.New("密钥长度必须为 32 字节（256 位）")
    }
    return &AESGCM{key: key}, nil
}

// Encrypt 加密数据
func (a *AESGCM) Encrypt(plaintext []byte) (string, error) {
    // 创建 AES cipher
    block, err := aes.NewCipher(a.key)
    if err != nil {
        return "", err
    }
    
    // 创建 GCM mode
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return "", err
    }
    
    // 生成随机 nonce（12 字节）
    nonce := make([]byte, gcm.NonceSize())
    if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
        return "", err
    }
    
    // 加密数据（包含认证标签）
    ciphertext := gcm.Seal(nonce, nonce, plaintext, nil)
    
    // 返回 Base64 编码的密文
    return base64.StdEncoding.EncodeToString(ciphertext), nil
}

// Decrypt 解密数据
func (a *AESGCM) Decrypt(encryptedBase64 string) ([]byte, error) {
    // 解码 Base64
    ciphertext, err := base64.StdEncoding.DecodeString(encryptedBase64)
    if err != nil {
        return nil, err
    }
    
    // 创建 AES cipher
    block, err := aes.NewCipher(a.key)
    if err != nil {
        return nil, err
    }
    
    // 创建 GCM mode
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    
    // 验证密文长度
    if len(ciphertext) < gcm.NonceSize() {
        return nil, errors.New("密文太短")
    }
    
    // 提取 nonce 和实际密文
    nonce := ciphertext[:gcm.NonceSize()]
    ciphertext = ciphertext[gcm.NonceSize():]
    
    // 解密数据
    plaintext, err := gcm.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        return nil, errors.New("解密失败（密钥错误或数据被篡改）")
    }
    
    return plaintext, nil
}

// GenerateKey 生成随机密钥
func GenerateAESKey() ([]byte, error) {
    key := make([]byte, 32) // 256 位
    _, err := rand.Read(key)
    if err != nil {
        return nil, err
    }
    return key, nil
}

func main() {
    // 生成密钥（实际应用中应安全存储）
    key, err := GenerateAESKey()
    if err != nil {
        fmt.Printf("密钥生成错误：%v\n", err)
        return
    }
    
    fmt.Printf("AES-256 密钥：%x\n\n", key)
    
    // 创建加密器
    aesgcm, err := NewAESGCM(key)
    if err != nil {
        fmt.Printf("错误：%v\n", err)
        return
    }
    
    // 加密
    plaintext := []byte("这是一条机密消息！")
    encrypted, err := aesgcm.Encrypt(plaintext)
    if err != nil {
        fmt.Printf("加密错误：%v\n", err)
        return
    }
    
    fmt.Printf("原文：%s\n", string(plaintext))
    fmt.Printf("密文：%s\n\n", encrypted)
    
    // 解密
    decrypted, err := aesgcm.Decrypt(encrypted)
    if err != nil {
        fmt.Printf("解密错误：%v\n", err)
        return
    }
    
    fmt.Printf("解密结果：%s\n", string(decrypted))
    
    // 验证完整性
    if string(decrypted) == string(plaintext) {
        fmt.Println("✓ 加密/解密成功，数据完整")
    }
    
    // 测试篡改检测
    fmt.Println("\n=== 测试篡改检测 ===")
    tampered := encrypted[:len(encrypted)-1] + "X" // 篡改密文
    _, err = aesgcm.Decrypt(tampered)
    if err != nil {
        fmt.Printf("✓ 正确检测到篡改：%v\n", err)
    }
}
```

#### Python 实现 - AES-GCM

```python
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os
import base64
from typing import Tuple

class AESGCMCipher:
    """AES-GCM 加密类"""
    
    def __init__(self, key: bytes = None):
        """
        初始化 AES-GCM 加密器
        
        Args:
            key: 32 字节密钥（256 位），如不提供则自动生成
        """
        if key is None:
            self.key = AESGCM.generate_key(bit_length=256)
        else:
            if len(key) not in [16, 24, 32]:
                raise ValueError("密钥长度必须为 16、24 或 32 字节")
            self.key = key
        
        self.aesgcm = AESGCM(self.key)
    
    def encrypt(self, plaintext: bytes, associated_data: bytes = None) -> Tuple[bytes, bytes]:
        """
        加密数据
        
        Args:
            plaintext: 明文数据
            associated_data: 关联数据（可选，用于认证但不加密）
        
        Returns:
            (nonce, ciphertext) 元组
        """
        # 生成随机 nonce（12 字节）
        nonce = os.urandom(12)
        
        # 加密（包含认证标签）
        ciphertext = self.aesgcm.encrypt(nonce, plaintext, associated_data)
        
        return nonce, ciphertext
    
    def decrypt(self, nonce: bytes, ciphertext: bytes, associated_data: bytes = None) -> bytes:
        """
        解密数据
        
        Args:
            nonce: 加密时使用的 nonce
            ciphertext: 密文（包含认证标签）
            associated_data: 关联数据（必须与加密时相同）
        
        Returns:
            解密后的明文
        
        Raises:
            cryptography.exceptions.InvalidTag: 密钥错误或数据被篡改
        """
        try:
            plaintext = self.aesgcm.decrypt(nonce, ciphertext, associated_data)
            return plaintext
        except Exception as e:
            raise ValueError(f"解密失败（密钥错误或数据被篡改）: {e}")
    
    def encrypt_string(self, text: str) -> str:
        """加密字符串并返回 Base64 编码"""
        nonce, ciphertext = self.encrypt(text.encode('utf-8'))
        # 组合 nonce 和密文
        combined = nonce + ciphertext
        return base64.b64encode(combined).decode('utf-8')
    
    def decrypt_string(self, encoded: str) -> str:
        """解密 Base64 编码的字符串"""
        combined = base64.b64decode(encoded)
        # 提取 nonce 和密文
        nonce = combined[:12]
        ciphertext = combined[12:]
        plaintext = self.decrypt(nonce, ciphertext)
        return plaintext.decode('utf-8')

# 使用示例
if __name__ == "__main__":
    # 创建加密器
    cipher = AESGCMCipher()
    
    print(f"密钥（256 位）: {cipher.key.hex()}\n")
    
    # 加密字符串
    plaintext = "这是一条机密消息！"
    encrypted = cipher.encrypt_string(plaintext)
    
    print(f"原文：{plaintext}")
    print(f"密文：{encrypted}\n")
    
    # 解密字符串
    decrypted = cipher.decrypt_string(encrypted)
    print(f"解密结果：{decrypted}")
    
    # 验证
    if decrypted == plaintext:
        print("✓ 加密/解密成功\n")
    
    # 测试篡改检测
    print("=== 测试篡改检测 ===")
    try:
        tampered = encrypted[:-1] + "X"  # 篡改密文
        cipher.decrypt_string(tampered)
        print("✗ 未检测到篡改（错误）")
    except ValueError as e:
        print(f"✓ 正确检测到篡改：{e}")
    
    # 带关联数据的加密
    print("\n=== 带关联数据的加密 ===")
    associated_data = b"user_id:12345"
    nonce, ciphertext = cipher.encrypt(
        b"敏感数据",
        associated_data=associated_data
    )
    
    # 正确解密
    plaintext = cipher.decrypt(nonce, ciphertext, associated_data=associated_data)
    print(f"✓ 正确解密：{plaintext.decode()}")
    
    # 错误的关联数据
    try:
        cipher.decrypt(nonce, ciphertext, associated_data=b"user_id:99999")
        print("✗ 未检测到关联数据错误")
    except Exception:
        print("✓ 正确检测到关联数据不匹配")
```

### 3.3 其他对称加密模式

#### Golang - AES-CBC（传统模式）

```go
package main

import (
    "bytes"
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "encoding/base64"
    "errors"
    "io"
)

// PKCS7 填充
func pkcs7Pad(data []byte, blockSize int) []byte {
    padding := blockSize - len(data)%blockSize
    padtext := bytes.Repeat([]byte{byte(padding)}, padding)
    return append(data, padtext...)
}

// PKCS7 去填充
func pkcs7Unpad(data []byte) ([]byte, error) {
    if len(data) == 0 {
        return nil, errors.New("数据为空")
    }
    
    padding := int(data[len(data)-1])
    if padding > len(data) {
        return nil, errors.New("填充错误")
    }
    
    for i := 0; i < padding; i++ {
        if data[len(data)-1-i] != byte(padding) {
            return nil, errors.New("填充错误")
        }
    }
    
    return data[:len(data)-padding], nil
}

// AES-CBC 加密
func encryptAES_CBC(key, plaintext []byte) (string, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return "", err
    }
    
    // PKCS7 填充
    plaintext = pkcs7Pad(plaintext, aes.BlockSize)
    
    // 生成随机 IV
    ciphertext := make([]byte, aes.BlockSize+len(plaintext))
    iv := ciphertext[:aes.BlockSize]
    if _, err := io.ReadFull(rand.Reader, iv); err != nil {
        return "", err
    }
    
    // CBC 模式加密
    mode := cipher.NewCBCEncrypter(block, iv)
    mode.CryptBlocks(ciphertext[aes.BlockSize:], plaintext)
    
    return base64.StdEncoding.EncodeToString(ciphertext), nil
}

// AES-CBC 解密
func decryptAES_CBC(key []byte, encryptedBase64 string) ([]byte, error) {
    ciphertext, err := base64.StdEncoding.DecodeString(encryptedBase64)
    if err != nil {
        return nil, err
    }
    
    if len(ciphertext) < aes.BlockSize {
        return nil, errors.New("密文太短")
    }
    
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    
    iv := ciphertext[:aes.BlockSize]
    ciphertext = ciphertext[aes.BlockSize:]
    
    // CBC 模式解密
    mode := cipher.NewCBCDecrypter(block, iv)
    mode.CryptBlocks(ciphertext, ciphertext)
    
    // 去除填充
    plaintext, err := pkcs7Unpad(ciphertext)
    if err != nil {
        return nil, err
    }
    
    return plaintext, nil
}

func main() {
    key := []byte("12345678901234567890123456789012") // 32 字节
    plaintext := []byte("Hello, AES-CBC!")
    
    encrypted, err := encryptAES_CBC(key, plaintext)
    if err != nil {
        panic(err)
    }
    
    decrypted, err := decryptAES_CBC(key, encrypted)
    if err != nil {
        panic(err)
    }
    
    println("原文:", string(plaintext))
    println("密文:", encrypted)
    println("解密:", string(decrypted))
}
```

#### Python - ChaCha20-Poly1305（移动设备推荐）

```python
from cryptography.hazmat.primitives.ciphers.aead import ChaCha20Poly1305
import os
import base64

class ChaCha20Cipher:
    """ChaCha20-Poly1305 加密类（适用于移动设备）"""
    
    def __init__(self, key: bytes = None):
        """
        初始化 ChaCha20 加密器
        
        Args:
            key: 32 字节密钥，如不提供则自动生成
        """
        if key is None:
            self.key = ChaCha20Poly1305.generate_key()
        else:
            if len(key) != 32:
                raise ValueError("ChaCha20 密钥必须为 32 字节")
            self.key = key
        
        self.cipher = ChaCha20Poly1305(self.key)
    
    def encrypt(self, plaintext: bytes, nonce: bytes = None) -> tuple:
        """
        加密数据
        
        Args:
            plaintext: 明文
            nonce: 12 字节 nonce（可选，自动生成）
        
        Returns:
            (nonce, ciphertext) 元组
        """
        if nonce is None:
            nonce = os.urandom(12)
        
        if len(nonce) != 12:
            raise ValueError("nonce 必须为 12 字节")
        
        ciphertext = self.cipher.encrypt(nonce, plaintext, None)
        return nonce, ciphertext
    
    def decrypt(self, nonce: bytes, ciphertext: bytes) -> bytes:
        """
        解密数据
        
        Args:
            nonce: 加密时使用的 nonce
            ciphertext: 密文（包含认证标签）
        
        Returns:
            解密后的明文
        """
        return self.cipher.decrypt(nonce, ciphertext, None)

# 使用示例
if __name__ == "__main__":
    cipher = ChaCha20Cipher()
    
    plaintext = b"ChaCha20 加密测试"
    nonce, ciphertext = cipher.encrypt(plaintext)
    
    print(f"原文：{plaintext.decode()}")
    print(f"密文：{base64.b64encode(ciphertext).decode()}")
    
    decrypted = cipher.decrypt(nonce, ciphertext)
    print(f"解密：{decrypted.decode()}")
```

---

## 四、非对称加密（Asymmetric Encryption）

### 4.1 非对称加密原理

非对称加密使用**密钥对**（公钥和私钥）：

```
┌─────────────────────────────────────────────────────────┐
│                  非对称加密流程                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  加密场景：                                              │
│  明文 ──→ [公钥加密] ──→ 密文 ──→ [私钥解密] ──→ 明文   │
│                                                         │
│  签名场景：                                              │
│  数据 ──→ [私钥签名] ──→ 签名 ──→ [公钥验证] ──→ 验证   │
│                                                         │
│  常见算法：RSA、ECC、Ed25519、ML-KEM（后量子）          │
└─────────────────────────────────────────────────────────┘
```

### 4.2 RSA 加密

#### Golang 实现

```go
package main

import (
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
    "crypto/x509"
    "encoding/base64"
    "encoding/pem"
    "fmt"
)

// RSAKeyPair RSA 密钥对
type RSAKeyPair struct {
    PrivateKey *rsa.PrivateKey
    PublicKey  *rsa.PublicKey
}

// GenerateRSAKeyPair 生成 RSA 密钥对
func GenerateRSAKeyPair(bits int) (*RSAKeyPair, error) {
    // 生成私钥
    privateKey, err := rsa.GenerateKey(rand.Reader, bits)
    if err != nil {
        return nil, err
    }
    
    return &RSAKeyPair{
        PrivateKey: privateKey,
        PublicKey:  &privateKey.PublicKey,
    }, nil
}

// PrivateKeyToPEM 私钥转 PEM 格式
func PrivateKeyToPEM(privateKey *rsa.PrivateKey) string {
    pemBytes := pem.EncodeToMemory(&pem.Block{
        Type:  "RSA PRIVATE KEY",
        Bytes: x509.MarshalPKCS1PrivateKey(privateKey),
    })
    return string(pemBytes)
}

// PublicKeyToPEM 公钥转 PEM 格式
func PublicKeyToPEM(publicKey *rsa.PublicKey) string {
    pubBytes, err := x509.MarshalPKIXPublicKey(publicKey)
    if err != nil {
        panic(err)
    }
    
    pemBytes := pem.EncodeToMemory(&pem.Block{
        Type:  "PUBLIC KEY",
        Bytes: pubBytes,
    })
    return string(pemBytes)
}

// EncryptWithPublicKey 使用公钥加密（OAEP 填充）
func EncryptWithPublicKey(publicKey *rsa.PublicKey, plaintext []byte) (string, error) {
    // 使用 SHA-256 和 OAEP 填充
    ciphertext, err := rsa.EncryptOAEP(
        sha256.New(),
        rand.Reader,
        publicKey,
        plaintext,
        nil, // label
    )
    if err != nil {
        return "", err
    }
    
    return base64.StdEncoding.EncodeToString(ciphertext), nil
}

// DecryptWithPrivateKey 使用私钥解密
func DecryptWithPrivateKey(privateKey *rsa.PrivateKey, encryptedBase64 string) ([]byte, error) {
    ciphertext, err := base64.StdEncoding.DecodeString(encryptedBase64)
    if err != nil {
        return nil, err
    }
    
    plaintext, err := rsa.DecryptOAEP(
        sha256.New(),
        rand.Reader,
        privateKey,
        ciphertext,
        nil,
    )
    if err != nil {
        return nil, err
    }
    
    return plaintext, nil
}

// SignWithPrivateKey 使用私钥签名
func SignWithPrivateKey(privateKey *rsa.PrivateKey, message []byte) (string, error) {
    // 计算消息哈希
    hashed := sha256.Sum256(message)
    
    // 签名
    signature, err := rsa.SignPKCS1v15(
        rand.Reader,
        privateKey,
        crypto.SHA256,
        hashed[:],
    )
    if err != nil {
        return "", err
    }
    
    return base64.StdEncoding.EncodeToString(signature), nil
}

// VerifyWithPublicKey 使用公钥验证签名
func VerifyWithPublicKey(publicKey *rsa.PublicKey, message []byte, signatureBase64 string) error {
    signature, err := base64.StdEncoding.DecodeString(signatureBase64)
    if err != nil {
        return err
    }
    
    hashed := sha256.Sum256(message)
    
    return rsa.VerifyPKCS1v15(
        publicKey,
        crypto.SHA256,
        hashed[:],
        signature,
    )
}

func main() {
    // 生成 RSA-2048 密钥对
    keyPair, err := GenerateRSAKeyPair(2048)
    if err != nil {
        fmt.Printf("密钥生成错误：%v\n", err)
        return
    }
    
    fmt.Println("=== RSA 密钥对 ===")
    fmt.Printf("私钥（PEM）:\n%s\n", PrivateKeyToPEM(keyPair.PrivateKey))
    fmt.Printf("公钥（PEM）:\n%s\n", PublicKeyToPEM(keyPair.PublicKey))
    
    // 加密/解密测试
    fmt.Println("\n=== 加密/解密测试 ===")
    plaintext := []byte("这是一条机密消息")
    
    encrypted, err := EncryptWithPublicKey(keyPair.PublicKey, plaintext)
    if err != nil {
        fmt.Printf("加密错误：%v\n", err)
        return
    }
    
    fmt.Printf("原文：%s\n", string(plaintext))
    fmt.Printf("密文：%s\n", encrypted)
    
    decrypted, err := DecryptWithPrivateKey(keyPair.PrivateKey, encrypted)
    if err != nil {
        fmt.Printf("解密错误：%v\n", err)
        return
    }
    
    fmt.Printf("解密：%s\n", string(decrypted))
    
    // 签名/验证测试
    fmt.Println("\n=== 签名/验证测试 ===")
    message := []byte("需要签名的消息")
    
    signature, err := SignWithPrivateKey(keyPair.PrivateKey, message)
    if err != nil {
        fmt.Printf("签名错误：%v\n", err)
        return
    }
    
    fmt.Printf("消息：%s\n", string(message))
    fmt.Printf("签名：%s\n", signature)
    
    err = VerifyWithPublicKey(keyPair.PublicKey, message, signature)
    if err != nil {
        fmt.Printf("验证失败：%v\n", err)
    } else {
        fmt.Println("✓ 签名验证通过")
    }
    
    // 篡改测试
    fmt.Println("\n=== 篡改测试 ===")
    tamperedMessage := []byte("被篡改的消息")
    err = VerifyWithPublicKey(keyPair.PublicKey, tamperedMessage, signature)
    if err != nil {
        fmt.Printf("✓ 正确检测到篡改：%v\n", err)
    }
}
```

#### Python 实现

```python
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.backends import default_backend
import base64

class RSAKeyPair:
    """RSA 密钥对"""
    
    def __init__(self, key_size: int = 2048):
        """
        生成 RSA 密钥对
        
        Args:
            key_size: 密钥长度（2048、3072、4096）
        """
        if key_size < 2048:
            raise ValueError("RSA 密钥长度至少为 2048 位")
        
        self.private_key = rsa.generate_private_key(
            public_exponent=65537,
            key_size=key_size,
            backend=default_backend()
        )
        self.public_key = self.private_key.public_key()
    
    def private_key_to_pem(self) -> str:
        """私钥转 PEM 格式"""
        pem = self.private_key.private_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PrivateFormat.PKCS8,
            encryption_algorithm=serialization.NoEncryption()
        )
        return pem.decode('utf-8')
    
    def public_key_to_pem(self) -> str:
        """公钥转 PEM 格式"""
        pem = self.public_key.public_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PublicFormat.SubjectPublicKeyInfo
        )
        return pem.decode('utf-8')
    
    @staticmethod
    def load_private_key_from_pem(pem_data: str):
        """从 PEM 加载私钥"""
        private_key = serialization.load_pem_private_key(
            pem_data.encode('utf-8'),
            password=None,
            backend=default_backend()
        )
        return private_key
    
    @staticmethod
    def load_public_key_from_pem(pem_data: str):
        """从 PEM 加载公钥"""
        public_key = serialization.load_pem_public_key(
            pem_data.encode('utf-8'),
            backend=default_backend()
        )
        return public_key
    
    def encrypt(self, plaintext: bytes) -> str:
        """使用公钥加密（OAEP 填充）"""
        ciphertext = self.public_key.encrypt(
            plaintext,
            padding.OAEP(
                mgf=padding.MGF1(algorithm=hashes.SHA256()),
                algorithm=hashes.SHA256(),
                label=None
            )
        )
        return base64.b64encode(ciphertext).decode('utf-8')
    
    def decrypt(self, encrypted_base64: str) -> bytes:
        """使用私钥解密"""
        ciphertext = base64.b64decode(encrypted_base64)
        plaintext = self.private_key.decrypt(
            ciphertext,
            padding.OAEP(
                mgf=padding.MGF1(algorithm=hashes.SHA256()),
                algorithm=hashes.SHA256(),
                label=None
            )
        )
        return plaintext
    
    def sign(self, message: bytes) -> str:
        """使用私钥签名（PSS 填充）"""
        signature = self.private_key.sign(
            message,
            padding.PSS(
                mgf=padding.MGF1(hashes.SHA256()),
                salt_length=padding.PSS.MAX_LENGTH
            ),
            hashes.SHA256()
        )
        return base64.b64encode(signature).decode('utf-8')
    
    def verify(self, message: bytes, signature_base64: str) -> bool:
        """使用公钥验证签名"""
        try:
            signature = base64.b64decode(signature_base64)
            self.public_key.verify(
                signature,
                message,
                padding.PSS(
                    mgf=padding.MGF1(hashes.SHA256()),
                    salt_length=padding.PSS.MAX_LENGTH
                ),
                hashes.SHA256()
            )
            return True
        except Exception:
            return False

# 使用示例
if __name__ == "__main__":
    # 生成密钥对
    key_pair = RSAKeyPair(key_size=2048)
    
    print("=== RSA 密钥对 ===")
    print(f"私钥（PEM）:\n{key_pair.private_key_to_pem()}")
    print(f"公钥（PEM）:\n{key_pair.public_key_to_pem()}")
    
    # 加密/解密测试
    print("\n=== 加密/解密测试 ===")
    plaintext = b"这是一条机密消息"
    
    encrypted = key_pair.encrypt(plaintext)
    print(f"原文：{plaintext.decode()}")
    print(f"密文：{encrypted}")
    
    decrypted = key_pair.decrypt(encrypted)
    print(f"解密：{decrypted.decode()}")
    
    # 签名/验证测试
    print("\n=== 签名/验证测试 ===")
    message = b"需要签名的消息"
    
    signature = key_pair.sign(message)
    print(f"消息：{message.decode()}")
    print(f"签名：{signature}")
    
    if key_pair.verify(message, signature):
        print("✓ 签名验证通过")
    else:
        print("✗ 签名验证失败")
    
    # 篡改测试
    print("\n=== 篡改测试 ===")
    tampered_message = b"被篡改的消息"
    if key_pair.verify(tampered_message, signature):
        print("✗ 未检测到篡改（错误）")
    else:
        print("✓ 正确检测到篡改")
    
    # 密钥持久化示例
    print("\n=== 密钥持久化 ===")
    # 保存私钥（应加密存储）
    with open('private_key.pem', 'w') as f:
        f.write(key_pair.private_key_to_pem())
    
    # 保存公钥
    with open('public_key.pem', 'w') as f:
        f.write(key_pair.public_key_to_pem())
    
    # 加载密钥
    with open('private_key.pem', 'r') as f:
        loaded_private_key = RSAKeyPair.load_private_key_from_pem(f.read())
    
    with open('public_key.pem', 'r') as f:
        loaded_public_key = RSAKeyPair.load_public_key_from_pem(f.read())
    
    print("✓ 密钥保存和加载成功")
```

### 4.3 ECC（椭圆曲线加密）

#### Golang - ECDSA 签名

```go
package main

import (
    "crypto/ecdsa"
    "crypto/elliptic"
    "crypto/rand"
    "crypto/sha256"
    "encoding/hex"
    "fmt"
)

func main() {
    // 生成 P-256 椭圆曲线密钥对
    privateKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
    if err != nil {
        panic(err)
    }
    
    publicKey := &privateKey.PublicKey
    
    fmt.Println("=== ECDSA 密钥对 ===")
    fmt.Printf("私钥 D: %x\n", privateKey.D)
    fmt.Printf("公钥 X: %x\n", publicKey.X)
    fmt.Printf("公钥 Y: %x\n\n", publicKey.Y)
    
    // 签名
    message := []byte("需要签名的消息")
    hashed := sha256.Sum256(message)
    
    r, s, err := ecdsa.Sign(rand.Reader, privateKey, hashed[:])
    if err != nil {
        panic(err)
    }
    
    fmt.Println("=== ECDSA 签名 ===")
    fmt.Printf("消息：%s\n", string(message))
    fmt.Printf("R: %x\n", r)
    fmt.Printf("S: %x\n\n", s)
    
    // 验证
    valid := ecdsa.Verify(publicKey, hashed[:], r, s)
    
    fmt.Println("=== 验证结果 ===")
    if valid {
        fmt.Println("✓ 签名验证通过")
    } else {
        fmt.Println("✗ 签名验证失败")
    }
}
```

#### Python - Ed25519（推荐用于签名）

```python
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey
from cryptography.hazmat.primitives import serialization
import base64

class Ed25519KeyPair:
    """Ed25519 密钥对（比 ECDSA 更快更安全）"""
    
    def __init__(self):
        """生成 Ed25519 密钥对"""
        self.private_key = Ed25519PrivateKey.generate()
        self.public_key = self.private_key.public_key()
    
    def sign(self, message: bytes) -> str:
        """签名"""
        signature = self.private_key.sign(message)
        return base64.b64encode(signature).decode('utf-8')
    
    def verify(self, message: bytes, signature_base64: str) -> bool:
        """验证签名"""
        try:
            signature = base64.b64decode(signature_base64)
            self.public_key.verify(signature, message)
            return True
        except Exception:
            return False
    
    def private_key_to_pem(self) -> str:
        """私钥转 PEM"""
        pem = self.private_key.private_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PrivateFormat.PKCS8,
            encryption_algorithm=serialization.NoEncryption()
        )
        return pem.decode('utf-8')
    
    def public_key_to_pem(self) -> str:
        """公钥转 PEM"""
        pem = self.public_key.public_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PublicFormat.SubjectPublicKeyInfo
        )
        return pem.decode('utf-8')

# 使用示例
if __name__ == "__main__":
    key_pair = Ed25519KeyPair()
    
    message = b"Ed25519 签名测试"
    signature = key_pair.sign(message)
    
    print(f"消息：{message.decode()}")
    print(f"签名：{signature}")
    
    if key_pair.verify(message, signature):
        print("✓ 签名验证通过")
    else:
        print("✗ 签名验证失败")
```

---

## 五、混合加密系统

实际应用中，通常结合对称加密和非对称加密：

```
┌─────────────────────────────────────────────────────────┐
│                  混合加密系统                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. 生成随机对称密钥（AES-256）                         │
│  2. 使用对称密钥加密数据（快速）                        │
│  3. 使用接收方公钥加密对称密钥（安全传输）              │
│  4. 发送：加密的数据 + 加密的对称密钥                   │
│                                                         │
│  接收方：                                                │
│  1. 使用私钥解密对称密钥                                │
│  2. 使用对称密钥解密数据                                │
└─────────────────────────────────────────────────────────┘
```

### Golang 实现

```go
package main

import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
    "encoding/base64"
    "fmt"
    "io"
)

// HybridEncrypt 混合加密
func HybridEncrypt(publicKey *rsa.PublicKey, plaintext []byte) (string, error) {
    // 1. 生成随机 AES 密钥
    aesKey := make([]byte, 32) // 256 位
    if _, err := io.ReadFull(rand.Reader, aesKey); err != nil {
        return "", err
    }
    
    // 2. 使用 AES-GCM 加密数据
    block, err := aes.NewCipher(aesKey)
    if err != nil {
        return "", err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return "", err
    }
    
    nonce := make([]byte, gcm.NonceSize())
    if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
        return "", err
    }
    
    ciphertext := gcm.Seal(nonce, nonce, plaintext, nil)
    
    // 3. 使用 RSA 加密 AES 密钥
    encryptedKey, err := rsa.EncryptOAEP(
        sha256.New(),
        rand.Reader,
        publicKey,
        aesKey,
        nil,
    )
    if err != nil {
        return "", err
    }
    
    // 4. 组合：加密的密钥 + nonce + 密文
    combined := append(encryptedKey, ciphertext...)
    
    return base64.StdEncoding.EncodeToString(combined), nil
}

// HybridDecrypt 混合解密
func HybridDecrypt(privateKey *rsa.PrivateKey, encryptedBase64 string) ([]byte, error) {
    combined, err := base64.StdEncoding.DecodeString(encryptedBase64)
    if err != nil {
        return nil, err
    }
    
    // 1. 提取加密的 AES 密钥（RSA-2048 为 256 字节）
    keySize := (privateKey.N.BitLen() + 7) / 8
    encryptedKey := combined[:keySize]
    ciphertext := combined[keySize:]
    
    // 2. 使用 RSA 解密 AES 密钥
    aesKey, err := rsa.DecryptOAEP(
        sha256.New(),
        rand.Reader,
        privateKey,
        encryptedKey,
        nil,
    )
    if err != nil {
        return nil, err
    }
    
    // 3. 使用 AES-GCM 解密数据
    block, err := aes.NewCipher(aesKey)
    if err != nil {
        return nil, err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    
    nonceSize := gcm.NonceSize()
    nonce := ciphertext[:nonceSize]
    ciphertext = ciphertext[nonceSize:]
    
    plaintext, err := gcm.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        return nil, err
    }
    
    return plaintext, nil
}

func main() {
    // 生成 RSA 密钥对
    rsaKey, _ := rsa.GenerateKey(rand.Reader, 2048)
    
    // 加密
    plaintext := []byte("这是一条很长的机密消息，使用混合加密系统...")
    encrypted, _ := HybridEncrypt(&rsaKey.PublicKey, plaintext)
    
    fmt.Printf("原文：%s\n", string(plaintext))
    fmt.Printf("密文：%s\n\n", encrypted)
    
    // 解密
    decrypted, _ := HybridDecrypt(rsaKey, encrypted)
    fmt.Printf("解密：%s\n", string(decrypted))
}
```

---

## 六、后量子密码学（Post-Quantum Cryptography）

### 6.1 为什么需要后量子密码学？

量子计算机对现有密码学的威胁：

| 算法 | 经典计算机安全 | 量子计算机威胁 | 后量子替代 |
|------|--------------|--------------|-----------|
| **RSA-2048** | 安全 | Shor 算法可破解 | ML-KEM、ML-DSA |
| **ECC P-256** | 安全 | Shor 算法可破解 | ML-KEM、SLH-DSA |
| **AES-256** | 安全 | Grover 算法减半安全 | 增加密钥长度 |
| **SHA-256** | 安全 | Grover 算法减半安全 | 使用 SHA-512 |

### 6.2 Golang - ML-KEM（NIST 标准化）

Go 1.24+ 已内置 ML-KEM（原 Kyber）：

```go
package main

import (
    "crypto/mlkem"
    "fmt"
)

func main() {
    // 生成密钥对（ML-KEM-768，NIST 安全级别 3）
    publicKey, privateKey, err := mlkem.GenerateKey(nil)
    if err != nil {
        panic(err)
    }
    
    fmt.Println("=== ML-KEM 密钥对 ===")
    fmt.Printf("公钥长度：%d 字节\n", len(publicKey.Bytes()))
    fmt.Printf("私钥长度：%d 字节\n\n", len(privateKey.Bytes()))
    
    // 封装（加密）
    ciphertext, sharedSecret, err := mlkem.Encapsulate(nil, publicKey)
    if err != nil {
        panic(err)
    }
    
    fmt.Println("=== 密钥封装 ===")
    fmt.Printf("密文长度：%d 字节\n", len(ciphertext))
    fmt.Printf("共享密钥：%x\n\n", sharedSecret)
    
    // 解封装（解密）
    sharedSecret2, err := mlkem.Decapsulate(privateKey, ciphertext)
    if err != nil {
        panic(err)
    }
    
    fmt.Println("=== 验证 ===")
    if string(sharedSecret) == string(sharedSecret2) {
        fmt.Println("✓ 共享密钥匹配")
    } else {
        fmt.Println("✗ 共享密钥不匹配")
    }
}
```

---

## 七、最佳实践总结

### 7.1 算法选择指南

| 用途 | 推荐算法 | 密钥长度 | 备注 |
|------|---------|---------|------|
| **密码哈希** | Argon2id | - | 时间=3，内存=64MB，并行=4 |
| **密码哈希（兼容）** | bcrypt | - | cost=12 |
| **数据加密** | AES-GCM | 256 位 | 优先选择 |
| **数据加密（移动）** | ChaCha20-Poly1305 | 256 位 | 移动设备优化 |
| **密钥交换** | ML-KEM-768 | - | 后量子安全 |
| **密钥交换（传统）** | X25519 | 256 位 | 经典安全 |
| **数字签名** | Ed25519 | 256 位 | 快速安全 |
| **数字签名（兼容）** | RSA-PSS | 3072+ 位 | 传统系统 |
| **哈希** | SHA-256 / SHA-3 | 256 位 | 通用 |
| **哈希（高性能）** | BLAKE3 | 256 位 | 速度最快 |

### 7.2 安全注意事项

1. **密钥管理**
   - 永远不要硬编码密钥
   - 使用密钥管理系统（KMS）
   - 定期轮换密钥
   - 最小权限原则

2. **随机数生成**
   - 始终使用加密安全的随机数生成器
   - Golang: `crypto/rand`
   - Python: `os.urandom()` 或 `secrets`

3. **nonce 使用**
   - 每次加密使用唯一 nonce
   - 永远不要重复使用 nonce
   - nonce 不需要保密，可与密文一起存储

4. **认证加密**
   - 优先选择 AEAD 模式（AES-GCM、ChaCha20-Poly1305）
   - 不要单独使用加密，要同时认证

5. **后量子迁移**
   - 新系统应支持混合模式（经典 + 后量子）
   - 关注 NIST 标准化进展
   - 制定迁移计划

### 7.3 常见错误

```go
// ❌ 错误：使用不安全的随机数
key := make([]byte, 32)
rand.Read(key) // math/rand 不是加密安全的

// ✅ 正确：使用加密安全的随机数
cryptoRand.Read(key) // crypto/rand

// ❌ 错误：重复使用 nonce
nonce := make([]byte, 12)
ciphertext1 := gcm.Seal(nonce, nonce, plaintext1, nil)
ciphertext2 := gcm.Seal(nonce, nonce, plaintext2, nil) // 危险！

// ✅ 正确：每次生成新 nonce
nonce1 := make([]byte, 12)
io.ReadFull(rand.Reader, nonce1)
nonce2 := make([]byte, 12)
io.ReadFull(rand.Reader, nonce2)

// ❌ 错误：使用 ECB 模式
block, _ := aes.NewCipher(key)
// ECB 模式会泄露模式信息

// ✅ 正确：使用 GCM 或 CBC 模式
gcm, _ := cipher.NewGCM(block)
```

---

## 八、总结

密码学是构建安全系统的基石。本文介绍了：

1. **哈希函数**：SHA-256、BLAKE3、Argon2
2. **对称加密**：AES-GCM、ChaCha20-Poly1305
3. **非对称加密**：RSA、ECC、Ed25519
4. **混合加密**：结合对称和非对称的优势
5. **后量子密码学**：ML-KEM 等抗量子算法

**关键要点：**
- 使用经过验证的库，不要自己实现密码学算法
- 优先选择 AEAD 模式（认证加密）
- 妥善管理密钥
- 为后量子时代做好准备

**参考资源：**
- NIST: https://csrc.nist.gov/
- Go crypto: https://pkg.go.dev/crypto
- Python cryptography: https://cryptography.io/
- IETF: https://www.ietf.org/

掌握这些密码学知识，你将能够构建安全可靠的系统。记住：**密码学很困难，错误可能导致严重后果**。如有疑问，请咨询安全专家！🔐
