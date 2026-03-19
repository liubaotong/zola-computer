+++
title = "Rust ORM 完全指南：Diesel、SQLx、SeaORM 实战"
date = "2026-03-19T05:36:00+08:00"

[taxonomies]
tags = ["rust", "orm", "database", "sqlx", "diesel", "sea-orm", "postgresql"]
categories = ["编程", "数据库"]

[extra]
summary = "本文详细介绍 Rust 生态中的主流 ORM 框架：Diesel、SQLx、SeaORM 和 SeaQuery，帮助你选择最适合的数据库访问方案。"
author = "博主"
+++

Rust 生态提供了多个成熟的数据库访问方案，从轻量级的 SQL 查询构建器到功能完整的 ORM 框架应有尽有。本文将详细介绍四种主流方案：Diesel、SQLx、SeaORM 和 SeaQuery，帮助你根据项目需求选择最合适的工具。

## Rust 数据库生态概览

### 主要方案对比

| 方案 | 类型 | 异步支持 | 编译时检查 | 学习曲线 |
|------|------|----------|------------|----------|
| Diesel | ORM | ❌ | ✅ 类型检查 | 中等 |
| SQLx | SQL 库 | ✅ | ✅ 实时连接检查 | 低 |
| SeaORM | ORM | ✅ | ❌ | 中等 |
| SeaQuery | 查询构建器 | ❌ | ❌ | 低 |

### 选择指南

- **需要完整 ORM 功能 + 异步？** → SeaORM
- **偏好写原生 SQL + 编译时检查？** → SQLx
- **需要同步 ORM + 强类型？** → Diesel
- **只需要查询构建器？** → SeaQuery

## SQLx：编译时安全的 SQL

SQLx 是一个异步 SQL 库，提供了独特的编译时 SQL 检查能力，无需运行数据库即可验证 SQL 语句的正确性。

### 项目设置

```toml
[dependencies]
sqlx = { version = "0.8", features = ["runtime-tokio", "postgres", "macros", "chrono"] }
tokio = { version = "1", features = ["full"] }
```

### 编译时查询验证

```rust
use sqlx::{FromRow, PgPool};

#[derive(Debug, FromRow)]
struct User {
    id: i64,
    username: String,
    email: String,
    created_at: chrono::DateTime<chrono::Utc>,
}

async fn get_user(pool: &PgPool, id: i64) -> Result<Option<User>, sqlx::Error> {
    // 编译时验证！无需运行数据库
    sqlx::query_as!(
        User,
        r#"
        SELECT id, username, email, created_at
        FROM users
        WHERE id = $1
        "#,
        id
    )
    .fetch_optional(pool)
    .await
}
```

### 连接池管理

```rust
use sqlx::postgres::{PgPool, PgPoolOptions};

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = PgPoolOptions::new()
        .max_connections(10)
        .min_connections(2)
        .acquire_timeout(std::time::Duration::from_secs(30))
        .idle_timeout(std::time::Duration::from_secs(600))
        .connect("postgres://user:password@localhost/mydb")
        .await?;

    println!("数据库连接池已建立");

    // 使用连接池
    let user = sqlx::query_as!(User, "SELECT * FROM users WHERE id = $1", 1)
        .fetch_one(&pool)
        .await?;

    println!("用户: {:?}", user);

    pool.close().await;
    Ok(())
}
```

### CRUD 操作

```rust
use sqlx::PgPool;
use chrono::Utc;

#[derive(Debug, sqlx::FromRow)]
struct User {
    id: i64,
    username: String,
    email: String,
    created_at: chrono::DateTime<chrono::Utc>,
}

// 创建用户
async fn create_user(
    pool: &PgPool,
    username: &str,
    email: &str,
) -> Result<i64, sqlx::Error> {
    let row: (i64,) = sqlx::query_as(
        "INSERT INTO users (username, email, created_at) VALUES ($1, $2, $3) RETURNING id"
    )
    .bind(username)
    .bind(email)
    .bind(Utc::now())
    .fetch_one(pool)
    .await?;

    Ok(row.0)
}

// 查询用户
async fn get_user_by_email(
    pool: &PgPool,
    email: &str,
) -> Result<Option<User>, sqlx::Error> {
    sqlx::query_as(
        "SELECT id, username, email, created_at FROM users WHERE email = $1"
    )
    .bind(email)
    .fetch_optional(pool)
    .await
}

// 更新用户
async fn update_username(
    pool: &PgPool,
    id: i64,
    new_username: &str,
) -> Result<bool, sqlx::Error> {
    let result = sqlx::query("UPDATE users SET username = $1 WHERE id = $2")
        .bind(new_username)
        .bind(id)
        .execute(pool)
        .await?;

    Ok(result.rows_affected() > 0)
}

// 删除用户
async fn delete_user(pool: &PgPool, id: i64) -> Result<bool, sqlx::Error> {
    let result = sqlx::query("DELETE FROM users WHERE id = $1")
        .bind(id)
        .execute(pool)
        .await?;

    Ok(result.rows_affected() > 0)
}
```

