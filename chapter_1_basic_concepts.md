# 第一章 - Rust 基礎入門

歡迎來到 Rust 程式語言的世界！本章將帶您認識 Rust 最核心的基礎概念，包括函式、變數和基本資料型態。這些是學習 Rust 的基石，也是後續章節的重要基礎。

## 1. 函式 (Functions)

**設計理念與重要性**

函式是 Rust 程式的基本建構塊，它們讓程式碼更模組化、可重用且易於維護。Rust 的函式設計特別強調：

- **表達式為本**：Rust 是基於表達式的語言，這影響了函式的寫法
- **明確的型別系統**：靜態型別檢查在編譯時捕獲錯誤
- **所有權系統**：管理記憶體安全，無需垃圾回收

**語法要點**

```rust
// 基本函式定義
fn greet() {
    println!("Hello, Rust!");
}

// 帶參數和返回值的函式
fn add(a: i32, b: i32) -> i32 {
    a + b  // 注意：沒有分號，這是一個表達式
}

// 多個參數
fn print_sum(a: i32, b: i32) {
    println!("{} + {} = {}", a, b, a + b);
}

// 提前返回
fn is_even(num: i32) -> bool {
    if num % 2 == 0 {
        return true;  // 使用 return 提前返回
    }
    false
}
```

**關鍵特性說明**

1. **參數型別註解**
   - 必須明確指定參數型別
   - 例如：`fn add(a: i32, b: i32)`

2. **返回值**
   - 使用 `->` 指定返回型別
   - 最後一個表達式會自動返回
   - 使用 `return` 關鍵字可以提前返回

3. **表達式 vs 語句**
   - 表達式會計算出一個值
   - 語句以分號結尾，不會返回值

**實際應用**

```rust
// 計算圓面積
fn calculate_circle_area(radius: f64) -> f64 {
    std::f64::consts::PI * radius.powi(2)
}

// 找出兩個數中的較大值
fn max(a: i32, b: i32) -> i32 {
    if a > b { a } else { b }
}
```

**練習題**

1. 編寫一個函式 `is_positive`，接收一個 i32 整數，如果是正數返回 `true`，否則返回 `false`

2. 實現一個函式 `factorial`，計算給定數字的階乘
   - 例如：`factorial(5)` 應該返回 120

3. 創建一個函式 `greet_user`，接收一個 `&str` 類型的名字，並印出 "Hello, [名字]!" 的問候語
   - 提示：使用 `format!` 巨集來格式化字串

## 2. 變數 (Variables)

**設計理念與重要性**

Rust 的變數系統設計強調安全性和明確性，主要特點包括：

- **不可變性優先**：預設變數不可變，避免意外修改
- **明確的可變性**：使用 `mut` 明確標示可變變數
- **型別安全**：編譯時檢查型別，避免執行時錯誤
- **變數遮蔽**：允許重新使用變數名，同時改變其型別或值

**語法要點**

```rust
// 不可變變數
let x = 5;
// x = 6;  // 錯誤：不可變變數不能修改

// 可變變數
let mut counter = 0;
counter += 1;  // 可變變數可以修改

// 常數（必須標註型別，全大寫命名）
const MAX_POINTS: u32 = 100_000;

// 變數遮蔽
let x = 5;
let x = x + 1;  // 遮蔽前一個 x
let x = "現在是字串";  // 可以改變型別
```

**關鍵特性說明**

1. **不可變性 (Immutability)**
   - 預設變數不可變，避免意外修改
   - 提高程式碼安全性和可預測性

2. **可變性 (Mutability)**
   - 使用 `mut` 關鍵字聲明可變變數
   - 明確標示可能變化的狀態

3. **常數 (Constants)**
   - 編譯時就確定的值
   - 必須標註型別
   - 命名慣例使用全大寫和底線

4. **變數遮蔽 (Shadowing)**
   - 允許重複使用變數名
   - 可以改變變數的型別
   - 作用域結束後恢復原值

**實際應用**

```rust
// 計算圓周長和面積
fn calculate_circle_properties(radius: f64) -> (f64, f64) {
    const PI: f64 = 3.14159;
    let circumference = 2.0 * PI * radius;
    let area = PI * radius.powi(2);
    (circumference, area)
}

// 溫度轉換
fn fahrenheit_to_celsius(f: f64) -> f64 {
    let c = (f - 32.0) * 5.0/9.0;
    c  // 返回轉換後的攝氏溫度
}
```

