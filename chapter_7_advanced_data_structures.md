# 第七章 - Rust 中的資料結構與行為模式

Rust 語言在設計上並未完全遵循傳統物件導向程式設計 (OOP) 的所有範式，例如它不提供類別繼承 (Class Inheritance)。然而，Rust 透過其獨特的特性組合，如結構體 (Structs)、列舉 (Enums) 以及強大的 Trait 系統，使得開發者能夠有效地組織資料、封裝行為，並實現許多物件導向設計原則所追求的目標，例如程式碼的模組化、可重用性和可維護性。本章將深入探討 Rust 如何運用這些核心特性來建構清晰且高效的程式碼。學完本章後，您將能夠理解 Rust 如何不依賴傳統 OOP 概念，而是透過其自身的特性來達到相似的設計目標。您將能熟練運用結構體、列舉和 Trait 來建立具有良好封裝、清晰抽象和高度模組化的 Rust 應用程式。

## 1. 結構體 (Structs)

結構體是用來組合相關資料的自訂型別。

#### 設計理念與重要性

Rust 的結構體 (Structs) 是一種自訂複合資料型別，允許你將多個相關的資料欄位組合在一起，形成一個有意義的單元。它類似於其他語言中的「類別」或「物件」，但 Rust 的結構體本身不包含行為（方法），行為是透過 `impl` 區塊來定義的。這種設計強調資料與行為的分離，使得資料的組織更加清晰，同時透過所有權系統確保記憶體安全。結構體是 Rust 中定義複雜資料模型的基礎，廣泛應用於表示實體、配置、訊息等。

#### 語法要點

- **定義結構體**：使用 `struct` 關鍵字，後跟結構體名稱和一對花括號 `{}`，內部定義欄位名稱和其型別。例如 `struct User { username: String, email: String, active: bool, sign_in_count: u64, }`。
- **實例化結構體**：使用結構體名稱和花括號，以 `key: value` 的形式為每個欄位賦值。例如 `let user1 = User { email: String::from("someone@example.com"), username: String::from("someusername123"), active: true, sign_in_count: 1, };`。
- **欄位存取**：使用點號 `.` 來存取結構體實例的特定欄位。例如 `user1.email`。
- **可變性**：如果需要修改結構體實例的欄位，整個實例必須是可變的 (`mut`)。Rust 不允許只將結構體的部分欄位標記為可變。
- **元組結構體 (Tuple Structs)**：類似於元組，但有命名。例如 `struct Color(u8, u8, u8);`。
- **單元式結構體 (Unit-Like Structs)**：沒有任何欄位，通常用於實現 Trait 或作為標記型別。例如 `struct AlwaysEqual;`。

#### 關鍵特性說明

1.  **複合資料型別**：結構體允許將不同型別的資料組合在一起，形成一個邏輯上的單元，提高了資料的組織性和可讀性。
2.  **資料所有權**：結構體擁有其內部欄位的所有權。當結構體實例離開作用域時，其內部所有欄位（包括堆上資料如 `String`）都會被自動清理，無需手動管理記憶體。
3.  **模式匹配**：結構體可以與 `match` 表達式結合使用，進行模式匹配，方便地解構和處理結構體中的資料。
4.  **靈活性**：除了標準結構體外，元組結構體和單元式結構體提供了不同的結構體形式，以適應不同的使用場景，例如元組結構體適用於簡單的資料組合，而單元式結構體則常用於實現標記或無狀態行為。
5.  **與 Trait 結合**：結構體是實現 Trait 的主要載體，透過為結構體實現 Trait，可以為其添加共享行為，並利用泛型和 Trait 約束來編寫更通用和可重用的程式碼。

#### 實際應用

```rust
// 定義一個簡單的結構體 Point
struct Point {
    x: f64,
    y: f64,
}

// 元組結構體 (Tuple Struct)
struct Color(u8, u8, u8); // RGB 顏色

// 單元式結構體 (Unit-like Struct) - 沒有欄位，可用於標記或實現 Trait
struct AlwaysEqual;

fn main() {
    // 創建 Point 結構體的實例
    let origin = Point { x: 0.0, y: 0.0 };
    let mut p1 = Point { x: 10.5, y: -3.2 };

    // 存取欄位
    println!("Origin: ({}, {})", origin.x, origin.y);
    println!("Point p1: ({}, {})", p1.x, p1.y);

    // 修改可變實例的欄位
    p1.x = 12.0;
    println!("Modified p1.x: {}", p1.x);

    // 創建 Color 元組結構體的實例
    let black = Color(0, 0, 0);
    let white = Color(255, 255, 255);
    println!("Black color: ({}, {}, {})", black.0, black.1, black.2);
    println!("White color: ({}, {}, {})", white.0, white.1, white.2);

    // 創建單元式結構體的實例
    let _subject = AlwaysEqual;
    // 通常用於泛型或 Trait 的情境
}
```

