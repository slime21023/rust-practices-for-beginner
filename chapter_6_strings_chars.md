# 第六章 - 文字處理 (Strings & Characters)

本章將介紹 Rust 中處理文字資料的三個核心型別：字元 (`char`)、可變字串 (`String`) 和字串切片 (`&str`)。Rust 的字串處理基於 UTF-8 編碼，強調記憶體安全和 Unicode 正確性。

## 1. 字元 (`char`)

### 核心概念

- Rust 的 `char` 型別代表一個 Unicode 純量值，固定佔用 4 個位元組
- 使用單引號定義：`'a'`、`'中'`、`'😻'`
- 與其他語言的單位元組 `char` 不同，能正確處理所有 Unicode 字元

### 基本用法

```rust
fn main() {
    let c1 = 'a';
    let c2 = '中';
    let c3 = '😻';
    
    println!("Size of char: {} bytes", std::mem::size_of::<char>());
    
    // 字元屬性檢查
    println!("'A' is alphabetic: {}", 'A'.is_alphabetic());
    println!("'7' is numeric: {}", '7'.is_numeric());
    println!("' ' is whitespace: {}", ' '.is_whitespace());
    
    // 大小寫轉換
    println!("Uppercase: {}", 'a'.to_uppercase().collect::<String>());
    println!("Lowercase: {}", 'A'.to_lowercase().collect::<String>());
}
```

## 2. 字串 (`String`)

### 核心概念

- `String` 是可變、可增長的字串型別，資料儲存在堆上
- 擁有資料的所有權，離開作用域時自動釋放記憶體
- 內部使用 UTF-8 編碼

### 創建和操作

```rust
fn main() {
    // 創建 String
    let mut s1 = String::new();
    let s2 = "hello".to_string();
    let s3 = String::from("world");
    
    // 修改 String
    s1.push_str("Hello");
    s1.push('!');
    
    // 字串連接
    let s4 = String::from("Hello, ");
    let s5 = String::from("world!");
    let s6 = s4 + &s5; // s4 被移動，不能再使用
    
    // 使用 format! 宏（不會移動所有權）
    let s7 = String::from("tic");
    let s8 = String::from("tac");
    let s9 = String::from("toe");
    let result = format!("{}-{}-{}", s7, s8, s9);
    println!("{}", result);
}
```

## 3. 字串切片 (`&str`)

### 核心概念

- `&str` 是字串的不可變引用，不擁有資料所有權
- 字串字面量的型別是 `&'static str`
- 可以從 `String` 創建切片，也可以是字串字面量的引用

### 基本用法

```rust
fn main() {
    let s = String::from("hello world");
    
    // 創建切片
    let hello = &s[0..5];
    let world = &s[6..11];
    
    // 字串字面量就是 &str
    let literal: &str = "I am a string literal";
    
    // 函式通常使用 &str 作為參數（更靈活）
    fn first_word(s: &str) -> &str {
        let bytes = s.as_bytes();
        for (i, &item) in bytes.iter().enumerate() {
            if item == b' ' {
                return &s[0..i];
            }
        }
        &s[..]
    }
    
    let word1 = first_word(&s);
    let word2 = first_word("hello rust");
    println!("First words: {}, {}", word1, word2);
}
```

## 4. UTF-8 編碼注意事項

### 重要概念

- Rust 字串使用 UTF-8 編碼，字元可能佔用 1-4 個位元組
- 不能直接用索引存取字元（如 `s[0]`），因為可能破壞字元邊界
- 切片索引必須位於有效的字元邊界上

### 安全的字串操作

```rust
fn main() {
    let s = "नमस्ते"; // 印地語，每個字元 3 個位元組
    
    println!("Length in bytes: {}", s.len());
    println!("Length in chars: {}", s.chars().count());
    
    // 遍歷字元
    for c in s.chars() {
        println!("Char: {}", c);
    }
    
    // 遍歷位元組
    for b in s.bytes() {
        print!("{} ", b);
    }
    println!();
    
    // 安全的字元存取
    let text = "abcde你好";
    if let Some(third_char) = text.chars().nth(2) {
        println!("Third char: {}", third_char);
    }
}
```

### 字元邊界問題

```rust
fn main() {
    let unicode_string = String::from("你好世界");
    
    // 正確：每個中文字元佔 3 個位元組
    let ni_hao = &unicode_string[0..6]; // "你好"
    println!("Correct slice: {}", ni_hao);
    
    // 錯誤：會 panic，因為 1 不是字元邊界
    // let invalid = &unicode_string[0..1];
}
```

