+++
title = "Axum 框架完全指南"
date = "2026-03-19T05:11:00+08:00"

[taxonomies]
tags = ["rust", "axum", "web", "框架", "教程", "tokio"]

[extra]
summary = "Axum 是 Tokio 团队开发的现代化 Rust Web 框架，本文详细介绍其核心概念、路由、提取器、中间件和实际应用。"
author = "博主"
+++

Axum 是由 Tokio 团队开发的现代化 Web 框架，专注于人体工程学和模块化设计。它构建在 Tokio、Hyper 和 Tower 之上，提供了优雅的 API 和出色的性能。本文将详细介绍 Axum 的核心概念和使用方法。

## Axum 概述

### 为什么选择 Axum？

- **基于 Tokio**：充分利用 Tokio 异步运行时
- **模块化设计**：使用 Tower 的 Service 和 Layer 抽象
- **人体工程学**：简洁的 API 设计
- **类型安全**：强大的编译时检查
- **高性能**：与 Hyper 相当的性能表现

### 项目设置

```toml
[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
tower = "0.4"
tower-http = { version = "0.5", features = ["trace", "cors"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
```

## 快速入门

### 第一个 Axum 应用

```rust
use axum::{
    routing::get,
    Router,
};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(handler));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    println!("Listening on http://{}", listener.local_addr().unwrap());
    axum::serve(listener, app).await.unwrap();
}

async fn handler() -> &'static str {
    "Hello, Axum!"
}
```

### 处理 JSON 请求和响应

```rust
use axum::{
    routing::{get, post},
    http::StatusCode,
    Json, Router,
};
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
struct CreateUser {
    username: String,
    email: String,
}

#[derive(Serialize)]
struct User {
    id: u64,
    username: String,
    email: String,
}

async fn create_user(
    Json(payload): Json<CreateUser>,
) -> (StatusCode, Json<User>) {
    let user = User {
        id: 1337,
        username: payload.username,
        email: payload.email,
    };

    (StatusCode::CREATED, Json(user))
}

async fn get_user() -> Json<User> {
    Json(User {
        id: 1,
        username: "alice".to_string(),
        email: "alice@example.com".to_string(),
    })
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/users", post(create_user))
        .route("/users/1", get(get_user));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

## 路由系统

### 基本路由

```rust
use axum::{
    routing::{get, post, put, delete},
    Router,
};

let app = Router::new()
    .route("/", get(index))
    .route("/users", get(list_users))
    .route("/users", post(create_user))
    .route("/users/:id", get(get_user))
    .route("/users/:id", put(update_user))
    .route("/users/:id", delete(delete_user));
```

### 路由方法

```rust
use axum::{
    routing::{get, post, put, patch, delete, head, options},
    Router,
};

let app = Router::new()
    .route("/resource", get(handler_get))
    .route("/resource", post(handler_post))
    .route("/resource", put(handler_put))
    .route("/resource", patch(handler_patch))
    .route("/resource", delete(handler_delete))
    .route("/resource", head(handler_head))
    .route("/resource", options(handler_options));
```

### 路径参数

```rust
use axum::{
    extract::Path,
    routing::get,
    Router,
};

async fn get_user(Path(user_id): Path<u64>) -> Json<User> {
    Json(User {
        id: user_id,
        username: "alice".to_string(),
    })
}

async fn get_user_posts(Path((user_id, post_id)): Path<(u64, u64)>) -> Json<Vec<Post>> {
    // 获取用户的特定文章
    Json(vec![])
}

let app = Router::new()
    .route("/users/:id", get(get_user))
    .route("/users/:user_id/posts/:post_id", get(get_user_posts));
```

### 嵌套路由

```rust
use axum::{
    extract::Path,
    routing::get,
    Router,
};

async fn list_user_articles(Path(user_id): Path<u64>) -> String {
    format!("Articles for user {}", user_id)
}

async fn get_article(
    Path((user_id, article_id)): Path<(u64, u64)>
) -> String {
    format!("Article {} by user {}", article_id, user_id)
}

