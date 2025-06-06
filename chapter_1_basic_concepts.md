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

}

fn main() {
    say_hello(); // 呼叫函式

    let num1 = 10;
    let num2 = 20;
    let sum_result = add_numbers(num1, num2);
    println!("The sum of {} and {} is: {}", num1, num2, sum_result);

    let calc_result = complex_calculation(num1, num2);
    println!("The result of complex calculation is: {}", calc_result);

    // 函式體本身也可以是一個表達式，如果它返回一個值
    // 例如，一個區塊 `{}` 可以是一個表達式
    let y = {
        let x_val = 3;
        x_val + 1 // 這個區塊的值是 x_val + 1，它被賦給 y
    };
    println!("The value of y (from a block expression) is: {}", y);
}
```

### 2. 變數 (Variables)

**設計理念、相關語法與應用場景**

變數是程式中用來儲存和引用資料的基本方式。它們允許我們為記憶體中的某個位置賦予一個有意義的名稱，方便後續操作。Rust 在變數處理上引入了幾個核心概念，旨在提升程式碼的安全性與清晰度，尤其是其預設不可變性。

**核心設計理念與重要性：**
*   **資料儲存與命名:** 為資料提供一個易於記憶和理解的名稱。
*   **不可變性優先 (Immutability by Default):** Rust 的一個核心特性。變數一旦被賦值，其值預設不能被改變。這有助於編寫更安全、更易於推理的程式碼，尤其在並行程式設計中能有效防止資料競爭。
*   **明確的可變性 (Explicit Mutability):** 如果確實需要改變變數的值，必須使用 `mut` 關鍵字明確聲明，這使得程式碼中潛在的狀態改變點更加清晰。
*   **型態安全 (Type Safety):** Rust 是靜態型別語言，每個變數都有一個確定的型態，在編譯時期就會進行檢查。
*   **遮蔽 (Shadowing):** 允許宣告一個與先前變數同名的新變數，新變數會「遮蔽」舊變數。這在需要改變值的型態或進行一系列轉換時非常有用，而無需創建許多不同名稱的變數。

**相關語法：**
*   使用 `let` 關鍵字宣告變數。例如：`let x = 5;`。
*   若要使變數可變，使用 `let mut`。例如：`let mut y = 10;`。
*   可以選擇性地為變數標註型態。例如：`let z: i32 = 15;`。如果沒有明確標註，Rust 通常能根據上下文推斷出型態。
*   **常數 (Constants):** 使用 `const` 關鍵字宣告，常數總是不可變的，且必須明確標註型態。常數名通常使用全大寫蛇形命名法（`SCREAMING_SNAKE_CASE`）。例如：`const MAX_POINTS: u32 = 100_000;`。常數只能被設定為常數表達式。
*   **遮蔽:** `let x = 5; let x = x + 1;` 第二個 `let` 宣告了一個新的 `x`，遮蔽了第一個 `x`。

**常見應用場景：**
*   儲存計算的中間結果或最終結果。
*   代表程式中的狀態，例如迴圈計數器、使用者輸入、設定值。
*   作為函式的參數和返回值。
*   在資料結構中儲存欄位值。

**程式碼範例**

在 Rust 中，我們使用 `let` 關鍵字來宣告變數。預設情況下，這些變數是不可變的 (immutable)。

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    // 下面這行如果取消註解會導致編譯錯誤
    // x = 6; 
}
```
上面的程式碼中，`x` 被賦值為 `5`。如果嘗試將 `x` 重新賦值為 `6`，編譯器會報錯，因為 `x` 是不可變的。

如果我們希望一個變數是可變的 (mutable)，我們需要在宣告時使用 `mut` 關鍵字。

```rust
fn main() {
    let mut y = 10;
    println!("The initial value of y is: {}", y);
    y = 20; // 這是允許的，因為 y 是可變的
    println!("The new value of y is: {}", y);
}
```

除了變數，Rust 還有常數 (constants) 的概念。常數總是不可變的，並且在宣告時必須明確標註型態。常數名習慣上使用全大寫字母和底線。

```rust
fn main() {
    const PI_APPROXIMATION: f64 = 3.14159;
    println!("The approximate value of PI is: {}", PI_APPROXIMATION);
}
```
常數可以在任何作用域中宣告，包括全域作用域。它們的值必須是一個常數表達式，不能是函式呼叫的結果或任何其他只能在執行時計算得到的值。

Rust 中的一個有趣特性是「遮蔽」(shadowing)。它允許我們宣告一個與之前變數同名的新變數，這個新變數會「遮蔽」掉舊的變數。

```rust
fn main() {
    let z = 5;
    println!("The initial value of z is: {}", z); // z = 5

    let z = z + 1; // z 被遮蔽，新的 z 是 6
    println!("After first shadowing, z is: {}", z); // z = 6

    {
        let z = z * 2; // 在內部作用域，z 再次被遮蔽，新的 z 是 12
        println!("Inside inner scope, z is: {}", z); // z = 12
    } // 內部作用域結束，內部的 z 被銷毀

    // 此時，z 回復到上一個被遮蔽的值
    println!("Outside inner scope, z is back to: {}", z); // z = 6

    // 遮蔽也允許我們改變變數的型態
    let spaces = "   "; // spaces 是字串型態
    println!("Spaces string: '{}'", spaces);

    let spaces = spaces.len(); // spaces 被遮蔽，新的 spaces 是數字型態 (長度)
    println!("Number of spaces (after shadowing to length): {}", spaces);
}
```
在上面的遮蔽範例中，我們首先宣告了 `z`，然後透過 `let z = ...` 重新宣告了它。每次宣告都會創建一個新的變數 `z`，它會遮蔽掉前一個。當一個作用域結束時，該作用域內遮蔽的變數也會結束其生命週期。遮蔽與將變數標記為 `mut` 不同。遮蔽是創建一個全新的變數，而 `mut` 允許改變同一個變數的值。遮蔽的一個重要用途是可以在保持變數名稱不變的情況下改變其型態，如 `spaces` 範例所示。

