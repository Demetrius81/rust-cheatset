# 📘 Diesel ORM — полная шпаргалка (Rust)

---

[Полная шпаргалка по Rust — синтаксис, инициализация, функции](README.md)

[Вопросы к собеседованию Rust разработчика 2023](questions_2023.md)

[Rust — Шпаргалка по `Vec` и `Iterator`](rust_vec_iterator_cheatsheet.md)

[Actix-web — энциклопедия](actix_web_cheatset.md)

---

## 1. Основы и настройка

| API / Команда                                                        |                              Описание | Пример                                           |
| -------------------------------------------------------------------- | ------------------------------------: | ------------------------------------------------ |
| `cargo install diesel_cli --no-default-features --features postgres` | Установка CLI для работы с миграциями | `diesel setup`                                   |
| `.env`                                                               |                 Настройка подключения | `DATABASE_URL=postgres://user:pass@localhost/db` |
| `diesel setup`                                                       |      Создать БД и папку `migrations/` | —                                                |
| `diesel migration generate NAME`                                     |                      Создать миграцию | `diesel migration generate create_users`         |
| `diesel migration run` / `redo`                                      |       Применить или откатить миграцию | —                                                |

---

## 2. Схема и модели

| API                      |                                     Описание | Пример                                                                                   |
| ------------------------ | -------------------------------------------: | ---------------------------------------------------------------------------------------- |
| `diesel print-schema`    |                        Генерация `schema.rs` | —                                                                                        |
| `table!`                 | Макрос описания таблицы (авто в `schema.rs`) | `rust table! { users (id) { id -> Int4, name -> Varchar, } } `                           |
| `#[derive(Queryable)]`   |                                 Чтение из БД | `rust #[derive(Queryable)] struct User { id: i32, name: String } `                       |
| `#[derive(Insertable)]`  |                                 Вставка в БД | `rust #[derive(Insertable)] #[table_name="users"] struct NewUser<'a> { name: &'a str } ` |
| `#[derive(AsChangeset)]` |                                   Обновление | `#[derive(AsChangeset)] #[table_name="users"]`                                           |

---

## 3. Подключение и выполнение запросов

| API                       |               Описание | Пример                                                                   |
| ------------------------- | ---------------------: | ------------------------------------------------------------------------ |
| `PgConnection::establish` |            Подключение | `rust let conn = PgConnection::establish(&env::var("DATABASE_URL")?)?; ` |
| `connection.execute`      | Выполнить SQL напрямую | `conn.execute("TRUNCATE users")?;`                                       |

---

## 4. Запросы (Query Builder)

| API / Функция               |                 Описание | Пример                                                   |
| --------------------------- | -----------------------: | -------------------------------------------------------- |
| `users.load::<User>(&conn)` |     SELECT \* FROM users | `let results = users.load::<User>(&conn)?;`              |
| `users.find(id)`            |              Поиск по PK | `users.find(1).first::<User>(&conn)?;`                   |
| `filter(col.eq(val))`       |          WHERE col = val | `users.filter(name.eq("John")).load(&conn)?`             |
| `order(col.desc())`         |                 ORDER BY | `users.order(created_at.desc()).load(&conn)?`            |
| `limit(n)` / `offset(n)`    |                Пагинация | `users.limit(10).offset(20).load(&conn)?`                |
| `select(cols)`              | Выбор определённых полей | `users.select((id, name)).load::<(i32, String)>(&conn)?` |
| `count_star()`              |                COUNT(\*) | `users.count().get_result(&conn)?`                       |
| `distinct()`                |          SELECT DISTINCT | `users.select(name).distinct().load(&conn)?`             |

---

## 5. Вставка, обновление, удаление

| API                                       | Описание | Пример                                                               |
| ----------------------------------------- | -------: | -------------------------------------------------------------------- |
| `insert_into(users).values(&new_user)`    |   INSERT | `diesel::insert_into(users).values(&new_user).execute(&conn)?;`      |
| `update(users.find(id)).set(col.eq(val))` |   UPDATE | `diesel::update(users.find(1)).set(name.eq("New")).execute(&conn)?;` |
| `delete(users.find(id))`                  |   DELETE | `diesel::delete(users.find(1)).execute(&conn)?;`                     |