#### 練習題

1.  定義一個結構體 `Book`，包含 `title` (String), `author` (String), `page_count` (u32) 和 `is_published` (bool) 欄位。創建一個 `Book` 實例並列印其所有資訊。
2.  編寫一個函式 `create_user`，接收 `email` 和 `username` (都是 `String`) 作為參數，返回一個新的 `User` 結構體實例，其中 `active` 預設為 `true`，`sign_in_count` 預設為 `1`。
3.  為 `Rectangle` 結構體 (包含 `width: u32, height: u32`) 實現一個關聯函式 `can_hold(&self, other: &Rectangle) -> bool`，判斷一個矩形是否可以完全容納另一個矩形。

## 2. 結構體上的方法 (Methods on Structs)

方法是附加到結構體、列舉或 Trait 物件的函式。第一個參數通常是 `self`，代表呼叫該方法的實例。

#### 設計理念與重要性

在 Rust 中，方法 (Methods) 是與特定資料型別（如結構體或列舉）相關聯的函式。它們允許你為這些型別定義行為，從而實現資料和操作的封裝。與傳統 OOP 語言中的類別方法類似，Rust 的方法通過 `impl` 區塊與型別綁定。這種設計使得程式碼更具組織性，將操作邏輯緊密地與其所操作的資料結合，同時利用 Rust 的所有權和借用規則確保記憶體安全，避免了空指針和資料競爭等問題。

#### 語法要點

- **`impl` 區塊**：使用 `impl` 關鍵字為特定的結構體或列舉定義方法。例如 `impl Rectangle { ... }`。
- **實例方法**：第一個參數通常是 `self`、`&self` 或 `&mut self`，代表呼叫該方法的實例。這決定了方法對實例的所有權或借用權限。
    - `self`：方法取得實例的所有權，實例在方法結束後會被銷毀（或轉移）。
    - `&self`：方法不可變地借用實例，可以在不取得所有權的情況下讀取實例的資料。
    - `&mut self`：方法可變地借用實例，可以在不取得所有權的情況下修改實例的資料。
- **關聯函式 (Associated Functions)**：不以 `self` 作為第一個參數的函式。它們與型別本身相關聯，而不是與型別的特定實例相關聯。通常用作建構函式（例如 `String::from()` 或 `Rectangle::new()`）。
- **呼叫方法**：使用點號 `.` 語法呼叫實例方法（例如 `rect.area()`）。
- **呼叫關聯函式**：使用雙冒號 `::` 語法呼叫關聯函式（例如 `Rectangle::new()`）。

#### 關鍵特性說明

1.  **資料與行為的綁定**：方法將操作邏輯直接與其所操作的資料型別綁定，提高了程式碼的內聚性和可讀性。
2.  **所有權和借用**：Rust 的所有權和借用規則嚴格控制了方法對資料的存取權限，確保了記憶體安全，避免了許多常見的程式錯誤。
3.  **靈活的 `self` 參數**：通過 `self`、`&self` 和 `&mut self`，開發者可以精確控制方法對實例的所有權和可變性，滿足不同的設計需求。
4.  **關聯函式作為建構子**：關聯函式常用作型別的建構子，提供多種創建實例的方式，例如 `new` 或 `default` 方法。
5.  **方法鏈接**：如果方法返回 `self` 或 `&mut self`，可以實現方法鏈接 (Method Chaining)，使得程式碼更加簡潔流暢。

#### 實際應用

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

// `impl` 關鍵字用於為結構體定義方法
impl Rectangle {
    // 實例方法 (Instance method) - 借用不可變的 self
    // &self 是 self: &Self 的縮寫
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // 實例方法 - 借用可變的 self
    // &mut self 是 self: &mut Self 的縮寫
    fn set_width(&mut self, width: u32) {
        self.width = width;
    }

    // 實例方法 - 取得所有權
    // self 是 self: Self 的縮寫 (較少見，通常用於轉換或消耗實例)
    fn destroy(self) {
        println!("Destroying rectangle with width {} and height {}", self.width, self.height);
        // self 在此方法結束後被銷毀
    }

