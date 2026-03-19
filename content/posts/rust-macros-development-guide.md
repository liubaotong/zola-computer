+++
title = "Rust 宏开发完全指南"
date = "2026-03-19T04:52:00+08:00"

[taxonomies]
tags = ["rust", "宏", "metaprogramming", "proc-macro", "教程"]
categories = ["编程"]

[extra]
summary = "Rust 的宏系统是其最强大的特性之一，本文详细介绍声明式宏和过程宏的开发方法，帮助你掌握编译时代码生成技术。"
author = "博主"
+++

Rust 的宏系统是其最强大的特性之一，能够在编译时生成代码，大大减少样板代码并提高代码的可维护性。本文将深入介绍 Rust 宏开发的各个方面，包括声明式宏（`macro_rules!`）和过程宏（Procedural Macros）。

## 宏概述

### 什么是宏？

宏是一种在编译时执行代码生成的技术。与函数不同：
- **函数**：复用**行为**
- **宏**：复用**语法和结构**

宏接受 Rust 代码作为输入，返回新的 Rust 代码作为输出。这使得我们可以自动化编写重复性的代码。

### Rust 宏的两种类型

1. **声明式宏（Declarative Macros）**：使用 `macro_rules!` 定义，基于模式匹配
2. **过程宏（Procedural Macros）**：像函数一样工作，操作 TokenStream

## 声明式宏

声明式宏使用 `macro_rules!` 定义，是 Rust 中最基本的宏形式。

### 基本语法

```rust
macro_rules! my_macro {
    () => {
        // 空参数匹配
    };
    ($val:expr) => {
        // 匹配表达式
    };
}
```

### 第一个声明式宏

```rust
macro_rules! say_hello {
    () => {
        println!("Hello, World!");
    };
}

fn main() {
    say_hello!();
}
```

### 模式匹配

```rust
macro_rules! print_if_true {
    ($condition:expr, $message:expr) => {
        if $condition {
            println!("{}", $message);
        }
    };
}

fn main() {
    print_if_true!(true, "Condition was true!");
    print_if_true!(false, "This won't print");
}
```

### 重复模式

`$()*` 用于匹配零个或多个，`$()+` 用于一个或多个：

```rust
macro_rules! vec_of_strings {
    ($($val:expr),*) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push(format!("{:?}", $val));
            )*
            temp_vec
        }
    };
}

fn main() {
    let v = vec_of_strings!(1, "hello", 3.14, true);
    println!("{:?}", v);
}
```

### 生成多个函数

```rust
macro_rules! define_functions {
    ($($name:ident),*) => {
        $(
            fn $name() {
                println!(stringify!($name));
            }
        )*
    };
}

define_functions!(foo, bar, baz);

fn main() {
    foo();
    bar();
    baz();
}
```

### 哈希映射宏

```rust
use std::collections::HashMap;

macro_rules! hash_map {
    ($($key:expr => $value:expr),* $(,)?) => {
        {
            let mut map = HashMap::new();
            $(
                map.insert($key, $value);
            )*
            map
        }
    };
}

fn main() {
    let scores = hash_map! {
        "Alice" => 100,
        "Bob" => 85,
        "Charlie" => 92,
    };

    println!("{:?}", scores);
}
```

## 过程宏基础

过程宏是 Rust 中更强大的宏形式，它们像函数一样操作代码的 TokenStream。

### 三种过程宏类型

1. **derive 宏**：`#[derive(...)]` - 为结构体和枚举自动实现 trait
2. **属性宏**：`#[attribute]` - 修改代码项的行为
3. **函数式宏**：`macro!()` - 像函数调用一样使用

### 项目设置

过程宏必须定义在独立的 crate 中：

```toml
# Cargo.toml for the macro crate
[package]
name = "my_macro_derive"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true

[dependencies]
syn = { version = "2.0", features = ["full", "extra-traits"] }
quote = "1.0"
proc-macro2 = "1.0"
darling = "0.20"  # 可选：简化属性解析
```

### 核心依赖说明

- **proc_macro2**：`proc_macro` crate 的 wrapper，提供兼容的 TokenStream 类型
- **syn**：解析 Rust 代码为 AST，提供可操作的结构
- **quote**：使用模板语法生成 Rust 代码

## Derive 宏开发

Derive 宏是最常用的过程宏类型，用于自动实现 trait。

### 第一个 Derive 宏

**Step 1: 创建 trait crate**

```rust
// hello_macro/src/lib.rs
pub trait Hello {
    fn hello();
}
```

**Step 2: 创建 derive 宏 crate**