---

## 6. Ассоциации (JOIN)

| API            |                   Описание | Пример                                                      |
| -------------- | -------------------------: | ----------------------------------------------------------- |
| `inner_join`   |                 INNER JOIN | `users.inner_join(posts).select((users::id, posts::title))` |
| `left_join`    |                  LEFT JOIN | `users.left_join(posts)`                                    |
| `belonging_to` | Загрузить связанные записи | `Post::belonging_to(&user).load::<Post>(&conn)?`            |

---

## 7. Транзакции

| API                              | Описание | Пример     |                             |                                                               |   |                                                                                        |
| -------------------------------- | -------: | ---------- | --------------------------- | ------------------------------------------------------------- | - | -------------------------------------------------------------------------------------- |
| \`conn.transaction::\<T, E, \_>( |          | { ... })\` | Выполнить блок в транзакции | \`\`\`rust conn.transaction::<\_, diesel::result::Error, \_>( |   | { diesel::insert\_into(users).values(\&new\_user).execute(\&conn)?; Ok(()) })?; \`\`\` |

---

## 8. Raw SQL

| API                |                   Описание | Пример                                                                           |
| ------------------ | -------------------------: | -------------------------------------------------------------------------------- |
| `sql_query("...")` | Выполнить произвольный SQL | `rust let res = diesel::sql_query("SELECT * FROM users").load::<User>(&conn)?; ` |

---

# 🛠 Готовые шаблоны проектов с Diesel

## 1. Basic CRUD + PostgreSQL

**Cargo.toml**

```toml
[package]
name = "diesel_crud"
version = "0.1.0"
edition = "2021"

[dependencies]
diesel = { version = "2.1.0", features = ["postgres"] }
dotenvy = "0.15"
serde = { version = "1.0", features = ["derive"] }
```

**src/main.rs**

```rust
use diesel::prelude::*;
use diesel_crud::models::*;
use dotenvy::dotenv;
use std::env;

mod schema;
mod models;

fn main() {
    dotenv().ok();
    let database_url = env::var("DATABASE_URL").unwrap();
    let conn = PgConnection::establish(&database_url).unwrap();

    // Create
    let new_user = NewUser { name: "John" };
    diesel::insert_into(schema::users::table)
        .values(&new_user)
        .execute(&conn)
        .unwrap();

    // Read
    let results = schema::users::table.load::<User>(&conn).unwrap();
    println!("{:?}", results);
}
```

**src/models.rs**

```rust
use super::schema::users;
use serde::{Serialize, Deserialize};

#[derive(Queryable, Serialize, Deserialize, Debug)]
pub struct User {
    pub id: i32,
    pub name: String,
}

#[derive(Insertable)]
#[diesel(table_name = users)]
pub struct NewUser<'a> {
    pub name: &'a str,
}
```

---

## 2. Фильтрация и сортировка

```rust
use diesel::prelude::*;
use myproj::schema::users::dsl::*;

fn main() {
    let conn = establish_connection();
    let results = users
        .filter(active.eq(true))
        .order(created_at.desc())
        .limit(10)
        .load::<User>(&conn)
        .unwrap();
    println!("{:?}", results);
}
```

---

## 3. Миграции и заполнение БД

**up.sql**

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL,
    active BOOLEAN DEFAULT true
);
```

**down.sql**

```sql
DROP TABLE users;
```

**seeds.rs**

```rust
let data = vec!["Alice", "Bob", "Charlie"];
for name in data {
    diesel::insert_into(users).values(NewUser { name }).execute(&conn)?;
}
```

---

---

---

Базовый пример использования Diesel с PostgreSQL в Rust, включающий:

- подключение к базе
- настройку миграций
- реализацию паттерна репозиторий (CRUD)

---

## 1. Настройка проекта

Создайте новый проект:

```bash
cargo new diesel_postgres_example
cd diesel_postgres_example
```

Добавьте зависимости в `Cargo.toml`:

```toml
[dependencies]
diesel = { version = "2.0.0", features = ["postgres"] }
dotenvy = "0.15"
tokio = { version = "1", features = ["full"] }
```

---

## 2. Настройка базы данных и миграции

