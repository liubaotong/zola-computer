+++
title = "Tokio 运行时完全指南：Rust 异步生态核心"
date = "2026-03-19T05:22:00+08:00"

[taxonomies]
tags = ["rust", "tokio", "async", "runtime", "并发", "异步"]

[extra]
summary = "Tokio 是 Rust 生态中最核心的异步运行时，本文详细介绍其架构、任务调度、通道、计时器和实际应用。"
author = "博主"
+++

Tokio 是 Rust 生态系统中最重要的异步运行时，它为 Rust 提供了高效、可靠的异步编程能力。几乎所有现代 Rust Web 框架（Axum、Actix-web、Rocket）都建立在 Tokio 之上。理解 Tokio 是掌握现代 Rust 异步编程的关键。

## Tokio 概述

### 什么是 Tokio？

Tokio 是一个用于构建可靠、无锁、高性能异步应用程序的框架。它提供了：

- **异步运行时**：多线程和单线程运行时
- **异步 I/O**：TCP/UDP/Unix Socket
- **任务调度**：futures 和 async/await
- **同步原语**：Mutex、RwLock、Semaphore 等
- **时间管理**：定时器和延迟

### Tokio 的优势

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

```rust
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    println!("Tokio: 快速、可靠、内存安全");
    sleep(Duration::from_millis(100)).await;
    println!("完成!");
}
```

**为什么选择 Tokio：**

- **速度快**：基于 Rust，性能卓越
- **可靠**：经过生产环境验证
- **零成本抽象**：async/await 不会增加额外开销
- **活跃生态**：庞大的社区和库支持

## 核心概念：Future 和 async/await

### Future 基础

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};

// Future 是一个可以在未来某个时刻产生值的计算
struct MyFuture {
    ready: bool,
}

impl Future for MyFuture {
    type Output = String;

    fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        if self.ready {
            Poll::Ready("Future completed!".to_string())
        } else {
            self.ready = true;
            // 模拟异步操作
            cx.waker().wake_by_ref();
            Poll::Pending
        }
    }
}
```

### async/await 语法

```rust
#[tokio::main]
async fn main() {
    // async fn 返回一个 Future
    let result = fetch_data().await;
    println!("Received: {}", result);
}

async fn fetch_data() -> String {
    // 模拟异步操作
    tokio::time::sleep(tokio::time::Duration::from_secs(1)).await;
    "Data fetched!".to_string()
}
```

### Future 执行流程

```rust
#[tokio::main]
async fn main() {
    println!("1. 开始执行");
    
    // 创建 Future（此时不执行）
    let future = async {
        println!("3. Future 内部开始");
        tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
        println!("5. Future 完成");
        42
    };

    println!("2. 等待 Future");
    let result = future.await;
    println!("4. 结果: {}", result);
}
```

## 运行时配置

### 单线程运行时

```rust
use tokio::runtime::Runtime;

fn main() {
    // 创建单线程运行时
    let rt = Runtime::new().unwrap();
    
    rt.block_on(async {
        println!("单线程运行时中执行");
        tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
    });
}
```

### 多线程运行时

```rust
use tokio::runtime::Runtime;

fn main() {
    // 创建多线程运行时（默认）
    let rt = tokio::runtime::Runtime::new().unwrap();
    
    rt.block_on(async {
        // 可以使用 tokio::spawn 在多线程中调度任务
        let handle = tokio::spawn(async {
            "在 worker 线程中执行"
        });
        
        let result = handle.await.unwrap();
        println!("{}", result);
    });
}
```

### 自定义运行时

```rust
use tokio::runtime::{Builder, Runtime};

fn create_runtime() -> Runtime {
    Builder::new_multi_thread()
        .worker_threads(8)                    // 8 个工作线程
        .max_blocking_threads(256)             // 最多 256 个阻塞线程
        .thread_name("my-worker")              // 线程名称前缀
        .thread_stack_size(3 * 1024 * 1024)   // 3MB 栈大小
        .enable_all()                          // 启用所有功能
        .build()
        .unwrap()
}
```

### 运行时特征

```rust
use tokio::runtime::Builder;