```rust
// hello_macro_derive/src/lib.rs
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(Hello)]
pub fn hello_derive(input: TokenStream) -> TokenStream {
    // 1. 解析输入 TokenStream 为 AST
    let ast = parse_macro_input!(input as DeriveInput);

    // 2. 获取结构体/枚举名称
    let name = &ast.ident;

    // 3. 使用 quote! 生成代码
    let expanded = quote! {
        impl Hello for #name {
            fn hello() {
                println!("Hello from {}!", stringify!(#name));
            }
        }
    };

    // 4. 转换为 TokenStream
    expanded.into()
}
```

**Step 3: 使用宏**

```rust
use hello_macro::Hello;
use hello_macro_derive::Hello;

#[derive(Hello)]
struct Pancakes;

#[derive(Hello)]
enum MyEnum {
    VariantA,
    VariantB,
}

fn main() {
    Pancakes::hello();  // 输出: Hello from Pancakes!
}
```

### 访问结构体字段

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput, Data, Fields};

#[proc_macro_derive(Describe)]
pub fn describe_derive(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);
    let name = &ast.ident;

    let fields_description = match &ast.data {
        Data::Struct(data) => {
            match &data.fields {
                Fields::Named(fields) => {
                    let field_names: Vec<_> = fields
                        .named
                        .iter()
                        .map(|f| f.ident.as_ref().unwrap())
                        .collect();

                    quote! {
                        let mut desc = String::new();
                        $(
                            desc.push_str(&format!("{}: {:?}\n", stringify!(#field_names), self.#field_names));
                        )*
                        desc
                    }
                }
                Fields::Unnamed(fields) => {
                    quote! {
                        format!("{} with {} fields", stringify!(#name), fields.len())
                    }
                }
                Fields::Unit => {
                    quote! { stringify!(#name).to_string() }
                }
            }
        }
        Data::Enum(_) | Data::Union(_) => {
            quote! { stringify!(#name).to_string() }
        }
    };

    let expanded = quote! {
        impl Describe for #name {
            fn describe(&self) -> String {
                #fields_description
            }
        }
    };

    expanded.into()
}
```

### 解析 Derive 宏参数

```rust
use proc_macro::TokenStream;
use quote::{format_ident, quote};
use syn::{parse_macro_input, Attribute, Data, DeriveInput, Fields, Ident, Meta};

#[proc_macro_derive(Builder, attributes(builder))]
pub fn derive(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);
    let name = &ast.ident;

    // 解析 builder 属性的参数
    let builder_name = if let Some(builder_attr) = ast.attrs.iter().find(|a| a.path().is_ident("builder")) {
        let meta = builder_attr.meta();
        if let Meta::NameValue(nv) = meta {
            if let syn::Expr::Lit(syn::ExprLit { lit: syn::Lit::Str(s), .. }) = &nv.value {
                Some(s.value())
            } else {
                None
            }
        } else {
            None
        }
    } else {
        None
    };

    let builder_ident = match builder_name {
        Some(n) => format_ident!("{}", n),
        None => format_ident!("{}Builder", name),
    };

    let fields = if let Data::Struct(data) = &ast.data {
        if let Fields::Named(fields) = &data.fields {
            &fields.named
        } else {
            panic!("Builder only works on structs with named fields");
        }
    } else {
        panic!("Builder only works on structs");
    };

    let field_names: Vec<_> = fields.iter().map(|f| f.ident.as_ref().unwrap()).collect();
    let field_types: Vec<_> = fields.iter().map(|f| &f.ty).collect();

    let expanded = quote! {
        pub struct #builder_ident {
            #(#field_names: Option<#field_types>,)*
        }

        impl #builder_ident {
            pub fn new() -> Self {
                Self {
                    #(#field_names: None,)*
                }
            }

            $(
                pub fn #field_names(mut self, val: #field_types) -> Self {
                    self.#field_names = Some(val);
                    self
                }
            )*

            pub fn build(&self) -> #name {
                #name {
                    #(#field_names: self.#field_names.clone().unwrap(),)*
                }
            }
        }

        impl Default for #builder_ident {
            fn default() -> Self {
                Self::new()
            }
        }
    };

    expanded.into()
}
```

使用：

```rust
use builder::Builder;

#[derive(Builder)]
struct CreateUserRequest {
    name: String,
    email: String,
    age: u32,
}

