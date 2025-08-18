## Гид по темам для изучения Actix Web

### 1. **Начало работы**

* **Установка и базовый пример**
  Подключите актуальную версию:

  ```toml
  [dependencies]
  actix-web = "4"
  ```

  Официальная инструкция по созданию первого приложения: ([actix.rs][1])
* **Мини-пример “Hello, world!”**

  ```rust
  use actix_web::{get, App, HttpServer, Responder};

  #[get("/hey/{name}")]
  async fn greet(name: actix_web::web::Path<String>) -> impl Responder {
      format!("Hello {}!", name)
  }

  #[actix_web::main]
  async fn main() -> std::io::Result<()> {
      HttpServer::new(|| App::new().service(greet))
          .bind(("127.0.0.1", 8080))?
          .run()
          .await
  }
  ```

  Пример доступен в документации: ([Docs.rs][2])

---

### 2. **Роутинг & Обработчики (Handlers)**

* **Маршруты с использованием макросов** (`#[get]`, `#[post]` и т.д.), ручное добавление через `.route()` или `.service(...)`. ([actix.rs][1], [Docs.rs][2])
* **Обработчики** могут быть `async fn`, возвращающими тип, реализующий `Responder`. ([actix.rs][3])

---

### 3. **Extractors (Извлечение данных)**

* Типобезопасное извлечение данных из URL-параметров, query, тела запроса.
* Поддерживаются `Path<T>`, `Query<T>`, `Json<T>`, `Form<T>`, где `T: Deserialize`. ([Shuttle][4])
* Вдобавок можно создавать пользовательские извлекатели, реализуя `FromRequest`. ([Shuttle][4])

---

### 4. **Состояние приложения (App State)**

* Общий контекст через `web::Data<T>` — строка к базе, конфигурации, и т.д.

  ```rust
  struct AppState { counter: std::sync::Mutex<i32> }
  .app_data(web::Data::new(AppState { counter: Mutex::new(0) }))
  ```

  Детальный пример: ([Shuttle][4])

---

### 5. **Middleware**

* Встроенные: `Logger`, `Compression`, `CORS`, и др. ([Docs.rs][2])
* Пользовательские middleware можно создавать вручную или через `actix-web-lab::from_fn`. ([Shuttle][4])

---

### 6. **Статические файлы и шаблоны**

* **Статика** — `actix_files::Files`:

  ```rust
  .service(actix_files::Files::new("/static", "./static").use_last_modified(true))
  ```

  ([Shuttle][4])
* **Шаблоны** — интеграция с Askama или Tera:

  ```rust
  #[derive(Template)]
  #[template(path = "index.html")]
  struct Index { name: String }
  ```

  ([Shuttle][4])

---

### 7. **Работа с WebSocket**

* **Актор-основа**: WebSocket поддерживается через актор-модель (`actix-web-actors`) и Actor API. ([GitHub][5], [Reddit][6])
* Общий туториал доступен на Reddit: ([Reddit][6])

---

### 8. **Асинхронность и совместимость с Tokio**

* Actix Web построен на async/await и совместим с Tokio runtime. ([Docs.rs][2])

---

### 9. **Компрессия, HTTP/2, SSL**

* Поддержка `gzip`, `brotli`, `zstd`.
* Работа с `rustls` или `openssl` для HTTPS: ([Docs.rs][2])

---

### 10. **Клиент HTTP (awc)**

* Встроенный асинхронный HTTP-клиент для внутренних запросов.

---

### 11. **Базы данных и интеграции**

* Примеры с SQLx, Diesel, Mongo: ([GitHub][5])
* Немножко приложений с микросервисами, JWT, Docker: ([GitHub][5])

---

### 12. **Логирование и трассировка**

* Логгер через `env_logger`, `tracing`, middleware Logger. ([Shuttle][4])

---

### 13. **Тестирование**

* Использование `actix_web::test` для юнит- и интеграционных тестов (не нашли напрямую, но доступны в `examples/`). ([GitHub][5])