fn main() {
    // 单线程运行时 - 适合需要严格顺序执行的场景
    let single = Builder::new_current_thread()
        .build()
        .unwrap();

    // 多线程运行时 - 适合大多数场景
    let multi = Builder::new_multi_thread()
        .worker_threads(4)
        .build()
        .unwrap();

    // 自动选择（单线程如果没有启用 rt-multi-thread 特性）
    let auto = Builder::new_multi_thread()
        .enable_all()
        .build()
        .unwrap();
}
```

## 任务 (Tasks)

### tokio::spawn

```rust
#[tokio::main]
async fn main() {
    // Spawn 一个独立的任务
    let handle = tokio::spawn(async {
        tokio::time::sleep(tokio::time::Duration::from_secs(1)).await;
        "任务完成"
    });

    // 在主任务中继续执行
    println!("主任务继续执行...");

    // 等待并获取结果
    match handle.await {
        Ok(result) => println!("结果: {}", result),
        Err(e) => println!("任务 panic: {}", e),
    }
}
```

### spawn多个任务

```rust
#[tokio::main]
async fn main() {
    let handles: Vec<_> = (0..10)
        .map(|i| {
            tokio::spawn(async move {
                tokio::time::sleep(tokio::time::Duration::from_millis(i * 100)).await;
                i
            })
        })
        .collect();

    // 等待所有任务完成
    let results: Vec<_> = futures::future::join_all(handles)
        .await
        .into_iter()
        .filter_map(|r| r.ok())
        .collect();

    println!("结果: {:?}", results);
}
```

### join! 和 try_join!

```rust
#[tokio::main]
async fn main() {
    // 并行执行多个 Future
    let (result1, result2, result3) = tokio::join!(
        async { 1 },
        async { 2 },
        async { 3 }
    );

    println!("{} {} {}", result1, result2, result3);
}
```

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // try_join! 会在任何一个 Future 返回 Err 时立即返回
    let (result1, result2) = tokio::try_join!(
        async { Ok::<_, String>(1) },
        async { Ok::<_, String>(2) },
    )?;

    println!("{} {}", result1, result2);
    Ok(())
}
```

### select!

```rust
#[tokio::main]
async fn main() {
    let start = tokio::time::Instant::now();
    
    tokio::select! {
        _ = tokio::time::sleep(tokio::time::Duration::from_secs(5)) => {
            println!("5秒超时");
        }
        _ = tokio::time::sleep(tokio::time::Duration::from_secs(1)) => {
            println!("1秒触发!");
        }
        _ = tokio::time::sleep(tokio::time::Duration::from_secs(3)) => {
            println!("3秒触发");
        }
    }

    println!("select! 完成，耗时: {:?}", start.elapsed());
}
```

### abort 任务

```rust
#[tokio::main]
async fn main() {
    let handle = tokio::spawn(async {
        loop {
            println!("任务运行中...");
            tokio::time::sleep(tokio::time::Duration::from_secs(1)).await;
        }
    });

    // 让任务运行一会儿
    tokio::time::sleep(tokio::time::Duration::from_secs(3)).await;

    // 取消任务
    handle.abort();
    
    // 检查是否已取消
    match handle.await {
        Ok(_) => println!("任务正常完成"),
        Err(e) if e.is_cancelled() => println!("任务被取消"),
        Err(e) => println!("任务 panic: {}", e),
    }
}
```

## 通道 (Channels)

### mpsc 通道 (多生产者单消费者)

```rust
#[tokio::main]
async fn main() {
    // 创建容量为 32 的通道
    let (tx, mut rx) = tokio::sync::mpsc::channel::<String>(32);

    // Spawn 生产者任务
    let producer = tokio::spawn(async move {
        for i in 0..5 {
            let msg = format!("Message {}", i);
            if tx.send(msg).await.is_err() {
                println!("Receiver 已关闭");
                return;
            }
            tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
        }
    });

    // 在主任务中接收消息
    while let Some(msg) = rx.recv().await {
        println!("Received: {}", msg);
    }

    producer.await.unwrap();
}
```

### 多个生产者