    // 關聯函式 (Associated function) - 不以 self 作為第一個參數
    // 通常用作建構函式 (constructor)
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }

    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}

fn main() {
    let rect1 = Rectangle::new(30, 50);
    let mut rect2 = Rectangle { width: 10, height: 40 };
    let sq = Rectangle::square(20);

    println!("rect1 area: {}", rect1.area());
    println!("rect2 area: {}", rect2.area());
    println!("Square area: {}", sq.area());

    rect2.set_width(25);
    println!("rect2 new width: {}, new area: {}", rect2.width, rect2.area());

    // rect1.set_width(5); // 錯誤! rect1 是不可變的

    rect1.destroy(); // rect1 的所有權被轉移
    // println!("rect1 area after destroy: {}", rect1.area()); // 錯誤! rect1 已被移動
}
```

#### 練習題

1.  為 `Circle` 結構體（包含 `radius: f64`）定義一個關聯函式 `new(radius: f64) -> Circle` 和一個實例方法 `circumference(&self) -> f64` 計算圓的周長。
2.  為 `User` 結構體（在「結構體」練習題中定義）添加一個實例方法 `change_email(&mut self, new_email: String)`，用於修改用戶的電子郵件地址。
3.  定義一個 `TrafficLight` 列舉，包含 `Red`, `Yellow`, `Green` 三種狀態。為 `TrafficLight` 實現一個實例方法 `next_state(&self) -> TrafficLight`，使其可以循環切換狀態（Red -> Green -> Yellow -> Red）。

## 3. 列舉 (Enums)

列舉允許你定義一個可以表示多種可能變體的型別。

#### 設計理念與重要性

列舉 (Enums) 在 Rust 中是一種強大的自訂資料型別，它允許你定義一個型別，該型別可以是一組預定義的、不同的變體中的一個。每個變體可以選擇性地包含相關的資料。這種設計模式非常適合表示有限的、離散的狀態或資料集，例如網路請求的狀態、不同類型的訊息或錯誤碼。列舉與模式匹配 (Pattern Matching) 結合使用時，能夠提供詳盡的檢查，確保程式碼處理了所有可能的狀態，從而提高程式碼的健壯性和安全性，避免了許多潛在的運行時錯誤。

#### 語法要點

- **定義列舉**：使用 `enum` 關鍵字，後跟列舉名稱和一對花括號 `{}`，內部定義各個變體。例如 `enum IpAddrKind { V4, V6, }`。
- **帶有資料的變體**：列舉的每個變體都可以選擇性地包含資料，這些資料可以是匿名結構體、元組或單個值。例如 `enum Message { Quit, Move { x: i32, y: i32 }, Write(String), ChangeColor(u8, u8, u8), }`。
- **`match` 表達式**：列舉最常用於 `match` 表達式中，以詳盡地處理每個可能的變體。`match` 必須覆蓋所有可能的變體，否則會導致編譯錯誤。
- **`if let` 語法**：當只需要處理列舉的一個特定變體而忽略其他變體時，可以使用 `if let` 語法，這是一種更簡潔的 `match` 語法。
- **標準庫列舉**：Rust 標準庫提供了許多有用的列舉，例如 `Option<T>`（用於處理可能存在或不存在的值）和 `Result<T, E>`（用於處理可能成功或失敗的操作）。

#### 關鍵特性說明

1.  **窮舉性檢查**：`match` 表達式要求處理列舉的所有可能變體，這在編譯時提供了強大的窮舉性檢查，防止了未處理的情況導致的錯誤。
2.  **資料綁定**：列舉變體可以攜帶不同型別和數量的值，這使得列舉能夠靈活地表示複雜的資料結構，並在模式匹配時方便地解構這些值。
3.  **狀態機**：列舉非常適合實現狀態機，每個變體代表一個狀態，並可以根據當前狀態和輸入來轉換到下一個狀態。
4.  **錯誤處理**：`Result<T, E>` 列舉是 Rust 錯誤處理的基石，它明確地表示操作可能成功並返回一個值 (`Ok(T)`)，或者失敗並返回一個錯誤 (`Err(E)`)。
5.  **可選值處理**：`Option<T>` 列舉用於表示一個值可能存在 (`Some(T)`) 或不存在 (`None`)，這避免了空指針 (null pointer) 的問題，提高了程式碼的安全性。

#### 實際應用

```rust
// 定義一個簡單的列舉
enum Message {
    Quit,                        // 沒有附加資料
    Move { x: i32, y: i32 },     // 包含匿名結構體
    Write(String),               // 包含一個 String
    ChangeColor(u8, u8, u8),     // 包含三個 u8 值
}

