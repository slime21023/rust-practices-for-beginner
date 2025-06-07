# 第八章 - 智慧指標 (Smart Pointers)

智慧指標是具有額外元資料和功能的指標，在 Rust 中用於實現複雜的記憶體管理模式。本章介紹五個核心智慧指標：`Box<T>`、`Deref` Trait、`Drop` Trait、`Rc<T>` 和 `RefCell<T>`。

## 1. `Box<T>` - 堆記憶體分配

### 核心概念

- `Box<T>` 將資料儲存在堆上而非棧上
- 提供單一所有權的堆分配
- 主要用於遞迴型別和大型資料結構

### 使用場景

- 編譯時大小未知的型別
- 大量資料避免棧溢出
- 遞迴資料結構（鏈表、樹）
- Trait 物件

### 基本語法

```rust
fn main() {
    // 基本用法
    let b = Box::new(5);
    println!("b = {}", *b);

    // 遞迴型別 - 鏈表
    #[derive(Debug)]
    enum List {
        Cons(i32, Box<List>),
        Nil,
    }

    use List::{Cons, Nil};
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
    println!("{:?}", list);

    // Trait 物件
    trait Draw {
        fn draw(&self);
    }

    struct Button;
    impl Draw for Button {
        fn draw(&self) {
            println!("Drawing button");
        }
    }

    let button: Box<dyn Draw> = Box::new(Button);
    button.draw();
}
```

## 2. `Deref` Trait - 自訂解引用

### 核心概念

- 允許自訂 `*` 運算子的行為
- 實現 Deref 強制轉換 (Deref Coercion)
- 讓智慧指標像普通引用一樣使用

### Deref 強制轉換

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

fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    
    // Deref 強制轉換：&MyBox<String> -> &String -> &str
    hello(&m);
    
    // 手動解引用
    println!("Value: {}", *m);
}
```

### DerefMut Trait

```rust
use std::ops::{Deref, DerefMut};

struct MyVec<T> {
    data: Vec<T>,
}

impl<T> MyVec<T> {
    fn new() -> Self {
        MyVec { data: Vec::new() }
    }
}

impl<T> Deref for MyVec<T> {
    type Target = Vec<T>;

    fn deref(&self) -> &Self::Target {
        &self.data
    }
}

impl<T> DerefMut for MyVec<T> {
    fn deref_mut(&mut self) -> &mut Self::Target {
        &mut self.data
    }
}

fn main() {
    let mut v = MyVec::new();
    v.push(1); // 透過 DerefMut 使用 Vec 的方法
    println!("Length: {}", v.len());
}
```

## 3. `Drop` Trait - 自動清理

### 核心概念

- 定義值離開作用域時的清理行為
- 自動呼叫，確保資源釋放
- 實現 RAII (Resource Acquisition Is Initialization) 模式

### 基本語法

```rust
struct FileHandler {
    filename: String,
}

impl FileHandler {
    fn new(filename: &str) -> Self {
        println!("Opening file: {}", filename);
        FileHandler {
            filename: filename.to_string(),
        }
    }
}

impl Drop for FileHandler {
    fn drop(&mut self) {
        println!("Closing file: {}", self.filename);
    }
}

fn main() {
    let _file1 = FileHandler::new("config.txt");
    
    {
        let _file2 = FileHandler::new("data.txt");
        // file2 在此處被 drop
    }
    
    // 使用 std::mem::drop 提前釋放
    let file3 = FileHandler::new("temp.txt");
    std::mem::drop(file3);
    println!("File3 已被手動釋放");
    
    // file1 在 main 結束時被 drop
}
```

### 實際應用 - 資源管理

```rust
use std::sync::Mutex;

struct DatabaseConnection {
    id: u32,
}

impl DatabaseConnection {
    fn new(id: u32) -> Self {
        println!("建立資料庫連線 {}", id);
        DatabaseConnection { id }
    }

    fn execute_query(&self, query: &str) {
        println!("連線 {} 執行查詢: {}", self.id, query);
    }
}

impl Drop for DatabaseConnection {
    fn drop(&mut self) {
        println!("關閉資料庫連線 {}", self.id);
    }
}