```rust
#[tokio::main]
async fn main() {
    let (tx, mut rx) = tokio::sync::mpsc::channel::<i32>(100);

    // Spawn 多个生产者
    for i in 0..3 {
        let tx = tx.clone();
        tokio::spawn(async move {
            for j in 0..5 {
                tx.send(i * 10 + j).await.unwrap();
            }
        });
    }

    // 主任务接收所有消息
    drop(tx);  // 显式丢弃主发送者
    let mut total = Vec::new();
    while let Some(msg) = rx.recv().await {
        total.push(msg);
    }

    total.sort();
    println!("收到 {} 条消息: {:?}", total.len(), total);
}
```

### oneshot 通道 (单次通信)

```rust
#[tokio::main]
async fn main() {
    // 创建 oneshot 通道
    let (tx, rx) = tokio::sync::oneshot::channel::<i32>();

    // Spawn 发送者任务
    tokio::spawn(async move {
        tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
        tx.send(42).unwrap();
    });

    // 等待接收
    match rx.await {
        Ok(value) => println!("收到值: {}", value),
        Err(_) => println!("发送者已关闭"),
    }
}
```

### broadcast 通道 (广播)

```rust
#[tokio::main]
async fn main() {
    let (tx, _rx) = tokio::sync::broadcast::channel::<String>(16);

    // 创建多个订阅者
    let mut rx1 = tx.subscribe();
    let mut rx2 = tx.subscribe();

    // Spawn 接收者
    tokio::spawn(async move {
        while let Ok(msg) = rx1.recv().await {
            println!("Receiver 1: {}", msg);
        }
    });

    tokio::spawn(async move {
        while let Ok(msg) = rx2.recv().await {
            println!("Receiver 2: {}", msg);
        }
    });

    // 广播消息
    tx.send("Hello".to_string()).unwrap();
    tx.send("World".to_string()).unwrap();

    tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
}
```

### watch 通道 (观察者模式)

```rust
#[tokio::main]
async fn main() {
    let (tx, rx) = tokio::sync::watch::channel::<String>("initial".to_string());

    // Spawn 观察者
    tokio::spawn(async move {
        let mut prev = rx;
        while let Ok(msg) = prev.changed().await {
            println!("收到更新: {}", *msg);
        }
    });

    // 发送更新
    tx.send("update 1".to_string()).unwrap();
    tx.send("update 2".to_string()).unwrap();

    tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
}
```

## 同步原语

### Mutex

```rust
use tokio::sync::Mutex;
use std::sync::Arc;

#[tokio::main]
async fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        handles.push(tokio::spawn(async move {
            let mut num = counter.lock().await;
            *num += 1;
        }));
    }

    for handle in handles {
        handle.await.unwrap();
    }

    println!("Result: {}", *counter.lock().await);
}
```

### RwLock

```rust
use tokio::sync::RwLock;
use std::sync::Arc;

#[tokio::main]
async fn main() {
    let data = Arc::new(RwLock::new(vec![1, 2, 3]));

    // 多个读取锁
    let read_handles: Vec<_> = (0..3)
        .map(|_| {
            let data = Arc::clone(&data);
            tokio::spawn(async move {
                let read = data.read().await;
                format!("Read: {:?}", &*read)
            })
        })
        .collect();

    // 一个写入锁
    let write_handle = {
        let data = Arc::clone(&data);
        tokio::spawn(async move {
            let mut write = data.write().await;
            write.push(4);
            format!("Wrote: {:?}", &*write)
        })
    };

    for handle in read_handles {
        println!("{}", handle.await.unwrap());
    }
    println!("{}", write_handle.await.unwrap());
}
```

### Semaphore

```rust
use tokio::sync::Semaphore;

#[tokio::main]
async fn main() {
    let sem = Arc::new(Semaphore::new(3));  // 最多 3 个并发
    let mut handles = vec![];

    for i in 0..10 {
        let sem = Arc::clone(&sem);
        handles.push(tokio::spawn(async move {
            let _permit = sem.acquire().await.unwrap();
            println!("Task {} 开始", i);
            tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
            println!("Task {} 结束", i);
        }));
    }

    for handle in handles {
        handle.await.unwrap();
    }
}
```

### Barrier

