+++
title = "Tower 库完全指南：Service 与 Layer 架构"
date = "2026-03-19T05:06:00+08:00"

[taxonomies]
tags = ["rust", "tower", "middleware", "service", "web", "框架"]
categories = ["编程"]

[extra]
summary = "Tower 是 Rust 生态中重要的中间件库，本文详细介绍其 Service 和 Layer 核心抽象，以及如何构建可组合的中间件系统。"
author = "博主"
+++

Tower 是 Rust 生态中用于构建健壮网络客户端和服务器的核心库。它提供了两个关键抽象：`Service`（异步请求/响应处理）和 `Layer`（中间件组合）。Axum、Hyper 等流行框架都基于 Tower 构建。理解 Tower 的架构对于深入掌握现代 Rust Web 开发至关重要。

## Tower 概述

### 什么是 Tower？

Tower 的核心设计理念是：

```rust
// async fn(Request) -> Result<Response, Error>
```

这是一个简洁而强大的抽象，代表了一个异步函数，接受请求并返回响应或错误。

### Tower 生态系统

Tower 生态系统由以下 crate 组成：

- **tower**：核心库，提供 Service 和 Layer trait
- **tower-service**：Service trait 定义
- **tower-layer**：Layer trait 定义
- **tower-test**：测试工具
- **tower-http**：HTTP 特定的中间件

### 为什么使用 Tower？

1. **模块化**：每个中间件都是独立的可组合组件
2. **协议无关**：Service trait 可以建模各种网络协议
3. **可复用**：社区提供的丰富中间件库
4. **类型安全**：充分利用 Rust 的类型系统

## 核心抽象：Service Trait

### Service Trait 定义

```rust
use std::future::Future;
use std::task::{Context, Poll};

pub trait Service<Request> {
    /// 响应的未来类型
    type Response;
    
    /// 可能发生的错误类型
    type Error;
    
    /// 响应 Future 类型
    type Future: Future<Output = Result<Self::Response, Self::Error>>;

    /// 检查服务是否准备好处理请求
    fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>>;

    /// 处理请求并返回响应
    fn call(&mut self, req: Request) -> Self::Future;
}
```

### 实现第一个 Service

```rust
use std::future::{Ready, ready};
use std::task::{Context, Poll};
use tower::{Layer, Service};
use http::{Request, Response};
use http_body::Body;

pub struct HelloService;

impl Service<Request<hyper::Body>> for HelloService {
    type Response = Response<String>;
    type Error = std::convert::Infallible;
    type Future = Ready<Result<Self::Response, Self::Error>>;

    fn poll_ready(&mut self, _cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
        Poll::Ready(Ok(()))
    }

    fn call(&mut self, _req: Request<hyper::Body>) -> Self::Future {
        let response = Response::builder()
            .status(200)
            .body("Hello, Tower!".to_string())
            .unwrap();

        ready(Ok(response))
    }
}
```

### Service 方法

```rust
impl<S, Request> Service<Request> for S
where
    S: tower::Service<Request>,
{
    type Response = S::Response;
    type Error = S::Error;
    type Future = S::Future;

    fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
        self.poll_ready(cx)
    }

    fn call(&mut self, req: Request) -> Self::Future {
        self.call(req)
    }
}
```

### Service 的状态管理

```rust
use std::sync::{Arc, Mutex};
use std::future::{Ready, ready};
use std::task::{Context, Poll};
use tower::{Service, Layer};
use http::{Request, Response};

pub struct CounterService {
    counter: Arc<Mutex<usize>>,
}

impl CounterService {
    pub fn new() -> Self {
        CounterService {
            counter: Arc::new(Mutex::new(0)),
        }
    }
}

impl Service<Request<()>> for CounterService {
    type Response = Response<String>;
    type Error = std::convert::Infallible;
    type Future = Ready<Result<Self::Response, Self::Error>>;

    fn poll_ready(&mut self, _cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
        Poll::Ready(Ok(()))
    }

    fn call(&mut self, _req: Request<()>) -> Self::Future {
        let mut counter = self.counter.lock().unwrap();
        *counter += 1;
        let count = *counter;

        let response = Response::builder()
            .status(200)
            .body(format!("Request count: {}", count))
            .unwrap();

        ready(Ok(response))
    }
}
```

## 核心抽象：Layer Trait

### Layer Trait 定义

