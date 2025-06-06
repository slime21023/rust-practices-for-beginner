# 第三章 - 基本資料結構

**設計理念與重要性**

Rust 的資料結構設計強調：
- **記憶體安全**：編譯時檢查記憶體存取
- **所有權系統**：明確的資料所有權和生命週期
- **零成本抽象**：高階抽象不帶來運行時開銷
- **明確性**：型別系統確保資料結構的正確使用

## 1. 元組 (Tuples)

**語法要點**

```rust
// 基本語法
let tup: (i32, f64, u8) = (500, 6.4, 1);

// 解構元組
let (x, y, z) = tup;

// 直接存取元素
let first = tup.0;
let second = tup.1;
```

**關鍵特性**

1. **異質集合**
   - 可以包含不同型別的資料
   - 最大長度為12（可以通過特徵擴展）

2. **模式匹配**
   ```rust
   let point = (10, 20);
   match point {
       (0, 0) => println!("原點"),
       (x, 0) => println!("x軸: {}", x),
       (0, y) => println!("y軸: {}", y),
       (x, y) => println!("點: ({}, {})", x, y),
   }
   ```

3. **函數返回多值**
   ```rust
   fn calculate_stats(numbers: &[i32]) -> (i32, f64) {
       let sum: i32 = numbers.iter().sum();
       let avg = sum as f64 / numbers.len() as f64;
       (sum, avg)
   }
   ```

**實際應用**

```rust
// 座標轉換
fn transform_coords((x, y): (f64, f64), dx: f64, dy: f64) -> (f64, f64) {
    (x + dx, y + dy)
}

// 處理多返回值
let (sum, avg) = calculate_stats(&[1, 2, 3, 4, 5]);
println!("總和: {}, 平均: {}", sum, avg);
```

## 2. 陣列 (Arrays)

**語法要點**

```rust
// 基本語法
let a: [i32; 5] = [1, 2, 3, 4, 5];

// 初始化相同值
let b = [3; 5]; // [3, 3, 3, 3, 3]

// 切片
let slice = &a[1..3]; // [2, 3]
```

**關鍵特性**

1. **固定大小**
   - 編譯時確定長度
   - 儲存在堆疊上

2. **型別標註**
   - 語法：`[T; N]`，T 是元素型別，N 是長度
   - 必須明確指定大小

3. **邊界檢查**
   - 編譯器和運行時都會檢查索引
   - 越界訪問會導致 panic

**實際應用**

```rust
// 矩陣運算
fn matrix_transpose(matrix: [[i32; 3]; 3]) -> [[i32; 3]; 3] {
    let mut result = [[0; 3]; 3];
    for i in 0..3 {
        for j in 0..3 {
            result[j][i] = matrix[i][j];
        }
    }
    result
}

// 緩衝區
struct Buffer {
    data: [u8; 1024],
    len: usize,
}
```

## 3. 結構體 (Structs)

**設計理念**

結構體是 Rust 中自訂型別的基礎，用於：
- 封裝相關資料
- 實現方法和關聯函數
- 建立領域模型

### 3.1 傳統結構體

**語法要點**

```rust
// 定義
struct User {
    username: String,
    email: String,
    active: bool,
    sign_in_count: u64,
}

// 建立實例
let user1 = User {
    email: String::from("user@example.com"),
    username: String::from("user123"),
    active: true,
    sign_in_count: 1,
};

// 更新語法
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("user456"),
    ..user1 // 使用其他實例的剩餘欄位
};
```

**關鍵特性**

1. **欄位可變性**
   - 整個實例必須都是可變的
   - 不能只標記單個欄位

2. **方法語法**
   ```rust
   impl User {
       // 關聯函數
       fn new(username: String, email: String) -> Self {
           Self {
               username,
               email,
               active: true,
               sign_in_count: 0,
           }
       }

       // 方法
       fn deactivate(&mut self) {
           self.active = false;
       }
   }
   ```