fn main() {
    let request = CreateUserRequest::builder()
        .name("Alice".to_string())
        .email("alice@example.com".to_string())
        .age(30)
        .build();
}
```

### Derive 宏辅助属性

```rust
#[proc_macro_derive(ToQueryParams, attributes(query_param))]
pub fn derive_to_query_params(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);
    let name = &ast.ident;

    let fields = if let Data::Struct(data) = &ast.data {
        &data.fields
    } else {
        panic!("ToQueryParams only works on structs");
    };

    let field_impls: Vec<_> = fields.iter().map(|f| {
        let field_name = f.ident.as_ref().unwrap();
        let field_str = field_name.to_string();

        // 检查是否有 query_param 属性
        let rename = f.attrs.iter()
            .find(|a| a.path().is_ident("query_param"))
            .and_then(|a| {
                let meta = a.meta();
                if let Meta::NameValue(nv) = meta {
                    if let syn::Expr::Lit(syn::ExprLit { lit: syn::Lit::Str(s), .. }) = &nv.value {
                        return Some(s.value());
                    }
                }
                None
            });

        let param_name = rename.unwrap_or(field_str);

        quote! {
            if let Some(ref v) = self.#field_name {
                params.push((#param_name, v.to_string()));
            }
        }
    }).collect();

    let expanded = quote! {
        impl #name {
            pub fn to_query_params(&self) -> Vec<(&str, String)> {
                let mut params = Vec::new();
                #(#field_impls)*
                params
            }
        }
    };

    expanded.into()
}
```

使用辅助属性：

```rust
#[derive(ToQueryParams)]
struct SearchParams {
    #[query_param(rename = "q")]
    query: Option<String>,
    #[query_param(rename = "page_size")]
    size: Option<u32>,
}
```

## 属性宏开发

属性宏可以附加到任何代码项上（函数、结构体、模块等）。

### 基本属性宏

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, ItemFn};

#[proc_macro_attribute]
pub fn log_execution(attr: TokenStream, item: TokenStream) -> TokenStream {
    // attr: 属性括号内的内容
    // item: 属性附加到的代码项

    let input = parse_macro_input!(item as ItemFn);
    let func_name = &input.sig.ident;
    let func_body = &input.block;

    let expanded = quote! {
        fn #func_name() {
            println!("Calling {}...", stringify!(#func_name));
            let start = std::time::Instant::now();
            let result = (|| #func_body)();
            println!("{} took {:?}", stringify!(#func_name), start.elapsed());
            result
        }
    };

    expanded.into()
}
```

使用：

```rust
#[log_execution]
fn my_function() {
    println!("Doing work...");
}

fn main() {
    my_function();
}
```

### 解析属性参数

```rust
use proc_macro::{TokenStream, TokenTree, Delimiter};
use quote::quote;
use syn::{parse_macro_input, Attribute, ItemFn, Lit, Meta};

#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
    let attr_tokens: Vec<_> = attr.clone().into_iter().collect();

    // 解析 "GET", "/path" 格式
    let method = &attr_tokens[0];
    let path = &attr_tokens[1];

    let input = parse_macro_input!(item as ItemFn);
    let func_name = &input.sig.ident;
    let func_body = &input.block;

    let expanded = quote! {
        fn #func_name(req: actix_web::HttpRequest) -> impl actix_web::Responder {
            println!("Route {} {}", #method, #path);
            (|| #func_body)()
        }
    };

    expanded.into()
}
```

使用：

```rust
#[route(GET, "/users")]
fn get_users() -> String {
    "List of users".to_string()
}

#[route(POST, "/users")]
fn create_user() -> String {
    "User created".to_string()
}
```

### 验证属性宏

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, Data, DeriveInput, Fields};