// 列舉也可以有方法
impl Message {
    fn call(&self) {
        match self {
            Message::Quit => println!("Quit message received."),
            Message::Move { x, y } => println!("Move to x: {}, y: {}", x, y),
            Message::Write(text) => println!("Text message: {}", text),
            Message::ChangeColor(r, g, b) => println!("Change color to R:{}, G:{}, B:{}", r, g, b),
        }
    }
}

// Option<T> 是一個標準庫提供的常用列舉
// enum Option<T> {
//     Some(T),
//     None,
// }

fn main() {
    let msg1 = Message::Quit;
    let msg2 = Message::Move { x: 10, y: 20 };
    let msg3 = Message::Write(String::from("Hello from enum"));
    let msg4 = Message::ChangeColor(255, 0, 128);

    msg1.call();
    msg2.call();
    msg3.call();
    msg4.call();

    // 使用 Option<T>
    let some_number = Some(5);
    let some_string = Some("a string");
    let absent_number: Option<i32> = None;

    match some_number {
        Some(val) => println!("Got a number: {}", val),
        None => println!("No number."),
    }
    match absent_number {
        Some(val) => println!("Got a number: {}", val), // 不會執行
        None => println!("No number here."),
    }
}
```

#### 練習題

1.  定義一個列舉 `Coin`，包含 `Penny`, `Nickel`, `Dime`, `Quarter` 四種變體，並為每個變體附加其對應的美元價值（例如 Penny 為 1 美分）。編寫一個函式 `value_in_cents(coin: Coin) -> u8`，返回硬幣的價值。
2.  定義一個列舉 `WebEvent`，包含 `PageLoad`, `PageUnload`, `KeyPress(char)`, `Paste(String)`, `Click { x: i32, y: i32 }` 等變體。編寫一個函式 `handle_event(event: WebEvent)`，使用 `match` 表達式處理不同類型的事件並印出相關資訊。
3.  使用 `Option<T>` 和 `match` 表達式，編寫一個函式 `divide(numerator: f64, denominator: f64) -> Option<f64>`，如果分母為零，則返回 `None`，否則返回 `Some(結果)`。

## 4. Trait (用於共享行為)

Trait 類似其他語言中的介面 (Interface)，它定義了某些型別為了實現特定功能所必須提供的行為。

#### 設計理念與重要性

Trait 是 Rust 實現共享行為的核心機制，類似於其他語言中的介面 (Interfaces) 或抽象類別。它定義了一組方法簽名，任何型別只要實現了這些方法，就被認為實現了該 Trait。這種設計使得 Rust 能夠在不使用繼承的情況下實現多型 (Polymorphism) 和程式碼重用。Trait 強調行為的抽象和分離，使得程式碼更具模組化和可擴展性，同時與泛型結合，能夠編寫出既安全又高效的通用程式碼。

#### 語法要點

- **定義 Trait**：使用 `trait` 關鍵字，後跟 Trait 名稱和一對花括號 `{}`，內部定義方法簽名。方法可以有預設實作。例如 `trait Summary { fn summarize(&self) -> String; }`。
- **為型別實作 Trait**：使用 `impl Trait for Type` 語法來為特定型別實作 Trait。例如 `impl Summary for NewsArticle { ... }`。
- **Trait 作為參數 (Trait Bounds)**：在函式或結構體的泛型參數上使用 Trait Bounds 來約束型別必須實現特定的 Trait。例如 `fn notify(item: &impl Summary)` 或 `fn notify<T: Summary>(item: &T)`。
- **多個 Trait Bounds**：可以使用 `+` 運算子指定多個 Trait Bounds。例如 `fn notify_multi<T: Summary + Display>(item: &T)`。
- **Trait 物件 (Trait Objects)**：使用 `dyn Trait` 語法來創建 Trait 物件，實現動態分派。這允許在運行時處理不同但實現了相同 Trait 的型別。例如 `Box<dyn Summary>`。

#### 關鍵特性說明

1.  **行為抽象**：Trait 允許你定義一組行為，而不關心實現這些行為的具體型別，從而實現程式碼的抽象和通用性。
2.  **多型**：通過 Trait Bounds 和 Trait 物件，Rust 實現了靜態多型（編譯時分派）和動態多型（運行時分派），使得程式碼能夠處理多種不同型別的資料，只要它們實現了相同的 Trait。
3.  **程式碼重用**：預設實作允許 Trait 提供基本功能，減少了重複程式碼，同時允許具體型別覆寫預設行為以滿足特定需求。
4.  **模組化和可擴展性**：Trait 使得新的行為可以被添加到現有的型別上，而無需修改原始型別的定義，這極大地提高了程式碼的模組化和可擴展性。
5.  **與所有權系統協同**：Trait 與 Rust 的所有權和借用系統緊密協同，確保了在實現和使用 Trait 時的記憶體安全。

#### 實際應用

```rust
// 定義一個 Trait
trait Summary {
    fn summarize_author(&self) -> String; // 必須實現的方法

