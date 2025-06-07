# 第四章 - 泛型與特徵約束 (Generics & Trait Bounds)

**設計理念**

泛型是 Rust 的核心特性之一，它讓程式碼更具彈性和重用性，同時保持型別安全和零成本抽象。Rust 的泛型系統在編譯時進行單態化（monomorphization），這意味著使用泛型的程式碼在運行時不會有任何性能損失。

## 1. 泛型基礎

### 設計理念

Rust 的泛型系統設計著重於：
- **型別安全**：在編譯時捕獲型別錯誤
- **零成本抽象**：泛型代碼與手寫的特定型別代碼性能相同
- **表達力**：通過特徵約束明確指定型別必須實現的行為

### 基本語法

#### 函式中的泛型

```rust
// 找到 slice 中最大的元素
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

#### 結構體中的泛型

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

#### 列舉中的泛型

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

## 2. 特徵約束 (Trait Bounds)

### 基本用法

```rust
use std::fmt::Display;

fn print_item<T: Display>(item: T) {
    println!("Item: {}", item);
}
```

### Where 子句

當有多個泛型參數或複雜的約束時，可以使用 `where` 子句提高可讀性：

```rust
fn some_function<T, U>(t: T, u: U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // 函式實作
}
```

## 3. 生命週期與泛型

生命週期參數也是一種泛型，用於確保引用有效：

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

## 4. 實際應用

### 實現泛型結構體

```rust
struct Pair<T> {
    first: T,
    second: T,
}

impl<T> Pair<T> {
    fn new(first: T, second: T) -> Self {
        Self { first, second }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.first >= self.second {
            println!("The larger member is first: {}", self.first);
        } else {
            println!("The larger member is second: {}", self.second);
        }
    }
}
```

---

**練習題**

1. **泛型函式實作**
   - 實現一個泛型函式 `swap<T>`，交換兩個變數的值
   - 確保函式可以處理任何可複製的型別

2. **泛型結構體**
   - 創建一個泛型結構體 `Stack<T>`，實現以下方法：
     - `push`：添加元素到堆疊頂部
     - `pop`：移除並返回堆疊頂部元素
     - `peek`：查看堆疊頂部元素但不移除
     - `is_empty`：檢查堆疊是否為空

3. **特徵約束**
   - 實現一個泛型函式 `print_summary<T: Summary>`，接受一個實現了 `Summary` 特徵的類型
   - 為自定義類型實現 `Summary` 特徵