let app = Router::new()
    .route("/users/:user_id/articles", get(list_user_articles))
    .route("/users/:user_id/articles/:article_id", get(get_article));
```

### 路由前缀 (nest)

```rust
use axum::{routing::get, Router};

async fn api_status() -> &'static str { "OK" }
async fn api_health() -> &'static str { "Healthy" }
async fn list_users() -> &'static str { "Users list" }
async fn get_user() -> &'static str { "User detail" }

let api_router = Router::new()
    .route("/status", get(api_status))
    .route("/health", get(api_health));

let users_router = Router::new()
    .route("/", get(list_users))
    .route("/:id", get(get_user));

let app = Router::new()
    .nest("/api", api_router)
    .nest("/users", users_router);
```

### 路由合并 (merge)

```rust
use axum::{routing::get, Router};

async fn public_handler() -> &'static str { "Public" }
async fn admin_handler() -> &'static str { "Admin" }

let public_routes = Router::new()
    .route("/about", get(public_handler));

let admin_routes = Router::new()
    .route("/dashboard", get(admin_handler));

let app = Router::new()
    .merge(public_routes)
    .merge(admin_routes);
```

## 提取器 (Extractors)

Axum 的核心特性之一是使用提取器从请求中获取数据。

### 常用提取器

```rust
use axum::{
    extract::{Path, Query, Json, Form, HeaderMap, Extension},
    routing::get,
    Router,
};
use serde::{Deserialize, Serialize;

// 路径参数
async fn handler(
    Path(id): Path<u64>,
    // 查询参数
    Query(params): Query<SearchParams>,
    // JSON body
    Json(data): Json<RequestData>,
    // Form data
    Form(form): Form<FormData>,
    // 请求头
    headers: HeaderMap,
    // 扩展数据
    Extension(state): Extension<AppState>,
) -> Json<Response> {
    // 处理请求
    Json(Response::default())
}

#[derive(Deserialize)]
struct SearchParams {
    q: Option<String>,
    page: Option<u32>,
    limit: Option<u32>,
}
```

### Query 提取器

```rust
use axum::{
    extract::Query,
    routing::get,
    Router,
};
use serde::Deserialize;

#[derive(Deserialize, Debug)]
struct Pagination {
    page: Option<u32>,
    per_page: Option<u32>,
}

#[derive(Deserialize, Debug)]
struct Filters {
    category: Option<String>,
    tag: Option<String>,
}

#[derive(Deserialize, Debug)]
struct SearchParams {
    #[serde(flatten)]
    pagination: Pagination,
    #[serde(flatten)]
    filters: Filters,
}

async fn search(Query(params): Query<SearchParams>) -> String {
    format!("Page: {:?}, Filters: {:?}", params.pagination, params.filters)
}

let app = Router::new()
    .route("/search", get(search));
```

### Form 提取器

```rust
use axum::{
    extract::Form,
    routing::post,
    Router,
};
use serde::Deserialize;

#[derive(Deserialize)]
struct LoginForm {
    username: String,
    password: String,
}

async fn login(Form(form): Form<LoginForm>) -> String {
    format!("Login: {}", form.username)
}

let app = Router::new()
    .route("/login", post(login));
```

### 多部分表单

```rust
use axum::{
    extract::Multipart,
    routing::post,
    Router,
};

async fn upload(mut multipart: Multipart) -> String {
    let mut filenames = Vec::new();

    while let Some(field) = multipart.next_field().await.unwrap() {
        let filename = field.file_name().unwrap_or("unknown").to_string();
        let data = field.bytes().await.unwrap();
        println!("Received file: {} ({} bytes)", filename, data.len());
        filenames.push(filename);
    }

    format!("Uploaded: {:?}", filenames)
}

let app = Router::new()
    .route("/upload", post(upload));
```

### HeaderMap 提取器

```rust
use axum::{
    extract::HeaderMap,
    routing::get,
    Router,
};

async fn get_headers(headers: HeaderMap) -> Json<Vec<(String, String)>> {
    let header_tuples: Vec<(String, String)> = headers
        .iter()
        .map(|(name, value)| {
            (
                name.to_string(),
                value.to_str().unwrap_or("").to_string(),
            )
        })
        .collect();

    Json(header_tuples)
}

async fn check_auth(headers: HeaderMap) -> Result<&'static str, StatusCode> {
    let auth = headers
        .get("Authorization")
        .and_then(|v| v.to_str().ok());

    match auth {
        Some(token) if token.starts_with("Bearer ") => Ok("Authorized"),
        _ => Err(StatusCode::UNAUTHORIZED),
    }
}