### 事务处理

```rust
use sqlx::{PgPool, Postgres, Transaction};

async fn transfer_funds(
    pool: &PgPool,
    from_id: i64,
    to_id: i64,
    amount: i64,
) -> Result<(), sqlx::Error> {
    let mut tx: Transaction<'_, Postgres> = pool.begin().await?;

    // 扣除发送者余额
    sqlx::query("UPDATE accounts SET balance = balance - $1 WHERE id = $2")
        .bind(amount)
        .bind(from_id)
        .execute(&mut *tx)
        .await?;

    // 增加接收者余额
    sqlx::query("UPDATE accounts SET balance = balance + $1 WHERE id = $2")
        .bind(amount)
        .bind(to_id)
        .execute(&mut *tx)
        .await?;

    // 提交事务
    tx.commit().await?;

    Ok(())
}
```

### 批量操作

```rust
use sqlx::PgPool;
use futures::stream;

async fn batch_insert_users(
    pool: &PgPool,
    users: Vec<(String, String)>,
) -> Result<u64, sqlx::Error> {
    let mut total = 0u64;

    // 事务中批量插入
    let mut tx = pool.begin().await?;

    for (username, email) in users {
        let result = sqlx::query(
            "INSERT INTO users (username, email) VALUES ($1, $2)"
        )
        .bind(&username)
        .bind(&email)
        .execute(&mut *tx)
        .await?;
        
        total += result.rows_affected();
    }

    tx.commit().await?;
    Ok(total)
}
```

## SeaORM：异步 ORM 框架

SeaORM 是一个异步动态 ORM，构建在 SeaQuery 之上，提供了完整的 ORM 功能。

### 项目设置

```toml
[dependencies]
sea-orm = { version = "1.0", features = ["runtime-tokio-native-tls", "postgres", "chrono", "uuid"] }
tokio = { version = "1", features = ["full"] }
```

### 定义 Entity

```rust
use sea_orm::entity::prelude::*;

#[derive(Clone, Debug, DeriveEntityModel)]
#[sea_orm(table_name = "users")]
pub struct Model {
    #[sea_orm(primary_key)]
    pub id: i64,
    #[sea_orm(column_type = "Text")]
    pub username: String,
    #[sea_orm(column_type = "Text")]
    pub email: String,
    #[sea_orm(column_type = "Timestamp")]
    pub created_at: DateTimeUtc,
    #[sea_orm(column_type = "Timestamp", nullable)]
    pub updated_at: Option<DateTimeUtc>,
}

#[derive(Copy, Clone, Debug, EnumIter, DeriveRelation)]
pub enum Relation {
    #[sea_orm(has_many = "super::post::Entity")]
    Posts,
}

impl ActiveModelBehavior for ActiveModel {}
```

### 定义 Relation

```rust
use sea_orm::entity::prelude::*;

#[derive(Clone, Debug, DeriveEntityModel)]
#[sea_orm(table_name = "posts")]
pub struct Model {
    #[sea_orm(primary_key)]
    pub id: i64,
    #[sea_orm(column_type = "BigInteger")]
    pub user_id: i64,
    #[sea_orm(column_type = "Text")]
    pub title: String,
    #[sea_orm(column_type = "Text", nullable)]
    pub content: Option<String>,
    #[sea_orm(column_type = "Timestamp")]
    pub created_at: DateTimeUtc,
}

#[derive(Copy, Clone, Debug, EnumIter, DeriveRelation)]
pub enum Relation {
    #[sea_orm(
        belongs_to = "super::user::Entity",
        from = "Column::UserId",
        to = "super::user::Column::Id"
    )]
    User,
}

impl Related<super::user::Entity> for Entity {
    fn relation() -> RelationDef {
        Relation::User.def()
    }
}

impl ActiveModelBehavior for ActiveModel {}
```

