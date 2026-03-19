+++
title = "Actix Web 框架完全指南"
date = "2026-03-19T04:40:00+08:00"

[taxonomies]
tags = ["rust", "actix", "web", "框架", "教程"]
categories = ["编程", "网络"]

[extra]
summary = "Actix 是 Rust 生态中最受欢迎的 Web 框架之一，本文详细介绍 Actix 和 Actix-web 的核心概念、使用方法和最佳实践。"
author = "博主"
+++

Actix 是 Rust 生态中最受欢迎的高性能 Web 框架之一，由 Actor 模型提供支持，以其出色的性能和优雅的 API 设计著称。本文将详细介绍 Actix 生态系统的两个核心库：**Actix**（Actor 框架）和 **Actix-web**（Web 框架）。

## Actix 生态系统概述

Actix 生态系统包含多个 crate：

- **actix**：Actor 框架核心库
- **actix-web**：基于 Actix 的 Web 框架
- **actix-rt**：Actix 运行时
- **actix-derive**：派生宏支持

## Actix Actor 框架基础

Actix 基于 Actor 模型，每个 Actor 是一个独立的计算单元，通过消息传递进行通信。

### 基本概念

Actor 的核心概念包括：

- **Actor**：独立的计算单元
- **Message**：Actor 之间传递的消息
- **Handler**：处理消息的逻辑
- **Context**：Actor 的执行上下文

### 第一个 Actor

```rust
use actix::{Actor, Context, System};

struct MyActor;

impl Actor for MyActor {
    type Context = Context<Self>;

    fn started(&mut self, _ctx: &mut Self::Context) {
        println!("I am alive!");
        System::current().stop(); // 停止系统
    }
}

fn main() {
    let system = System::new();

    let _addr = system.block_on(async { MyActor.start() });

    system.run().unwrap();
}
```

### 定义和处理消息

```rust
use actix::{Actor, Context, Handler, System};
use actix_derive::{Message, MessageResponse};

// 定义消息及响应类型
#[derive(MessageResponse)]
struct Added(usize);

#[derive(Message)]
#[rtype(Added)]
struct Sum(usize, usize);

// 定义 Actor
#[derive(Default)]
struct Adder;

impl Actor for Adder {
    type Context = Context<Self>;
}

// 实现消息处理器
impl Handler<Sum> for Adder {
    type Result = <Sum as actix::Message>::Result;

    fn handle(&mut self, msg: Sum, _: &mut Self::Context) -> Added {
        Added(msg.0 + msg.1)
    }
}

#[actix::main] // 简化异步 main 函数
async fn main() {
    let addr = Adder.start();
    let res = addr.send(Sum(10, 5)).await;

    match res {
        Ok(result) => println!("SUM: {}", result.0),
        _ => println!("通信失败"),
    }
}
```

### Actor 状态管理

```rust
use actix::prelude::*;
use std::time::Duration;

#[derive(Message)]
#[rtype(result = "()")]
struct Ping {
    pub id: usize,
}

// 带状态的 Actor
struct Game {
    counter: usize,
    name: String,
    recipient: Recipient<Ping>,
}

impl Actor for Game {
    type Context = Context<Game>;
}

// 消息处理器
impl Handler<Ping> for Game {
    type Result = ();

    fn handle(&mut self, msg: Ping, ctx: &mut Context<Self>) {
        self.counter += 1;

        if self.counter > 10 {
            System::current().stop();
        } else {
            println!("[{}] 收到 Ping {}", self.name, msg.id);

            // 延迟发送消息
            ctx.run_later(Duration::new(0, 100), move |act, _| {
                act.recipient.do_send(Ping { id: msg.id + 1 });
            });
        }
    }
}
```

## Actix-web Web 框架

Actix-web 是基于 Actix 的高性能 Web 框架，支持 HTTP/1.x 和 HTTP/2、WebSocket 等功能。

### 快速入门

```rust
use actix_web::{get, web, App, HttpServer, Responder};

#[get("/hello/{name}")]
async fn greet(name: web::Path<String>) -> impl Responder {
    format!("Hello {}!", name)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(greet)
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

### 路由配置

Actix-web 提供了灵活的路由系统：

```rust
use actix_web::{web, App, HttpResponse, HttpServer};
use serde::Deserialize;