let app = Router::new()
    .route("/headers", get(get_headers))
    .route("/check-auth", get(check_auth));
```

### 自定义提取器

```rust
use axum::{
    extract::{FromRequestParts, State},
    http::StatusCode,
    routing::get,
    Router,
};
use async_trait::async_trait;

#[derive(Clone)]
struct AuthUser {
    id: u64,
    username: String,
}

#[async_trait]
impl<S> FromRequestParts<S> for AuthUser
where
    S: Send + Sync,
{
    type Rejection = StatusCode;

    async fn from_request_parts(
        parts: &mut Parts,
        _state: &S,
    ) -> Result<Self, Self::Rejection> {
        // 从请求头获取用户信息
        let auth_header = parts
            .headers
            .get("X-User-Id")
            .and_then(|v| v.to_str().ok())
            .and_then(|s| s.parse().ok());

        match auth_header {
            Some(id) => Ok(AuthUser {
                id,
                username: format!("User{}", id),
            }),
            None => Err(StatusCode::UNAUTHORIZED),
        }
    }
}

async fn profile(
    user: AuthUser,
) -> String {
    format!("Profile of {} (id: {})", user.username, user.id)
}

let app = Router::new()
    .route("/profile", get(profile));
```

## 状态管理

### 应用状态

```rust
use axum::{
    extract::State,
    routing::get,
    Router,
};
use std::sync::Arc;

// 定义状态结构
#[derive(Clone)]
struct AppState {
    db: DatabasePool,
    config: AppConfig,
}

#[derive(Clone)]
struct DatabasePool {
    connection_string: String,
}

#[derive(Clone)]
struct AppConfig {
    app_name: String,
    version: String,
}

async fn get_status(State(state): State<AppState>) -> Json<serde_json::Value> {
    Json(serde_json::json!({
        "app": state.config.app_name,
        "version": state.config.version,
        "db": state.db.connection_string,
    }))
}

