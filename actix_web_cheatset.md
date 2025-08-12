# 📘 Actix-web — энциклопедия (таблица API)

---

[Полная шпаргалка по Rust — синтаксис, инициализация, функции](README.md)

[Вопросы к собеседованию Rust разработчика 2023](questions_2023.md)

[Rust — Шпаргалка по `Vec` и `Iterator`](rust_vec_iterator_cheatsheet.md)

[Diesel ORM — полная шпаргалка (Rust)](diesel.md)

---

[1 rest api skeleton crud](#1-rest-api-skeleton-crud-по-items)

[2 auth skeleton jwt login](#2-auth-skeleton-jwt-login--protected-route)

[3 file upload skeleton multipart](#3-file-upload-skeleton-multipart-form-data)

---

## 1. Запуск сервера / базовая инициализация

| API / Конструкт                      |                                                 Описание | Пример                                                                                                                                     |   |                                                                                                 |
| ------------------------------------ | -------------------------------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------ | - | ----------------------------------------------------------------------------------------------- |
| `HttpServer::new`                    |  Создать сервер; принимает замыкание, возвращающее `App` | \`\`\`rust let server = HttpServer::new(                                                                                                   |   | App::new().route("/", web::get().to(index))); server.bind("0.0.0.0:8080")?.run().await?; \`\`\` |
| `App::new()`                         | Корневой контейнер для маршрутов, middleware и состояний | `App::new().wrap(Logger::default()).configure(config)`                                                                                     |   |                                                                                                 |
| `App::configure(fn)`                 |               Группировка регистрации маршрутов/сервисов | `rust fn config(cfg: &mut web::ServiceConfig) { cfg.service(web::resource("/u").route(web::get().to(u))); } App::new().configure(config) ` |   |                                                                                                 |
| `HttpServer::bind` / `bind_rustls`   |                   Привязка к адресу; `bind_rustls` — TLS | `HttpServer::new(...).bind("127.0.0.1:8080")?`                                                                                             |   |                                                                                                 |
| `HttpServer::workers(n)`             |                               Количество рабочих потоков | `HttpServer::new(...).workers(4)`                                                                                                          |   |                                                                                                 |
| `HttpServer::shutdown_timeout(secs)` |                         Время ожидания graceful shutdown | `.shutdown_timeout(30)`                                                                                                                    |   |                                                                                                 |
| `HttpServer::run` / `.await`         |                             Запуск / ожидание завершения | `server.run().await`                                                                                                                       |   |                                                                                                 |

---

## 2. Регистрация маршрутов / сервисов / scope

| API                                    |                                       Описание | Пример                                                                                 |
| -------------------------------------- | ---------------------------------------------: | -------------------------------------------------------------------------------------- |
| `App::route(path, method.to(handler))` |                Быстрая регистрация одного пути | `.route("/hello", web::get().to(hello))`                                               |
| `web::resource("/path")`               |         Ресурс с несколькими методами и guards | `rust web::resource("/item").route(web::get().to(get)).route(web::post().to(create)) ` |
| `web::scope("/api")`                   |            Группировка маршрутов под префиксом | `App::new().service(web::scope("/api").route("/users", web::get().to(list_users)))`    |
| `web::service()`                       | Регистрирует `Service` (например, actix-files) | `.service(fs::Files::new("/static", "./static"))`                                      |
| `web::to(handler_fn)`                  |                   Привязать функцию-обработчик | `.route("/", web::get().to(index))`                                                    |
| `guard::Get() / Post()`                |              Guards для HTTP-методов и условий | `.route("/p", web::route().guard(guard::Post()).to(handler))`                          |

---

## 3. Состояние приложения (shared state)

| API                             |                                                    Описание | Пример                                                                                                                                                             |
| ------------------------------- | ----------------------------------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `web::Data<T>`                  |               Обёртка для шаренного состояния (Arc-подобно) | `rust let app_state = web::Data::new(MyState{ pool }); App::new().app_data(app_state.clone()); async fn idx(data: web::Data<MyState>) { let pool = &data.pool; } ` |
| `app_data` vs `app_data_config` | `app_data` — клонируемое состояние; конфигурационные данные | `App::new().app_data(web::Data::new(cfg))`                                                                                                                         |
| `App::app_data` + extractor     |   Автоматически доступно в обработчике через `web::Data<T>` | см. пример выше                                                                                                                                                    |

---

## 4. Extractors — извлечение данных из запроса

| Extractor                               |                                                             Описание | Пример                                                                                           |
| --------------------------------------- | -------------------------------------------------------------------: | ------------------------------------------------------------------------------------------------ |
| `web::Path<T>`                          |                       Парсит параметры пути (generic T: Deserialize) | `async fn get(Path((id,)): Path<(u32,)>)` или `Path<MyPath>`                                     |
| `web::Query<T>`                         |                                      Парсит query-string в структуру | `async fn list(q: Query<ListQuery>)`                                                             |
| `web::Json<T>`                          |                            Парсит JSON тело (с `serde::Deserialize`) | `async fn create(item: Json<CreateDto>) -> impl Responder`                                       |
| `web::Form<T>`                          |                           Парсит `application/x-www-form-urlencoded` | `async fn submit(f: Form<Login>)`                                                                |
| `HttpRequest`                           |                  Низкоуровневый доступ к headers, uri, method и т.д. | `fn h(req: HttpRequest) { let hdr = req.headers(); }`                                            |
| `HttpResponseBuilder`                   |                                            Создание ответа (builder) | `HttpResponse::Ok().json(obj)`                                                                   |
| `web::Bytes` / `web::Payload`           |                                          Для streaming body / байтов | `async fn upload(mut payload: Payload) { while let Some(chunk) = payload.next().await { ... } }` |
| `actix_web::web::Data<T>`               |                                                      state, см. выше | —                                                                                                |
| `Identity` / `CookieSession` extractors |                                для сессий/идентификации (middleware) | `async fn who(id: Identity) { let user = id.identity(); }`                                       |
| `HttpRequest::match_info()`             |                                      Доступ к параметрам из маршрута | `req.match_info().get("id")`                                                                     |
| `web::FormConfig`, `JsonConfig`         |                                  Настройка лимитов и ошибки парсинга | `App::new().app_data(web::JsonConfig::default().limit(4096))`                                    |
| `actix_web::web::ReqData<T>`            | Извлечение данных, установленных middleware через `req.extensions()` | middleware пишет `req.extensions_mut().insert(MyCtx)`; handler: `fn h(data: ReqData<MyCtx>)`     |

---

## 5. Responders — формирование ответов

| API / Трейт                                |                                                                              Описание | Пример                                                                           |
| ------------------------------------------ | ------------------------------------------------------------------------------------: | -------------------------------------------------------------------------------- |
| `impl Responder`                           | Типы, которые могут быть возвращены из handler (String, \&str, HttpResponse, Json<T>) | `async fn h() -> impl Responder { "ok" }`                                        |
| `HttpResponse::Ok()` etc.                  |                                                               Билдер для всех ответов | `HttpResponse::Created().json(obj)`                                              |
| `HttpResponse::build(StatusCode::XXX)`     |                                               Полный контроль над кодом и заголовками | `HttpResponse::build(StatusCode::ACCEPTED).insert_header(("X-H", "v")).finish()` |
| `web::Json<T>`                             |                                              Возврат JSON: автоматически Content-Type | `HttpResponse::Ok().json(&obj)`                                                  |
| `HttpResponse::Streaming`                  |                                                     Стриминг ответа (stream of Bytes) | `HttpResponse::Ok().streaming(stream)`                                           |
| `NamedFile` (actix\_files)                 |                                                            Отдания статического файла | `NamedFile::open("path")?.into_response(&req)`                                   |
| `Either` / `Result<impl Responder, Error>` |                                                     Возврат разных вариантов и ошибок | `-> Result<impl Responder, actix_web::Error>`                                    |

---

## 6. Middleware (регистрация и типы)

| Middleware                  |                                                  Описание | Пример                                                                              |
| --------------------------- | --------------------------------------------------------: | ----------------------------------------------------------------------------------- |
| `Logger`                    |                   Логирование запросов (actix-web logger) | `.wrap(Logger::default())`                                                          |
| `Cors`                      |                                         CORS конфигурация | `.wrap(Cors::default().allow_any_origin().allowed_methods(vec!["GET","POST"]))`     |
| `Compress`                  |                                            Сжатие ответов | `.wrap(Compress::default())`                                                        |
| `NormalizePath`             |                               Приведение слешей (без/с /) | `.wrap(NormalizePath::new(TrailingSlash::Trim))`                                    |
| `Condition`                 |                                       Условный middleware | `wrap(Condition::new(flag, MyMiddleware))`                                          |
| Пользовательский middleware | Реализовать Middleware trait или использовать `Transform` | `rust struct MyMw; impl<S, B> Transform<S, ServiceRequest> for MyMw { /* ... */ } ` |
| `ErrorHandlers`             |                            Перехват и кастомизация ошибок | `.wrap(ErrorHandlers::new().handler(StatusCode::NOT_FOUND, render_404))`            |

---

## 7. Guards — условий маршрутизации

| API                            |                                   Описание | Пример                                                 |     |        |
| ------------------------------ | -----------------------------------------: | ------------------------------------------------------ | --- | ------ |
| `guard::Get()` / `Post()`      | Переходит, только если method = GET / POST | `.route("/x", web::route().guard(guard::Get()).to(h))` |     |        |
| `guard::Header("X-API", "v1")` |                         Guard по заголовку | `.guard(guard::Header("X-API", "v1"))`                 |     |        |
| Комбинирование guards          |                          Логическое AND/OR | \`guard::fn\_guard(                                    | ctx | ...)\` |

---

## 8. Работа с файлами и статикой

| Функция / Crate                                  |                                       Описание | Пример                                                                          |
| ------------------------------------------------ | ---------------------------------------------: | ------------------------------------------------------------------------------- |
| `actix_files::Files::new("/static", "./public")` |         Статика с кэшированием и range support | `.service(actix_files::Files::new("/static", "./public").show_files_listing())` |
| `NamedFile`                                      |                          Отдать отдельный файл | `NamedFile::open("index.html")?.use_last_modified(true)`                        |
| `fs::NamedFile::open_async`                      |             Асинхронная отправка файла (опции) | —                                                                               |
| Range и partial content                          | actix-files поддерживает Ranges (vids, resume) | —                                                                               |

---

## 9. Multipart / загрузка файлов

| API / Extractor              |                        Описание | Пример                                                                                                                                                                                           |
| ---------------------------- | ------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `actix_multipart::Multipart` | Чтение multipart stream (файлы) | `rust async fn upload(mut payload: Multipart) { while let Some(field) = payload.next().await { let mut f = field.unwrap(); while let Some(chunk) = f.next().await { /* записать байты */ } } } ` |
| `MultipartConfig` / limits   |               Настройка лимитов | `App::new().app_data(web::PayloadConfig::new(262_144))`                                                                                                                                          |

---

## 10. WebSocket (actix-web + tokio/tungstenite)

| API                              |                                  Описание | Пример                                                                                                                              |
| -------------------------------- | ----------------------------------------: | ----------------------------------------------------------------------------------------------------------------------------------- |
| `actix_web_actors::ws`           |     Поддержка WebSocket на основе акторов | `rust async fn ws_index(req: HttpRequest, stream: web::Payload) -> Result<HttpResponse, Error>{ ws::start(MyWs{}, &req, stream) } ` |
| `ws::start`                      |                Запустить WebSocket-актора | см. выше                                                                                                                            |
| `actix_ws` / `tokio_tungstenite` |        Альтернативы для неблокирующего ws | —                                                                                                                                   |
| `Message` события                | `Text`, `Binary`, `Ping`, `Pong`, `Close` | В `Actor::handle` обрабатывай `ws::Message`                                                                                         |

---

## 11. SSE / Server-Sent Events и Streaming

| API                                    |                                     Описание | Пример                               |   |                                                                                                                                                     |
| -------------------------------------- | -------------------------------------------: | ------------------------------------ | - | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `HttpResponse::Ok().streaming(stream)` | Отдать `Stream<Item = Result<Bytes, Error>>` | \`\`\`rust let s = stream::unfold(0, | i | async move { Some((Ok(Bytes::from(format!("data: {}\n\n", i))), i+1)) }); HttpResponse::Ok().content\_type("text/event-stream").streaming(s) \`\`\` |
| `web::Bytes`                           |                            байтовый фрагмент | —                                    |   |                                                                                                                                                     |

---

## 12. Авторизация / Аутентификация

| API / Подход                      |                                               Описание | Пример                                                                               |
| --------------------------------- | -----------------------------------------------------: | ------------------------------------------------------------------------------------ |
| `Identity` (actix-identity)       |                           Cookie-based identity helper | `Identity::login(&id, user_id.to_string())`                                          |
| JWT                               |               Проверка токена в middleware / extractor | middleware читает авторизационный header, верифицирует JWT                           |
| `req.extensions()` + `ReqData<T>` | Middleware сохраняет контекст пользователя для handler | middleware: `req.extensions_mut().insert(UserCtx{...})`; handler: `ReqData<UserCtx>` |

---

## 13. Ошибки и обработка

| API                                            |                                               Описание | Пример                                                                                 |   |                                        |
| ---------------------------------------------- | -----------------------------------------------------: | -------------------------------------------------------------------------------------- | - | -------------------------------------- |
| `actix_web::error::Error`                      |                    Стандартный тип ошибок для handlers | `-> Result<HttpResponse, Error>`                                                       |   |                                        |
| `HttpResponse::InternalServerError().finish()` |                                            Возврат 500 | —                                                                                      |   |                                        |
| `map_err` / `?`                                | Преобразование ошибок с `map_err` в `actix_web::Error` | \`fs::read().await.map\_err(                                                           | e | error::ErrorInternalServerError(e))?\` |
| `ResponseError` trait                          |   Реализовать для custom error -> HttpResponse mapping | `rust impl ResponseError for MyError { fn status_code(&self) -> StatusCode { ... } } ` |   |                                        |
| `ErrorHandlers` middleware                     |       Перехват и кастомизация статичных страниц ошибок | `.wrap(ErrorHandlers::new().handler(StatusCode::INTERNAL_SERVER_ERROR, render500))`    |   |                                        |

---

## 14. Тестирование handler'ов

| API                              |                            Описание | Пример                                                                                                |
| -------------------------------- | ----------------------------------: | ----------------------------------------------------------------------------------------------------- |
| `actix_web::test::TestRequest`   |      Формирование тестового запроса | `let req = TestRequest::get().uri("/").to_request(); let resp = test::call_service(&app, req).await;` |
| `test::init_service(App::new())` | Инициализация приложения для тестов | `let app = test::init_service(App::new().route("/", web::get().to(index))).await;`                    |
| `test::call_and_read_body`       |          Вызов и чтение тела ответа | `let body = test::call_and_read_body(&app, req).await;`                                               |

---

## 15. Логирование и трассировка

| API / Crate                     |                           Описание | Пример                                                                       |
| ------------------------------- | ---------------------------------: | ---------------------------------------------------------------------------- |
| `Logger` middleware             |       Простое логирование запросов | `.wrap(Logger::default())`                                                   |
| `tracing` + `tracing-actix-web` |      Интеграция structured tracing | `tracing_subscriber::fmt::init(); App::new().wrap(TracingLogger::default())` |
| `RequestId` middleware          | Проставляет request-id в заголовок | `wrap(RequestId::new())`                                                     |

---

## 16. CORS и безопасность заголовков

| API               |                                          Описание | Пример                                                                                                   |
| ----------------- | ------------------------------------------------: | -------------------------------------------------------------------------------------------------------- |
| `Cors::default()` |                                    Настройка CORS | `rust .wrap(Cors::default().allowed_origin("https://example.com").allowed_methods(vec!["GET","POST"])) ` |
| `DefaultHeaders`  |                       Добавление security headers | `.wrap(DefaultHeaders::new().add(("X-Frame-Options", "DENY")))`                                          |
| `Rate-limiting`   | Часто реализуется на middleware или reverse-proxy | встроенной реализации нет; использовать `tower`/rate-limiter перед приложением                           |

---

## 17. Performance / настройки / release

| Тема                           |                                                        Рекомендации |
| ------------------------------ | ------------------------------------------------------------------: |
| `workers`                      |        Количество рабочих потоков под нагрузку (обычно = CPU cores) |
| `keep_alive`, `client_timeout` |                                          tune для долгих соединений |
| `enable-lto`, `opt-level`      |                           Собирай `--release` с LTO и оптимизациями |
| `use-actix-web` + `tokio`      | actix-web использует tokio runtime — следи за feature-ми и версиями |
| Static files                   |                        offload to CDN / nginx при высоких нагрузках |

---

## 18. Deployment / Docker / TLS

| Тема               |                                                             Пример / Совет |
| ------------------ | -------------------------------------------------------------------------: |
| Docker multi-stage | Сборка `cargo build --release` в builder и копирование бинаря в slim образ |
| TLS termination    |                          TLS на LB/nginx или `bind_rustls` в самом сервере |
| Health checks      |                            `/healthz` endpoints и readiness probes для k8s |
| Logging            |                      structured JSON logs + central collector (ELK/Vector) |

---

## 19. Полезные шаблоны кода (короткие примеры)

**Простой handler с state, JSON и error:**

```rust
use actix_web::{web, HttpResponse, Error};

async fn create_item(data: web::Data<AppState>, payload: web::Json<CreateDto>) -> Result<HttpResponse, Error> {
    let dto = payload.into_inner();
    let id = data.db.insert(dto).await.map_err(|e| actix_web::error::ErrorInternalServerError(e))?;
    Ok(HttpResponse::Created().json(CreateResp { id }))
}
```

**Регистрация scope и middleware:**

```rust
App::new()
    .wrap(Logger::default())
    .service(
        web::scope("/api")
           .wrap(AuthMiddleware)
           .route("/users", web::get().to(list_users))
    )
```

**WebSocket actor:**

```rust
use actix::prelude::*;
use actix_web_actors::ws;

struct MyWs;
impl Actor for MyWs { type Context = ws::WebsocketContext<Self>; }
impl StreamHandler<Result<ws::Message, ws::ProtocolError>> for MyWs {
    fn handle(&mut self, msg: Result<ws::Message, ws::ProtocolError>, ctx: &mut Self::Context) {
        if let Ok(ws::Message::Text(t)) = msg { ctx.text(format!("Echo: {}", t)); }
    }
}
```

---

## 20. Частые ошибки и советы

* **Не пропускай `.await`** при вызовах асинхронных операций — компилятор подскажет.
* Для больших боди задавай лимиты через `JsonConfig` / `PayloadConfig`.
* Используй `web::Data` для общих ресурсов (DB pool) и избегай `static mut`.
* В проде — TLS termination и rate-limiting на уровне LB.
* Тестируй handlers через `actix_web::test` — это позволяет покрыть real routing logic.
* При высоких нагрузках отдавай статические ресурсы не через app, а через CDN / nginx.

---

# 1-rest-api-skeleton-crud-по-items

**Cargo.toml**

```toml
[package]
name = "rest_api_skeleton"
version = "0.1.0"
edition = "2021"

[dependencies]
actix-web = "4"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tokio = { version = "1", features = ["macros", "rt-multi-thread"] }
```

**src/main.rs**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Clone)]
struct Item {
    id: u32,
    name: String,
}

async fn list_items() -> impl Responder {
    HttpResponse::Ok().json(vec![
        Item { id: 1, name: "Item1".into() },
        Item { id: 2, name: "Item2".into() },
    ])
}

async fn get_item(path: web::Path<u32>) -> impl Responder {
    HttpResponse::Ok().json(Item { id: path.into_inner(), name: "Example".into() })
}

async fn create_item(item: web::Json<Item>) -> impl Responder {
    HttpResponse::Created().json(item.into_inner())
}

async fn delete_item(path: web::Path<u32>) -> impl Responder {
    HttpResponse::NoContent().finish()
}

#[tokio::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/items", web::get().to(list_items))
            .route("/items/{id}", web::get().to(get_item))
            .route("/items", web::post().to(create_item))
            .route("/items/{id}", web::delete().to(delete_item))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

---

# 2-auth-skeleton-jwt-login--protected-route

**Cargo.toml**

```toml
[package]
name = "auth_skeleton"
version = "0.1.0"
edition = "2021"

[dependencies]
actix-web = "4"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
jsonwebtoken = "8"
tokio = { version = "1", features = ["macros", "rt-multi-thread"] }
```

**src/main.rs**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse, HttpRequest};
use serde::{Deserialize, Serialize};
use jsonwebtoken::{encode, decode, Header, Validation, EncodingKey, DecodingKey};

const SECRET: &[u8] = b"supersecretkey";

#[derive(Serialize, Deserialize)]
struct Claims {
    sub: String,
    exp: usize,
}

#[derive(Deserialize)]
struct Login {
    username: String,
    password: String,
}

async fn login(creds: web::Json<Login>) -> impl Responder {
    if creds.username == "admin" && creds.password == "password" {
        let claims = Claims { sub: creds.username.clone(), exp: 2000000000 };
        let token = encode(&Header::default(), &claims, &EncodingKey::from_secret(SECRET)).unwrap();
        HttpResponse::Ok().json(serde_json::json!({ "token": token }))
    } else {
        HttpResponse::Unauthorized().finish()
    }
}

async fn protected(req: HttpRequest) -> impl Responder {
    if let Some(auth) = req.headers().get("Authorization") {
        let token = auth.to_str().unwrap().replace("Bearer ", "");
        if decode::<Claims>(&token, &DecodingKey::from_secret(SECRET), &Validation::default()).is_ok() {
            return HttpResponse::Ok().body("Welcome to protected area");
        }
    }
    HttpResponse::Unauthorized().finish()
}

#[tokio::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/login", web::post().to(login))
            .route("/protected", web::get().to(protected))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

---

# 3-file-upload-skeleton-multipart-form-data

**Cargo.toml**

```toml
[package]
name = "file_upload_skeleton"
version = "0.1.0"
edition = "2021"

[dependencies]
actix-web = "4"
actix-multipart = "0.4"
tokio = { version = "1", features = ["macros", "rt-multi-thread"] }
futures-util = "0.3"
```

**src/main.rs**

```rust
use actix_multipart::Multipart;
use actix_web::{web, App, HttpServer, Responder, HttpResponse, Error};
use futures_util::StreamExt;
use std::io::Write;

async fn upload_file(mut payload: Multipart) -> Result<HttpResponse, Error> {
    while let Some(field) = payload.next().await {
        let mut field = field?;
        let content_disposition = field.content_disposition().unwrap();
        let filename = content_disposition.get_filename().unwrap();
        let filepath = format!("./uploads/{}", sanitize_filename::sanitize(&filename));

        let mut f = std::fs::File::create(filepath)?;
        while let Some(chunk) = field.next().await {
            f.write_all(&chunk?)?;
        }
    }
    Ok(HttpResponse::Ok().body("File uploaded"))
}

#[tokio::main]
async fn main() -> std::io::Result<()> {
    std::fs::create_dir_all("./uploads")?;

    HttpServer::new(|| {
        App::new()
            .route("/upload", web::post().to(upload_file))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

---