```rust
pub trait Layer<S> {
    /// 应用此层后产生的 Service 类型
    type Service;

    /// 将 Layer 应用到 Service 上
    fn layer(&self, inner: S) -> Self::Service;
}
```

### 实现第一个 Layer

```rust
use std::future::{Future, ready};
use std::task::{Context, Poll};
use std::pin::Pin;
use tower::{Service, Layer};
use http::{Request, Response};

#[derive(Clone)]
pub struct TimingLayer;

impl<S> Layer<S> for TimingLayer {
    type Service = TimingService<S>;

    fn layer(&self, inner: S) -> Self::Service {
        TimingService { inner }
    }
}

#[derive(Clone)]
pub struct TimingService<S> {
    inner: S,
}

impl<S, B> Service<Request<B>> for TimingService<S>
where
    S: Service<Request<B>, Response = Response<String>> + Clone + Send + 'static,
    S::Future: Send + 'static,
    B: Send + 'static,
{
    type Response = Response<String>;
    type Error = S::Error;
    type Future = Pin<Box<dyn Future<Output = Result<Self::Response, Self::Error>> + Send>>;

    fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
        self.inner.poll_ready(cx)
    }

    fn call(&mut self, req: Request<B>) -> Self::Future {
        let start = std::time::Instant::now();
        let mut inner = self.inner.clone();

        Box::pin(async move {
            let mut res = inner.call(req).await?;
            let elapsed = start.elapsed();

            println!("Request took: {:?}", elapsed);

            Ok(res)
        })
    }
}
```

### 组合多个 Layer

```rust
use tower::ServiceBuilder;

let service = ServiceBuilder::new()
    .layer(TimingLayer)
    .layer(LoggingLayer)
    .layer(AuthLayer)
    .service(HelloService)
    .clone();
```

## 内置中间件

### Timeout 中间件

```rust
use tower::timeout::Timeout;
use std::time::Duration;

let service = Timeout::new(
    my_service,
    Duration::from_secs(30)
);
```

### Retry 中间件

```rust
use tower::retry::Retry;
use tower::retry::Policy;
use std::sync::Arc;

// 定义重试策略
#[derive(Clone)]
struct RetryPolicy;

impl<B, E> Policy<http::Request<B>, Response<String>, E> for RetryPolicy
where
    B: Send + 'static,
    E: std::fmt::Debug + Send + 'static,
{
    // ... 实现 Policy trait
}

let service = Retry::new(
    RetryPolicy,
    my_service,
);
```

### Rate Limit 中间件

```rust
use tower::limit::RateLimit;
use std::num::NonZeroU64;

// 每秒最多 100 个请求
let service = RateLimit::new(
    my_service,
    NonZeroU64::new(100).unwrap(),
);
```

### Load Shedding 中间件

```rust
use tower::load_shed::LoadShed;

let service = LoadShed::new(my_service);
```

### Buffer 中间件

```rust
use tower::buffer::Buffer;
use std::sync::Arc;

let service = Buffer::new(
    my_service,
    256,  // 缓冲区大小
).unwrap();
```

## 实际应用示例

### 认证中间件

```rust
use std::future::{Future, ready};
use std::pin::Pin;
use std::task::{Context, Poll};
use tower::{Layer, Service};
use http::{Request, Response, StatusCode};
use http_body::Body;

#[derive(Clone)]
pub struct AuthLayer {
    expected_token: String,
}

impl AuthLayer {
    pub fn new(expected_token: impl Into<String>) -> Self {
        Self {
            expected_token: expected_token.into(),
        }
    }
}

impl<S> Layer<S> for AuthLayer {
    type Service = AuthMiddleware<S>;

    fn layer(&self, inner: S) -> Self::Service {
        AuthMiddleware {
            inner,
            expected_token: self.expected_token.clone(),
        }
    }
}

#[derive(Clone)]
pub struct AuthMiddleware<S> {
    inner: S,
    expected_token: String,
}

impl<S, B, ResB, ResE> Service<Request<B>> for AuthMiddleware<S>
where
    S: Service<Request<B>, Response = Response<ResB>, Error = ResE> + Clone + Send + 'static,
    S::Future: Send + 'static,
    B: Send + 'static,
    ResB: Body::Data + 'static,
    ResE: Send + 'static,
{
    type Response = Response<ResB>;
    type Error = ResE;
    type Future = Pin<Box<dyn Future<Output = Result<Self::Response, Self::Error>> + Send>>;

    fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
        self.inner.poll_ready(cx)
    }

    fn call(&mut self, req: Request<B>) -> Self::Future {
        // 检查 Authorization 头
        let auth_header = req
            .headers()
            .get("Authorization")
            .and_then(|v| v.to_str().ok())
            .map(|s| s.to_string());

        let is_authorized = auth_header
            .as_ref()
            .map(|token| token.strip_prefix("Bearer ").unwrap_or(token))
            .map(|token| token == &self.expected_token)
            .unwrap_or(false);

        let mut inner = self.inner.clone();

        if is_authorized {
            Box::pin(async move {
                inner.call(req).await
            })
        } else {
            let response = Response::builder()
                .status(StatusCode::UNAUTHORIZED)
                .body("Unauthorized".into())
                .unwrap();

            Box::pin(ready(Ok(response)))
        }
    }
}
```