// 定义查询参数结构
#[derive(Deserialize)]
struct QueryInfo {
    name: Option<String>,
    age: Option<u32>,
}

// 定义 JSON 请求体
#[derive(serde::Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

// GET 路由 - 路径参数
#[get("/users/{id}")]
async fn get_user(path: web::Path<u64>) -> HttpResponse {
    let user_id = path.into_inner();
    HttpResponse::Ok().json(serde_json::json!({
        "id": user_id,
        "name": "张三",
        "email": "zhangsan@example.com"
    }))
}

// GET 路由 - 查询参数
async fn search_users(query: web::Query<QueryInfo>) -> HttpResponse {
    let name = query.name.as_deref().unwrap_or("all");
    HttpResponse::Ok().json(serde_json::json!({
        "query": name,
        "age": query.age,
        "results": []
    }))
}

// POST 路由 - JSON 请求体
async fn create_user(body: web::Json<CreateUser>) -> HttpResponse {
    HttpResponse::Created().json(serde_json::json!({
        "id": 1,
        "name": body.name,
        "email": body.email
    }))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/search", web::get().to(search_users))
            .service(get_user)
            .service(
                web::resource("/users")
                    .route(web::post().to(create_user))
            )
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### 使用 Scope 组织路由

```rust
use actix_web::{web, App, HttpResponse, HttpServer};

async fn index() -> HttpResponse {
    HttpResponse::Ok().body("Welcome to API")
}

async fn health() -> HttpResponse {
    HttpResponse::Ok().json(serde_json::json!({"status": "healthy"}))
}

async fn list_users() -> HttpResponse {
    HttpResponse::Ok().json(serde_json::json!([
        {"id": 1, "name": "张三"},
        {"id": 2, "name": "李四"}
    ]))
}

async fn get_user(path: web::Path<u64>) -> HttpResponse {
    HttpResponse::Ok().json(serde_json::json!({
        "id": *path
    }))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(index))
            // API v1 版本
            .service(
                web::scope("/api/v1")
                    .route("/health", web::get().to(health))
                    .service(
                        web::scope("/users")
                            .route("", web::get().to(list_users))
                            .route("/{id}", web::get().to(get_user))
                    )
            )
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### 状态管理

```rust
use actix_web::{web, App, HttpResponse, HttpServer};
use std::sync::Mutex;

struct AppState {
    db: Mutex<Vec<String>>,
}

async fn get_items(data: web::Data<AppState>) -> HttpResponse {
    let items = data.db.lock().unwrap();
    HttpResponse::Ok().json(items.clone())
}

async fn add_item(
    item: web::Json<String>,
    data: web::Data<AppState>,
) -> HttpResponse {
    let mut items = data.db.lock().unwrap();
    items.push(item.into_inner());
    HttpResponse::Created().json(serde_json::json!({"status": "added"}))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let app_state = web::Data::new(AppState {
        db: Mutex::new(vec![]),
    });

    HttpServer::new(move || {
        App::new()
            .app_data(app_state.clone())
            .route("/items", web::get().to(get_items))
            .route("/items", web::post().to(add_item))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

## 中间件

Actix-web 支持丰富的中间件功能。

### Logger 中间件

```rust
use actix_web::{web, App, HttpServer, HttpResponse};
use actix_web::middleware::Logger;
use std::env;

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // 设置日志格式
    env::set_var("RUST_LOG", "info");
    env_logger::init();

    HttpServer::new(|| {
        App::new()
            .wrap(Logger::default())
            .wrap(Logger::new("%a %{User-Agent}i"))
            .route("/", web::get().to(HttpResponse::Ok))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### Compress 中间件

```rust
use actix_web::{web, App, HttpServer, HttpResponse};
use actix_web::middleware::Compress;

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .wrap(Compress::default())
            .route("/", web::get().to(|| async {
                HttpResponse::Ok().body("Large response body...")
            }))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### 错误处理中间件

```rust
use actix_web::{
    dev::ServiceResponse,
    http::{header, StatusCode},
    middleware::{ErrorHandlerResponse, ErrorHandlers},
    web, App, HttpResponse, HttpServer, Result,
};

fn add_error_header<B>(
    mut res: ServiceResponse<B>
) -> Result<ErrorHandlerResponse<B>> {
    res.response_mut().headers_mut().insert(
        header::CONTENT_TYPE,
        header::HeaderValue::from_static("application/json"),
    );
    Ok(ErrorHandlerResponse::Response(
        res.map_into_left_body()
    ))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .wrap(
                ErrorHandlers::new()
                    .handler(StatusCode::NOT_FOUND, add_error_header)
                    .handler(StatusCode::INTERNAL_SERVER_ERROR, add_error_header)
            )
            .route("/", web::get().to(HttpResponse::Ok))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### 自定义中间件

```rust
use actix_web::{
    dev::{Service, ServiceRequest, ServiceResponse, Transform},
    Error, HttpResponse,
};
use futures_util::future::LocalBoxFuture;
use std::future::{ready, Ready};
use std::rc::Rc;

pub struct SayHi;

impl<S, B> Transform<S, ServiceRequest> for SayHi
where
    S: Service<ServiceRequest, Response = ServiceResponse<B>, Error = Error>,
    S::Future: 'static,
    B: 'static,
{
    type Response = ServiceResponse<B>;
    type Error = Error;
    type InitError = ();
    type Transform = SayHiMiddleware<S>;
    type Future = Ready<Result<Self::Transform, Self::InitError>>;

    fn new_transform(&self, service: S) -> Self::Future {
        ready(Ok(SayHiMiddleware { service }))
    }
}

pub struct SayHiMiddleware<S> {
    service: S,
}

impl<S, B> Service<ServiceRequest> for SayHiMiddleware<S>
where
    S: Service<ServiceRequest, Response = ServiceResponse<B>, Error = Error>,
    S::Future: 'static,
    B: 'static,
{
    type Response = ServiceResponse<B>;
    type Error = Error;
    type Future = LocalBoxFuture<'static, Result<Self::Response, Self::Error>>;

    fn poll_ready(
        &self,
        cx: &mut std::task::Context<'_>
    ) -> std::task::Poll<Result<(), Self::Error>> {
        self.service.poll_ready(cx)
    }

    fn call(&self, req: ServiceRequest) -> Self::Future {
        println!("Hi from start. You requested: {}", req.path());
        let fut = self.service.call(req);
        Box::pin(async move {
            let res = fut.await?;
            println!("Hi from response");
            Ok(res)
        })
    }
}
```

## 错误处理

### 自定义错误类型

```rust
use actix_web::{
    error, http::StatusCode, App, HttpResponse, HttpServer, ResponseError,
};
use serde::Serialize;
use std::fmt;

#[derive(Debug)]
enum MyError {
    NotFound(String),
    BadRequest(String),
    InternalError(String),
}

#[derive(Serialize)]
struct ErrorResponse {
    error: String,
    message: String,
}

impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            MyError::NotFound(msg) => write!(f, "Not Found: {}", msg),
            MyError::BadRequest(msg) => write!(f, "Bad Request: {}", msg),
            MyError::InternalError(msg) => write!(f, "Internal Error: {}", msg),
        }
    }
}

impl ResponseError for MyError {
    fn status_code(&self) -> StatusCode {
        match self {
            MyError::NotFound(_) => StatusCode::NOT_FOUND,
            MyError::BadRequest(_) => StatusCode::BAD_REQUEST,
            MyError::InternalError(_) => StatusCode::INTERNAL_SERVER_ERROR,
        }
    }

    fn error_response(&self) -> HttpResponse {
        let status = self.status_code();
        HttpResponse::build(status).json(ErrorResponse {
            error: status.to_string(),
            message: self.to_string(),
        })
    }
}

async fn find_user(id: web::Path<u64>) -> Result<HttpResponse, MyError> {
    if *id == 0 {
        Err(MyError::NotFound(format!("User {} not found", id)))
    } else if *id > 1000 {
        Err(MyError::BadRequest("Invalid user ID range".to_string()))
    } else {
        Ok(HttpResponse::Ok().json(serde_json::json!({
            "id": *id,
            "name": "张三"
        })))
    }
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new().route("/users/{id}", web::get().to(find_user))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### 使用 anyhow 进行错误处理

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Result};
use anyhow::Context;

fn read_config() -> anyhow::Result<String> {
    std::fs::read_to_string("config.json")
        .context("无法读取配置文件")
}

async fn handler() -> Result<HttpResponse> {
    let config = read_config()
        .map_err(|e| actix_web::error::InternalError::from_response(
            e,
            HttpResponse::InternalServerError().body(e.to_string()),
        ))?;

    Ok(HttpResponse::Ok().json(serde_json::json!({
        "config": config
    })))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new().route("/", web::get().to(handler))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

## 数据库集成

### 使用 sqlx

```rust
use actix_web::{web, App, HttpResponse, HttpServer};
use sqlx::{mysql::MySqlPoolOptions, MySql, Pool};
use serde::{Deserialize, Serialize};

#[derive(Serialize, sqlx::FromRow)]
struct User {
    id: i64,
    name: String,
    email: String,
}

#[derive(Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

struct AppState {
    db: Pool<MySql>,
}

async fn get_users(data: web::Data<AppState>) -> Result<HttpResponse, sqlx::Error> {
    let users: Vec<User> = sqlx::query_as("SELECT id, name, email FROM users")
        .fetch_all(&data.db)
        .await?;

    Ok(HttpResponse::Ok().json(users))
}

async fn create_user(
    body: web::Json<CreateUser>,
    data: web::Data<AppState>,
) -> Result<HttpResponse, sqlx::Error> {
    let result = sqlx::query(
        "INSERT INTO users (name, email) VALUES (?, ?)"
    )
    .bind(&body.name)
    .bind(&body.email)
    .execute(&data.db)
    .await?;

    Ok(HttpResponse::Created().json(serde_json::json!({
        "id": result.last_insert_id()
    })))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let pool = MySqlPoolOptions::new()
        .max_connections(5)
        .connect("mysql://user:password@localhost/database")
        .await
        .expect("无法连接数据库");

    let app_state = web::Data::new(AppState { db: pool });

    HttpServer::new(move || {
        App::new()
            .app_data(app_state.clone())
            .route("/users", web::get().to(get_users))
            .route("/users", web::post().to(create_user))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

## WebSocket 支持

```rust
use actix::{Actor, StreamHandler};
use actix_web::{web, App, Error, HttpRequest, HttpResponse, HttpServer};
use actix_web_actors::ws;

const HEARTBEAT_INTERVAL: std::time::Duration = std::time::Duration::from_secs(5);
const CLIENT_TIMEOUT: std::time::Duration = std::time::Duration::from_secs(30);

struct MyWs;

impl Actor for MyWs {
    type Context = ws::WebsocketContext<Self>;

    fn started(&mut self, ctx: &mut Self::Context) {
        self.hb(ctx);
    }
}

impl MyWs {
    fn hb(&self, ctx: &mut <Self as Actor>::Context) {
        ctx.run_interval(HEARTBEAT_INTERVAL, |act, ctx| {
            ctx.ping(b"ping");
            if Instant::now().duration_since(act.hb) > CLIENT_TIMEOUT {
                println!("WebSocket Client heartbeat failed, disconnecting!");
                ctx.stop();
            }
        });
    }
}

impl StreamHandler<Result<ws::Message, ws::ProtocolError>> for MyWs {
    fn handle(&mut self, msg: Result<ws::Message, ws::ProtocolError>, ctx: &mut Self::Context) {
        match msg {
            Ok(ws::Message::Ping(msg)) => ctx.pong(&msg),
            Ok(ws::Message::Text(text)) => {
                println!("Received: {}", text);
                ctx.text(format!("Echo: {}", text))
            }
            Ok(ws::Message::Binary(bin)) => ctx.binary(bin),
            Ok(ws::Message::Close(reason)) => ctx.close(reason),
            _ => ctx.stop(),
        }
    }
}

async fn ws_route(req: HttpRequest, stream: web::Payload) -> Result<HttpResponse, Error> {
    let ws = MyWs { hb: Instant::now() };
    ws::start(ws, &req, stream)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new().route("/ws", web::get().to(ws_route))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

## 测试

### 单元测试

```rust
use actix_web::{test, web, App};

#[cfg(test)]
mod tests {
    use super::*;

    #[actix_web::test]
    async fn test_index() {
        let app = test::init_service(
            App::new().route("/", web::get().to(|| async {
                actix_web::HttpResponse::Ok().body("Hello")
            }))
        ).await;

        let req = test::TestRequest::get().uri("/").to_request();
        let resp = test::call_service(&app, req).await;

        assert!(resp.status().is_success());
    }

    #[actix_web::test]
    async fn test_json_response() {
        let app = test::init_service(
            App::new().route("/user", web::get().to(|| async {
                actix_web::web::Json(serde_json::json!({
                    "id": 1,
                    "name": "张三"
                }))
            }))
        ).await;

        let req = test::TestRequest::get().uri("/user").to_request();
        let resp = test::call_service(&app, req).await;

        assert!(resp.status().is_success());
    }
}
```

### 集成测试

```rust
#[cfg(test)]
mod tests {
    use actix_web::{test, web, App, HttpResponse};

    async fn health() -> HttpResponse {
        HttpResponse::Ok().json(serde_json::json!({
            "status": "healthy"
        }))
    }

    #[actix_web::test]
    async fn test_health_endpoint() {
        let app = test::init_service(
            App::new().route("/health", web::get().to(health))
        ).await;

        let req = test::TestRequest::get()
            .uri("/health")
            .to_request();

        let resp = test::call_service(&app, req).await;
        assert!(resp.status().is_success());

        let body: serde_json::Value = test::read_body_json(resp).await;
        assert_eq!(body["status"], "healthy");
    }
}
```

## 最佳实践

### 1. 使用 `#[get]`、`#[post]` 等属性宏

```rust
use actix_web::{get, post, web, HttpResponse, Responder};

// 推荐使用宏定义路由
#[get("/users/{id}")]
async fn get_user(path: web::Path<u64>) -> impl Responder {
    HttpResponse::Ok().json(serde_json::json!({
        "id": *path
    }))
}

#[post("/users")]
async fn create_user(body: web::Json<serde_json::Value>) -> impl Responder {
    HttpResponse::Created().json(serde_json::json!({
        "status": "created"
    }))
}
```

### 2. 合理使用状态

```rust
use std::sync::Arc;
use tokio::sync::Mutex;

// 推荐使用 Arc<Mutex<T>> 或 Arc<RwLock<T>>
struct AppState {
    counter: Arc<Mutex<i32>>,
}

async fn increment(data: web::Data<Arc<Mutex<i32>>>) -> HttpResponse {
    let mut counter = data.lock().await;
    *counter += 1;
    HttpResponse::Ok().json(serde_json::json!({
        "counter": *counter
    }))
}
```

### 3. 配置管理

```rust
use serde::Deserialize;

#[derive(Deserialize, Clone)]
struct Config {
    host: String,
    port: u16,
    database_url: String,
}

impl Config {
    fn from_env() -> Result<Self, env_logger::Err> {
        Ok(Config {
            host: std::env::var("HOST").unwrap_or_else(|_| "127.0.0.1".to_string()),
            port: std::env::var("PORT")
                .unwrap_or_else(|_| "8080".to_string())
                .parse()
                .expect("PORT must be a number"),
            database_url: std::env::var("DATABASE_URL")
                .expect("DATABASE_URL must be set"),
        })
    }
}
```

## 总结

Actix 生态系统为 Rust 开发者提供了强大的工具集：

- **Actix**：Actor 模型实现，适合构建高并发系统
- **Actix-web**：高性能 Web 框架，支持完整的 HTTP 功能
- **丰富的中间件**：日志、压缩、认证、CORS 等
- **优秀的错误处理**：自定义错误类型和错误中间件
- **WebSocket 支持**：实时双向通信
- **完善的测试支持**：单元测试和集成测试

Actix 以其极致的性能和优雅的 API 设计，成为 Rust Web 开发的首选框架之一。无论是构建微服务、API 后端还是实时应用，Actix 都能提供出色的支持。

快乐编程，大家来 Rust! 🦀