Создайте файл `.env` в корне проекта:

```env
DATABASE_URL=postgres://user:password@localhost/mydb
```

Замените `user`, `password`, `mydb` на свои параметры.

Инициализируйте миграции:

```bash
diesel setup
```

Создайте миграцию для таблицы `users`:

```bash
diesel migration generate create_users
```

В файле `migrations/<timestamp>_create_users/up.sql` напишите:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL,
    email VARCHAR UNIQUE NOT NULL
);
```

В файле `migrations/<timestamp>_create_users/down.sql` — откат:

```sql
DROP TABLE users;
```

Запустите миграции:

```bash
diesel migration run
```

---

## 3. Создание моделей и схемы

В `src/schema.rs` сгенерируется автоматически после миграций, или создайте вручную:

```rust
// src/schema.rs
diesel::table! {
    users (id) {
        id -> Int4,
        name -> Varchar,
        email -> Varchar,
    }
}
```

В `src/models.rs` создайте модель пользователя:

```rust
// src/models.rs
use super::schema::users;

#[derive(Queryable)]
pub struct User {
    pub id: i32,
    pub name: String,
    pub email: String,
}

#[derive(Insertable)]
#[diesel(table_name = super::schema::users)]
pub struct NewUser<'a> {
    pub name: &'a str,
    pub email: &'a str,
}
```

---

## 4. Реализация репозитория (CRUD)

Создайте файл `src/repository.rs`:

```rust
// src/repository.rs

use diesel::prelude::*;
use crate::{models::{User, NewUser}, schema::users};
use diesel::PgConnection;

pub struct UserRepository<'a> {
    conn: &'a PgConnection,
}

impl<'a> UserRepository<'a> {
    pub fn new(conn: &'a PgConnection) -> Self {
        Self { conn }
    }

    // Create
    pub fn create(&self, name: &str, email: &str) -> QueryResult<User> {
        let new_user = NewUser { name, email };
        diesel::insert_into(users::table)
            .values(&new_user)
            .get_result(self.conn)
    }

    // Read all
    pub fn get_all(&self) -> QueryResult<Vec<User>> {
        users::table.load::<User>(self.conn)
    }

    // Read by id
    pub fn get_by_id(&self, user_id: i32) -> QueryResult<User> {
        users::table.find(user_id).get_result(self.conn)
    }

    // Update (пример для обновления имени)
    pub fn update_name(&self, user_id: i32, new_name: &str) -> QueryResult<User> {
        diesel::update(users::table.find(user_id))
            .set(users::name.eq(new_name))
            .get_result(self.conn)
    }

    // Delete
    pub fn delete(&self, user_id: i32) -> QueryResult<usize> {
        diesel::delete(users::table.find(user_id))
            .execute(self.conn)
    }
}
```

---

## 5. Основной пример использования

В `src/main.rs`:

```rust
// src/main.rs

use diesel::prelude::*;
use dotenvy::dotenv;
use std::env;

mod schema;
mod models;
mod repository;

fn establish_connection() -> PgConnection {
    dotenv().ok();
    let database_url = env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    PgConnection::establish(&database_url).expect("Error connecting to database")
}

fn main() {
    let connection = establish_connection();
    
    let repo = repository::UserRepository::new(&connection);

    // Создаем нового пользователя
    let user = repo.create("Alice", "alice@example.com").expect("Failed to create user");
    println!("Created user: {} with id {}", user.name, user.id);

    // Получаем всех пользователей
    let users = repo.get_all().expect("Failed to load users");
    println!("All users:");
    for u in users {
        println!("{} - {}", u.id, u.name);
    }

    // Обновляем имя пользователя по id
    let updated_user = repo.update_name(user.id, "Alice Smith").expect("Failed to update");
    println!("Updated user: {} - {}", updated_user.id, updated_user.name);

    // Удаляем пользователя по id
    let deleted_rows = repo.delete(user.id).expect("Failed to delete");
    println!("Deleted {} row(s)", deleted_rows);
}
```

---

## Итог

Этот пример показывает базовую структуру проекта с Diesel + PostgreSQL:
- подключение через `.env`
- миграции для создания таблицы
- модели и схемы для ORM-операций
- репозиторий с CRUD-методами