#[tokio::main]
async fn main() {
    let app_state = AppState {
        db: DatabasePool {
            connection_string: "postgres://localhost/mydb".to_string(),
        },
        config: AppConfig {
            app_name: "MyApp".to_string(),
            version: "1.0.0".to_string(),
        },
    };

    let app = Router::new()
        .route("/status", get(get_status))
        .with_state(app_state);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

### 可变状态

```rust
use axum::{
    extract::State,
    routing::get,
    Router,
};
use std::sync::Arc;
use tokio::sync::RwLock;

#[derive(Clone)]
struct CounterState {
    counter: Arc<RwLock<u64>>,
}

async fn increment(State(state): State<CounterState>) -> String {
    let mut count = state.counter.write().await;
    *count += 1;
    format!("Counter: {}", *count)
}

async fn get_count(State(state): State<CounterState>) -> String {
    let count = state.counter.read().await;
    format!("Counter: {}", *count)
}

#[tokio::main]
async fn main() {
    let counter_state = CounterState {
        counter: Arc::new(RwLock::new(0)),
    };

    let app = Router::new()
        .route("/increment", get(increment))
        .route("/count", get(get_count))
        .with_state(counter_state);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

### Extension 提取器

```rust
use axum::{
    extract::{Extension, Path},
    routing::get,
    Router,
};
use std::sync::Arc;
use std::collections::HashMap;

#[derive(Clone)]
struct CurrentUser {
    id: u64,
    username: String,
}

// 模拟用户数据库
fn get_user_from_db(user_id: u64) -> Option<CurrentUser> {
    Some(CurrentUser {
        id: user_id,
        username: format!("User{}", user_id),
    })
}

async fn get_user_profile(
    Path(user_id): Path<u64>,
    Extension(user): Extension<Arc<CurrentUser>>,
) -> String {
    format!("Profile: {} (ID: {})", user.username, user.id)
}

// 在中间件或前置处理中设置 Extension
async fn auth_middleware(
    Path(user_id): Path<u64>,
    mut request: axum::extract::Request,
    next: axum::middleware::Next,
) -> axum::response::Response {
    if let Some(user) = get_user_from_db(user_id) {
        request.extensions_mut().insert(Arc::new(user));
    }
    next.run(request).await
}

let app = Router::new()
    .route("/users/:id/profile", get(get_user_profile))
    .layer(axum::middleware::from_fn(auth_middleware));
```

## 中间件

### 函数式中间件

```rust
use axum::{
    body::Body,
    extract::Request,
    http::StatusCode,
    middleware::{self, Next},
    response::Response,
    routing::get,
    Router,
};
use std::time::Instant;

// 日志中间件
async fn logging_middleware(request: Request, next: Next) -> Response {
    let method = request.method().clone();
    let uri = request.uri().clone();
    let start = Instant::now();

    let response = next.run(request).await;

    let duration = start.elapsed();
    println!("{} {} - {:?} - {}ms", method, uri, response.status(), duration.as_millis());

    response
}

// 认证中间件
async fn auth_middleware(
    request: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    let auth_header = request
        .headers()
        .get("Authorization")
        .and_then(|h| h.to_str().ok());

    match auth_header {
        Some(token) if token.starts_with("Bearer ") => {
            Ok(next.run(request).await)
        }
        _ => Err(StatusCode::UNAUTHORIZED),
    }
}

async fn public_handler() -> &'static str {
    "Public content"
}

async fn protected_handler() -> &'static str {
    "Protected content"
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/public", get(public_handler))
        .route("/protected", get(protected_handler))
        .route_layer(middleware::from_fn(auth_middleware))  // 只应用于 /protected
        .layer(middleware::from_fn(logging_middleware));    // 应用于所有路由

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

### 带状态的中间件

```rust
use axum::{
    extract::{Request, State},
    http::StatusCode,
    middleware::{self, Next},
    response::Response,
    routing::get,
    Router,
};
use std::sync::Arc;

#[derive(Clone)]
struct AppState {
    api_key: String,
}

// 带状态的认证中间件
async fn api_key_middleware(
    State(state): State<Arc<AppState>>,
    request: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    let api_key = request
        .headers()
        .get("X-API-Key")
        .and_then(|h| h.to_str().ok());

    match api_key {
        Some(key) if key == state.api_key => Ok(next.run(request).await),
        _ => Err(StatusCode::FORBIDDEN),
    }
}

async fn api_handler() -> &'static str {
    "API content"
}

#[tokio::main]
async fn main() {
    let state = Arc::new(AppState {
        api_key: "secret-key".to_string(),
    });

    let app = Router::new()
        .route("/api", get(api_handler))
        .layer(middleware::from_fn_with_state(state.clone(), api_key_middleware))
        .with_state(state);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

### 请求扩展

```rust
use axum::{
    extract::{Extension, Request},
    http::StatusCode,
    middleware::{self, Next},
    response::Response,
    routing::get,
    Router,
};
use std::sync::Arc;

#[derive(Clone)]
struct CurrentUser {
    id: u64,
    username: String,
}

#[derive(Clone)]
struct RequestId(String);

// 添加用户到请求扩展
async fn auth_middleware(
    mut request: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    // 模拟认证逻辑
    let user = CurrentUser {
        id: 1,
        username: "alice".to_string(),
    };
    request.extensions_mut().insert(Arc::new(user));
    Ok(next.run(request).await)
}

// 添加请求 ID
async fn request_id_middleware(
    mut request: Request,
    next: Next,
) -> Response {
    let id = uuid::Uuid::new_v4().to_string();
    request.extensions_mut().insert(RequestId(id));
    next.run(request).await
}