```rust
#[tokio::main]
async fn main() {
    let barrier = Arc::new(tokio::sync::Barrier::new(5));
    let mut handles = vec![];

    for i in 0..5 {
        let barrier = Arc::clone(&barrier);
        handles.push(tokio::spawn(async move {
            println!("Task {} 准备中...", i);
            barrier.wait().await;
            println!("Task {} 继续执行!", i);
        }));
    }

    for handle in handles {
        handle.await.unwrap();
    }
}
```

### Condvar (条件变量)

```rust
use tokio::sync::Mutex;
use std::sync::Arc;

#[tokio::main]
async fn main() {
    let flag = Arc::new(Mutex::new(false));
    let flag_clone = Arc::clone(&flag);

    // 等待者
    let waiter = tokio::spawn(async move {
        let mut flag = flag_clone.lock().await;
        while !*flag {
            flag = Arc::new(Mutex::new(false)).lock().await;
        }
        println!("Flag is now true!");
    });

    // 设置者
    tokio::spawn(async move {
        tokio::time::sleep(tokio::time::Duration::from_secs(1)).await;
        let mut flag = flag.lock().await;
        *flag = true;
    });

    waiter.await.unwrap();
}
```

## 时间管理

### sleep

```rust
#[tokio::main]
async fn main() {
    println!("开始睡眠...");
    
    tokio::time::sleep(tokio::time::Duration::from_secs(2)).await;
    
    println!("2秒后醒来");
}
```

### interval

```rust
#[tokio::main]
async fn main() {
    let mut interval = tokio::time::interval(tokio::time::Duration::from_secs(1));
    
    for i in 0..5 {
        interval.tick().await;
        println!("Tick {}", i);
    }
}
```

### timeout

```rust
use tokio::time::timeout;

#[tokio::main]
async fn main() {
    let result = timeout(
        tokio::time::Duration::from_millis(100),
        tokio::time::sleep(tokio::time::Duration::from_secs(1))
    ).await;

    match result {
        Ok(_) => println!("操作完成"),
        Err(_) => println!("操作超时!"),
    }
}
```

### Instant

```rust
use tokio::time::Instant;

#[tokio::main]
async fn main() {
    let start = Instant::now();
    
    tokio::time::sleep(tokio::time::Duration::from_secs(1)).await;
    
    println!("耗时: {:?}", start.elapsed());
}
```

### 延迟初始化

```tokio::spawn(async move {
    let _timer = tokio::time::sleep(std::time::Duration::from_secs(10));
    tokio::time::timeout(Duration::from_secs(10), long_operation()).await
});
```

## I/O 操作

### TCP

```rust
use tokio::net::TcpListener;
use tokio::net::TcpStream;

#[tokio::main]
async fn main() -> std::io::Result<()> {
    // TCP 服务器
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("服务器监听在 127.0.0.1:8080");

    loop {
        let (mut socket, addr) = listener.accept().await?;
        println!("新连接: {}", addr);

        tokio::spawn(async move {
            let mut buf = vec![0u8; 1024];
            loop {
                match socket.read(&mut buf).await {
                    Ok(0) => {
                        println!("客户端断开: {}", addr);
                        return;
                    }
                    Ok(n) => {
                        println!("收到 {} 字节", n);
                        if socket.write_all(&buf[..n]).await.is_err() {
                            return;
                        }
                    }
                    Err(_) => return,
                }
            }
        });
    }
}
```

### TCP 客户端

```rust
#[tokio::main]
async fn main() -> std::io::Result<()> {
    let mut stream = TcpStream::connect("127.0.0.1:8080").await?;
    
    let msg = b"Hello, Server!";
    stream.write_all(msg).await?;
    
    let mut buf = vec![0u8; 1024];
    let n = stream.read(&mut buf).await?;
    
    println!("收到: {}", String::from_utf8_lossy(&buf[..n]));
    
    Ok(())
}
```

### UDP

```rust
use tokio::net::UdpSocket;

#[tokio::main]
async fn main() -> std::io::Result<()> {
    let socket = UdpSocket::bind("127.0.0.1:0").await?;
    println!("UDP 绑定到 {}", socket.local_addr()?);

    socket.connect("127.0.0.1:8080").await?;

    socket.send(b"Hello UDP").await?;
    
    let mut buf = vec![0u8; 1024];
    let (len, _) = socket.recv_from(&mut buf).await?;
    
    println!("收到: {}", String::from_utf8_lossy(&buf[..len]));
    
    Ok(())
}
```