### CRUD 操作

```rust
use sea_orm::{entity::*, DatabaseConnection, DbErr, ActiveModel};

async fn create_user(
    db: &DatabaseConnection,
    username: &str,
    email: &str,
) -> Result<user::ActiveModel, DbErr> {
    let user = user::ActiveModel {
        username: Set(username.to_owned()),
        email: Set(email.to_owned()),
        created_at: Set(chrono::Utc::now()),
        ..Default::default()
    };

    user.insert(db).await
}

async fn get_user_by_id(
    db: &DatabaseConnection,
    id: i64,
) -> Result<Option<user::Model>, DbErr> {
    User::find_by_id(id).one(db).await
}

async fn get_user_by_email(
    db: &DatabaseConnection,
    email: &str,
) -> Result<Option<user::Model>, DbErr> {
    User::find()
        .filter(user::Column::Email.eq(email))
        .one(db)
        .await
}

async fn update_user(
    db: &DatabaseConnection,
    id: i64,
    new_username: &str,
) -> Result<user::Model, DbErr> {
    let user: user::ActiveModel = User::find_by_id(id)
        .one(db)
        .await?
        .ok_or(DbErr::RecordNotFound)?
        .into();

    let mut user = user;
    user.username = Set(new_username.to_owned());
    user.updated_at = Set(Some(chrono::Utc::now()));

    user.update(db).await
}

async fn delete_user(db: &DatabaseConnection, id: i64) -> Result<(), DbErr> {
    User::delete_by_id(id).exec(db).await?;
    Ok(())
}
```

### 查询构建器

```rust
use sea_orm::{entity::*, DbErr, Order};

async fn complex_queries(db: &DatabaseConnection) -> Result<(), DbErr> {
    // 条件查询
    let users = User::find()
        .filter(user::Column::Email.like("%@example.com%"))
        .order_by(user::Column::CreatedAt, Order::Desc)
        .limit(10)
        .offset(0)
        .all(db)
        .await?;

    // 聚合查询
    let count: i64 = User::find()
        .filter(user::Column::Email.like("%@example.com%"))
        .count(db)
        .await?;

    // 分组查询
    let posts_with_authors = Post::find()
        .find_also_related(User)
        .all(db)
        .await?;

    for (post, author) in posts_with_authors {
        println!("Post: {} by {}", post.title, author.username);
    }

    Ok(())
}
```

### 预加载关联

```rust
use sea_orm::{entity::*, DbErr};

async fn eager_loading(db: &DatabaseConnection) -> Result<(), DbErr> {
    // 一对多：获取用户及其所有文章
    let users_with_posts = User::find()
        .find_with_related(Post)
        .all(db)
        .await?;

    for (user, posts) in users_with_posts {
        println!("User: {} has {} posts", user.username, posts.len());
        for post in posts {
            println!("  - {}", post.title);
        }
    }

    // 多对一：获取文章及其作者
    let posts_with_users = Post::find()
        .find_also_related(User)
        .all(db)
        .await?;

    for (post, user) in posts_with_users {
        println!("Post '{}' by {}", post.title, user.username);
    }

    Ok(())
}
```

### 事务处理

```rust
use sea_orm::{DatabaseConnection, DbErr, TransactionTrait};

async fn transfer_funds(
    db: &DatabaseConnection,
    from_id: i64,
    to_id: i64,
    amount: i64,
) -> Result<(), DbErr> {
    let tx = db.begin().await?;

    // 使用事务执行更新
    let result = tx.execute(
        query::Query::update()
            .table(AccountTable)
            .values([
                (Column::Balance, Expr::col(Column::Balance) - amount),
            ])
            .and_where(Column::Id.eq(from_id))
    ).await?;

    if result.rows_affected() == 0 {
        return Err(DbErr::RecordNotFound);
    }

    tx.execute(
        query::Query::update()
            .table(AccountTable)
            .values([
                (Column::Balance, Expr::col(Column::Balance) + amount),
            ])
            .and_where(Column::Id.eq(to_id))
    ).await?;

    tx.commit().await
}
```

## Diesel：同步 ORM

Diesel 是 Rust 生态中最成熟的同步 ORM，提供了强大的编译时类型检查。

### 项目设置

