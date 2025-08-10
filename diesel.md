# 📘 Diesel ORM — полная шпаргалка (Rust)

---

[Полная шпаргалка по Rust — синтаксис, инициализация, функции](README.md)

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
