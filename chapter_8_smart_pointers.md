# 第八章 - 智慧指標 (Smart Pointers)

智慧指標是 Rust 中一個核心概念，它們不僅能存儲記憶體地址，還提供額外的功能和元資料。對於有其他程式語言經驗的開發者來說，智慧指標是理解 Rust 所有權系統的關鍵。

在傳統語言中，指標就是記憶體地址。但在 Rust 中，智慧指標是一種資料結構，除了包含記憶體地址外，還有額外的元資料和功能。它們通常實作了 `Deref` 和 `Drop` trait。

**與其他語言的對比：**
- **C/C++**：需要手動管理記憶體（malloc/free, new/delete）
- **Java/C#**：有垃圾回收器自動管理
- **Rust**：使用智慧指標在編譯時確保記憶體安全

## 1. `Box<T>` - 堆記憶體分配

### 基本概念

`Box<T>` 是最簡單的智慧指標，用於將資料存儲在堆上而不是棧上。

```rust
fn main() {
  // 在棧上存儲
  let stack_value = 5;
  
  // 在堆上存儲
  let heap_value = Box::new(5);
  
  println!("棧上的值: {}", stack_value);
  println!("堆上的值: {}", *heap_value); // 使用 * 解引用
}
```

### 使用場景

**1. 大型資料結構**
```rust
struct LargeData {
  data: [i32; 1000000], // 很大的陣列
}

fn main() {
  // 避免棧溢出
  let large = Box::new(LargeData {
      data: [0; 1000000],
  });
  println!("資料已分配到堆上");
}
```

**2. 遞迴資料結構**
```rust
// 鏈表節點
#[derive(Debug)]
enum List {
  Cons(i32, Box<List>), // 必須使用 Box，否則編譯器無法確定大小
  Nil,
}

fn main() {
  use List::{Cons, Nil};
  
  let list = Cons(1, 
      Box::new(Cons(2, 
          Box::new(Cons(3, 
              Box::new(Nil))))));
  
  println!("{:?}", list);
}
```

**3. Trait 物件**
```rust
trait Animal {
  fn make_sound(&self);
}

struct Dog;
struct Cat;

impl Animal for Dog {
  fn make_sound(&self) {
      println!("汪汪！");
  }
}

impl Animal for Cat {
  fn make_sound(&self) {
      println!("喵喵！");
  }
}

fn main() {
  // 將不同類型存儲在同一個集合中
  let animals: Vec<Box<dyn Animal>> = vec![
      Box::new(Dog),
      Box::new(Cat),
  ];
  
  for animal in animals {
      animal.make_sound();
  }
}
```

## 2. `Rc<T>` - 引用計數（單執行緒）

### 基本概念

`Rc<T>`（Reference Counted）允許一個值有多個所有者。當最後一個所有者被丟棄時，值才會被清理。

```rust
use std::rc::Rc;

fn main() {
  let data = Rc::new(String::from("共享資料"));
  
  let owner1 = Rc::clone(&data); // 增加引用計數
  let owner2 = Rc::clone(&data); // 再次增加引用計數
  
  println!("資料: {}", data);
  println!("owner1: {}", owner1);
  println!("owner2: {}", owner2);
  println!("引用計數: {}", Rc::strong_count(&data)); // 輸出: 3
}
```

### 實際應用

**共享配置物件**
```rust
use std::rc::Rc;

#[derive(Debug)]
struct Config {
  database_url: String,
  timeout: u32,
}

struct Service {
  name: String,
  config: Rc<Config>,
}

impl Service {
  fn new(name: &str, config: Rc<Config>) -> Self {
      Service {
          name: name.to_string(),
          config,
      }
  }
  
  fn start(&self) {
      println!("服務 {} 啟動，使用資料庫: {}", 
               self.name, self.config.database_url);
  }
}

fn main() {
  let config = Rc::new(Config {
      database_url: "postgresql://localhost/mydb".to_string(),
      timeout: 30,
  });
  
  let service1 = Service::new("用戶服務", Rc::clone(&config));
  let service2 = Service::new("訂單服務", Rc::clone(&config));
  
  service1.start();
  service2.start();
  
  println!("配置引用計數: {}", Rc::strong_count(&config));
}
```