```toml
[dependencies]
diesel = { version = "2.1", features = ["postgres", "r2d2", "chrono"] }
```

### 定义 Schema

```rust
// schema.rs
diesel::table! {
    users (id) {
        id -> Int8,
        username -> Varchar,
        email -> Varchar,
        created_at -> Timestamp,
    }
}

diesel::table! {
    posts (id) {
        id -> Int8,
        user_id -> Int8,
        title -> Varchar,
        content -> Nullable<Text>,
        created_at -> Timestamp,
    }
}

diesel::joinable!(posts -> users (user_id));
diesel::allow_tables_to_appear_in_same_query!(users, posts);
```

### 定义 Model

```rust
// models.rs
use diesel::prelude::*;
use chrono::{DateTime, Utc};
use crate::schema::{users, posts};

#[derive(Queryable, Selectable, Debug)]
#[diesel(table_name = users)]
#[diesel(check_for_backend(diesel::pg::Pg))]
pub struct User {
    pub id: i64,
    pub username: String,
    pub email: String,
    pub created_at: DateTime<Utc>,
}

#[derive(Insertable, Debug)]
#[diesel(table_name = users)]
pub struct NewUser<'a> {
    pub username: &'a str,
    pub email: &'a str,
}

#[derive(Queryable, Selectable, Associations, Debug)]
#[diesel(belongs_to(User))]
#[diesel(table_name = posts)]
#[diesel(check_for_backend(diesel::pg::Pg))]
pub struct Post {
    pub id: i64,
    pub user_id: i64,
    pub title: String,
    pub content: Option<String>,
    pub created_at: DateTime<Utc>,
}

#[derive(Insertable, Debug)]
#[diesel(table_name = posts)]
pub struct NewPost<'a> {
    pub user_id: i64,
    pub title: &'a str,
    pub content: Option<&'a str>,
}
```

### CRUD 操作

```rust
use diesel::prelude::*;
use crate::schema::{users, posts};
use crate::models::{User, NewUser, Post, NewPost};
use crate::establish_connection;

pub fn create_user(conn: &mut PgConnection, username: &str, email: &str) -> User {
    let new_user = NewUser { username, email };

    diesel::insert_into(users::table)
        .values(&new_user)
        .get_result(conn)
        .unwrap()
}

pub fn get_user_by_id(conn: &mut PgConnection, user_id: i64) -> Option<User> {
    users::table
        .find(user_id)
        .first::<User>(conn)
        .optional()
        .unwrap()
}

pub fn get_user_by_email(conn: &mut PgConnection, email: &str) -> Option<User> {
    users::table
        .filter(users::email.eq(email))
        .first::<User>(conn)
        .optional()
        .unwrap()
}

pub fn update_username(conn: &mut PgConnection, user_id: i64, new_username: &str) -> User {
    diesel::update(users::find(user_id))
        .set(users::username.eq(new_username))
        .get_result(conn)
        .unwrap()
}

pub fn delete_user(conn: &mut PgConnection, user_id: i64) -> bool {
    diesel::delete(users::find(user_id))
        .execute(conn)
        .unwrap() > 0
}
```

### 关联查询

```rust
use diesel::prelude::*;
use crate::schema::{users, posts};
use crate::models::{User, Post};

// 获取用户的所有文章
pub fn get_user_posts(conn: &mut PgConnection, user_id: i64) -> Vec<Post> {
    Post::belonging_to(&User { id: user_id, username: "".to_string(), email: "".to_string(), created_at: chrono::Utc::now() })
        .load(conn)
        .unwrap()
}

// 获取所有用户及其文章
pub fn get_all_users_with_posts(conn: &mut PgConnection) -> Vec<(User, Vec<Post>)> {
    let users = users::table.load::<User>(conn).unwrap();
    let posts = Post::belonging_to(&users)
        .load::<Post>(conn)
        .unwrap()
        .grouped_by(&users);

    users.into_iter().zip(posts).collect()
}
```

### 事务处理

```rust
use diesel::prelude::*;
use diesel::r2d2::ConnectionManager;
use crate::schema::{accounts, users};

pub fn transfer_funds(
    conn: &mut PgConnection,
    from_id: i64,
    to_id: i64,
    amount: i64,
) -> Result<(), diesel::result::Error> {
    conn.transaction(|conn| {
        // 扣除发送者余额
        diesel::update(accounts::table.filter(accounts::user_id.eq(from_id)))
            .set(accounts::balance.eq(accounts::balance - amount))
            .execute(conn)?;

        // 增加接收者余额
        diesel::update(accounts::table.filter(accounts::user_id.eq(to_id)))
            .set(accounts::balance.eq(accounts::balance + amount))
            .execute(conn)?;

        Ok(())
    })
}
```