---

### 14. **Роутинг, шаблоны и практики**

* Полезные статьи и сравнительные гайты (LogRocket): ([LogRocket Blog][7])
* Книги: “Actix Web: The Ultimate Guide (2023)”: ([Mastering Backend][8])

---

### 15. **Примеры и шаблоны**

* \[actix/examples на GitHub] — огромное количество готовых примеров: серверы, WebSocket, TLS, базы данных, шаблоны, и многое другое: ([GitHub][5])

---

## Тематическая таблица

| №  | Тема                  | Зачем изучить                       | Ссылки                       |
| -- | --------------------- | ----------------------------------- | ---------------------------- |
| 1  | Getting Started       | Основы и первый сервер              | ([actix.rs][1])              |
| 2  | Роутинг и Extractors  | Обработка запросов                  | ([Shuttle][4], [Docs.rs][2]) |
| 3  | App State             | Общий контекст приложения           | ([Shuttle][4])               |
| 4  | Middleware            | Обработка запросов до/после Handler | ([Shuttle][4])               |
| 5  | Статика и шаблоны     | Отдача файлов, динамика             | ([Shuttle][4])               |
| 6  | WebSocket (Actors)    | Реальное время, WebSockets          | ([GitHub][5], [Reddit][6])   |
| 7  | SSL & HTTP/2          | Безопасность, новейшие протоколы    | ([Docs.rs][2])               |
| 8  | БД & Интеграции       | Сохранение и обработка данных       | ([GitHub][5])                |
| 9  | Логирование и tracing | Мониторинг, отладка                 | ([Shuttle][4])               |
| 10 | Примеры и Boilerplate | Быстрый старт и готовые шаблоны     | ([GitHub][5])                |

---

[1]: https://actix.rs/docs/getting-started/?utm_source=chatgpt.com "Getting Started"
[2]: https://docs.rs/actix-web?utm_source=chatgpt.com "actix_web - Rust"
[3]: https://actix.rs/?utm_source=chatgpt.com "Actix Web"
[4]: https://www.shuttle.dev/blog/2023/12/15/using-actix-rust?utm_source=chatgpt.com "Getting Started with Actix Web in Rust"
[5]: https://github.com/actix/examples?utm_source=chatgpt.com "Community showcase and examples of Actix Web ..."
[6]: https://www.reddit.com/r/rust/comments/k28w0v/a_tutorial_for_websockets_in_actix_web/?utm_source=chatgpt.com "A tutorial for websockets in actix web! : r/rust"
[7]: https://blog.logrocket.com/actix-web-adoption-guide/?utm_source=chatgpt.com "Actix Web adoption guide: Overview, examples, and ..."
[8]: https://masteringbackend.com/posts/actix-web-the-ultimate-guide?utm_source=chatgpt.com "Actix Web: The Ultimate Guide (2023)"

---

---

---

Пример базового API на `actix-web`, который использует ранее созданный репозиторий для доступа к базе данных PostgreSQL. В этом примере реализованы основные CRUD-операции для ресурса `User`.

---

## 1. Обновление зависимостей

Добавьте в `Cargo.toml`:

```toml
[dependencies]
actix-web = "4"
diesel = { version = "2.0.0", features = ["postgres"] }
dotenvy = "0.15"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tokio = { version = "1", features = ["full"] }
```

---

## 2. Структура проекта

Обновим структуру, чтобы включить API:

```
src/
├── main.rs
├── schema.rs
├── models.rs
├── repository.rs
└── handlers.rs
```

---

## 3. Реализация API в `src/handlers.rs`

Создайте файл `src/handlers.rs`:

```rust
use actix_web::{web, HttpResponse, Responder};
use crate::repository::UserRepository;
use crate::models::{User, NewUser};
use diesel::prelude::*;
use serde::{Deserialize, Serialize};

#[derive(Serialize)]
struct ApiResponse<T> {
    data: T,
}

// Создать пользователя
#[derive(Deserialize)]
struct CreateUserRequest {
    name: String,
    email: String,
}

pub async fn create_user(
    repo: web::Data<UserRepository>,
    info: web::Json<CreateUserRequest>,
) -> impl Responder {
    let user = repo.create(&info.name, &info.email);
    match user {
        Ok(u) => HttpResponse::Ok().json(ApiResponse { data: u }),
        Err(e) => HttpResponse::InternalServerError().body(e.to_string()),
    }
}

// Получить всех пользователей
pub async fn get_users(repo: web::Data<UserRepository>) -> impl Responder {
    match repo.get_all() {
        Ok(users) => HttpResponse::Ok().json(ApiResponse { data: users }),
        Err(e) => HttpResponse::InternalServerError().body(e.to_string()),
    }
}

// Получить пользователя по id
pub async fn get_user_by_id(
    repo: web::Data<UserRepository>,
    path: web::Path<i32>,
) -> impl Responder {
    let user_id = path.into_inner();
    match repo.get_by_id(user_id) {
        Ok(user) => HttpResponse::Ok().json(ApiResponse { data: user }),
        Err(diesel::result::Error::NotFound) => HttpResponse::NotFound().body("User not found"),
        Err(e) => HttpResponse::InternalServerError().body(e.to_string()),
    }
}

// Обновить имя пользователя по id
#[derive(Deserialize)]
struct UpdateUserName {
    name: String,
}

pub async fn update_user_name(
    repo: web::Data<UserRepository>,
    path: web::Path<i32>,
    info: web::Json<UpdateUserName>,
) -> impl Responder {
    let user_id = path.into_inner();
    match repo.update_name(user_id, &info.name) {
        Ok(user) => HttpResponse::Ok().json(ApiResponse { data: user }),
        Err(diesel::result::Error::NotFound) => HttpResponse::NotFound().body("User not found"),
        Err(e) => HttpResponse::InternalServerError().body(e.to_string()),
    }
}

// Удалить пользователя по id
pub async fn delete_user(
    repo: web::Data<UserRepository>,
    path: web::Path<i32>,
) -> impl Responder {
    let user_id = path.into_inner();
    match repo.delete(user_id) {
        Ok(count) => {
            if count > 0 {
                HttpResponse::Ok().body("User deleted")
            } else {
                HttpResponse::NotFound().body("User not found")
            }
        }
        Err(e) => HttpResponse::InternalServerError().body(e.to_string()),
    }
}
```

---

## 4. Основной файл `src/main.rs`

Обновите `main.rs`:

```rust
use actix_web::{web, App, HttpServer};
use dotenvy::dotenv;
use std::env;

mod schema;
mod models;
mod repository;
mod handlers;

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    dotenv().ok();

    // Установление соединения с базой данных
    let database_url =
        env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    
    // Создаем пул соединений (можно использовать diesel r2d2 или просто одно соединение)
    
    // Для простоты используем одно соединение:
    let conn = repository::<diesel_async_postgres>::establish_connection(&database_url);
    
    // Оборачиваем в Arc или Data для передачи в обработчики
    let repo = web::Data::new(repository::<diesel_async_postgres>(&database_url));

    println!("Starting server at http://127.0.0.1:8080");

    HttpServer::new(move || {
        App::new()
            .app_data(repo.clone())
            .service(
                web scope("/api")
                    .route("/users", web.post().to(handlers::create_user))
                    .route("/users", web.get().to(handlers::get_users))
                    .route("/users/{id}", web.get().to(handlers::get_user_by_id))
                    .route("/users/{id}", web->put().to(handlers->update_user_name))
                    .route("/users/{id}", web->delete().to(handlers->delete_user))
            )
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Обратите внимание:** В реальности лучше использовать пул соединений (например, через `r2d2`) и асинхронные драйверы (`diesel_async`). Для простоты здесь показан синхронный пример.

---