### Unix Socket

```rust
use tokio::net::UnixStream;

#[tokio::main]
async fn main() -> std::io::Result<()> {
    let mut stream = UnixStream::connect("/tmp/example.sock").await?;
    
    stream.write_all(b"Hello").await?;
    
    let mut buf = vec![0u8; 1024];
    let n = stream.read(&mut buf).await?;
    
    println!("收到: {}", String::from_utf8_lossy(&buf[..n]));
    
    Ok(())
}
```

### 文件操作

```rust
use tokio::fs::{self, File};
use tokio::io::{AsyncReadExt, AsyncWriteExt};

#[tokio::main]
async fn main() -> std::io::Result<()> {
    // 异步读取文件
    let contents = fs::read_to_string("example.txt").await?;
    println!("文件内容: {}", contents);
    
    // 异步写入文件
    let mut file = File::create("output.txt").await?;
    file.write_all(b"Hello, Tokio!").await?;
    
    // 逐行读取
    let contents = fs::read_to_string("large_file.txt").await?;
    for line in contents.lines().take(10) {
        println!("{}", line);
    }
    
    Ok(())
}
```

## 进程管理

### Spawn 子进程

```rust
use tokio::process::Command;

#[tokio::main]
async fn main() -> std::result::Result<(), Box<dyn std::error::Error>> {
    // 运行外部命令
    let output = Command::new("ls")
        .arg("-la")
        .output()
        .await?;

    println!("stdout: {}", String::from_utf8_lossy(&output.stdout));
    println!("stderr: {}", String::from_utf8_lossy(&output.stderr));
    
    Ok(())
}
```

### 管道

```rust
#[tokio::main]
async fn main() -> std::io::Result<()> {
    let mut child = Command::new("ps")
        .arg("aux")
        .stdout(std::process::Stdio::piped())
        .stderr(std::process::Stdio::piped())
        .spawn()?;

    let stdout = child.stdout.take().unwrap();
    let mut lines = tokio::io::BufReader::new(stdout).lines();
    
    while let Some(line) = lines.next_line().await? {
        if line.contains("tokio") {
            println!("{}", line);
        }
    }

    let status = child.wait().await?;
    println!("进程退出码: {:?}", status);
    
    Ok(())
}
```

## 错误处理

### Result 和 ?

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let data = fetch_data().await?;
    println!("数据: {}", data);
    Ok(())
}

async fn fetch_data() -> Result<String, Box<dyn std::error::Error>> {
    let mut stream = tokio::net::TcpStream::connect("127.0.0.1:8080").await?;
    let mut buf = vec![0u8; 1024];
    let n = stream.read(&mut buf).await?;
    Ok(String::from_utf8_lossy(&buf[..n]).to_string())
}
```

### Anyhow 简化错误

```toml
[dependencies]
anyhow = "1"
```

```rust
use anyhow::{Context, Result};

#[tokio::main]
async fn main() -> Result<()> {
    let config = load_config()
        .context("无法加载配置文件")?;
    
    println!("配置: {:?}", config);
    Ok(())
}

async fn load_config() -> Result<serde_json::Value> {
    let contents = tokio::fs::read_to_string("config.json")
        .await
        .context("读取配置文件失败")?;
    
    serde_json::from_str(&contents)
        .context("解析配置文件失败")
}
```

### thiserror 自定义错误

```toml
[dependencies]
thiserror = "1"
```

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("连接失败: {0}")]
    ConnectionError(String),
    
    #[error("资源未找到: {0}")]
    NotFound(String),
    
    #[error("超时")]
    Timeout,
    
    #[error(transparent)]
    Io(#[from] std::io::Error),
}

#[tokio::main]
async fn main() -> Result<(), AppError> {
    let result = connect().await?;
    Ok(())
}

async fn connect() -> Result<String, AppError> {
    let stream = tokio::net::TcpStream::connect("127.0.0.1:9999")
        .await
        .map_err(|e| AppError::ConnectionError(e.to_string()))?;
    
    Ok("Connected".to_string())
}
```

## 最佳实践

### 1. 避免阻塞主线程