### 日志中间件

```rust
use std::future::{Future, ready};
use std::pin::Pin;
use std::task::{Context, Poll};
use tower::{Layer, Service};
use http::Request;

#[derive(Clone)]
pub struct LoggingLayer;

impl<S> Layer<S> for LoggingLayer {
    type Service = LoggingService<S>;

    fn layer(&self, inner: S) -> Self::Service {
        LoggingService { inner }
    }
}

#[derive(Clone)]
pub struct LoggingService<S> {
    inner: S,
}

impl<S, B, ResB, ResE> Service<Request<B>> for LoggingService<S>
where
    S: Service<Request<B>, Response = Response<ResB>, Error = ResE> + Clone + Send + 'static,
    S::Future: Send + 'static,
    B: Send + 'static,
    ResB: Default + Send + 'static,
    ResE: Send + 'static,
{
    type Response = Response<ResB>;
    type Error = ResE;
    type Future = Pin<Box<dyn Future<Output = Result<Self::Response, Self::Error>> + Send>>;

    fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
        self.inner.poll_ready(cx)
    }

    fn call(&mut self, req: Request<B>) -> Self::Future {
        let method = req.method().clone();
        let uri = req.uri().clone();
        let start = std::time::Instant::now();
        let mut inner = self.inner.clone();

        Box::pin(async move {
            println!("--> {} {}", method, uri);
            
            let result = inner.call(req).await;
            
            let elapsed = start.elapsed();
            let status = match &result {
                Ok(res) => res.status().as_u16(),
                Err(_) => 500,
            };
            
            println!("<-- {} {} ({:?})", status, uri, elapsed);
            
            result
        })
    }
}
```

### 请求ID中间件

```rust
use std::future::{Future, ready};
use std::pin::Pin;
use std::sync::atomic::{AtomicU64, Ordering};
use std::task::{Context, Poll};
use tower::{Layer, Service};
use http::{Request, header};
use http_body::Body;

static REQUEST_ID_COUNTER: AtomicU64 = AtomicU64::new(0);

#[derive(Clone)]
pub struct RequestIdLayer;

impl<S> Layer<S> for RequestIdLayer {
    type Service = RequestIdService<S>;

    fn layer(&self, inner: S) -> Self::Service {
        RequestIdService { inner }
    }
}

pub struct RequestIdService<S> {
    inner: S,
}

impl<S, B, ResB, ResE> Service<Request<B>> for RequestIdService<S>
where
    S: Service<Request<B>, Response = Response<ResB>, Error = ResE> + Clone + Send + 'static,
    S::Future: Send + 'static,
    B: Send + 'static,
    ResB: Default + Send + 'static,
    ResE: Send + 'static,
{
    type Response = Response<ResB>;
    type Error = ResE;
    type Future = Pin<Box<dyn Future<Output = Result<Self::Response, Self::Error>> + Send>>;

    fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
        self.inner.poll_ready(cx)
    }

    fn call(&mut self, mut req: Request<B>) -> Self::Future {
        // 生成请求 ID
        let request_id = REQUEST_ID_COUNTER.fetch_add(1, Ordering::SeqCst);
        
        // 添加到请求头
        req.headers_mut().insert(
            header::HeaderName::from_static("x-request-id"),
            header::HeaderValue::from(format!("{}", request_id).parse().unwrap()),
        );

        let mut inner = self.inner.clone();

        Box::pin(async move {
            inner.call(req).await
        })
    }
}
```