fn main() {
    let conn = DatabaseConnection::new(1);
    conn.execute_query("SELECT * FROM users");
    // 連線在此自動關閉
}
```

## 4. `Rc<T>` - 多所有權引用計數

### 核心概念

- 允許多個所有者共享同一份資料
- 使用引用計數追蹤所有者數量
- 僅適用於單執行緒環境

### 基本語法

```rust
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

fn main() {
    use List::{Cons, Nil};

    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("a 的引用計數: {}", Rc::strong_count(&a));

    let b = Cons(3, Rc::clone(&a));
    println!("創建 b 後 a 的引用計數: {}", Rc::strong_count(&a));

    {
        let c = Cons(4, Rc::clone(&a));
        println!("創建 c 後 a 的引用計數: {}", Rc::strong_count(&a));
    }
    
    println!("c 離開作用域後 a 的引用計數: {}", Rc::strong_count(&a));
}
```

### 實際應用 - 圖結構

```rust
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    id: i32,
    children: Vec<Rc<Node>>,
}

impl Node {
    fn new(id: i32) -> Rc<Self> {
        Rc::new(Node {
            id,
            children: Vec::new(),
        })
    }

    fn add_child(mut self: Rc<Self>, child: Rc<Node>) -> Rc<Self> {
        if let Some(node) = Rc::get_mut(&mut self) {
            node.children.push(child);
        }
        self
    }
}

fn main() {
    let root = Node::new(1);
    let child1 = Node::new(2);
    let child2 = Node::new(3);
    
    // 共享子節點
    let shared_child = Node::new(4);
    
    println!("shared_child 引用計數: {}", Rc::strong_count(&shared_child));
    
    let root = root.add_child(child1);
    let root = root.add_child(child2);
    let root = root.add_child(Rc::clone(&shared_child));
    
    println!("shared_child 引用計數: {}", Rc::strong_count(&shared_child));
}
```

### 循環引用問題

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf 強引用計數: {}", Rc::strong_count(&leaf));
    println!("leaf 弱引用計數: {}", Rc::weak_count(&leaf));

    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!("branch 強引用計數: {}", Rc::strong_count(&branch));
        println!("leaf 強引用計數: {}", Rc::strong_count(&leaf));
    }

    println!("leaf 父節點 = {:?}", leaf.parent.borrow().upgrade());
}
```

## 5. `RefCell<T>` - 內部可變性

### 核心概念

- 在不可變引用下修改資料（內部可變性）
- 將借用檢查從編譯時移到執行時
- 違反借用規則會導致 panic

### 基本語法

```rust
use std::cell::RefCell;

fn main() {
    let data = RefCell::new(vec![1, 2, 3]);

    // 不可變借用
    {
        let borrowed = data.borrow();
        println!("Data: {:?}", *borrowed);
    } // borrowed 在此離開作用域

    // 可變借用
    {
        let mut borrowed_mut = data.borrow_mut();
        borrowed_mut.push(4);
        println!("Modified data: {:?}", *borrowed_mut);
    }

    // 同時借用會 panic
    // let r1 = data.borrow();
    // let r2 = data.borrow_mut(); // panic!
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

    fn get_value(&self) -> i32 {
        *self.value.borrow()
    }

    fn clone_counter(&self) -> Self {
        Counter {
            value: Rc::clone(&self.value),
        }
    }
}

fn main() {
    let counter1 = Counter::new(0);
    let counter2 = counter1.clone_counter();

    counter1.increment();
    counter2.increment();

    println!("Counter1 value: {}", counter1.get_value()); // 2
    println!("Counter2 value: {}", counter2.get_value()); // 2
    println!("引用計數: {}", Rc::strong_count(&counter1.value));
}
```

### Mock 物件模式

```rust
use std::cell::RefCell;

trait Messenger {
    fn send(&self, msg: &str);
}

struct MockMessenger {
    sent_messages: RefCell<Vec<String>>,
}

impl MockMessenger {
    fn new() -> MockMessenger {
        MockMessenger {
            sent_messages: RefCell::new(vec![]),
        }
    }
}

impl Messenger for MockMessenger {
    fn send(&self, message: &str) {
        // 在不可變 self 下修改內部狀態
        self.sent_messages.borrow_mut().push(String::from(message));
    }
}

fn main() {
    let mock_messenger = MockMessenger::new();
    mock_messenger.send("Hello");
    mock_messenger.send("World");

    println!("Messages sent: {:?}", mock_messenger.sent_messages.borrow());
}
```