### 循環引用問題

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
  value: i32,
  children: RefCell<Vec<Rc<Node>>>,
  parent: RefCell<Weak<Node>>, // 使用 Weak 避免循環引用
}

fn main() {
  let parent = Rc::new(Node {
      value: 1,
      children: RefCell::new(vec![]),
      parent: RefCell::new(Weak::new()),
  });
  
  let child = Rc::new(Node {
      value: 2,
      children: RefCell::new(vec![]),
      parent: RefCell::new(Rc::downgrade(&parent)), // 創建弱引用
  });
  
  parent.children.borrow_mut().push(Rc::clone(&child));
  
  println!("父節點: {:?}", parent.value);
  println!("子節點: {:?}", child.value);
}
```

## 3. `RefCell<T>` - 內部可變性

### 基本概念

`RefCell<T>` 允許在擁有不可變引用的情況下修改資料，將借用檢查從編譯時移到執行時。

```rust
use std::cell::RefCell;

fn main() {
  let data = RefCell::new(vec![1, 2, 3]);
  
  // 不可變借用
  {
      let borrowed = data.borrow();
      println!("資料: {:?}", *borrowed);
  } // borrowed 在此離開作用域
  
  // 可變借用
  {
      let mut borrowed_mut = data.borrow_mut();
      borrowed_mut.push(4);
      println!("修改後: {:?}", *borrowed_mut);
  }
  
  println!("最終資料: {:?}", *data.borrow());
}
```

### 與 Rc 結合使用

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct Counter {
  value: Rc<RefCell<i32>>,
}

impl Counter {
  fn new(initial: i32) -> Self {
      Counter {
          value: Rc::new(RefCell::new(initial)),
      }
  }
  
  fn increment(&self) {
      *self.value.borrow_mut() += 1;
  }
  
  fn get(&self) -> i32 {
      *self.value.borrow()
  }
  
  fn share(&self) -> Self {
      Counter {
          value: Rc::clone(&self.value),
      }
  }
}

fn main() {
  let counter1 = Counter::new(0);
  let counter2 = counter1.share(); // 共享同一個計數器
  
  counter1.increment();
  counter2.increment();
  
  println!("counter1: {}", counter1.get()); // 2
  println!("counter2: {}", counter2.get()); // 2
}
```

### 實際應用 - Mock 物件

```rust
use std::cell::RefCell;

trait Logger {
  fn log(&self, message: &str);
}

// 真實的日誌記錄器
struct FileLogger;

impl Logger for FileLogger {
  fn log(&self, message: &str) {
      println!("寫入檔案: {}", message);
  }
}

// 測試用的 Mock 物件
struct MockLogger {
  messages: RefCell<Vec<String>>,
}

impl MockLogger {
  fn new() -> Self {
      MockLogger {
          messages: RefCell::new(vec![]),
      }
  }
  
  fn get_messages(&self) -> Vec<String> {
      self.messages.borrow().clone()
  }
}

impl Logger for MockLogger {
  fn log(&self, message: &str) {
      // 在不可變 self 下修改內部狀態
      self.messages.borrow_mut().push(message.to_string());
  }
}

fn main() {
  let mock = MockLogger::new();
  
  mock.log("第一條訊息");
  mock.log("第二條訊息");
  
  println!("記錄的訊息: {:?}", mock.get_messages());
}
```

## 4. `Deref` Trait - 自訂解引用行為

### 基本概念

`Deref` trait 允許自訂 `*` 運算子的行為，讓智慧指標能像普通引用一樣使用。

```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
  fn new(x: T) -> MyBox<T> {
      MyBox(x)
  }
}

impl<T> Deref for MyBox<T> {
  type Target = T;
  
  fn deref(&self) -> &Self::Target {
      &self.0
  }
}

fn main() {
  let x = 5;
  let y = MyBox::new(x);
  
  assert_eq!(5, x);
  assert_eq!(5, *y); // 使用自訂的解引用
  
  println!("MyBox 中的值: {}", *y);
}
```

### Deref 強制轉換