```rust
// 不好：在 async 上下文中使用阻塞调用
#[tokio::main]
async fn bad_example() {
    let data = std::fs::read_to_string("file.txt").unwrap(); // 阻塞!
}

// 好：使用 tokio::task::spawn_blocking
#[tokio::main]
async fn good_example() {
    let data = tokio::task::spawn_blocking(|| {
        std::fs::read_to_string("file.txt").unwrap()
    }).await.unwrap();
}
```

### 2. 正确使用 Mutex

```rust
// 好：尽快释放锁
async fn good_lock(data: Arc<tokio::Mutex<Vec<i32>>>) {
    {
        let mut vec = data.lock().await;
        vec.push(1);
    } // 锁在这里释放
    // 继续其他工作
}

// 不好：持有锁期间进行异步操作
async fn bad_lock(data: Arc<tokio::Mutex<Vec<i32>>>) {
    let mut vec = data.lock().await;
    vec.push(1);
    some_async_operation().await; // 不要这样做！
}
```

### 3. 使用 join! 并行执行

```rust
// 不好：串行执行
#[tokio::main]
async fn bad_sequential() {
    let a = fetch_a().await;
    let b = fetch_b().await;
}

// 好：并行执行
#[tokio::main]
async fn good_parallel() {
    let (a, b) = tokio::join!(
        fetch_a(),
        fetch_b()
    );
}
```

### 4. 优雅关闭

```rust
use tokio::signal;

#[tokio::main]
async fn main() -> std::io::Result<()> {
    let listener = tokio::net::TcpListener::bind("127.0.0.1:8080").await?;
    
    println!("服务器运行中，按 Ctrl+C 关闭");
    
    match signal::ctrl_c().await {
        Ok(()) => println!("\n收到关闭信号，正在关闭..."),
        Err(e) => eprintln!("Error listening for shutdown signal: {}", e),
    }
    
    Ok(())
}
```

### 5. 资源清理

```rust
#[tokio::main]
async fn main() {
    let guard = SomeResource::new();
    
    // 使用 guard 进行工作
    if let Some(resource) = guard {
        // ...
    }
    
    // 显式清理
    drop(guard);
}
```

## 生态工具

### tracing (结构化日志)

```toml
[dependencies]
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
```

```rust
use tracing::{info, error, instrument};

#[instrument(skip(data))]
async fn process_data(data: &[u8]) -> Result<usize, std::io::Error> {
    info!("开始处理数据，长度: {}", data.len());
    
    let result = do_processing(data).await?;
    
    info!("处理完成，结果: {}", result);
    Ok(result)
}

#[tokio::main]
async fn main() {
    tracing_subscriber::fmt()
        .with_target(false)
        .init();
    
    process_data(b"hello").await.unwrap();
}
```

### tokio-console (调试工具)

```bash
cargo install tokio-console
```

```toml
[dependencies]
console-subscriber = "0.2"
```

```rust
fn main() {
    console_subscriber::init();
    // 你的应用代码
}
```

### metrics (指标)

```rust
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel::<u64>(100);
    
    // 生产指标
    tokio::spawn(async move {
        for i in 0.. {
            tx.send(i).await.unwrap();
        }
    });
    
    // 消费指标
    while let Some(metric) = rx.recv().await {
        println!("Metric: {}", metric);
    }
}
```

## 总结

Tokio 是 Rust 异步编程的核心运行时：

**核心组件：**
- **Future 和 async/await**：零成本抽象
- **任务调度**：tokio::spawn、join!、select!
- **通道**：mpsc、oneshot、broadcast、watch
- **同步原语**：Mutex、RwLock、Semaphore、Barrier
- **时间管理**：sleep、interval、timeout
- **I/O**：TCP/UDP/文件/进程

**关键特性：**
- **多线程运行时**：充分利用多核
- **工作窃取**：高效的任务调度
- **异步 I/O**：高性能网络编程
- **结构化并发**：安全地管理大量并发任务

**最佳实践：**
- 避免在 async 上下文中阻塞
- 使用 join! 并行执行独立任务
- 正确使用锁和通道
- 实现优雅关闭
- 使用 tracing 进行结构化日志

掌握 Tokio，你将能够构建高性能、可靠的 Rust 异步应用程序！

快乐编程，大家来 Rust! 🦀
