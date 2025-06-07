# 第二章 - 控制流程

**設計理念與重要性**

控制流程是程式設計的核心概念，Rust 提供了多種控制結構來管理程式的執行路徑。Rust 的控制流程設計強調：

- **表達式為本**：`if` 和 `match` 都是表達式，可以返回值
- **安全性**：編譯時檢查所有可能的分支
- **明確性**：強制處理所有可能的情況
- **零成本抽象**：高階控制結構在編譯後幾乎沒有運行時開銷

## 1. 條件表達式 (if/else)

**語法要點**

```rust
// 基本 if 表達式
let number = 7;
if number < 5 {
    println!("小於 5");
} else if number == 5 {
    println!("等於 5");
} else {
    println!("大於 5");
}

// if 作為表達式
let result = if number > 5 {
    "大於 5"
} else {
    "小於等於 5"
};
```

**關鍵特性說明**

1. **布林條件**
   - 條件必須是 `bool` 型別
   - 不會自動轉換為布林值

2. **表達式特性**
   - `if` 可以返回一個值
   - 所有分支必須返回相同型別

3. **模式匹配**
   - 可以與 `let` 綁定一起使用
   - 常用於處理 `Option` 和 `Result` 型別

**實際應用**

```rust
// 處理 Option 類型
fn divide(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 {
        None
    } else {
        Some(a / b)
    }
}

// 使用 if let 簡化模式匹配
fn process_optional_value(value: Option<i32>) {
    if let Some(x) = value {
        println!("Got value: {}", x);
    } else {
        println!("No value");
    }
}
```

**練習題**

1. 編寫一個函式 `is_even`，接收一個整數，如果是偶數返回 `true`，奇數返回 `false`
   - 使用 `if` 表達式實現
   - 使用 `match` 表達式實現

2. 實現一個函式 `max`，接收三個整數參數，返回最大的那個
   - 使用巢狀 `if` 表達式
   - 使用 `std::cmp::max` 函式

3. 編寫一個函式 `fizzbuzz`，接收一個整數 n：
   - 如果 n 是 3 的倍數，返回 "Fizz"
   - 如果 n 是 5 的倍數，返回 "Buzz"
   - 如果 n 同時是 3 和 5 的倍數，返回 "FizzBuzz"
   - 其他情況返回數字的字串表示

## 2. 迴圈 (Loops)

**設計理念與重要性**

Rust 的迴圈設計強調安全性和表達力：
- **確定性**：明確的進入和退出條件
- **所有權**：在迴圈中安全地處理數據
- **靈活性**：多種迴圈結構適用於不同場景

### 2.1 `loop` 迴圈

**語法要點**

```rust
// 基本用法
let mut count = 0;
let result = loop {
    count += 1;
    if count == 10 {
        break count * 2;  // 可以返回值
    }
};

// 迴圈標籤
'outer: loop {
    'inner: loop {
        break 'outer;  // 跳出外層迴圈
    }
}
```

**關鍵特性**
- 必須明確使用 `break` 退出
- 可以返回值
- 支援標籤用於巢狀迴圈

### 2.2 `while` 迴圈

**語法要點**

```rust
let mut number = 3;
while number != 0 {
    println!("{}!", number);
    number -= 1;
}
```

**關鍵特性**
- 條件為 `true` 時執行
- 常用於不確定執行次數的情況

### 2.3 `for` 迴圈

**語法要點**

```rust
// 範圍迭代
for i in 1..=5 {  // 1 到 5（包含5）
    println!("{}", i);
}

// 集合迭代
let a = [10, 20, 30, 40, 50];
for element in a.iter() {
    println!("值: {}", element);
}

// 使用 enumerate 獲取索引
for (i, &item) in a.iter().enumerate() {
    println!("索引 {} 的值: {}", i, item);
}
```

**實際應用**

```rust
// 反轉範圍
for number in (1..4).rev() {
    println!("倒數: {}!", number);
}

// 迭代字串字元
for c in "你好".chars() {
    println!("字元: {}", c);
}
```

## 3. 模式匹配 (Pattern Matching)

**設計理念與重要性**

`match` 是 Rust 中最強大的控制流運算子：
- **窮盡性檢查**：必須處理所有可能情況
- **模式解構**：可以解構複雜的數據結構
- **表達式特性**：可以返回值

**語法要點**

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("幸運硬幣！");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

**關鍵特性**

1. **模式綁定**
   ```rust
   fn plus_one(x: Option<i32>) -> Option<i32> {
       match x {
           None => None,
           Some(i) => Some(i + 1),
       }
   }
   ```

2. **通配模式**
   ```rust
   let some_u8_value = 3u8;
   match some_u8_value {
       1 => println!("一"),
       3 => println!("三"),
       5 => println!("五"),
       7 => println!("七"),
       _ => (),  // 處理其他所有情況
   }
   ```

3. **if let 語法糖**
   ```rust
   let some_value = Some(3u8);
   if let Some(3) = some_value {
       println!("三");
   }
   ```

**實際應用**

```rust
// 處理不同的錯誤類型
fn process_result(result: Result<i32, String>) {
    match result {
        Ok(value) => println!("成功: {}", value),
        Err(e) => println!("錯誤: {}", e),
    }
}

// 解構元組
let point = (3, 5);
match point {
    (0, 0) => println!("原點"),
    (x, 0) => println!("x軸上的點: {}", x),
    (0, y) => println!("y軸上的點: {}", y),
    (x, y) => println!("其他點: ({}, {})", x, y),
}
```

**練習題**

1. **費氏數列**
   - 使用 `match` 實現費氏數列函式
   - 使用 `for` 迴圈印出前 10 個費氏數

2. **字串處理**
   - 統計字串中的母音字母數量
   - 使用 `for` 迴圈和模式匹配
   