### 3. 基本資料型態 (Primitive Types)

Rust 有幾種基本資料型態，用於表示不同種類的數值。

#### 整數型態 (Integer Types)

整數型態根據其大小（位元數）和是否有符號進行分類。
`i` 開頭表示有符號 (signed)，`u` 開頭表示無符號 (unsigned)。

| 長度  | 有符號 | 無符號 |
| :---- | :----- | :----- |
| 8-bit | `i8`   | `u8`   |
| 16-bit| `i16`  | `u16`  |
| 32-bit| `i32`  | `u32`  |
| 64-bit| `i64`  | `u64`  |
| 128-bit|`i128` | `u128` |
| arch  | `isize`| `usize`| (大小取決於目標平台的架構，通常是 32 位元或 64 位元)

```rust
fn main() {
    let a: i32 = -10;      // 有符號 32 位元整數
    let b: u64 = 100_000;  // 無符號 64 位元整數 (可以使用 _ 作為分隔符提高可讀性)
    let c = 98_222;        // Rust 通常可以推斷型態，這裡是 i32 (預設整數型態)
    let d: u8 = b'A';      // u8 也可以表示 ASCII 字元

    println!("a = {}, b = {}, c = {}, d = {}", a, b, c, d);

    // 整數溢位 (Integer Overflow)
    // 在 debug 模式下，整數溢位會導致 panic。
    // 在 release 模式下 (--release)，Rust 不會檢查可能導致 panic 的整數溢位。
    // 相反，如果發生溢位，Rust 會執行二補數環繞 (two’s complement wrapping)。
    // 例如，u8 型態的值 255 + 1 會變成 0。
    // 若要明確處理溢位，可以使用標準庫提供的 wrapping_* 方法，如 wrapping_add。
    let overflow_val: u8 = 255;
    // let panics = overflow_val + 1; // 這在 debug 模式下會 panic
    let wrapped = overflow_val.wrapping_add(1);
    println!("255 + 1 (wrapped) = {}", wrapped); // 輸出 0
}
```

#### 浮點數型態 (Floating-Point Types)

Rust 有兩種主要的浮點數型態：`f32` (單精度) 和 `f64` (雙精度，預設)。

```rust
fn main() {
    let x = 2.0;    // f64 (預設)
    let y: f32 = 3.0; // f32

    println!("x = {}, y = {}", x, y);

    // 數學運算
    let sum = 5 + 10;          // 加法
    let difference = 95.5 - 4.3; // 減法
    let product = 4 * 30;      // 乘法
    let quotient = 56.7 / 32.2;  // 除法
    let remainder = 43 % 5;      // 取餘

    println!("Sum: {}", sum);
    println!("Difference: {}", difference);
    println!("Product: {}", product);
    println!("Quotient: {}", quotient);
    println!("Remainder: {}", remainder);
}
```

#### 布林型態 (Boolean Type)

Rust 的布林型態 `bool` 只有兩個可能的值：`true` 和 `false`。

```rust
fn main() {
    let t = true;
    let f: bool = false; // 可以明確標註型態

    println!("t is {}, f is {}", t, f);
}
```

#### 字元型態 (Character Type)

Rust 的 `char` 型態用於表示單個 Unicode 純量值，因此它可以表示的不僅僅是 ASCII。
字元使用單引號 `'`。

```rust
fn main() {
    let c = 'z';
    let z = 'ℤ'; // Unicode 字元
    let heart_eyed_cat = '😻'; // Emoji 也是字元

    println!("c = {}", c);
    println!("z = {}", z);
    println!("Heart-eyed cat: {}", heart_eyed_cat);
}
```

## 練習題

1.  **計算圓的面積和周長:**
    *   編寫一個程式，宣告一個表示圓半徑的變數 (例如 `let radius: f64 = 5.0;`)。
    *   定義兩個函式：`calculate_area(radius: f64) -> f64` 和 `calculate_circumference(radius: f64) -> f64`。
    *   在 `main` 函式中呼叫這兩個函式，並印出結果。
    *   提示：圓周率 π 可以使用 `std::f64::consts::PI`。面積公式：π * r^2，周長公式：2 * π * r。

2.  **溫度轉換:**
    *   編寫一個程式，包含兩個函式：
        *   `celsius_to_fahrenheit(celsius: f64) -> f64` (攝氏轉華氏，公式：F = C * 9/5 + 32)
        *   `fahrenheit_to_celsius(fahrenheit: f64) -> f64` (華氏轉攝氏，公式：C = (F - 32) * 5/9)
    *   在 `main` 函式中，宣告一個攝氏溫度值和一個華氏溫度值，分別呼叫轉換函式，並印出轉換後的結果。

3.  **變數與遮蔽練習:**
    *   在 `main` 函式中，宣告一個不可變變數 `x` 並賦值為 `10`。
    *   嘗試修改 `x` 的值，觀察編譯器的錯誤訊息。
    *   將 `x` 改為可變變數，並成功修改其值為 `20`，然後印出。
    *   使用遮蔽，重新宣告一個名為 `x` 的變數，使其值為字串 `"hello"`，然後印出。
    *   在一個新的內部作用域中，再次遮蔽 `x`，使其值為布林值 `true`，然後印出。
    *   離開內部作用域後，再次印出 `x` 的值，觀察其值和型態。