## 6. 智慧指標組合模式

### 常見組合

```rust
use std::rc::Rc;
use std::cell::RefCell;

// 1. Rc<RefCell<T>> - 多所有權 + 內部可變性
type SharedMutable<T> = Rc<RefCell<T>>;

// 2. Box<dyn Trait> - 堆分配 + Trait 物件
trait Animal {
    fn make_sound(&self);
}

struct Dog;
impl Animal for Dog {
    fn make_sound(&self) {
        println!("Woof!");
    }
}

fn main() {
    // 多所有權的可變資料
    let shared_data: SharedMutable<Vec<i32>> = Rc::new(RefCell::new(vec![1, 2, 3]));
    let data_ref1 = Rc::clone(&shared_data);
    let data_ref2 = Rc::clone(&shared_data);

    data_ref1.borrow_mut().push(4);
    data_ref2.borrow_mut().push(5);

    println!("Shared data: {:?}", shared_data.borrow());

    // Trait 物件集合
    let animals: Vec<Box<dyn Animal>> = vec![
        Box::new(Dog),
    ];

    for animal in animals {
        animal.make_sound();
    }
}
```

## 7. 效能考量

### 成本比較

```rust
use std::rc::Rc;
use std::cell::RefCell;
use std::time::Instant;

fn benchmark_smart_pointers() {
    let n = 1_000_000;

    // Box<T> - 最低開銷
    let start = Instant::now();
    let mut boxes = Vec::new();
    for i in 0..n {
        boxes.push(Box::new(i));
    }
    println!("Box<T> 時間: {:?}", start.elapsed());

    // Rc<T> - 引用計數開銷
    let start = Instant::now();
    let mut rcs = Vec::new();
    for i in 0..n {
        rcs.push(Rc::new(i));
    }
    println!("Rc<T> 時間: {:?}", start.elapsed());

    // RefCell<T> - 執行時檢查開銷
    let start = Instant::now();
    let mut refcells = Vec::new();
    for i in 0..n {
        refcells.push(RefCell::new(i));
    }
    println!("RefCell<T> 時間: {:?}", start.elapsed());
}
```

## 8. 練習題

### 基礎練習

1. **二元樹實作**：
   ```rust
   enum BinaryTree {
       Node(i32, Box<BinaryTree>, Box<BinaryTree>),
       Leaf,
   }
   
   // 實作 insert 和 search 方法
   ```

2. **自訂智慧指標**：
   ```rust
   struct MySmartPointer<T> {
       data: T,
   }
   
   // 實作 Deref 和 Drop trait
   ```

3. **共享計數器**：
   ```rust
   // 使用 Rc<RefCell<i32>> 實作多所有權計數器
   ```

### 進階練習

4. **圖結構**：實作一個有向圖，節點可以有多個父節點和子節點
5. **觀察者模式**：使用智慧指標實作觀察者模式
6. **記憶體池**：實作一個簡單的物件池模式

## 總結

### 選擇指南

| 需求 | 智慧指標 | 使用場景 |
|------|----------|----------|
| 堆分配 | `Box<T>` | 大型資料、遞迴型別 |
| 多所有權 | `Rc<T>` | 共享唯讀資料 |
| 內部可變性 | `RefCell<T>` | 在不可變引用下修改 |
| 多所有權+可變 | `Rc<RefCell<T>>` | 共享可變資料 |
| Trait 物件 | `Box<dyn Trait>` | 動態分派 |

### 關鍵概念

- **所有權管理**：智慧指標擴展了 Rust 的所有權系統
- **記憶體安全**：在編譯時或執行時確保記憶體安全
- **零成本抽象**：大部分智慧指標沒有執行時開銷
- **組合使用**：不同智慧指標可以組合使用

### 最佳實踐

- 優先使用 `Box<T>` 進行簡單的堆分配
- 只在需要多所有權時使用 `Rc<T>`
- 謹慎使用 `RefCell<T>`，注意執行時 panic 風險
- 避免循環引用，必要時使用 `Weak<T>`
- 在多執行緒環境中使用 `Arc<T>` 和 `Mutex<T>`

掌握這些智慧指標將幫助你構建更複雜、更安全的 Rust 應用程式。