## ServiceBuilder

### 基本用法

```rust
use tower::ServiceBuilder;

let service = ServiceBuilder::new()
    .rate_limit(100, std::time::Duration::from_secs(1))
    .concurrency_limit(256)
    .timeout(std::time::Duration::from_secs(30))
    .service(my_service);
```

### 组合多个中间件

```rust
use tower::ServiceBuilder;
use tower::limit::RateLimit;
use tower::timeout::Timeout;

let service = ServiceBuilder::new()
    // 添加认证
    .layer(AuthLayer::new("secret-token"))
    // 添加日志
    .layer(LoggingLayer)
    // 添加请求 ID
    .layer(RequestIdLayer)
    // 限制速率
    .rate_limit(100, Duration::from_secs(1))
    // 添加超时
    .timeout(Duration::from_secs(30))
    // 应用目标服务
    .service(MyService);
```

### 自定义中间件组合

```rust
use tower::ServiceBuilder;

pub fn with_middleware<S>(service: S) -> impl Service<Request<()>>
where
    S: Service<Request<()>> + Clone,
{
    ServiceBuilder::new()
        .layer(TimingLayer)
        .layer(LoggingLayer)
        .layer(AuthLayer::new("token"))
        .service(service)
}
```

## 与 Axum 集成

### 在 Axum 中使用 Tower 中间件

```rust
use axum::{
    Router,
    routing::get,
    body::Body,
    http::{Request, StatusCode},
};
use tower::{Layer, ServiceExt};
use tower_http::trace::TraceLayer;
use std::net::SocketAddr;

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(handler))
        .layer(TraceLayer::new_for_http());

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("Listening on {}", addr);

    let listener = tokio::net::TcpListener::bind(addr).await.unwrap();
    axum::serve(listener, app).await.unwrap();
}

async fn handler() -> &'static str {
    "Hello, Tower!"
}
```

### 自定义 Tower 中间件在 Axum 中

```rust
use axum::{
    Router,
    routing::get,
    extract::Request,
    response::Response,
    body::Body,
};
use tower::{Service, Layer};
use std::future::{Ready, ready};
use std::task::{Context, Poll};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(handler))
        .layer(MyTowerLayer::new());

    // 启动服务...
}

async fn handler() -> &'static str {
    "Hello, Tower Middleware!"
}

#[derive(Clone)]
pub struct MyTowerLayer;

impl MyTowerLayer {
    pub fn new() -> Self {
        Self
    }
}

impl<S> Layer<S> for MyTowerLayer {
    type Service = MyTowerMiddleware<S>;

    fn layer(&self, inner: S) -> Self::Service {
        MyTowerMiddleware { inner }
    }
}

#[derive(Clone)]
pub struct MyTowerMiddleware<S> {
    inner: S,
}

impl<S> Service<Request<Body>> for MyTowerMiddleware<S>
where
    S: Service<Request<Body>, Response = Response> + Clone + Send + 'static,
    S::Future: Send + 'static,
{
    type Response = Response;
    type Error = S::Error;
    type Future = S::Future;

    fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
        self.inner.poll_ready(cx)
    }

    fn call(&mut self, mut req: Request<Body>) -> Self::Future {
        println!("Processing request: {} {}", req.method(), req.uri());
        self.inner.call(req)
    }
}
```

## Tower HTTP

### Trace 中间件

```rust
use tower_http::trace::TraceLayer;
use tracing_subscriber;

let service = TraceLayer::new_for_http()
    .on_request(|request, _| {
        tracing::info!("Started {} {}", request.method(), request.uri());
    })
    .on_response(|response, latency| {
        tracing::info!("Completed with {} in {:?}", response.status(), latency);
    })
    .on_failure(|error, _| {
        tracing::error!("Request failed: {}", error);
    });
```

### CORS 中间件

```rust
use tower_http::cors::{CorsLayer, Any};

let cors = CorsLayer::new()
    .allow_origin(Any)
    .allow_methods(Any)
    .allow_headers(Any);

let service = my_service.layer(cors);
```

### Compression 中间件

```rust
use tower_http::compression::CompressionLayer;

let service = CompressionLayer::new()
    .gzip(true, 12)
    .deflate(true);
```

### Sensitive Headers

