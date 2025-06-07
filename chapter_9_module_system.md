# 第九章 - 專案組織與模組系統

Rust 的模組系統是組織程式碼的核心機制，提供命名空間、封裝性和程式碼重用功能。

## 1. 基本概念

### 模組定義與可見性

```rust
mod math {
  pub fn add(a: i32, b: i32) -> i32 {
      a + b
  }
  
  // 私有函數，外部無法存取
  fn internal_calculation() -> i32 {
      42
  }
}

fn main() {
  let result = math::add(5, 3);
  println!("5 + 3 = {}", result);
}
```

### 結構體的可見性控制

```rust
mod library {
  pub struct Book {
      pub title: String,
      pub author: String,
      isbn: String, // 私有欄位
  }
  
  impl Book {
      pub fn new(title: &str, author: &str, isbn: &str) -> Book {
          Book {
              title: title.to_string(),
              author: author.to_string(),
              isbn: isbn.to_string(),
          }
      }
      
      pub fn display_info(&self) {
          println!("《{}》 by {}", self.title, self.author);
      }
  }
}

fn main() {
  let book = library::Book::new("Rust程式設計", "作者", "123456789");
  book.display_info();
  println!("書名: {}", book.title); // 可存取公開欄位
  // println!("ISBN: {}", book.isbn); // 編譯錯誤：私有欄位
}
```

### 巢狀模組

```rust
mod network {
  pub mod client {
      pub fn connect() {
          println!("連接到伺服器");
      }
      
      pub mod http {
          pub fn get(url: &str) {
              println!("GET {}", url);
          }
      }
  }
  
  pub mod server {
      pub fn start() {
          println!("啟動伺服器");
      }
  }
}

fn main() {
  network::client::connect();
  network::client::http::get("https://example.com");
  network::server::start();
}
```

## 2. `use` 關鍵字

### 基本用法

```rust
mod math {
  pub fn add(a: i32, b: i32) -> i32 { a + b }
  pub fn multiply(a: i32, b: i32) -> i32 { a * b }
  
  pub mod advanced {
      pub fn power(base: i32, exp: u32) -> i32 {
          base.pow(exp)
      }
  }
}

// 引入特定函數
use math::add;
use math::advanced::power;

fn main() {
  println!("2 + 3 = {}", add(2, 3));
  println!("2^3 = {}", power(2, 3));
  println!("4 * 5 = {}", math::multiply(4, 5)); // 完整路徑
}
```

### 重新命名與批量引入

```rust
mod graphics {
  pub mod shapes {
      pub fn draw_circle() { println!("畫圓形"); }
  }
  
  pub mod colors {
      pub fn draw_circle() { println!("畫彩色圓形"); }
  }
}

// 重新命名
use graphics::shapes::draw_circle as draw_shape_circle;
use graphics::colors::draw_circle as draw_color_circle;

// 巢狀引入
use graphics::{
  shapes::draw_circle,
  colors,
};

fn main() {
  draw_shape_circle();
  draw_color_circle();
  colors::draw_circle();
}
```

## 3. 檔案系統中的模組

### 基本檔案結構

```
src/
├── main.rs
├── math.rs
└── network/
  ├── mod.rs
  ├── client.rs
  └── server.rs
```

**src/math.rs**
```rust
pub fn add(a: i32, b: i32) -> i32 {
  a + b
}

pub fn multiply(a: i32, b: i32) -> i32 {
  a * b
}
```

**src/main.rs**
```rust
mod math; // 宣告模組，自動尋找 math.rs

fn main() {
  println!("2 + 3 = {}", math::add(2, 3));
  println!("4 * 6 = {}", math::multiply(4, 6));
}
```

### 目錄模組

**src/network/mod.rs**
```rust
pub mod client;
pub mod server;

pub fn initialize() {
  println!("初始化網路模組");
}
```

**src/network/client.rs**
```rust
pub fn connect(address: &str) {
  println!("連接到 {}", address);
}
```

**src/main.rs**
```rust
mod network;

use network::{client, server};

fn main() {
  network::initialize();
  client::connect("127.0.0.1:8080");
}
```

### 現代檔案結構（Rust 2018+）

```
src/
├── main.rs
├── network.rs      // 取代 network/mod.rs
└── network/
  ├── client.rs
  └── server.rs
```

## 4. 路徑和作用域

### 絕對路徑與相對路徑

```rust
mod restaurant {
  pub mod hosting {
      pub fn add_to_waitlist() {
          println!("加入等候名單");
      }
  }
}

pub fn eat_at_restaurant() {
  // 絕對路徑
  crate::restaurant::hosting::add_to_waitlist();
  
  // 相對路徑
  restaurant::hosting::add_to_waitlist();
  
  // 使用 use 簡化
  use restaurant::hosting;
  hosting::add_to_waitlist();
}
```

