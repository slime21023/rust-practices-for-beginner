# 第七章 - 資料結構與行為模式

Rust 不採用傳統的類別繼承，而是透過結構體 (Structs)、列舉 (Enums) 和 Trait 系統來實現資料組織和行為封裝。本章將介紹這些核心概念，幫助你理解 Rust 如何實現程式碼的模組化、重用性和可維護性。

## 1. 結構體 (Structs)

### 核心概念

- 結構體用於組合相關資料的自訂型別
- 資料與行為分離：結構體定義資料，方法透過 `impl` 區塊定義
- 支援三種形式：標準結構體、元組結構體、單元式結構體

### 基本語法

```rust
// 標準結構體
struct User {
    username: String,
    email: String,
    active: bool,
    sign_in_count: u64,
}

// 元組結構體
struct Color(u8, u8, u8);

// 單元式結構體
struct AlwaysEqual;

fn main() {
    // 創建實例
    let user1 = User {
        email: String::from("user@example.com"),
        username: String::from("username123"),
        active: true,
        sign_in_count: 1,
    };

    // 存取欄位
    println!("User: {}", user1.username);

    // 可變實例
    let mut user2 = User {
        email: String::from("another@example.com"),
        username: String::from("another"),
        active: true,
        sign_in_count: 1,
    };
    user2.email = String::from("newemail@example.com");

    // 元組結構體
    let black = Color(0, 0, 0);
    println!("RGB: ({}, {}, {})", black.0, black.1, black.2);
}
```

## 2. 方法 (Methods)

### 核心概念

- 方法是與特定型別相關聯的函式
- 第一個參數通常是 `self`、`&self` 或 `&mut self`
- 關聯函式不以 `self` 作為參數，常用作建構函式

### 基本語法

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // 關聯函式（建構函式）
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }

    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }

    // 實例方法 - 不可變借用
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // 實例方法 - 可變借用
    fn set_width(&mut self, width: u32) {
        self.width = width;
    }

    // 實例方法 - 取得所有權
    fn destroy(self) {
        println!("Destroying rectangle");
    }
}

fn main() {
    let rect = Rectangle::new(30, 50);
    let mut square = Rectangle::square(20);
    
    println!("Area: {}", rect.area());
    square.set_width(25);
    
    rect.destroy(); // rect 被移動，不能再使用
}
```

## 3. 列舉 (Enums)

### 核心概念

- 列舉定義一個可以表示多種變體的型別
- 每個變體可以攜帶不同型別的資料
- 與模式匹配結合使用，提供詳盡的狀態檢查

### 基本語法

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(u8, u8, u8),
}

impl Message {
    fn process(&self) {
        match self {
            Message::Quit => println!("Quit"),
            Message::Move { x, y } => println!("Move to ({}, {})", x, y),
            Message::Write(text) => println!("Text: {}", text),
            Message::ChangeColor(r, g, b) => println!("Color: RGB({}, {}, {})", r, g, b),
        }
    }
}

fn main() {
    let messages = vec![
        Message::Quit,
        Message::Move { x: 10, y: 20 },
        Message::Write(String::from("Hello")),
        Message::ChangeColor(255, 0, 0),
    ];

    for msg in messages {
        msg.process();
    }
}
```

### Option 和 Result

```rust
fn main() {
    // Option<T> - 處理可能不存在的值
    let some_number = Some(5);
    let no_number: Option<i32> = None;

    match some_number {
        Some(n) => println!("Number: {}", n),
        None => println!("No number"),
    }

    // if let 語法糖
    if let Some(n) = some_number {
        println!("Got number: {}", n);
    }

    // Result<T, E> - 處理可能失敗的操作
    fn divide(a: f64, b: f64) -> Result<f64, String> {
        if b == 0.0 {
            Err(String::from("Division by zero"))
        } else {
            Ok(a / b)
        }
    }

    match divide(10.0, 2.0) {
        Ok(result) => println!("Result: {}", result),
        Err(error) => println!("Error: {}", error),
    }
}
```

## 4. Trait (共享行為)

### 核心概念

- Trait 類似於其他語言的介面，定義共享行為
- 支援預設實作和必須實作的方法
- 與泛型結合實現靜態多型

### 基本語法

```rust
trait Summary {
    // 必須實作的方法
    fn summarize_author(&self) -> String;

    // 有預設實作的方法
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

struct NewsArticle {
    headline: String,
    author: String,
    content: String,
}

struct Tweet {
    username: String,
    content: String,
}

impl Summary for NewsArticle {
    fn summarize_author(&self) -> String {
        format!("@{}", self.author)
    }
}

impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }

    // 覆寫預設實作
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

// Trait 作為參數
fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// 泛型語法
fn notify_generic<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

fn main() {
    let article = NewsArticle {
        headline: String::from("News Title"),
        author: String::from("John"),
        content: String::from("Content..."),
    };

    let tweet = Tweet {
        username: String::from("user123"),
        content: String::from("Hello world!"),
    };

    notify(&article);
    notify(&tweet);
}
```

### Trait Bounds 和多重約束

```rust
use std::fmt::Display;

// 多重 Trait 約束
fn notify_multi<T: Summary + Display>(item: &T) {
    println!("{}", item);
}

// where 子句（複雜約束時更清晰）
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // 函式實作
    0
}

// 返回實作 Trait 的型別
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
    }
}
```

## 5. Trait 物件 (動態分派)

### 核心概念