### 3.2 元組結構體

**語法要點**

```rust
struct Color(i32, i32, i32);
let black = Color(0, 0, 0);
```

### 3.3 單元結構體

**語法要點**

```rust
struct AlwaysEqual;
let subject = AlwaysEqual;
```

## 4. 列舉 (Enums)

**設計理念**

列舉允許你定義一個可以有多種變體的型別，每個變體可以：
- 不關聯資料
- 關聯元組
- 關聯命名欄位

**語法要點**

```rust
// 基本列舉
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

// 使用 match 處理
fn handle_message(msg: Message) {
    match msg {
        Message::Quit => println!("退出"),
        Message::Move { x, y } => println!("移動到 ({}, {})", x, y),
        Message::Write(text) => println!("訊息: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!("顏色變更: R{}, G{}, B{}", r, g, b)
        }
    }
}
```

**關鍵特性**

1. **Option 列舉**
   ```rust
   enum Option<T> {
       Some(T),
       None,
   }
   ```

2. **Result 列舉**
   ```rust
   enum Result<T, E> {
       Ok(T),
       Err(E),
   }
   ```

**實際應用**

```rust
// 檔案操作
fn read_file(path: &str) -> Result<String, std::io::Error> {
    std::fs::read_to_string(path)
}

// 處理結果
match read_file("hello.txt") {
    Ok(content) => println!("檔案內容: {}", content),
    Err(e) => println!("讀取檔案失敗: {}", e),
}
```

**練習題**

1. **幾何計算**
   - 定義 `Point` 和 `Rectangle` 結構體
   - 實現計算面積和周長的方法
   - 實現檢查點是否在矩形內的方法

2. **購物車系統**
   - 定義 `Product` 和 `ShoppingCart` 結構體
   - 實現添加/刪除商品的方法
   - 計算總價和顯示購物車內容

3. **狀態機**
   - 定義 `Connection` 列舉，包含 `Closed`、`Listening`、`Established` 狀態
   - 實現狀態轉換方法
   - 處理無效狀態轉換

4. **日誌系統**
   - 定義 `LogLevel` 列舉（Error, Warn, Info, Debug）
   - 實現格式化日誌訊息的功能
   - 過濾和顯示不同級別的日誌


### 4. 列舉 (Enums)

列舉 (enumerations) 允許你定義一個型態，該型態可以是一組可能的變體 (variants) 中的任何一個。

```rust
// 定義一個代表 IP 位址種類的列舉
enum IpAddrKind {
    V4,
    V6,
}

// 列舉的變體可以儲存不同型態和數量的資料
enum IpAddr {
    V4(u8, u8, u8, u8), // V4 變體儲存四個 u8 值
    V6(String),         // V6 變體儲存一個 String
}

// 另一個列舉範例，代表不同的訊息
#[derive(Debug)] // 為了能印出 Message
enum Message {
    Quit,                             // 沒有關聯資料
    Move { x: i32, y: i32 },          // 包含一個匿名結構體
    Write(String),                    // 包含一個 String
    ChangeColor(i32, i32, i32),       // 包含三個 i32 值
}

// 列舉也可以有方法
impl Message {
    fn call(&self) {
        // 方法內容可以根據 self 的變體執行不同操作
        match self {
            Message::Quit => println!("Quit message received."),
            Message::Move { x, y } => println!("Move to x: {}, y: {}", x, y),
            Message::Write(text) => println!("Text message: {}", text),
            Message::ChangeColor(r, g, b) => println!("Change color to R:{}, G:{}, B:{}", r, g, b),
        }
    }
}

fn main() {
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;

    route(four);
    route(six);

    let home = IpAddr::V4(127, 0, 0, 1);
    let loopback = IpAddr::V6(String::from("::1"));

    print_ip(home);
    print_ip(loopback);

    let m1 = Message::Quit;
    let m2 = Message::Move { x: 10, y: 20 };
    let m3 = Message::Write(String::from("hello"));
    let m4 = Message::ChangeColor(255, 0, 0);

    m1.call();
    m2.call();
    m3.call();
    m4.call();

    // Option<T> 是一個標準庫提供的列舉，用於處理可能為空的值
    // enum Option<T> {
    //     Some(T), // 存在值 T
    //     None,    // 不存在值
    // }
    let some_number = Some(5);
    let some_string = Some("a string");
    let absent_number: Option<i32> = None; // 需要明確指定型態，因為 None 沒有值可以推斷

    println!("Some number: {:?}", some_number);
    println!("Absent number: {:?}", absent_number);
}

fn route(ip_kind: IpAddrKind) {
    // 可以使用 match 來處理列舉
    match ip_kind {
        IpAddrKind::V4 => println!("Routing IPv4"),
        IpAddrKind::V6 => println!("Routing IPv6"),
    }
}

fn print_ip(ip_addr: IpAddr) {
    match ip_addr {
        IpAddr::V4(a, b, c, d) => println!("IPv4: {}.{}.{}.{}", a, b, c, d),
        IpAddr::V6(s) => println!("IPv6: {}", s),
    }
}
```

