+++
title = "Rust 语言常用第三方库指南"
date = "2026-03-16T22:35:00+08:00"

[taxonomies]
tags = ["rust", "第三方库", "编程", "crates"]
categories = ["编程"]

[extra]
summary = "Rust 拥有丰富的 crates 生态，本文介绍 Web 框架、异步运行时、数据库、序列化等常用库，帮助你快速构建 Rust 应用。"
author = "博主"
+++

Rust 语言以其内存安全、零成本抽象和出色的性能而闻名。除了强大的标准库外，Rust 还拥有丰富的 crates 生态系统。本文将介绍各类常用第三方库，帮助你更高效地开发 Rust 应用。

## Web 框架

### Axum

[Axum](https://github.com/tokio-rs/axum) 是 Tokio 团队开发的 Web 框架，与 Tokio 生态系统深度集成。

```rust
use axum::{
    routing::{get, post},
    Router,
    extract::{Path, Json},
    response::Json as ResponseJson,
};
use serde::{Deserialize, Serialize};
use std::net::SocketAddr;

#[derive(Serialize)]
struct HelloResponse {
    message: String,
}

#[derive(Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

#[derive(Serialize)]
struct User {
    id: u64,
    name: String,
    email: String,
}

async fn hello() -> ResponseJson<HelloResponse> {
    ResponseJson(HelloResponse {
        message: "Hello, World!".to_string(),
    })
}

async fn get_user(Path(id): Path<u64>) -> ResponseJson<User> {
    ResponseJson(User {
        id,
        name: "张三".to_string(),
        email: "zhangsan@example.com".to_string(),
    })
}

async fn create_user(Json(payload): Json<CreateUser>) -> ResponseJson<User> {
    ResponseJson(User {
        id: 1,
        name: payload.name,
        email: payload.email,
    })
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(hello))
        .route("/users/:id", get(get_user))
        .route("/users", post(create_user));

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("Listening on {}", addr);
    
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

### Actix-web

[Actix-web](https://github.com/actix/actix-web) 是性能极高的 Web 框架，基于 Actor 模型。

```rust
use actix_web::{get, post, web, App, HttpResponse, HttpServer, Responder};
use serde::{Deserialize, Serialize};

#[derive(Serialize)]
struct HelloResponse {
    message: String,
}

#[derive(Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

#[derive(Serialize)]
struct User {
    id: u64,
    name: String,
    email: String,
}

#[get("/")]
async fn hello() -> impl Responder {
    HttpResponse::Ok().json(HelloResponse {
        message: "Hello, World!".to_string(),
    })
}

#[get("/users/{id}")]
async fn get_user(path: web::Path<u64>) -> impl Responder {
    let id = path.into_inner();
    HttpResponse::Ok().json(User {
        id,
        name: "张三".to_string(),
        email: "zhangsan@example.com".to_string(),
    })
}

#[post("/users")]
async fn create_user(body: web::Json<CreateUser>) -> impl Responder {
    HttpResponse::Created().json(User {
        id: 1,
        name: body.name.clone(),
        email: body.email.clone(),
    })
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(hello)
            .service(get_user)
            .service(create_user)
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### Rocket

[Rocket](https://rocket.rs/) 是注重易用性和类型安全的 Web 框架。

```rust
#[macro_use]
extern crate rocket;

use rocket::serde::{json::Json, Deserialize, Serialize};

#[derive(Serialize)]
#[serde(crate = "rocket::serde")]
struct Message {
    message: String,
}

#[derive(Deserialize)]
#[serde(crate = "rocket::serde")]
struct UserInput {
    name: String,
    age: u8,
}

#[get("/")]
fn index() -> Json<Message> {
    Json(Message {
        message: "Hello, Rocket!".to_string(),
    })
}

#[get("/hello/<name>")]
fn hello(name: &str) -> String {
    format!("Hello, {}!", name)
}

#[post("/user", format = "json", data = "<input>")]
fn create_user(input: Json<UserInput>) -> String {
    format!("Created user: {} (age: {})", input.name, input.age)
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![index, hello, create_user])
}
```

## 异步运行时

### Tokio

[Tokio](https://tokio.rs/) 是 Rust 最流行的异步运行时，提供 I/O、调度、定时器等功能。

```rust
use tokio::time::{sleep, Duration};
use tokio::task;

#[tokio::main]
async fn main() {
    // 创建并发任务
    let task1 = task::spawn(async {
        println!("Task 1 started");
        sleep(Duration::from_secs(1)).await;
        println!("Task 1 completed");
        42
    });

    let task2 = task::spawn(async {
        println!("Task 2 started");
        sleep(Duration::from_millis(500)).await;
        println!("Task 2 completed");
        "Hello"
    });

    // 等待所有任务完成
    let result1 = task1.await.unwrap();
    let result2 = task2.await.unwrap();

    println!("Results: {}, {}", result1, result2);

    // TCP 服务器示例
    use tokio::net::TcpListener;
    use tokio::io::{AsyncReadExt, AsyncWriteExt};

    let listener = TcpListener::bind("127.0.0.1:8080").await.unwrap();
    println!("Server listening on 127.0.0.1:8080");

    loop {
        let (mut socket, _) = listener.accept().await.unwrap();
        
        tokio::spawn(async move {
            let mut buf = [0; 1024];
            
            match socket.read(&mut buf).await {
                Ok(n) if n == 0 => return,
                Ok(n) => {
                    socket.write_all(&buf[0..n]).await.unwrap();
                }
                Err(e) => {
                    eprintln!("Failed to read from socket: {}", e);
                }
            }
        });
    }
}
```

### async-std

[async-std](https://async.rs/) 提供类似标准库的异步 API。

```rust
use async_std::task;
use async_std::fs::File;
use async_std::io::prelude::*;
use async_std::net::TcpListener;

#[async_std::main]
async fn main() -> std::io::Result<()> {
    // 异步文件操作
    let mut file = File::create("example.txt").await?;
    file.write_all(b"Hello, async-std!").await?;
    
    let mut file = File::open("example.txt").await?;
    let mut contents = String::new();
    file.read_to_string(&mut contents).await?;
    println!("File contents: {}", contents);

    // TCP 服务器
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("Listening on 127.0.0.1:8080");

    while let Ok((mut stream, _)) = listener.accept().await {
        task::spawn(async move {
            let mut buf = vec![0u8; 1024];
            if let Ok(n) = stream.read(&mut buf).await {
                stream.write_all(&buf[..n]).await.unwrap();
            }
        });
    }

    Ok(())
}
```

## 数据库

### Diesel

[Diesel](https://diesel.rs/) 是安全的 ORM 和查询构建器。

```rust
use diesel::prelude::*;
use diesel::sqlite::SqliteConnection;
use diesel::Queryable;

// 定义模型
#[derive(Queryable)]
struct User {
    id: i32,
    name: String,
    email: String,
}

#[derive(Insertable)]
#[table_name = "users"]
struct NewUser {
    name: String,
    email: String,
}

// 定义表结构
table! {
    users (id) {
        id -> Integer,
        name -> Text,
        email -> Text,
    }
}

fn establish_connection() -> SqliteConnection {
    SqliteConnection::establish("database.sqlite")
        .expect("Error connecting to database")
}

fn main() {
    use self::users::dsl::*;

    let connection = establish_connection();

    // 插入数据
    let new_user = NewUser {
        name: "张三".to_string(),
        email: "zhangsan@example.com".to_string(),
    };

    diesel::insert_into(users)
        .values(&new_user)
        .execute(&connection)
        .expect("Error saving new user");

    // 查询数据
    let results = users
        .filter(name.like("%张%"))
        .limit(5)
        .load::<User>(&connection)
        .expect("Error loading users");

    println!("找到 {} 个用户", results.len());
    for user in results {
        println!("{}: {} ({})", user.id, user.name, user.email);
    }

    // 更新数据
    diesel::update(users.find(1))
        .set(name.eq("李四"))
        .execute(&connection)
        .expect("Error updating user");

    // 删除数据
    diesel::delete(users.filter(id.eq(1)))
        .execute(&connection)
        .expect("Error deleting user");
}
```

### sqlx

[sqlx](https://github.com/launchbadge/sqlx) 是异步的、编译时检查的 SQL 库。

```rust
use sqlx::{sqlite::SqlitePool, Row};

#[derive(Debug)]
struct User {
    id: i64,
    name: String,
    email: String,
}

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    // 创建连接池
    let pool = SqlitePool::connect("sqlite://database.sqlite").await?;

    // 创建表
    sqlx::query(
        r#"
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            email TEXT NOT NULL
        )
        "#
    )
    .execute(&pool)
    .await?;

    // 插入数据
    let user_id: i64 = sqlx::query(
        "INSERT INTO users (name, email) VALUES (?, ?) RETURNING id"
    )
    .bind("张三")
    .bind("zhangsan@example.com")
    .fetch_one(&pool)
    .await?
    .get(0);

    println!("Created user with id: {}", user_id);

    // 查询单行
    let user: User = sqlx::query_as(
        "SELECT id, name, email FROM users WHERE id = ?"
    )
    .bind(user_id)
    .fetch_one(&pool)
    .await?;

    println!("User: {:?}", user);

    // 查询多行
    let users: Vec<User> = sqlx::query_as(
        "SELECT id, name, email FROM users"
    )
    .fetch_all(&pool)
    .await?;

    for user in users {
        println!("User: {:?}", user);
    }

    Ok(())
}
```

### redis-rs

[redis-rs](https://github.com/redis-rs/redis-rs) 是 Redis 的 Rust 客户端。

```rust
use redis::{AsyncCommands, Client};

#[tokio::main]
async fn main() -> redis::RedisResult<()> {
    // 创建客户端
    let client = Client::open("redis://127.0.0.1:6379/")?;
    let mut con = client.get_async_connection().await?;

    // 设置键值
    con.set("my_key", "my_value").await?;

    // 获取值
    let value: String = con.get("my_key").await?;
    println!("my_key: {}", value);

    // 设置过期时间
    con.set_ex("temp_key", "temp_value", 60).await?;

    // 使用哈希
    con.hset("user:1", "name", "张三").await?;
    con.hset("user:1", "age", 25).await?;

    let name: String = con.hget("user:1", "name").await?;
    println!("User name: {}", name);

    // 列表操作
    con.lpush("my_list", "item1").await?;
    con.lpush("my_list", "item2").await?;

    let item: String = con.rpop("my_list").await?;
    println!("Popped item: {}", item);

    // 发布订阅
    let mut pubsub = con.into_pubsub();
    pubsub.subscribe("my_channel").await?;

    Ok(())
}
```

## 序列化

### Serde

[Serde](https://serde.rs/) 是 Rust 的序列化和反序列化框架。

```rust
use serde::{Deserialize, Serialize};
use serde_json;

#[derive(Serialize, Deserialize, Debug)]
struct User {
    name: String,
    email: String,
    age: u8,
    #[serde(default)]
    address: Option<String>,
    #[serde(rename = "user_id")]
    id: u64,
}

fn main() {
    // 序列化为 JSON
    let user = User {
        name: "张三".to_string(),
        email: "zhangsan@example.com".to_string(),
        age: 25,
        address: Some("北京市".to_string()),
        id: 1,
    };

    let json = serde_json::to_string(&user).unwrap();
    println!("JSON: {}", json);

    // 反序列化
    let json_str = r#"{
        "name": "李四",
        "email": "lisi@example.com",
        "age": 30,
        "user_id": 2
    }"#;

    let user: User = serde_json::from_str(json_str).unwrap();
    println!("User: {:?}", user);

    // 序列化为其他格式
    // YAML
    let yaml = serde_yaml::to_string(&user).unwrap();
    println!("YAML:\n{}", yaml);

    // TOML
    let toml = toml::to_string(&user).unwrap();
    println!("TOML:\n{}", toml);

    // MessagePack
    let msgpack = rmp_serde::to_vec(&user).unwrap();
    println!("MessagePack length: {}", msgpack.len());
}
```

## 错误处理

### thiserror

[thiserror](https://github.com/dtolnay/thiserror) 用于定义错误类型。

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum MyError {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
    
    #[error("Parse error: {0}")]
    Parse(#[from] std::num::ParseIntError),
    
    #[error("Custom error: {message}")]
    Custom { message: String },
    
    #[error("Not found: {0}")]
    NotFound(String),
}

fn may_fail() -> Result<i32, MyError> {
    // 自动转换错误类型
    let content = std::fs::read_to_string("file.txt")?;
    let num: i32 = content.trim().parse()?;
    
    if num < 0 {
        return Err(MyError::Custom {
            message: "Number must be positive".to_string(),
        });
    }
    
    Ok(num)
}

fn find_user(id: u64) -> Result<User, MyError> {
    if id == 0 {
        return Err(MyError::NotFound(format!("User with id {}", id)));
    }
    
    Ok(User {
        id,
        name: "张三".to_string(),
    })
}
```

### anyhow

[anyhow](https://github.com/dtolnay/anyhow) 用于简化错误处理。

```rust
use anyhow::{Context, Result};

fn read_config() -> Result<String> {
    // 自动转换任何错误
    let config = std::fs::read_to_string("config.json")
        .context("Failed to read config file")?;
    
    Ok(config)
}

fn parse_number(s: &str) -> Result<i32> {
    let num: i32 = s.parse()
        .with_context(|| format!("Failed to parse '{}' as number", s))?;
    
    Ok(num)
}

fn main() -> Result<()> {
    let config = read_config()?;
    let number = parse_number("42")?;
    
    println!("Config: {}, Number: {}", config, number);
    
    // 添加上下文
    let result = some_operation()
        .context("Operation failed")?;
    
    Ok(())
}

fn some_operation() -> Result<()> {
    anyhow::bail!("Something went wrong");
}
```

## HTTP 客户端

### reqwest

[reqwest](https://github.com/seanmonstar/reqwest) 是易用的 HTTP 客户端。

```rust
use reqwest;
use serde::{Deserialize, Serialize};

#[derive(Serialize)]
struct CreateUserRequest {
    name: String,
    email: String,
}

#[derive(Deserialize, Debug)]
struct UserResponse {
    id: u64,
    name: String,
    email: String,
}

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = reqwest::Client::new();

    // GET 请求
    let response: serde_json::Value = client
        .get("https://api.github.com/users/rust-lang")
        .header("User-Agent", "reqwest")
        .send()
        .await?
        .json()
        .await?;
    
    println!("Response: {:?}", response);

    // POST JSON
    let user = CreateUserRequest {
        name: "张三".to_string(),
        email: "zhangsan@example.com".to_string(),
    };

    let created: UserResponse = client
        .post("https://api.example.com/users")
        .json(&user)
        .send()
        .await?
        .json()
        .await?;
    
    println!("Created user: {:?}", created);

    // 表单提交
    let params = [("key", "value"), ("foo", "bar")];
    let response = client
        .post("https://httpbin.org/post")
        .form(&params)
        .send()
        .await?;

    // 下载文件
    let bytes = client
        .get("https://example.com/file.pdf")
        .send()
        .await?
        .bytes()
        .await?;
    
    std::fs::write("file.pdf", &bytes)?;

    Ok(())
}
```

## 日志

### tracing

[tracing](https://github.com/tokio-rs/tracing) 是异步感知的结构化日志框架。

```rust
use tracing::{info, warn, error, debug, span, Level};
use tracing_subscriber;

#[tracing::instrument]
async fn process_user(user_id: u64) -> Result<(), String> {
    info!(user_id, "Processing user");
    
    // 模拟异步操作
    tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
    
    if user_id == 0 {
        error!(user_id, "Invalid user ID");
        return Err("Invalid user ID".to_string());
    }
    
    info!(user_id, "User processed successfully");
    Ok(())
}

#[tokio::main]
async fn main() {
    // 初始化订阅者
    tracing_subscriber::fmt::init();
    
    // 创建 span
    let span = span!(Level::INFO, "main", version = "1.0");
    let _enter = span.enter();
    
    info!("Application started");
    
    // 带字段的日志
    info!(
        target: "app",
        user_count = 42,
        "Users loaded"
    );
    
    // 异步函数自动追踪
    if let Err(e) = process_user(123).await {
        error!(error = %e, "Failed to process user");
    }
    
    // 手动创建 span
    let db_span = span!(Level::DEBUG, "database_query", query = "SELECT * FROM users");
    {
        let _enter = db_span.enter();
        debug!("Executing query");
        // 数据库操作
        debug!("Query completed");
    }
}
```

### log + env_logger

标准日志接口的简单实现。

```rust
use log::{info, debug, error, warn};

fn main() {
    // 初始化日志
    env_logger::init();
    
    // 不同级别的日志
    debug!("This is a debug message");
    info!("Application started");
    warn!("This is a warning");
    error!("This is an error: {}", "something failed");
    
    // 带目标模块的日志
    log::info!(target: "database", "Connected to database");
}
```

## CLI 工具

### clap

[clap](https://github.com/clap-rs/clap) 是强大的命令行参数解析器。

```rust
use clap::{Parser, Subcommand, Args};

#[derive(Parser)]
#[command(name = "myapp")]
#[command(about = "A CLI application")]
#[command(version = "1.0")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    /// 添加新用户
    Add(AddArgs),
    /// 列出所有用户
    List,
    /// 删除用户
    Delete { id: u64 },
}

#[derive(Args)]
struct AddArgs {
    /// 用户名
    name: String,
    /// 邮箱地址
    #[arg(short, long)]
    email: Option<String>,
    /// 是否设为管理员
    #[arg(short, long)]
    admin: bool,
}

fn main() {
    let cli = Cli::parse();

    match cli.command {
        Commands::Add(args) => {
            println!("Adding user: {}", args.name);
            if let Some(email) = args.email {
                println!("Email: {}", email);
            }
            if args.admin {
                println!("Admin: true");
            }
        }
        Commands::List => {
            println!("Listing all users");
        }
        Commands::Delete { id } => {
            println!("Deleting user with ID: {}", id);
        }
    }
}
```

### structopt (已合并到 clap v3)

使用派生宏定义 CLI。

```rust
use clap::Parser;

#[derive(Parser, Debug)]
#[clap(name = "example")]
#[clap(about = "An example CLI")]
struct Opts {
    /// 输入文件
    #[clap(short, long, default_value = "input.txt")]
    input: String,
    
    /// 输出文件
    #[clap(short, long, default_value = "output.txt")]
    output: String,
    
    /// 详细模式
    #[clap(short, long)]
    verbose: bool,
    
    /// 要处理的文件
    files: Vec<String>,
}

fn main() {
    let opts = Opts::parse();
    
    println!("Options: {:?}", opts);
}
```

## 工具类

### uuid

[uuid](https://github.com/uuid-rs/uuid) 用于生成 UUID。

```rust
use uuid::Uuid;

fn main() {
    // 生成随机 UUID v4
    let id = Uuid::new_v4();
    println!("UUID v4: {}", id);
    
    // 生成 UUID v1
    let context = uuid::Context::new(0);
    let id = Uuid::new_v1(
        uuid::Timestamp::from_unix(&context, 1497624119, 1234),
        &[1, 2, 3, 4, 5, 6],
    );
    
    // 解析 UUID
    let parsed = Uuid::parse_str("550e8400-e29b-41d4-a716-446655440000").unwrap();
    println!("Parsed: {}", parsed);
    
    // 转换为不同格式
    println!("Simple: {}", id.simple());
    println!("URN: {}", id.urn());
    println!("Hyphenated: {}", id.hyphenated());
}
```

### chrono

[chrono](https://github.com/chronotope/chrono) 是日期和时间处理库。

```rust
use chrono::{DateTime, Utc, Local, NaiveDate, Duration, TimeZone};

fn main() {
    // 当前时间
    let now: DateTime<Utc> = Utc::now();
    println!("UTC now: {}", now);
    
    let local: DateTime<Local> = Local::now();
    println!("Local now: {}", local);
    
    // 解析日期
    let date = NaiveDate::from_ymd_opt(2024, 3, 15).unwrap();
    println!("Date: {}", date);
    
    // 时间计算
    let tomorrow = now + Duration::days(1);
    println!("Tomorrow: {}", tomorrow);
    
    let two_hours_ago = now - Duration::hours(2);
    println!("Two hours ago: {}", two_hours_ago);
    
    // 格式化
    let formatted = now.format("%Y-%m-%d %H:%M:%S").to_string();
    println!("Formatted: {}", formatted);
    
    // 解析字符串
    let parsed = DateTime::parse_from_str(
        "2024-03-15 14:30:00",
        "%Y-%m-%d %H:%M:%S"
    ).unwrap();
    println!("Parsed: {}", parsed);
    
    // 时区转换
    let utc = Utc::now();
    let beijing = utc.with_timezone(&chrono::FixedOffset::east_opt(8 * 3600).unwrap());
    println!("Beijing time: {}", beijing);
}
```

### regex

[regex](https://github.com/rust-lang/regex) 是正则表达式库。

```rust
use regex::Regex;

fn main() {
    // 编译正则表达式
    let re = Regex::new(r"\d+").unwrap();
    
    // 查找匹配
    let text = "abc123def456";
    for mat in re.find_iter(text) {
        println!("Found: {} at {}", mat.as_str(), mat.start());
    }
    
    // 捕获组
    let re = Regex::new(r"(\w+)@(\w+)\.(\w+)").unwrap();
    let email = "user@example.com";
    
    if let Some(caps) = re.captures(email) {
        println!("Username: {}", &caps[1]);
        println!("Domain: {}", &caps[2]);
        println!("TLD: {}", &caps[3]);
    }
    
    // 替换
    let result = re.replace_all("Contact: user1@example.com or user2@test.org", "$1@hidden.com");
    println!("Replaced: {}", result);
    
    // 验证邮箱格式
    let email_re = Regex::new(r"^[\w.-]+@[\w.-]+\.\w+$").unwrap();
    println!("Is valid: {}", email_re.is_match("test@example.com"));
}
```

## 测试

### tokio-test

用于测试异步代码。

```rust
#[cfg(test)]
mod tests {
    use tokio_test;
    use std::time::Duration;
    
    #[tokio::test]
    async fn test_async_function() {
        let result = async {
            tokio::time::sleep(Duration::from_millis(10)).await;
            42
        }.await;
        
        assert_eq!(result, 42);
    }
    
    #[test]
    fn test_async_block() {
        tokio_test::block_on(async {
            let result = some_async_function().await;
            assert!(result.is_ok());
        });
    }
    
    #[test]
    #[should_panic]
    fn test_timeout() {
        tokio_test::block_on(async {
            tokio::time::timeout(
                Duration::from_millis(10),
                async {
                    tokio::time::sleep(Duration::from_secs(1)).await;
                }
            ).await.unwrap();
        });
    }
}

async fn some_async_function() -> Result<(), ()> {
    Ok(())
}
```

### mockall

[mockall](https://github.com/asomers/mockall) 用于创建 Mock 对象。

```rust
use mockall::automock;

#[automock]
trait Database {
    fn get_user(&self, id: u64) -> Option<User>;
    fn save_user(&mut self, user: &User) -> Result<(), String>;
}

struct User {
    id: u64,
    name: String,
}

struct UserService<T: Database> {
    db: T,
}

impl<T: Database> UserService<T> {
    fn get_user_name(&self, id: u64) -> Option<String> {
        self.db.get_user(id).map(|u| u.name)
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_get_user_name() {
        let mut mock = MockDatabase::new();
        
        mock.expect_get_user()
            .with(mockall::predicate::eq(1))
            .times(1)
            .returning(|_| Some(User {
                id: 1,
                name: "张三".to_string(),
            }));
        
        let service = UserService { db: mock };
        let name = service.get_user_name(1);
        
        assert_eq!(name, Some("张三".to_string()));
    }
}
```

## 总结

本文介绍了 Rust 生态中常用的第三方库，涵盖了：

- **Web 框架**：Axum、Actix-web、Rocket
- **异步运行时**：Tokio、async-std
- **数据库**：Diesel、sqlx、redis-rs
- **序列化**：Serde
- **错误处理**：thiserror、anyhow
- **HTTP 客户端**：reqwest
- **日志**：tracing、log
- **CLI 工具**：clap
- **工具类**：uuid、chrono、regex
- **测试**：tokio-test、mockall

这些 crates 都经过广泛使用和验证，可以帮助你更高效地开发 Rust 应用。建议根据项目需求选择合适的库，并始终关注 [crates.io](https://crates.io/) 和官方文档获取最新信息。

快乐编程，大家来 Rust! 🦀