## 5. 常用字串方法

### String 方法

```rust
fn main() {
    let mut s = String::from("hello");
    
    // 基本操作
    s.push(' ');
    s.push_str("world");
    println!("Length: {}", s.len());
    println!("Is empty: {}", s.is_empty());
    
    // 容量管理
    println!("Capacity: {}", s.capacity());
    s.reserve(10);
    
    // 清空和截斷
    let mut s2 = s.clone();
    s2.clear();
    s.truncate(5);
}
```

### &str 方法

```rust
fn main() {
    let s = "  Hello, World!  ";
    
    // 字串檢查
    println!("Contains 'World': {}", s.contains("World"));
    println!("Starts with 'Hello': {}", s.starts_with("Hello"));
    println!("Ends with '!': {}", s.ends_with("!"));
    
    // 字串處理
    println!("Trimmed: '{}'", s.trim());
    println!("Uppercase: {}", s.to_uppercase());
    println!("Lowercase: {}", s.to_lowercase());
    
    // 分割字串
    for word in s.split_whitespace() {
        println!("Word: {}", word);
    }
}
```

## 6. 實用範例

### 字串處理函式

```rust
fn reverse_string(s: &str) -> String {
    s.chars().rev().collect()
}

fn count_words(text: &str) -> usize {
    text.split_whitespace().count()
}

fn get_file_extension(filename: &str) -> Option<&str> {
    filename.rfind('.').map(|i| &filename[i + 1..])
}

fn main() {
    println!("Reversed: {}", reverse_string("hello"));
    println!("Word count: {}", count_words("hello world rust"));
    
    if let Some(ext) = get_file_extension("file.txt") {
        println!("Extension: {}", ext);
    }
}
```

## 7. 最佳實踐

### 函式參數選擇

```rust
// 推薦：使用 &str 作為參數（接受 String 和 &str）
fn process_text(text: &str) -> String {
    text.to_uppercase()
}

// 避免：除非需要所有權
fn process_text_owned(text: String) -> String {
    text.to_uppercase()
}

fn main() {
    let s1 = String::from("hello");
    let s2 = "world";
    
    // 兩種都可以傳給 process_text
    println!("{}", process_text(&s1));
    println!("{}", process_text(s2));
}
```

### 效能考慮

```rust
fn main() {
    // 預分配容量避免重複分配
    let mut s = String::with_capacity(100);
    
    // 使用 format! 而非多次 + 操作
    let name = "Alice";
    let age = 30;
    let message = format!("Hello, {}! You are {} years old.", name, age);
    
    // 避免不必要的 String 轉換
    fn print_text(text: &str) {
        println!("{}", text);
    }
    
    let s = String::from("hello");
    print_text(&s); // 而非 print_text(&s.to_string())
}
```
---

### 練習題

#### 基礎練習

1. 實作一個函式檢查字元是否為英文母音：
   ```rust
   fn is_vowel(c: char) -> bool {
       // 你的實作
   }
   ```

2. 實作一個函式從 email 地址提取域名：
   ```rust
   fn get_domain(email: &str) -> Option<&str> {
       // 你的實作
   }
   ```

3. 實作一個函式計算字串中特定字元的出現次數：
   ```rust
   fn count_char(text: &str, target: char) -> usize {
       // 你的實作
   }
   ```

#### 進階練習

4. 實作一個函式將字串轉換為標題格式（每個單詞首字母大寫）：
   ```rust
   fn to_title_case(text: &str) -> String {
       // 你的實作
   }
   ```

5. 實作一個函式檢查字串是否為回文：
   ```rust
   fn is_palindrome(text: &str) -> bool {
       // 你的實作
   }
   ```
   
---

## 總結

### 關鍵要點

- **`char`**：4 位元組 Unicode 純量值，用單引號定義
- **`String`**：可變、擁有所有權的字串，儲存在堆上
- **`&str`**：不可變字串切片，不擁有所有權
- **UTF-8**：所有字串都使用 UTF-8 編碼，注意字元邊界
- **函式設計**：優先使用 `&str` 作為參數型別

### 記憶口訣

- 需要修改 → `String`
- 只需讀取 → `&str`
- 遍歷字元 → `.chars()`
- 字串連接 → `format!` 或 `+`
- 切片操作 → 注意字元邊界

掌握這些概念後，你就能在 Rust 中安全高效地處理字串了。