## 練習題

1.  **學生資訊 (元組):**
    *   創建一個元組來儲存學生的姓名 (`String`)、年齡 (`u32`) 和平均成績 (`f32`)。
    *   解構該元組，並分別印出學生的姓名、年齡和平均成績。

2.  **月份與天數 (陣列與元組):**
    *   創建一個陣列，其中每個元素都是一個元組，代表 (月份名稱: `&str`, 天數: `u8`)。例如 `[("January", 31), ("February", 28), ...]` (暫不考慮閏年)。
    *   遍歷這個陣列，並印出每個月的名稱和對應的天數。

3.  **計算機 (結構體與方法):**
    *   定義一個 `Calculator` 結構體，不包含任何欄位 (可以是單元結構體)。
    *   為 `Calculator` 實現以下方法：
        *   `add(a: f64, b: f64) -> f64`
        *   `subtract(a: f64, b: f64) -> f64`
        *   `multiply(a: f64, b: f64) -> f64`
        *   `divide(a: f64, b: f64) -> Option<f64>` (如果除數為 0，則返回 `None`，否則返回 `Some(result)`)
    *   在 `main` 函式中創建一個 `Calculator` 實例，並測試所有方法，包括除以零的情況。

4.  **交通號誌狀態 (列舉與 `match`):**
    *   定義一個 `TrafficLight` 列舉，包含變體 `Red`、`Yellow`、`Green`。
    *   編寫一個函式 `duration(light: TrafficLight) -> u32`，該函式使用 `match` 表達式返回不同燈號的建議持續時間 (秒)：Red (30秒), Yellow (5秒), Green (45秒)。
    *   在 `main` 函式中，創建每種交通號誌的實例，並印出它們的建議持續時間。

5.  **訂單系統 (結構體與列舉):**
    *   定義一個 `Product` 結構體，包含 `id` (`u32`)、`name` (`String`) 和 `price` (`f64`)。
    *   定義一個 `OrderStatus` 列舉，包含 `Pending`、`Processing`、`Shipped(String)` (其中 `String` 是追蹤號碼)、`Delivered`、`Cancelled`。
    *   定義一個 `Order` 結構體，包含 `order_id` (`u64`)、`product` (`Product`)、`quantity` (`u32`) 和 `status` (`OrderStatus`)。
    *   編寫一個函式，接收一個 `Order` 實例，並根據其 `status` 印出訂單的詳細資訊和當前狀態描述。
    *   在 `main` 函式中創建一個 `Product` 實例和一個 `Order` 實例 (例如，狀態為 `Shipped` 並帶有追蹤號碼)，然後印出訂單資訊。
