# 第五章 - 所有權 (Ownership) 與借用 (Borrowing)

## 學習目標

通過本章學習，你將能夠：
- 理解 Rust 的記憶體管理模型
- 正確使用所有權和借用機制
- 避免常見的記憶體安全問題
- 編寫高效且安全的 Rust 代碼

## 核心概念

### 所有權與移動語義

#### 設計理念與重要性

Rust 的所有權系統是其實現記憶體安全的關鍵創新，通過編譯時規則管理記憶體，無需垃圾回收器。核心目標：

1. **記憶體安全**：防止空指針、懸垂指針和數據競爭
2. **執行時性能**：零成本抽象，無運行時開銷
3. **明確的資源管理**：資源獲取和釋放時機清晰可預測

#### 語法要點

Rust 所有權系統的三個基本規則：
1. **單一所有權**：每個值有且只有一個擁有者
2. **作用域綁定**：擁有者離開作用域時，值自動釋放
3. **移動語義**：賦值、參數傳遞和返回值會轉移所有權

- **移動 (Move)**：賦值操作轉移所有權，原變數無法再使用
- **複製 (Copy)**：基本數值類型自動複製，不移動所有權

#### 實際應用

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // 所有權轉移
    // println!("{}", s1); // 錯誤！s1 不再有效
    
    let s3 = s2.clone(); // 深拷貝
    println!("s2: {}, s3: {}", s2, s3);
    
    takes_ownership(s3);
    // println!("{}", s3); // 錯誤！所有權已移動
    
    let x = 5;
    makes_copy(x);
    println!("x is still: {}", x); // 基本類型複製
}

fn takes_ownership(s: String) {
    println!("Taking ownership of: {}", s);
}

fn makes_copy(i: i32) {
    println!("Making copy of: {}", i);
}

fn gives_ownership() -> String {
    let some_string = String::from("yours");
    some_string // 所有權被移動出去
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len();
    (s, length) // 返回元組，轉移所有權
}
```

### 借用與引用

#### 設計理念與重要性

借用允許引用數據而無需獲取所有權，實現高效且安全的數據共享。借用檢查器在編譯時強制執行規則，確保所有引用都有效。

#### 語法要點

- **不可變引用 (`&T`)**：只讀訪問，可同時有多個
- **可變引用 (`&mut T`)**：可修改數據，同時只能有一個
- **引用規則**：
  - 同一作用域內可有多個不可變引用
  - 同一作用域內只能有一個可變引用
  - 不能同時有可變和不可變引用
  - 引用必須始終有效

#### 實際應用

```rust
fn main() {
    let s1 = String::from("hello");
    
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
    
    let mut s = String::from("hello");
    change(&mut s);
    println!("After change: {}", s);
    
    let mut s = String::from("hello");
    let r1 = &s;     // 沒問題
    let r2 = &s;     // 沒問題
    println!("{} and {}", r1, r2);
    
    let r3 = &mut s; // 現在可以了
    r3.push_str(", world");
    println!("{}", r3);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}

fn change(s: &mut String) {
    s.push_str(", world");
}
```

### 生命週期 (Lifetimes)

#### 設計理念與重要性

生命週期確保所有引用都有效，解決懸垂引用問題。生命週期註解告訴編譯器不同引用之間的有效時間關係，實現編譯時檢查。

#### 語法要點

- 生命週期註解以 `'` 開頭，如 `'a`
- 用於函式簽名中，表示輸入引用和輸出引用的關係
- 大多數情況下編譯器可自動推導

#### 實際應用

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}.", result);
}
```

## 切片 (Slices)

### 設計理念與重要性

切片是不擁有數據所有權的引用類型，允許引用集合中連續的一部分。提供安全且高效的方式處理數據部分，無需複製數據。

### 語法要點

- **字符串切片 (`&str`)**：引用 `String` 或字符串字面量的一部分，語法 `&s[start..end]`
- **數組切片 (`&[T]`)**：引用數組或 `Vec` 的一部分，語法 `&a[start..end]`
- `start` 索引包含，`end` 索引不包含
- 可省略起始或結束索引

### 實際應用

```rust
fn main() {
    let s = String::from("hello world");
    
    let hello = &s[0..5];
    let world = &s[6..11];
    println!("{} {}", hello, world);
    
    let s_literal = "hello"; // &'static str 類型
    
    let a = [1, 2, 3, 4, 5];
    let slice = &a[1..3]; // 類型是 &[i32]
    
    for i in slice {
        println!("{}", i);
    }
}
```

## 練習題

### 1. 所有權轉移與函式
- 創建一個函式 `process_string(s: String)`，它接收一個 `String` 並印出它
- 在 `main` 函式中，創建一個 `String`，將它傳遞給 `process_string`
- 嘗試在傳遞之後再次印出 `main` 中的 `String`。觀察會發生什麼，並解釋原因
- 修改 `process_string`，使其在印出後返回 `String` 的所有權。在 `main` 中接收返回的 `String` 並再次印出

### 2. 可變借用
- 創建一個函式 `modify_string(s: &mut String)`，它接收一個 `String` 的可變引用並向其追加一些文本
- 在 `main` 函式中，創建一個可變 `String`，將其傳遞給 `modify_string`，然後印出修改後的 `String`

### 3. 切片練習
- 編寫一個函式 `get_first_word(s: &str) -> &str`，它接收一個字符串切片並返回其第一個單詞的切片
- 在 `main` 函式中，測試此函式，使用 `String` 和字符串字面量作為輸入

