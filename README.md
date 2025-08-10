# 📚 Полная шпаргалка по Rust — синтаксис, инициализация, функции

## 1. **Переменные и константы**

```rust
let x = 5;        // неизменяемая
let mut y = 10;   // изменяемая
const PI: f64 = 3.1415; // константа времени компиляции
static GLOBAL: i32 = 42; // глобальная статическая переменная
```

---

## 2. **Типы данных**

```rust
let a: i32 = 10;      // целое
let b: f64 = 3.14;    // число с плавающей точкой
let c: bool = true;   // логический
let d: char = 'R';    // символ
let s: &str = "Hi";   // строковый литерал
let st: String = String::from("Hello"); // строка в куче
```

---

## 3. **Массивы и векторы**

```rust
let arr = [1, 2, 3];           // фиксированный массив
let mut v = vec![1, 2, 3];     // динамический вектор
v.push(4);
```

---

## 4. **Кортежи**

```rust
let tup: (i32, f64, &str) = (1, 2.0, "hi");
let (x, y, z) = tup;
println!("{}", tup.0);
```

---

## 5. **Условные выражения**

```rust
if x > 0 {
    println!("Positive");
} else if x < 0 {
    println!("Negative");
} else {
    println!("Zero");
}
```

---

## 6. **Циклы**

```rust
for i in 0..5 { println!("{}", i); }
while x < 10 { x += 1; }
loop { break; }
```

---

## 7. **Функции**

```rust
fn sum(a: i32, b: i32) -> i32 {
    a + b
}
```

---

## 8. **Структуры**

```rust
struct User {
    name: String,
    age: u32,
}

impl User {
    fn new(name: String, age: u32) -> Self {
        Self { name, age }
    }
}
```

---

## 9. **Enum**

```rust
enum Color {
    Red,
    Green,
    Rgb(u8, u8, u8),
}

let c = Color::Rgb(255, 0, 0);
```

---

## 10. **Pattern Matching**

```rust
match c {
    Color::Red => println!("Red"),
    Color::Rgb(r, g, b) => println!("RGB({}, {}, {})", r, g, b),
    _ => println!("Other"),
}
```

---

## 11. **Option и Result**

```rust
fn find(x: i32) -> Option<i32> {
    if x > 0 { Some(x) } else { None }
}

fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 { Err(String::from("Division by zero")) }
    else { Ok(a / b) }
}
```

---

## 12. **Итераторы и коллекции**

```rust
let nums = vec![1, 2, 3];
let sum: i32 = nums.iter().sum();
let squares: Vec<_> = nums.iter().map(|x| x * x).collect();
```

---

## 13. **Строки**

```rust
let mut s = String::from("Hi");
s.push('!');
s.push_str(" Rust");
println!("{}", s.len());
```

---

## 14. **Borrowing**

```rust
fn print_str(s: &String) { println!("{}", s); }
fn modify(s: &mut String) { s.push_str(" world"); }
```

---

## 15. **Smart Pointers**

```rust
use std::rc::Rc;
let a = Rc::new(5);
let b = Rc::clone(&a);
println!("{}", Rc::strong_count(&a));
```

---

## 16. **Обработка ошибок**

```rust
let file = std::fs::File::open("data.txt")
    .expect("Cannot open file");

let content = std::fs::read_to_string("data.txt")?;
```

---

## 17. **Модули**

```rust
mod utils {
    pub fn hello() { println!("Hello"); }
}

utils::hello();
```

---

## 18. **Async/Await**

```rust
use tokio::time::{sleep, Duration};

async fn task() {
    sleep(Duration::from_secs(1)).await;
    println!("Done");
}
```

---

## 19. **Макросы**

```rust
println!("Hello {}", "Rust");
vec![1, 2, 3];
```

---

## 20. **Часто используемые методы**

```rust
let v = vec![1, 2, 3];
v.contains(&2);
v.is_empty();
"abc".to_uppercase();
"abc,def".split(',').collect::<Vec<_>>();
```

---

# 🚀 Продвинутые фишки Rust

## 21. **Generics**

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}
```

---

## 22. **Trait Bounds**

```rust
trait Summary {
    fn summarize(&self) -> String;
}

fn notify<T: Summary>(item: T) {
    println!("{}", item.summarize());
}
```

---

## 23. **Lifetimes**

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

---

## 24. **Associated Types**

```rust
trait IteratorExt {
    type Item;
    fn next_val(&mut self) -> Option<Self::Item>;
}
```

---

## 25. **Unsafe**

```rust
unsafe {
    let mut x: i32 = 10;
    let r: *mut i32 = &mut x;
    *r += 1;
}
```

---

## 26. **macro\_rules!**

```rust
macro_rules! say_hello {
    () => {
        println!("Hello!");
    };
}

say_hello!();
```

---

## 27. **Builder Pattern**

```rust
struct Config { port: u16, host: String }

impl Config {
    fn new() -> Self { Self { port: 8080, host: "localhost".into() } }
    fn port(mut self, port: u16) -> Self { self.port = port; self }
}
```

---

## 28. **Iterator Chaining**

```rust
let sum: i32 = (1..=5).filter(|x| x % 2 == 0).map(|x| x * x).sum();
```

---

## 29. **Custom Iterator**

```rust
struct Counter { count: u32 }
impl Iterator for Counter {
    type Item = u32;
    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        if self.count < 6 { Some(self.count) } else { None }
    }
}
```

---

## 30. **Arc + Mutex**

```rust
use std::sync::{Arc, Mutex};
let counter = Arc::new(Mutex::new(0));
```

---