**練習題**

1. 宣告一個可變變數 `count`，初始值為 0，然後將其增加 5 並印出結果

2. 使用變數遮蔽實現以下功能：
   - 宣告一個變數 `value` 並賦值為 10
   - 遮蔽 `value` 並將其轉換為字串
   - 印出轉換後的結果

3. 編寫一個函式 `swap_values`，接收兩個 `i32` 參數 `a` 和 `b`，返回一個元組 `(b, a)`
   - 提示：使用元組來交換兩個變數的值

## 3. 基本資料型態 (Primitive Types)

**設計理念與重要性**

Rust 的型別系統設計注重安全性和效率：

- **靜態型別**：所有變數的型別在編譯時就已知
- **零成本抽象**：高階抽象不會帶來執行時開銷
- **明確的數值運算**：避免隱式轉換，減少錯誤

### 整數型態 (Integer Types)

| 長度  | 有符號 | 無符號 | 範圍 |
|------|-------|-------|------|
| 8-bit | `i8` | `u8` | -128..127 / 0..255 |
| 16-bit | `i16` | `u16` | -32,768..32,767 / 0..65,535 |
| 32-bit | `i32` | `u32` | -2.1B..2.1B / 0..4.2B |
| 64-bit | `i64` | `u64` | -9.2e18..9.2e18 / 0..1.8e19 |
| 128-bit | `i128` | `u128` | -1.7e38..1.7e38 / 0..3.4e38 |
| 系統架構 | `isize` | `usize` | 取決於目標平台 |

```rust
// 整數字面值
let decimal = 98_222;       // 十進制
let hex = 0xff;             // 十六進制
let octal = 0o77;           // 八進制
let binary = 0b1111_0000;   // 二進制
let byte = b'A';            // byte (u8 only)
```

### 浮點數型態 (Floating-Point Types)

| 型態 | 精確度 | 範圍 |
|------|-------|------|
| `f32` | 6-7 位有效數字 | ±1.2e-38 到 ±3.4e38 |
| `f64` | 15-17 位有效數字 | ±2.2e-308 到 ±1.8e308 |

```rust
let x = 2.0;        // f64 (預設)
let y: f32 = 3.0;    // f32
let z = 1.0e10;     // 科學記號

// 數值運算
let sum = 5 + 10;
let difference = 95.5 - 4.3;
let product = 4 * 30;
let quotient = 56.7 / 32.2;
let remainder = 43 % 5;
```

### 布林型態 (Boolean Type)

```rust
let t = true;           // 推斷為 bool
let f: bool = false;    // 明確標註型別

// 條件判斷
if t {
    println!("這是真的！");
}
```

### 字元型態 (Character Type)

Rust 的 `char` 型態是 4 個字節大小，表示一個 Unicode 純量值。

```rust
let c = 'z';
let z = 'ℤ';            // Unicode 字元
let heart_eyed_cat = '😻'; // Emoji
let newline = '\n';     // 跳脫字元
```

### 複合型態 (Compound Types)

#### 元組 (Tuples)

```rust
// 建立元組
let tup: (i32, f64, u8) = (500, 6.4, 1);

// 解構
let (x, y, z) = tup;
println!("y 的值是: {}", y);  // 6.4

// 使用索引
let five_hundred = tup.0;
let six_point_four = tup.1;
let one = tup.2;
```

#### 陣列 (Arrays)

```rust
// 固定大小陣列
let a = [1, 2, 3, 4, 5];
let months = ["一月", "二月", "三月", "四月", "五月"];

// 明確指定型別和大小
let b: [i32; 5] = [1, 2, 3, 4, 5];

// 初始化相同值的陣列
let c = [3; 5];  // 等同於 [3, 3, 3, 3, 3]

// 訪問元素
let first = a[0];
let second = a[1];

// 編譯時檢查越界
// let element = a[10];  // 編譯錯誤
```

**練習題**

1. **溫度轉換**
   - 編寫函式 `celsius_to_fahrenheit` 和 `fahrenheit_to_celsius`
   - 在 `main` 中測試 0°C 和 32°F 的轉換

2. **斐波那契數列**
   - 編寫函式 `fibonacci` 計算第 n 個斐波那契數
   - 使用迴圈或遞迴實現

3. **陣列操作**
   - 建立一個包含 5 個元素的陣列
   - 編寫函式計算陣列總和
   - 找出陣列中的最大值和最小值
   - 嘗試修改陣列中的元素（注意可變性）