async fn profile(
    Extension(user): Extension<Arc<CurrentUser>>,
    Extension(RequestId(req_id)): Extension<RequestId>,
) -> String {
    format!("[{}] User: {} (ID: {})", req_id, user.username, user.id)
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/profile", get(profile))
        .layer(middleware::from_fn(auth_middleware))
        .layer(middleware::from_fn(request_id_middleware));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

## 错误处理

### 使用 Result 类型

```rust
use axum::{
    extract::Path,
    http::StatusCode,
    routing::get,
    Json, Router,
};
use serde::Serialize;
use thiserror::Error;

#[derive(Error, Debug)]
enum AppError {
    #[error("User not found")]
    NotFound,
    
    #[error("Invalid input: {0}")]
    BadRequest(String),
    
    #[error("Internal server error")]
    InternalError,
}

impl axum::response::IntoResponse for AppError {
    fn into_response(self) -> axum::response::Response {
        let (status, message) = match &self {
            AppError::NotFound => (StatusCode::NOT_FOUND, self.to_string()),
            AppError::BadRequest(msg) => (StatusCode::BAD_REQUEST, msg.clone()),
            AppError::InternalError => (StatusCode::INTERNAL_SERVER_ERROR, self.to_string()),
        };

        let body = Json(serde_json::json!({
            "error": message,
        }));

        (status, body).into_response()
    }
}

async fn get_user(Path(id): Path<u64>) -> Result<Json<User>, AppError> {
    if id == 0 {
        Err(AppError::NotFound)
    } else if id > 1000 {
        Err(AppError::BadRequest("Invalid user ID range".to_string()))
    } else {
        Ok(Json(User {
            id,
            username: "alice".to_string(),
        }))
    }
}

#[derive(Serialize)]
struct User {
    id: u64,
    username: String,
}

let app = Router::new()
    .route("/users/:id", get(get_user));
```

### 使用 ? 运算符

```rust
use axum::{
    extract::Path,
    routing::get,
    Json, Router,
};
use thiserror::Error;

#[derive(Error, Debug)]
enum AppError {
    #[error("Database error")]
    DatabaseError(#[from] sqlx::Error),
    
    #[error("User not found")]
    NotFound,
}

impl axum::response::IntoResponse for AppError {
    fn into_response(self) -> axum::response::Response {
        (StatusCode::INTERNAL_SERVER_ERROR, self.to_string()).into_response()
    }
}

async fn get_user(Path(id): Path<u64>) -> Result<Json<User>, AppError> {
    let user = find_user_in_db(id).await?;  // 使用 ? 自动转换错误
    Ok(Json(user))
}

async fn find_user_in_db(id: u64) -> Result<User, AppError> {
    // 数据库查询
    todo!()
}

let app = Router::new()
    .route("/users/:id", get(get_user));
```

## 数据库集成

### 与 sqlx 集成

```rust
use axum::{
    extract::State,
    routing::get,
    Json, Router,
};
use sqlx::{PgPool, PgPoolOptions};
use std::sync::Arc;

#[derive(Clone)]
struct AppState {
    db: PgPool,
}

#[derive(serde::Serialize, sqlx::FromRow)]
struct User {
    id: i64,
    name: String,
    email: String,
}

async fn list_users(
    State(state): State<Arc<AppState>>,
) -> Result<Json<Vec<User>>, sqlx::Error> {
    let users = sqlx::query_as::<_, User>("SELECT id, name, email FROM users")
        .fetch_all(&state.db)
        .await?;

    Ok(Json(users))
}

async fn create_user(
    State(state): State<Arc<AppState>>,
    Json(payload): Json<CreateUser>,
) -> Result<Json<User>, sqlx::Error> {
    let user = sqlx::query_as::<_, User>(
        "INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id, name, email"
    )
    .bind(&payload.name)
    .bind(&payload.email)
    .fetch_one(&state.db)
    .await?;

    Ok(Json(user))
}

#[derive(serde::Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect("postgres://user:password@localhost/mydb")
        .await?;

    let app_state = Arc::new(AppState { db: pool });

    let app = Router::new()
        .route("/users", get(list_users))
        .route("/users", post(create_user))
        .with_state(app_state);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await?;
    axum::serve(listener, app).await?;

    Ok(())
}
```

## Tower HTTP 中间件

### Trace 中间件

```rust
use axum::{
    routing::get,
    Router,
};
use tower_http::trace::TraceLayer;
use tracing_subscriber;

let app = Router::new()
    .route("/", get(handler))
    .layer(
        TraceLayer::new_for_http()
            .on_request(|request, _| {
                tracing::info!("Started {} {}", request.method(), request.uri());
            })
            .on_response(|response, latency| {
                tracing::info!("Completed with {} in {:?}", response.status(), latency);
            })
            .on_failure(|error, _| {
                tracing::error!("Request failed: {}", error);
            }),
    );

tracing_subscriber::fmt()
    .with_target(false)
    .init();

async fn handler() -> &'static str {
    "Hello, Trace!"
}
```

### CORS 中间件

```rust
use axum::{
    routing::get,
    Router,
};
use tower_http::cors::{CorsLayer, Any, AllowCredentials};

let app = Router::new()
    .route("/", get(handler))
    .layer(
        CorsLayer::new()
            .allow_origin(Any)
            .allow_methods(Any)
            .allow_headers(Any)
            .allow_credentialss(true),
    );

async fn handler() -> &'static str {
    "Hello, CORS!"
}
```

### Compression 中间件

```rust
use axum::{
    routing::get,
    Router,
};
use tower_http::compression::CompressionLayer;

let app = Router::new()
    .route("/", get(handler))
    .layer(CompressionLayer::new());

async fn handler() -> String {
    "A".repeat(10000)  // 会被压缩
}
```

## WebSocket 支持

```rust
use axum::{
    extract::ws::{Message, WebSocket, WebSocketUpgrade},
    routing::get,
    response::Response,
    Router,
};
use futures_util::{SinkExt, StreamExt};

async fn websocket_handler(
    ws: WebSocketUpgrade,
) -> Response {
    ws.on_upgrade(handle_socket)
}

async fn handle_socket(socket: WebSocket) {
    let (mut sender, mut receiver) = socket.split();

    while let Some(msg) = receiver.next().await {
        let msg = msg.unwrap();
        if msg.is_text() {
            println!("Received: {:?}", msg);
            let _ = sender.send(Message::Text("Echo: ".to_string() + msg.to_str().unwrap())).await;
        } else if msg.is_close() {
            break;
        }
    }
}

let app = Router::new()
    .route("/ws", get(websocket_handler));
```

## 测试

### 单元测试

```rust
#[cfg(test)]
mod tests {
    use axum::{
        body::Body,
        routing::get,
        Router,
    };
    use http::Request;
    use tower::ServiceExt;

    async fn handler() -> &'static str {
        "Hello, Test!"
    }

    #[tokio::test]
    async fn test_route() {
        let app = Router::new()
            .route("/test", get(handler));

        let response = app
            .oneshot(
                Request::builder()
                    .uri("/test")
                    .body(Body::empty())
                    .unwrap(),
            )
            .await
            .unwrap();

        assert_eq!(response.status(), StatusCode::OK);
    }

    #[tokio::test]
    async fn test_not_found() {
        let app = Router::new()
            .route("/exists", get(handler));

        let response = app
            .oneshot(
                Request::builder()
                    .uri("/not-found")
                    .body(Body::empty())
                    .unwrap(),
            )
            .await
            .unwrap();

        assert_eq!(response.status(), StatusCode::NOT_FOUND);
    }
}
```

### 集成测试

```rust
#[cfg(test)]
mod tests {
    use axum::{
        body::Body,
        routing::{get, post},
        extract::Json,
        Router,
    };
    use http::Request;
    use tower::ServiceExt;
    use serde::{Deserialize, Serialize};

    #[derive(Deserialize, Serialize)]
    struct CreateUser {
        name: String,
    }

    async fn create_user(
        Json(payload): Json<CreateUser>,
    ) -> &'static str {
        "Created"
    }

    #[tokio::test]
    async fn test_create_user() {
        let app = Router::new()
            .route("/users", post(create_user));

        let response = app
            .oneshot(
                Request::builder()
                    .method("POST")
                    .uri("/users")
                    .header("Content-Type", "application/json")
                    .body(Body::from(r#"{"name":"Alice"}"#))
                    .unwrap(),
            )
            .await
            .unwrap();

        assert_eq!(response.status(), StatusCode::OK);
    }
}
```

## 最佳实践

### 1. 使用结构化路由

```rust
// 好：将相关路由组织在一起
fn users_router() -> Router {
    Router::new()
        .route("/", get(list_users))
        .route("/", post(create_user))
        .route("/:id", get(get_user))
        .route("/:id", put(update_user))
        .route("/:id", delete(delete_user))
}

fn articles_router() -> Router {
    Router::new()
        .route("/", get(list_articles))
        .route("/", post(create_article))
}

let app = Router::new()
    .nest("/users", users_router())
    .nest("/articles", articles_router());
```

### 2. 统一错误处理

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("Not found: {0}")]
    NotFound(String),
    
    #[error("Validation error: {0}")]
    Validation(String),
    
    #[error(transparent)]
    Internal(#[from] anyhow::Error),
}