```rust
use tower_http::set_header::SetResponseHeaderLayer;
use http::{header::AUTHORIZATION, header::HeaderValue};

let service = my_service
    .layer(SetResponseHeaderLayer::if_present(
        AUTHORIZATION,
        HeaderValue::from_static("****"),
    ));
```

## 测试

### 单元测试

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use http::Request;
    use std::convert::Infallible;

    #[tokio::test]
    async fn test_auth_middleware_authorized() {
        let mut service = AuthMiddleware {
            inner: PassThroughService,
            expected_token: "secret".to_string(),
        };

        let req = Request::builder()
            .header("Authorization", "Bearer secret")
            .body(())
            .unwrap();

        let res = service.call(req).await.unwrap();
        assert_eq!(res.status(), StatusCode::OK);
    }

    #[tokio::test]
    async fn test_auth_middleware_unauthorized() {
        let mut service = AuthMiddleware {
            inner: PassThroughService,
            expected_token: "secret".to_string(),
        };

        let req = Request::builder()
            .header("Authorization", "Bearer wrong")
            .body(())
            .unwrap();

        let res = service.call(req).await.unwrap();
        assert_eq!(res.status(), StatusCode::UNAUTHORIZED);
    }

    // 用于测试的穿透服务
    struct PassThroughService;

    impl Service<Request<()>> for PassThroughService {
        type Response = Response<String>;
        type Error = Infallible;
        type Future = Ready<Result<Self::Response, Self::Error>>;

        fn poll_ready(&mut self, _: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
            Poll::Ready(Ok(()))
        }

        fn call(&mut self, _: Request<()>) -> Self::Future {
            std::future::ready(Ok(Response::builder()
                .status(200)
                .body("OK".to_string())
                .unwrap()))
        }
    }
}
```

### 使用 Mock 进行测试

```rust
use tower_test::{mock, Mock};

#[tokio::test]
async fn test_with_mock() {
    let (mock_service, handle) = Mock::new();
    
    // 设置期望
    handle.allow(0);  // 允许 0 个请求
    
    let mut layered = MyLayer.layer(mock_service);
    
    // 测试逻辑...
}
```

## 最佳实践

### 1. 保持中间件简单

```rust
// 好：单一职责
struct RateLimitLayer { /* ... */ }
struct AuthLayer { /* ... */ }
struct LoggingLayer { /* ... */ }

// 不好：承担太多职责
struct MegaLayer { /* ... */ }  // 做了所有事情
```

### 2. 使用 Clone 实现可复制

```rust
#[derive(Clone)]
pub struct MyLayer;

impl<S> Layer<S> for MyLayer {
    type Service = MyMiddleware<S>;
    fn layer(&self, inner: S) -> Self::Service {
        MyMiddleware { inner }
    }
}
```

### 3. 正确实现 poll_ready

```rust
fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
    // 确保内部服务准备好
    self.inner.poll_ready(cx)
}
```

### 4. 处理错误

```rust
fn call(&mut self, req: Request) -> Self::Future {
    let mut inner = self.inner.clone();

    Box::pin(async move {
        inner.call(req).await
    })
}
```

### 5. 中间件顺序

```rust
ServiceBuilder::new()
    // 1. 先处理请求 ID（最先，记录所有请求）
    .layer(RequestIdLayer)
    // 2. 日志记录
    .layer(LoggingLayer)
    // 3. 认证（可能拒绝请求）
    .layer(AuthLayer)
    // 4. 限流（在认证之后）
    .rate_limit(100, Duration::from_secs(1))
    // 5. 目标服务
    .service(MyService)
```

## 总结

Tower 是 Rust 网络编程的核心基础设施：

**核心抽象：**
- **Service**：`async fn(Request) -> Result<Response, Error>`
- **Layer**：装饰 Service 产生新的 Service

**关键特性：**
- 模块化：每个中间件独立可组合
- 协议无关：适用于各种网络协议
- 类型安全：充分利用 Rust 类型系统
- 丰富的生态：tower-http、tower-go、tower-grpc 等

**实际应用：**
- 认证和授权
- 日志和追踪
- 限流和熔断
- 请求 ID 和调试
- CORS 和压缩

**最佳实践：**
- 保持中间件简单和单一职责
- 正确实现 poll_ready
- 注意中间件顺序
- 编写测试确保正确性

掌握 Tower，你将能够构建模块化、可维护的现代网络应用！

快乐编程，大家来 Rust! 🦀