    // 帶有預設實作的方法
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

struct NewsArticle {
    headline: String,
    location: String,
    author: String,
    content: String,
}

// 為 NewsArticle 實現 Summary Trait
impl Summary for NewsArticle {
    fn summarize_author(&self) -> String {
        format!("@{}", self.author)
    }
    // summarize 方法使用預設實作
}

struct Tweet {
    username: String,
    content: String,
    reply: bool,
    retweet: bool,
}

// 為 Tweet 實現 Summary Trait
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }

    // 覆寫預設的 summarize 方法
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

// 接受任何實現了 Summary Trait 的型別作為參數 (靜態分派)
// fn notify(item: &impl Summary) { // 簡寫語法
fn notify<T: Summary>(item: &T) { // 完整泛型語法 (Trait Bound)
    println!("Breaking news! {}", item.summarize());
}

// 函式返回實現了 Trait 的型別
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}

fn main() {
    let article = NewsArticle {
        headline: String::from("Penguins win the Stanley Cup Championship!"),
        location: String::from("Pittsburgh, PA, USA"),
        author: String::from("Iceburgh"),
        content: String::from("The Pittsburgh Penguins once again are the best hockey team in the NHL."),
    };

    let tweet = Tweet {
        username: String::from("john_doe"),
        content: String::from("Rust is awesome! #rustlang"),
        reply: false,
        retweet: false,
    };

    println!("Article: {}", article.summarize());
    println!("Tweet: {}", tweet.summarize());

    notify(&article);
    notify(&tweet);

    let returned_item = returns_summarizable();
    println!("Returned item summary: {}", returned_item.summarize());
}
```

### 5. Trait 物件 (Trait Objects - 動態分派)

Trait 物件允許你在執行時期處理不同型別的實例，只要它們都實現了相同的 Trait。

```rust
trait Drawable {
    fn draw(&self);
}

struct Button {
    width: u32,
    height: u32,
    label: String,
}

impl Drawable for Button {
    fn draw(&self) {
        println!("Drawing a button: label='{}', width={}, height={}", self.label, self.width, self.height);
    }
}

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Drawable for SelectBox {
    fn draw(&self) {
        println!("Drawing a select box: options={:?}, width={}, height={}", self.options, self.width, self.height);
    }
}

// Screen 結構體持有一個 Drawable Trait 物件的 Vector
// Box<dyn Drawable> 是一個 Trait 物件，它是一個指向堆上實現了 Drawable Trait 的型別的指標
struct Screen {
    components: Vec<Box<dyn Drawable>>,
}

impl Screen {
    fn run(&self) {
        for component in self.components.iter() {
            component.draw(); // 動態分派：在執行時期決定呼叫哪個 draw 方法
        }
    }
}

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No"),
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

### 6. 模組與可見性 (Modules and Visibility) - 簡介

模組用於組織程式碼並控制可見性 (哪些部分是公開的，哪些是私有的)。

```rust
// 想像這是一個名為 `my_library` 的 crate 的 `src/lib.rs` 或 `src/main.rs`

// 定義一個名為 `front_of_house` 的模組
mod front_of_house {
    // 預設情況下，模組內的項目是私有的
    // 使用 `pub` 使模組或其內的項目公開
    pub mod hosting {
        // 這個函式是公開的，因為 hosting 模組是公開的，且函式本身也是 pub
        pub fn add_to_waitlist() {
            println!("Added to waitlist in front_of_house::hosting");
            seat_at_table(); // 可以呼叫同一模組內的私有函式
        }

        fn seat_at_table() {
            println!("Seated at table in front_of_house::hosting");
        }
    }

    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}

// 使用 `use` 關鍵字將路徑引入作用域
use crate::front_of_house::hosting;
// 也可以使用相對路徑: use self::front_of_house::hosting;
// 或重新匯出: pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    // 絕對路徑
    crate::front_of_house::hosting::add_to_waitlist();

    // 使用 `use` 後的相對路徑
    hosting::add_to_waitlist();
    // hosting::seat_at_table(); // 錯誤! seat_at_table 是私有的

    // 結構體欄位的可見性
    // 如果結構體是 pub，其欄位預設是私有的，除非也標記為 pub
    mod back_of_house {
        pub struct Breakfast {
            pub toast: String,      // 這個欄位是公開的
            seasonal_fruit: String, // 這個欄位是私有的
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

    let mut meal = back_of_house::Breakfast::summer("Rye");
    meal.toast = String::from("Wheat"); // 可以修改，因為 toast 是 pub
    println!("I'd like {} toast please", meal.toast);
    // meal.seasonal_fruit = String::from("blueberries"); // 錯誤! seasonal_fruit 是私有的
}

fn main() {
    eat_at_restaurant();
}
```