impl axum::response::IntoResponse for AppError {
    fn into_response(self) -> axum::response::Response {
        let (status, message) = match &self {
            AppError::NotFound(msg) => (StatusCode::NOT_FOUND, msg.clone()),
            AppError::Validation(msg) => (StatusCode::BAD_REQUEST, msg.clone()),
            AppError::Internal(_) => (StatusCode::INTERNAL_SERVER_ERROR, self.to_string()),
        };

        (status, Json(serde_json::json!({
            "error": message,
        }))).into_response()
    }
}
```

### 3. 使用日志和追踪

```rust
use tower_http::trace::TraceLayer;
use tracing_subscriber::{layer::SubscriberExt, util::SubscriberInitExt};

#[tokio::main]
async fn main() {
    tracing_subscriber::registry()
        .with(EnvFilter::try_from_default_env().unwrap_or_else(|_| "example_todos=debug,tower_http=debug".into()))
        .with(tracing_subscriber::fmt::layer())
        .init();

    // ... 应用代码
}
```

### 4. 健康检查端点

```rust
use axum::{
    extract::State,
    routing::get,
    Json, Router,
};
use sqlx::PgPool;
use std::sync::Arc;

#[derive(Clone)]
struct HealthState {
    db: PgPool,
}

async fn health_check(
    State(state): State<Arc<HealthState>>,
) -> Json<serde_json::Value> {
    // 检查数据库连接
    let db_healthy = sqlx::query("SELECT 1")
        .fetch_one(&state.db)
        .await
        .is_ok();

    let status = if db_healthy { "healthy" } else { "unhealthy" };
    let status_code = if db_healthy { StatusCode::OK } else { StatusCode::SERVICE_UNAVAILABLE };

    (status_code, Json(serde_json::json!({
        "status": status,
        "checks": {
            "database": if db_healthy { "ok" } else { "failed" }
        }
    }))).into_response()
}

let app = Router::new()
    .route("/health", get(health_check));
```

## 总结

Axum 是 Rust 生态中最现代和灵活的 Web 框架之一：

**核心特性：**
- **提取器系统**：简洁地获取请求数据
- **模块化路由**：支持 nest、merge 等组合方式
- **Tower 集成**：充分利用中间件生态
- **类型安全**：编译时检查减少运行时错误

**关键组件：**
- **Router**：路由定义和管理
- **Layer**：中间件组合
- **State**：应用状态管理
- **Extractors**：请求数据提取

**实用工具：**
- **tower-http**：Trace、CORS、Compression 等中间件
- **serde**：JSON 序列化/反序列化
- **sqlx**：异步数据库访问
- **tracing**：结构化日志和追踪

**最佳实践：**
- 使用结构化路由组织代码
- 统一错误处理
- 添加健康检查端点
- 启用日志和追踪

掌握 Axum，你将能够构建高性能、结构清晰的 Rust Web 应用！

快乐编程，大家来 Rust! 🦀