```rust
use std::ops::Deref;

struct MyString {
  data: String,
}

impl MyString {
  fn new(s: &str) -> Self {
      MyString {
          data: s.to_string(),
      }
  }
}

impl Deref for MyString {
  type Target = String;
  
  fn deref(&self) -> &Self::Target {
      &self.data
  }
}

fn print_string(s: &str) {
  println!("字串: {}", s);
}

fn main() {
  let my_string = MyString::new("Hello, Rust!");
  
  // Deref 強制轉換：&MyString -> &String -> &str
  print_string(&my_string);
  
  // 也可以使用 String 的方法
  println!("長度: {}", my_string.len());
  println!("大寫: {}", my_string.to_uppercase());
}
```

## 5. `Drop` Trait - 自動清理

### 基本概念

`Drop` trait 定義了值離開作用域時的清理行為，類似於其他語言中的解構函數。

```rust
struct Resource {
  name: String,
}

impl Resource {
  fn new(name: &str) -> Self {
      println!("獲取資源: {}", name);
      Resource {
          name: name.to_string(),
      }
  }
}

impl Drop for Resource {
  fn drop(&mut self) {
      println!("釋放資源: {}", self.name);
  }
}

fn main() {
  let _resource1 = Resource::new("資料庫連線");
  
  {
      let _resource2 = Resource::new("檔案控制代碼");
      // resource2 在此處自動釋放
  }
  
  println!("主函數繼續執行");
  // resource1 在 main 結束時釋放
}
```

### 手動釋放

```rust
fn main() {
  let resource = Resource::new("重要資源");
  
  // 使用 std::mem::drop 提前釋放
  std::mem::drop(resource);
  
  println!("資源已提前釋放");
}
```

## 6. 智慧指標選擇指南

| 需求 | 使用的智慧指標 | 說明 |
|------|---------------|------|
| 堆上分配資料 | `Box<T>` | 單一所有權，最簡單 |
| 多個所有者共享資料 | `Rc<T>` | 引用計數，單執行緒 |
| 在不可變引用下修改 | `RefCell<T>` | 內部可變性，執行時檢查 |
| 多所有權+可變性 | `Rc<RefCell<T>>` | 常見組合模式 |
| 遞迴資料結構 | `Box<T>` 或 `Rc<T>` | 根據是否需要共享 |
| Trait 物件 | `Box<dyn Trait>` | 動態分派 |

## 7. 練習題

### 基礎練習

**練習 1：二元樹**
實作一個簡單的二元搜尋樹，支援插入和搜尋功能。使用 `Box<T>` 來處理遞迴結構。

**提示：**
- 定義一個 `BinaryTree` enum，包含 `Node` 和 `Empty` 變體
- `Node` 應該包含值和左右子樹
- 實作 `insert` 和 `contains` 方法

**練習 2：共享計數器**
使用 `Rc<RefCell<T>>` 實作一個可以被多個物件共享的計數器。

**要求：**
- 多個計數器實例共享同一個內部值
- 支援增加、減少和獲取當前值
- 任一實例的修改都會影響其他實例

**練習 3：自訂智慧指標**
實作一個具有存取計數功能的智慧指標，每次解引用時記錄存取次數。

**要求：**
- 實作 `Deref` trait
- 使用 `RefCell` 來存儲存取計數
- 提供方法獲取總存取次數

## 8. 總結

### 核心要點

1. **`Box<T>`**：最簡單的智慧指標，用於堆分配
2. **`Rc<T>`**：允許多個所有者，使用引用計數
3. **`RefCell<T>`**：提供內部可變性，執行時借用檢查
4. **`Deref`**：讓智慧指標像普通引用一樣使用
5. **`Drop`**：自動清理資源，確保記憶體安全

### 選擇建議

- **需要堆分配**：使用 `Box<T>`
- **需要多個所有者**：使用 `Rc<T>`
- **需要在不可變引用下修改**：使用 `RefCell<T>`
- **需要多所有權且可變**：使用 `Rc<RefCell<T>>`

### 注意事項

- `RefCell<T>` 的借用檢查在執行時進行，違反規則會導致 panic
- 注意循環引用問題，必要時使用 `Weak<T>`
- 智慧指標有少量的執行時開銷，但換來了記憶體安全

掌握這些智慧指標將幫助你更好地理解和使用 Rust 的所有權系統，寫出安全且高效的程式碼。