## 練習題

1.  **幾何形狀與方法:**
    *   定義一個 `Circle` 結構體，包含 `radius: f64` 欄位。
    *   為 `Circle` 實現一個 `area(&self) -> f64` 方法來計算面積 (`π * r^2`)。
    *   為 `Circle` 實現一個關聯函式 `new(radius: f64) -> Circle` 作為建構函式。
    *   在 `main` 函式中創建一個圓並印出其面積。

2.  **列舉與 `match` 應用:**
    *   定義一個 `Coin` 列舉，包含變體 `Penny`, `Nickel`, `Dime`, `Quarter`。
    *   編寫一個函式 `value_in_cents(coin: Coin) -> u8`，它接受一個 `Coin` 並使用 `match` 返回其美分價值 (Penny: 1, Nickel: 5, Dime: 10, Quarter: 25)。
    *   在 `main` 函式中測試此函式。

3.  **媒體播放器 Trait:**
    *   定義一個 `Playable` Trait，包含一個方法 `play(&self)` 和一個方法 `stop(&self)`。
    *   創建兩個結構體 `Audio` (包含 `title: String`, `duration_seconds: u32`) 和 `Video` (包含 `title: String`, `duration_seconds: u32`, `resolution: String`)。
    *   為 `Audio` 和 `Video` 實現 `Playable` Trait。`play` 方法可以簡單地印出正在播放的媒體標題和類型，`stop` 方法印出停止播放。
    *   編寫一個泛型函式 `start_playback<T: Playable>(media: &T)`，它呼叫 `media.play()`。
    *   在 `main` 中創建 `Audio` 和 `Video` 的實例，並使用 `start_playback` 播放它們。

4.  **動物園 Trait 物件:**
    *   定義一個 `Animal` Trait，包含一個方法 `make_sound(&self) -> String`。
    *   創建 `Dog` 和 `Cat` 結構體，並為它們實現 `Animal` Trait (`Dog` 返回 "Woof!", `Cat` 返回 "Meow!")。
    *   在 `main` 函式中，創建一個 `Vec<Box<dyn Animal>>`，並將一個 `Dog` 實例和一個 `Cat` 實例放入其中。
    *   遍歷這個 vector，並對每個動物呼叫 `make_sound()` 方法並印出結果。

5.  **簡單日誌模組 (組織與可見性):**
    *   創建一個名為 `my_logger` 的新模組。
    *   在 `my_logger` 中，定義一個私有的 `log_internal(level: &str, message: &str)` 函式，它會印出類似 `[LEVEL] message` 的格式。
    *   在 `my_logger` 中，定義一個公有的 `info(message: &str)` 函式，它呼叫 `log_internal` 並傳入 "INFO" 作為級別。
    *   在 `my_logger` 中，定義一個公有的 `warn(message: &str)` 函式，它呼叫 `log_internal` 並傳入 "WARN" 作為級別。
    *   在 `main` 函式中，使用 `my_logger::info` 和 `my_logger::warn` 來記錄一些訊息，展示模組的使用和可見性規則。
    *   思考：如果 `log_internal` 是公有的，會有什麼不同？為什麼將其設為私有是個好主意？

---

通過本章的學習，你已經掌握了 Rust 中定義和使用結構體、列舉、方法、Trait 以及模組的基本方式。這些是構建複雜應用程式的基石。Rust 雖然沒有傳統的類別繼承，但其 Trait 和泛型系統提供了強大且靈活的方式來實現程式碼的抽象和重用。模組系統則幫助我們組織程式碼，控制可見性，從而構建出易於維護和擴展的專案。

在下一章中，我們將探討 Rust 的錯誤處理機制，這是編寫健壯程式碼的關鍵部分。