- 允許在執行時處理不同型別的實例
- 使用 `dyn Trait` 語法創建 Trait 物件
- 實現動態多型，但有運行時成本

### 基本語法

```rust
trait Drawable {
    fn draw(&self);
}

struct Button {
    label: String,
}

struct TextField {
    placeholder: String,
}

impl Drawable for Button {
    fn draw(&self) {
        println!("Drawing button: {}", self.label);
    }
}

impl Drawable for TextField {
    fn draw(&self) {
        println!("Drawing text field: {}", self.placeholder);
    }
}

struct Screen {
    components: Vec<Box<dyn Drawable>>,
}

impl Screen {
    fn run(&self) {
        for component in self.components.iter() {
            component.draw(); // 動態分派
        }
    }
}

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(Button {
                label: String::from("OK"),
            }),
            Box::new(TextField {
                placeholder: String::from("Enter text..."),
            }),
        ],
    };

    screen.run();
}
```

## 6. 模組系統

### 核心概念

- 模組用於組織程式碼和控制可見性
- 預設情況下項目是私有的，使用 `pub` 公開
- `use` 關鍵字將路徑引入作用域

### 基本語法

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {
            println!("Added to waitlist");
        }

        fn seat_at_table() {
            println!("Seated at table");
        }
    }

    mod serving {
        fn take_order() {}
        fn serve_order() {}
    }
}

// 引入路徑
use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    // 絕對路徑
    crate::front_of_house::hosting::add_to_waitlist();

    // 使用 use 後的簡化路徑
    hosting::add_to_waitlist();
}

// 結構體欄位可見性
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,      // 公開欄位
        seasonal_fruit: String, // 私有欄位
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

fn main() {
    let mut meal = back_of_house::Breakfast::summer("Rye");
    meal.toast = String::from("Wheat"); // 可以修改公開欄位
    println!("I'd like {} toast", meal.toast);
    // meal.seasonal_fruit = String::from("berries"); // 錯誤：私有欄位
}
```

## 7. 實用範例

### 狀態機模式

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

impl TrafficLight {
    fn next(&self) -> TrafficLight {
        match self {
            TrafficLight::Red => TrafficLight::Green,
            TrafficLight::Yellow => TrafficLight::Red,
            TrafficLight::Green => TrafficLight::Yellow,
        }
    }

    fn duration(&self) -> u32 {
        match self {
            TrafficLight::Red => 30,
            TrafficLight::Yellow => 5,
            TrafficLight::Green => 25,
        }
    }
}
```

### 建造者模式

```rust
struct Config {
    host: String,
    port: u16,
    debug: bool,
}

impl Config {
    fn new() -> ConfigBuilder {
        ConfigBuilder::default()
    }
}

struct ConfigBuilder {
    host: Option<String>,
    port: Option<u16>,
    debug: bool,
}

impl ConfigBuilder {
    fn default() -> Self {
        ConfigBuilder {
            host: None,
            port: None,
            debug: false,
        }
    }

    fn host(mut self, host: &str) -> Self {
        self.host = Some(host.to_string());
        self
    }

    fn port(mut self, port: u16) -> Self {
        self.port = Some(port);
        self
    }

    fn debug(mut self, debug: bool) -> Self {
        self.debug = debug;
        self
    }

    fn build(self) -> Config {
        Config {
            host: self.host.unwrap_or_else(|| "localhost".to_string()),
            port: self.port.unwrap_or(8080),
            debug: self.debug,
        }
    }
}

fn main() {
    let config = Config::new()
        .host("example.com")
        .port(3000)
        .debug(true)
        .build();
}
```

## 8. 練習題

### 基礎練習

1. **幾何形狀**：
   ```rust
   struct Circle {
       radius: f64,
   }
   
   // 實作 new() 和 area() 方法
   ```

2. **硬幣列舉**：
   ```rust
   enum Coin {
       Penny,
       Nickel,
       Dime,
       Quarter,
   }
   
   // 實作 value_in_cents(coin: Coin) -> u8
   ```

3. **媒體播放器**：
   ```rust
   trait Playable {
       fn play(&self);
       fn stop(&self);
   }
   
   struct Audio { title: String }
   struct Video { title: String }
   
   // 為 Audio 和 Video 實作 Playable
   ```

### 進階練習

4. **動物園系統**：
   - 定義 `Animal` trait 包含 `make_sound(&self) -> String`
   - 實作 `Dog` 和 `Cat` 結構體
   - 使用 `Vec<Box<dyn Animal>>` 儲存不同動物

5. **日誌模組**：
   - 建立 `logger` 模組
   - 實作私有的 `log_internal` 函式
   - 實作公開的 `info` 和 `warn` 函式

## 總結

### 關鍵概念

- **結構體**：組織相關資料，支援三種形式
- **方法**：透過 `impl` 區塊為型別添加行為
- **列舉**：表示多種可能的變體，支援模式匹配
- **Trait**：定義共享行為，實現多型
- **模組**：組織程式碼和控制可見性

### 設計原則

- 資料與行為分離
- 優先使用組合而非繼承
- 透過 Trait 實現多型
- 使用模組控制可見性
- 利用型別系統確保安全性

### 選擇指南

- 需要組合資料 → 結構體
- 需要多種狀態 → 列舉
- 需要共享行為 → Trait
- 需要動態分派 → Trait 物件
- 需要組織程式碼 → 模組

掌握這些概念後，你就能設計出結構清晰、類型安全且易於維護的 Rust 程式。