## SeaQuery：轻量级查询构建器

SeaQuery 提供了灵活的 SQL 查询构建 API，适用于不需要完整 ORM 功能的场景。

### 项目设置

```toml
[dependencies]
sea-query = { version = "0.30", features = ["backend-postgres", "with-chrono"] }
```

### SELECT 查询

```rust
use sea_query::{*, PostgresQueryBuilder};

let query = Query::select()
    .columns([Glyph::Id, Glyph::Aspect, Glyph::Path])
    .from(Glyph::Table)
    .conditions(
        Some(true),
        |q| {
            q.and_where(Expr::col(Glyph::Aspect).eq(2.0))
        },
    )
    .and_where(Expr::col(Glyph::Aspect).is_not_null())
    .to_string(PostgresQueryBuilder);

assert_eq!(
    query,
    r#"SELECT "id", "aspect", "path" FROM "glyph" WHERE ("aspect" = 2.0) AND "aspect" IS NOT NULL"#
);
```

### JOIN 查询

```rust
use sea_query::{*, PostgresQueryBuilder};

let query = Query::select()
    .column((Font::Table, Font::Name))
    .column(Char::Character)
    .from(Char::Table)
    .left_join(
        Font::Table,
        Expr::col((Char::Table, Char::FontId)).equals((Font::Table, Font::Id)),
    )
    .to_string(PostgresQueryBuilder);

assert_eq!(
    query,
    r#"SELECT "font"."name", "character"."character" FROM "character" LEFT JOIN "font" ON "character"."font_id" = "font"."id""#
);
```

### INSERT 查询

```rust
use sea_query::{*, PostgresQueryBuilder};

let query = Query::insert()
    .into_table(Glyph::Table)
    .columns([Glyph::Aspect, Glyph::Path])
    .values([
        1.0.into(),
        "1".into(),
    ])
    .values([
        2.0.into(),
        "2".into(),
    ])
    .to_string(PostgresQueryBuilder);

assert_eq!(
    query,
    r#"INSERT INTO "glyph" ("aspect", "path") VALUES (1.0, '1'), (2.0, '2')"#
);
```

### UPDATE 查询

```rust
use sea_query::{*, PostgresQueryBuilder};

let query = Query::update()
    .table(Glyph::Table)
    .values([
        (Glyph::Aspect, 3.0.into()),
        (Glyph::Path, "3".into()),
    ])
    .and_where(Expr::col(Glyph::Id).eq(1))
    .to_string(PostgresQueryBuilder);

assert_eq!(
    query,
    r#"UPDATE "glyph" SET "aspect" = 3.0, "path" = '3' WHERE "id" = 1"#
);
```

### DELETE 查询

```rust
use sea_query::{*, PostgresQueryBuilder};

let query = Query::delete()
    .from_table(Glyph::Table)
    .and_where(Expr::col(Glyph::Id).eq(1))
    .to_string(PostgresQueryBuilder);

assert_eq!(
    query,
    r#"DELETE FROM "glyph" WHERE "id" = 1"#
);
```

### 动态查询

```rust
use sea_query::{*, PostgresQueryBuilder};

fn build_search_query(
    username: Option<&str>,
    email: Option<&str>,
    limit: Option<u64>,
) -> (String, Values) {
    let mut query = Query::select()
        .columns([users::Id, users::Username, users::Email])
        .from(users::Table);

    if let Some(name) = username {
        query = query.and_where(Expr::col(users::Username).like(format!("%{}%", name)));
    }

    if let Some(email_addr) = email {
        query = query.and_where(Expr::col(users::Email).eq(email_addr));
    }

    if let Some(l) = limit {
        query = query.limit(l);
    }

    query.build(PostgresQueryBuilder)
}
```

## 迁移管理

### SQLx 迁移

```bash
# 创建迁移目录
sqlx migrate add create_users_table
```

```sql
-- migrations/20240101000000_create_users_table.sql

CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(255) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
```

