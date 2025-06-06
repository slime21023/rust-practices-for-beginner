# 第九章 - 專案組織與模組系統

隨著程式碼規模的增長，將所有程式碼都放在單一檔案中會變得難以維護。Rust 提供了強大的模組系統，讓我們能夠將程式碼組織成更小、更易於理解的單元。本章將介紹如何有效地組織 Rust 專案。

## 1. 套件 (Packages) 與 Crates

### 基本概念
- **Crate**：Rust 的編譯單元，可以編譯成二進位檔或函式庫
- **Package**：包含一個或多個 Crate 的套件，由 `Cargo.toml` 定義

### 建立新專案
```bash
# 建立函式庫專案
cargo new my_library --lib

# 建立二進位專案
cargo new my_binary
```

## 2. 模組基礎

### 定義模組
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {
            println!("Added to waitlist!");
        }
    }
}
```

### 模組可見性
- `pub`：公開項目
- `pub(crate)`：僅在當前 crate 內可見
- `pub(super)`：僅在父模組中可見

## 3. 路徑與 use 語句

### 絕對路徑與相對路徑
```rust
// 絕對路徑
crate::front_of_house::hosting::add_to_waitlist();

// 相對路徑
front_of_house::hosting::add_to_waitlist();
```

### 使用 use 引入作用域
```rust
use crate::front_of_house::hosting;

fn main() {
    hosting::add_to_waitlist();
}
```

## 4. 將模組拆分到不同檔案

### 目錄結構
```
my_project/
├── Cargo.toml
└── src/
    ├── main.rs
    ├── lib.rs
    └── front_of_house/
        ├── mod.rs
        └── hosting.rs
```

### 檔案內容範例
**src/front_of_house/mod.rs**
```rust
pub mod hosting;
```

**src/front_of_house/hosting.rs**
```rust
pub fn add_to_waitlist() {
    println!("Added to waitlist!");
}
```

## 5. 常用慣例

### 函式庫與二進位 Crate
- 將主要邏輯放在 `src/lib.rs`
- 將二進位入口點放在 `src/main.rs`
- 使用 `use crate_name::module_name` 引入模組

### 測試組織
- 單元測試寫在原始碼檔案中
- 整合測試放在 `tests/` 目錄下

## 6. 練習題

1. 建立一個名為 `shapes` 的函式庫，實現計算不同形狀面積和周長的功能。將每種形狀放在獨立的模組中。

2. 建立一個名為 `school` 的專案，包含以下模組：
   - `student`：學生相關功能
   - `teacher`：教師相關功能
   - `course`：課程管理
   - `grade`：成績管理

3. 解釋 `pub use` 的用途，並提供一個實際應用場景。

## 7. 進階模組組織方法

### 7.1 條件式編譯與功能旗標

使用 `#[cfg]` 屬性根據不同的編譯條件包含或排除模組：

```rust
#[cfg(feature = "network")]
mod network {
    pub fn connect() { /* ... */ }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_connect() {
        // 測試程式碼
    }
}
```

在 `Cargo.toml` 中定義功能旗標：

```toml
[features]
default = []
network = []
```

### 7.2 使用 `pub use` 重新匯出

重新匯出可以創建更簡潔的公共 API：

```rust
// src/lib.rs
pub mod network;
pub mod client;

// 重新匯出，讓使用者可以直接使用 crate::Client
pub use client::Client;

// src/client.rs
pub struct Client { /* ... */ }
```

### 7.3 工作區 (Workspaces)

對於大型專案，可以使用工作區管理多個相關的套件：

```toml
# Cargo.toml
[workspace]
members = [
    "crates/core",
    "crates/utils",
    "examples/simple"
]
```

### 7.4 模組可見性設計模式

#### 私有模組的公共類型

```rust
mod network {
    // 模組私有，但類型公開
    pub struct NetworkConfig {
        pub timeout: u32,
        retry_count: u32,  // 私有欄位
    }
    
    impl NetworkConfig {
        // 公開建構函數
        pub fn new(timeout: u32) -> Self {
            NetworkConfig { timeout, retry_count: 3 }
        }
    }
}
```

#### 模組前向兼容性

使用私有模組實現細節，公開穩定接口：

```rust
// 公開穩定的 API
pub fn process_data(input: &str) -> String {
    internal::process(input)
}

// 私有實現細節
mod internal {
    pub(super) fn process(input: &str) -> String {
        // 實現細節可以隨意更改
        input.to_uppercase()
    }
}
```

### 7.5 測試組織策略

#### 整合測試

```
tests/
├── common/
│   └── mod.rs     # 測試輔助函數
├── integration_test.rs
└── another_test.rs
```

#### 文件測試 (Doc-tests)

```rust
/// 計算兩個數字的和
///
/// # 範例
/// ```
/// let result = my_crate::add(2, 2);
/// assert_eq!(result, 4);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

## 總結

- 使用 `mod` 關鍵字定義模組
- 使用 `pub` 控制可見性
- 使用 `use` 簡化路徑
- 將大型模組拆分到不同檔案
- 遵循 Rust 的模組慣例，保持程式碼整潔有序

掌握這些概念後，您將能夠：

1. 設計出模組化、可維護的程式碼結構
2. 使用進階模組特性來組織大型專案
3. 實現清晰的公共 API 和封裝細節
4. 有效管理測試和文件
5. 處理複雜的專案依賴關係

這些技巧將幫助您建立專業級的 Rust 專案，無論是小型工具還是大型應用程式。