### super 和 self

```rust
mod parent {
  fn parent_function() {
      println!("父模組函數");
  }
  
  pub mod child {
      pub fn child_function() {
          // 使用 super 存取父模組
          super::parent_function();
      }
      
      pub fn self_demo() {
          // 使用 self 明確指示當前模組
          self::child_function();
      }
  }
}

fn main() {
  parent::child::child_function();
  parent::child::self_demo();
}
```

## 5. Crate 和重新匯出

### 函式庫 Crate

**src/lib.rs**
```rust
pub mod math;
pub mod string_utils;

// 重新匯出常用項目
pub use math::{add, subtract};
pub use string_utils::capitalize;

/// 函式庫的主要功能
pub fn library_function() {
  println!("這是函式庫的主要功能");
}

// 建立 prelude 模組
pub mod prelude {
  pub use crate::{add, subtract, capitalize};
}
```

### pub use 重新匯出

```rust
mod internal {
  pub mod deeply {
      pub mod nested {
          pub fn important_function() {
              println!("重要功能");
          }
      }
  }
}

// 重新匯出，提供簡潔的 API
pub use internal::deeply::nested::important_function;
pub use internal::deeply::nested::important_function as main_feature;

fn main() {
  important_function(); // 簡潔的存取方式
  main_feature();
}
```

## 6. 實際專案範例

### Web 應用程式結構

```
src/
├── main.rs
├── lib.rs
├── handlers/
│   ├── mod.rs
│   ├── user.rs
│   └── auth.rs
├── models/
│   ├── mod.rs
│   └── user.rs
└── services/
  ├── mod.rs
  └── user_service.rs
```

**src/lib.rs**
```rust
pub mod handlers;
pub mod models;
pub mod services;

// 重新匯出常用項目
pub use models::User;

// 建立 prelude 模組
pub mod prelude {
  pub use crate::models::*;
  pub use crate::services::*;
}
```

**src/models/mod.rs**
```rust
pub mod user;
pub use user::User;
```

**src/models/user.rs**
```rust
#[derive(Debug)]
pub struct User {
  pub id: u32,
  pub username: String,
  pub email: String,
}

impl User {
  pub fn new(id: u32, username: String, email: String) -> Self {
      User { id, username, email }
  }
}
```

## 7. 條件編譯

```rust
// 測試模組
#[cfg(test)]
mod tests {
  use super::*;
  
  #[test]
  fn test_something() {
      assert_eq!(2 + 2, 4);
  }
}

// 平台特定程式碼
#[cfg(target_os = "windows")]
mod windows_specific {
  pub fn platform_function() {
      println!("Windows 特定功能");
  }
}

#[cfg(target_os = "linux")]
mod linux_specific {
  pub fn platform_function() {
      println!("Linux 特定功能");
  }
}

// 功能特性
#[cfg(feature = "advanced")]
mod advanced_features {
  pub fn advanced_function() {
      println!("進階功能");
  }
}
```

## 8. 最佳實踐

### 模組組織原則

1. **按功能分組**：相關功能放在同一模組
2. **控制可見性**：只公開必要的項目
3. **使用 prelude**：為常用項目建立 prelude
4. **重新匯出**：提供簡潔的 API

```rust
// 好的模組組織
mod user_management {
  mod user;
  mod authentication;
  
  // 只公開必要的介面
  pub use user::User;
  pub use authentication::{login, logout};
}

// 建立 prelude
pub mod prelude {
  pub use crate::user_management::*;
}
```

### 命名慣例

- **模組名稱**：使用 snake_case
- **檔案名稱**：與模組名稱一致
- **重新匯出**：使用清楚的名稱

## 9. 總結

### 核心概念

1. **`mod`**：定義模組
2. **`pub`**：控制可見性
3. **`use`**：引入項目
4. **`crate`、`super`、`self`**：路徑導航
5. **`pub use`**：重新匯出

### 選擇指南

| 需求 | 使用方式 | 說明 |
|------|---------|------|
| 組織相關功能 | `mod` | 建立模組 |
| 公開 API | `pub` | 控制可見性 |
| 簡化存取 | `use` | 引入常用項目 |
| 簡化 API | `pub use` | 重新匯出 |
| 大型專案 | 檔案模組 | 分離到不同檔案 |

### 注意事項

- 預設所有項目都是私有的
- 使用 `pub` 明確公開必要的項目
- 善用重新匯出建立清晰的 API
- 為大型專案建立 prelude 模組

掌握模組系統將幫助你建立可維護、可擴展的 Rust 專案。