#[proc_macro_derive(Validate, attributes(validate))]
pub fn validate_derive(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);
    let name = &ast.ident;

    let validation_code = if let Data::Struct(data) = &ast.data {
        if let Fields::Named(fields) = &data.fields {
            fields.named.iter().map(|f| {
                let field_name = f.ident.as_ref().unwrap();
                let mut checks = Vec::new();

                for attr in &f.attrs {
                    if attr.path().is_ident("validate") {
                        let meta = attr.meta();
                        if let syn::Meta::List(list) = meta {
                            for meta in &list.meta {
                                if let syn::Meta::NameValue(nv) = meta {
                                    if let syn::Expr::Path(p) = &nv.value {
                                        let constraint = p.path.last().unwrap();
                                        match constraint.to_string().as_str() {
                                            "non_empty" => {
                                                checks.push(quote! {
                                                    if self.#field_name.is_empty() {
                                                        return Err(format!("{} must not be empty", stringify!(#field_name)));
                                                    }
                                                });
                                            }
                                            "positive" => {
                                                checks.push(quote! {
                                                    if self.#field_name <= 0 {
                                                        return Err(format!("{} must be positive", stringify!(#field_name)));
                                                    }
                                                });
                                            }
                                            _ => {}
                                        }
                                    }
                                }
                            }
                        }
                    }
                }

                checks
            }).flatten().collect::<Vec<_>>()
        } else {
            Vec::new()
        }
    } else {
        Vec::new()
    };

    let expanded = quote! {
        impl Validate for #name {
            fn validate(&self) -> Result<(), String> {
                #(#validation_code)*
                Ok(())
            }
        }
    };

    expanded.into()
}
```

使用：

```rust
use validate::Validate;

#[derive(Validate)]
struct CreateUserRequest {
    #[validate(non_empty)]
    name: String,
    #[validate(positive)]
    age: u32,
}
```

## 函数式宏开发

函数式宏像函数调用一样使用，但操作 TokenStream。

### 基本函数式宏

```rust
use proc_macro::TokenStream;
use quote::quote;

#[proc_macro]
pub fn make_answer(_item: TokenStream) -> TokenStream {
    let expanded = quote! {
        fn answer() -> u32 {
            42
        }
    };

    expanded.into()
}
```

使用：

```rust
use make_answer::make_answer;

make_answer!();

fn main() {
    println!("{}", answer());  // 输出: 42
}
```

### SQL 查询宏

```rust
use proc_macro::TokenStream;
use quote::quote;

#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
    let sql_string = input.to_string();

    let expanded = quote! {
        {
            static STATEMENT: once_cell::sync::Lazy<tokio_postgres::Statement> =
                once_cell::sync::Lazy::new(|| {
                    let client = DATABASE.get().unwrap();
                    futures::executor::block_on(client.prepare(#sql_string))
                        .expect("Failed to prepare statement")
                });
            STATEMENT
        }
    };

    expanded.into()
}
```

使用：

```rust
let users = sql!(
    SELECT id, name, email
    FROM users
    WHERE active = true
).fetch_all(&*client).await?;
```

### 哈希宏

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::parse_str;

#[proc_macro]
pub fn hashmap(input: TokenStream) -> TokenStream {
    let input_str = input.to_string();

    let expanded = quote! {
        {
            let mut map = std::collections::HashMap::new();
            // 解析并填充 map
            let items: Vec<&str> = #input_str.split(',').collect();
            for item in items {
                let parts: Vec<&str> = item.split("=>").collect();
                if parts.len() == 2 {
                    map.insert(parts[0].trim(), parts[1].trim());
                }
            }
            map
        }
    };

    expanded.into()
}
```

## 测试宏

### 使用 trybuild 测试

```toml
[dev-dependencies]
trybuild = "1.0"
```

```rust
// tests/ui/simple.rs
use my_macro::MyTrait;

#[derive(MyTrait)]
struct TestStruct {
    field_a: i32,
    field_b: String,
}

fn main() {
    let instance = TestStruct {
        field_a: 42,
        field_b: "test".to_string(),
    };
    instance.my_trait_method();
}
```

```rust
// tests/my_macro_tests.rs
#[test]
fn test_derive() {
    let t = trybuild::TestCases::new();
    t.pass("tests/ui/simple.rs");
    t.compile_fail("tests/ui/error*.rs");
}
```

### 单元测试

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use quote::quote;

    #[test]
    fn test_derive_simple_struct() {
        let input = quote! {
            struct Point {
                x: f64,
                y: f64,
            }
        };

        let output = derive(input);
        assert!(output.to_string().contains("impl Describe for Point"));
    }

    #[test]
    fn test_derive_with_generics() {
        let input = quote! {
            struct Wrapper<T> {
                value: T,
            }
        };

        let output = derive(input);
        assert!(output.to_string().contains("impl<T> Describe for Wrapper<T>"));
    }
}
```

## 调试技巧

### 使用 cargo-expand

```bash
cargo install cargo-expand
cargo expand --lib
```

这会打印出宏展开后的代码：

```bash
$ cargo expand
#![feature(prelude_import)]
#[prelude_import]
pub use ::std::prelude::rust_2021::*;
...
```

### 打印调试信息

```rust
#[proc_macro_derive(Hello)]
pub fn hello_derive(input: TokenStream) -> TokenStream {
    eprintln!("Input: {}", input);  // 打印到 stderr

    let ast = syn::parse(input).unwrap();
    eprintln!("Parsed: {:?}", ast);

    // ... 宏逻辑
}
```

### 使用 proc-macro2 进行测试

```rust
#[cfg(test)]
mod tests {
    use proc_macro2::TokenStream;
    use quote::quote;
    use syn::parse_quote;

    #[test]
    fn test_macro() {
        let input: TokenStream = quote! {
            struct Test {
                field: i32,
            }
        };

        let ast: syn::DeriveInput = syn::parse2(input).unwrap();
        assert_eq!(ast.ident, "Test");
    }
}
```

## 最佳实践

### 1. 分离关注点

```rust
// 分离解析和代码生成
#[proc_macro_derive(MyTrait, attributes(my_attr))]
pub fn my_trait_derive(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);
    impl_my_trait(&ast).into()
}

fn impl_my_trait(ast: &DeriveInput) -> proc_macro2::TokenStream {
    // 代码生成逻辑
    let name = &ast.ident;
    // ...
}
```

### 2. 提供清晰的错误消息

```rust
#[proc_macro_derive(MyTrait)]
pub fn my_trait_derive(input: TokenStream) -> TokenStream {
    let ast = match syn::parse(input) {
        Ok(ast) => ast,
        Err(e) => return e.to_compile_error().into(),
    };

    if let syn::Data::Struct(_) = &ast.data {
        // OK
    } else {
        return syn::Error::new_spanned(
            &ast,
            "MyTrait only supports structs",
        ).to_compile_error().into();
    }

    // ...
}
```

### 3. 保持宏专注

```rust
// 好：单一职责
#[proc_macro_derive(Debug))]
#[proc_macro_derive(Clone))]
#[proc_macro_derive(Serialize)]

// 不好：试图做太多
#[proc_macro_derive(AllTheThings)]
```

### 4. 处理卫生性

```rust
// quote! 自动处理标识符卫生性
let var = quote!(value);  // 生成的标识符是卫生的

// 如果需要非卫生的标识符
let var = syn::Ident::new("value", proc_macro2::Span::混在一起());
```

### 5. 支持泛型和生命周期

```rust
#[proc_macro_derive(Describe)]
pub fn describe_derive(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);
    let name = &ast.ident;
    let (impl_generics, ty_generics, where_clause) = &ast.generics.split_for_impl();

    let expanded = quote! {
        impl #impl_generics Describe for #name #ty_generics #where_clause {
            // ...
        }
    };

    expanded.into()
}
```

## 常见宏库示例

### 实现 Default for Builder

```rust
#[proc_macro_derive(Builder, attributes(builder))]
pub fn builder_derive(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);
    let name = &ast.ident;
    let builder_name = format_ident!("{}Builder", name);

    let fields = if let Data::Struct(data) = &ast.data {
        if let Fields::Named(fields) = &data.fields {
            &fields.named
        } else {
            return quote! {
                compile_error!("Builder requires named fields");
            }.into();
        }
    } else {
        return quote! {
            compile_error!("Builder only works on structs");
        }.into();
    };

    let field_names: Vec<_> = fields.iter().map(|f| &f.ident).collect();
    let field_types: Vec<_> = fields.iter().map(|f| &f.ty).collect();

    let expanded = quote! {
        pub struct #builder_name {
            #(#field_names: core::option::Option<#field_types>,)*
        }

        impl #builder_name {
            pub fn new() -> Self {
                Self {
                    #(#field_names: core::option::Option::None,)*
                }
            }

            #(
                pub fn #field_names(mut self, val: #field_types) -> Self {
                    self.#field_names = core::option::Option::Some(val);
                    self
                }
            )*

            pub fn build(self) -> #name {
                #name {
                    #(#field_names: self.#field_names.unwrap(),)*
                }
            }
        }

        impl Default for #builder_name {
            fn default() -> Self {
                Self::new()
            }
        }
    };

    expanded.into()
}
```

## 总结

Rust 的宏系统提供了强大的编译时代码生成能力：

- **声明式宏**：`macro_rules!` - 适合简单的模式匹配和代码生成
- **过程宏**：
  - **Derive 宏** - 自动实现 trait（如 `#[derive(Debug)]`）
  - **属性宏** - 修改或增强代码项
  - **函数式宏** - 像函数一样使用的 DSL

关键工具：
- **syn**：解析 Rust 代码为 AST
- **quote**：使用模板生成 Rust 代码
- **proc-macro2**：TokenStream 的跨平台兼容层

最佳实践：
- 保持宏专注和单一职责
- 提供清晰的错误消息
- 使用 cargo-expand 调试
- 处理泛型和生命周期
- 编写测试确保正确性

掌握宏开发，你将能够构建像 serde、diesel、rocket 这样强大的 Rust 库！

快乐编程，大家来 Rust! 🦀