```rust
use sqlx::PgPool;
use sqlx::migrate::Migrator;
use std::path::Path;

static MIGRATOR: Migrator = sqlx::migrate!("./migrations");

async fn run_migrations(pool: &PgPool) -> Result<(), sqlx::Error> {
    MIGRATOR.run(pool).await?;
    Ok(())
}
```

### SeaORM 迁移

```bash
# 安装 CLI
cargo install sea-orm-cli

# 创建迁移
sea-orm-cli migrate generate create_users_table
```

```rust
// migration/src/m20240101_000001_create_users_table.rs

use sea_orm_migration::prelude::*;

pub struct Migration;

impl MigrationName for Migration {
    fn name(&self) -> &str {
        "m20240101_000001_create_users_table"
    }
}

#[async_trait::async_trait]
impl MigrationTrait for Migration {
    async fn up(&self, manager: &SchemaManager) -> Result<(), DbErr> {
        manager
            .create_table(
                Table::create()
                    .table(Users::Table)
                    .if_not_exists()
                    .col(ColumnDef::new(Users::Id).big_int().not_null().auto_increment().primary_key())
                    .col(ColumnDef::new(Users::Username).string().not_null().unique_key())
                    .col(ColumnDef::new(Users::Email).string().not_null().unique_key())
                    .col(ColumnDef::new(Users::CreatedAt).timestamp_with_time_zone().not_null().default(Expr::current_timestamp()))
                    .to_owned(),
            )
            .await?;

        manager
            .create_index(
                Index::create()
                    .name("idx_users_email")
                    .table(Users::Table)
                    .col(Users::Email)
                    .to_owned(),
            )
            .await?;

        Ok(())
    }

    async fn down(&self, manager: &SchemaManager) -> Result<(), DbErr> {
        manager
            .drop_table(Table::drop().table(Users::Table).to_owned())
            .await
    }
}
```

## 最佳实践

### 1. 连接池配置

```rust
// SQLx
let pool = PgPoolOptions::new()
    .max_connections(20)
    .min_connections(5)
    .acquire_timeout(Duration::from_secs(30))
    .idle_timeout(Duration::from_secs(600))
    .max_lifetime(Duration::from_secs(1800))
    .connect(&database_url)
    .await?;
```

### 2. 错误处理

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum DbError {
    #[error("Record not found")]
    NotFound,
    
    #[error("Unique constraint violation: {0}")]
    UniqueViolation(String),
    
    #[error("Database error: {0}")]
    QueryError(#[from] sqlx::Error),
}

impl From<sqlx::Error> for DbError {
    fn from(err: sqlx::Error) -> Self {
        match err {
            sqlx::Error::RowNotFound => DbError::NotFound,
            _ => DbError::QueryError(err),
        }
    }
}
```

### 3. 异步 vs 同步选择

```rust
// Web 服务 - 使用异步 (SQLx / SeaORM)
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let pool = PgPool::connect(&std::env::var("DATABASE_URL")?).await?;
    // ...
    Ok(())
}

// CLI 工具 - 可以使用同步 (Diesel)
fn main() {
    let mut conn = PgConnection::establish(&std::env::var("DATABASE_URL").unwrap()).unwrap();
    // ...
}
```

### 4. 索引优化

```sql
-- 经常查询的字段添加索引
CREATE INDEX idx_users_email ON users(email);

-- 复合索引
CREATE INDEX idx_posts_user_id_created_at ON posts(user_id, created_at DESC);

-- 部分索引
CREATE INDEX idx_posts_published ON posts(user_id) WHERE published = true;
```

### 5. N+1 查询问题

```rust
// 不好：N+1 查询
for user in users {
    let posts = get_user_posts(user.id); // 每个用户一次查询
}

// 好：批量查询
let posts = Post::find()
    .filter(user::Column::Id.is_in(users.iter().map(|u| u.id)))
    .all(db)
    .await?;
```

## 总结

Rust 提供了丰富的数据库访问方案：

**SQLx**：
- 编译时 SQL 验证
- 异步支持
- 适合偏好写原生 SQL 的开发者

**SeaORM**：
- 完整 ORM 功能
- 异步支持
- 适合需要快速开发的企业应用

**Diesel**：
- 同步操作
- 强大的类型系统
- 适合 CLI 工具和同步应用

**SeaQuery**：
- 轻量级查询构建
- 灵活动态查询
- 适合作为自定义 ORM 的基础

选择合适的工具，让数据库操作更高效！

快乐编程，大家来 Rust! 🦀
