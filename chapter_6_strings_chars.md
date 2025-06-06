# 第六章 - 文字處理 (Strings & Characters)

本章將深入探討 Rust 中處理文字資料的核心機制，包括字元 (`char`)、可變字串 (`String`) 和字串切片 (`&str`)。理解這些型別及其相互關係，對於編寫高效、安全且符合 Unicode 標準的 Rust 程式至關重要。Rust 的字串處理設計強調記憶體安全、UTF-8 編碼的正確性以及避免常見的字串操作陷阱。

## 1. 字元 (`char`)

#### 設計理念與重要性

Rust 的 `char` 型別設計旨在正確處理 Unicode 字元。與許多其他語言的 `char` 型別（通常是單個位元組）不同，Rust 的 `char` 始終代表一個 Unicode 純量值 (Unicode Scalar Value)，這意味著它可以表示地球上幾乎所有的文字和符號，包括表情符號。這種設計確保了在處理多語言文本時的正確性和一致性，避免了因編碼問題導致的錯誤。

#### 語法要點

- `char` 型別使用單引號 `' '` 來定義，例如 `'a'`、`'中'`、`'😻'`。
- 每個 `char` 佔用 4 個位元組的記憶體空間，無論它代表的是 ASCII 字元還是多位元組的 Unicode 字元。
- `char` 型別提供了多種方法來檢查字元的屬性，例如 `is_alphabetic()`、`is_numeric()`、`is_whitespace()`、`to_uppercase()` 等。

#### 關鍵特性說明

1.  **Unicode 純量值**：Rust 的 `char` 型別保證能表示任何有效的 Unicode 純量值，這意味著它能處理世界上絕大多數的文字系統和符號，包括表情符號。
2.  **固定大小**：每個 `char` 都固定佔用 4 個位元組的記憶體空間，這與其表示的字元是否為 ASCII 無關。這簡化了記憶體管理，但也意味著對於純 ASCII 文本，`char` 可能會比其他語言的單字節字元佔用更多空間。
3.  **豐富的方法**：`char` 型別提供了多種實用的方法，用於檢查字元的屬性（如 `is_alphabetic()`、`is_numeric()`、`is_whitespace()`）以及進行大小寫轉換（如 `to_uppercase()`、`to_lowercase()`）。

#### 實際應用

```rust
fn main() {
    let c1 = 'a'; // 英文字母
    let c2 = '中'; // 中文字元
    let c3 = '😻'; // Emoji (一個 Unicode 純量值)

    println!("c1: {}", c1);
    println!("c2: {}", c2);
    println!("c3: {}", c3);

    // char 佔用 4 個位元組
    println!("Size of char: {} bytes", std::mem::size_of::<char>());

    // 檢查字元屬性
    let is_alphabetic = 'A'.is_alphabetic();
    let is_numeric = '7'.is_numeric();
    let is_whitespace = ' '.is_whitespace();
    let is_uppercase = 'T'.is_uppercase();
    let is_lowercase = 't'.is_lowercase();

    println!("'A' is alphabetic: {}", is_alphabetic);
    println!("'7' is numeric: {}", is_numeric);
    println!("' ' is whitespace: {}", is_whitespace);
    println!("'T' is uppercase: {}", is_uppercase);
    println!("'t' is lowercase: {}", is_lowercase);
}
```

#### 練習題

1.  編寫一個函式 `count_vowels`，接收一個 `char` 參數，如果該字元是英文母音 (a, e, i, o, u，不區分大小寫) 則返回 `true`，否則返回 `false`。
2.  編寫一個函式 `is_punctuation`，接收一個 `char` 參數，判斷該字元是否為標點符號 (例如 `.` `,` `!` `?` 等)。
3.  編寫一個函式 `to_alternating_case`，接收一個 `char` 參數，如果該字元是小寫字母則轉換為大寫，如果是大寫字母則轉換為小寫，其他字元保持不變。

## 2. 字串 (`String`)

#### 設計理念與重要性

`String` 是 Rust 標準庫中用於處理可變、可增長文本數據的核心類型。它在堆上分配記憶體，並以 UTF-8 編碼儲存字元。`String` 擁有其數據的所有權，這意味著當 `String` 離開作用域時，其佔用的記憶體會被自動釋放。這種設計提供了靈活性，允許在運行時修改字串內容，同時通過所有權系統確保記憶體安全。

#### 語法要點

- **創建 `String`**：
  - `String::new()`：創建一個空的 `String`。
  - `to_string()`：從字串字面量或其他實現 `Display` trait 的類型轉換為 `String`。
  - `String::from()`：從字串字面量創建 `String`。
- **修改 `String`**：
  - `push_str(&str)`：附加一個字串切片。
  - `push(char)`：附加一個單個字元。
- **連接 `String`**：
  - `+` 運算子：用於連接 `String` 和 `&str`。注意，`+` 運算子會消耗左側 `String` 的所有權。
  - `format!` 宏：用於格式化和連接多個字串，不會消耗任何輸入字串的所有權。

#### 關鍵特性說明

1.  **堆上分配**：`String` 將其數據儲存在堆上，這使得它的大小在運行時可以動態增長或縮小。這與固定大小、儲存在棧上的字串字面量 (`&str`) 不同。
2.  **所有權**：`String` 擁有其包含數據的所有權。這意味著當 `String` 變數離開作用域時，Rust 會自動釋放其佔用的記憶體，無需手動管理，從而避免了記憶體洩漏和雙重釋放等問題。
3.  **UTF-8 編碼**：`String` 內部以 UTF-8 編碼儲存其字元。這確保了 Rust 能夠正確處理各種語言的文字，包括多位元組字元，並避免了許多因編碼問題導致的錯誤。
4.  **可變性**：`String` 是可變的，你可以使用 `push_str()`、`push()` 等方法向其追加內容，或者使用其他方法修改其內容。這使得 `String` 成為處理需要頻繁修改的文本數據的理想選擇。
5.  **與 `&str` 的互操作性**：`String` 可以很容易地轉換為 `&str` (通過 `&s[..]` 或自動解引用)，這使得 `String` 可以作為參數傳遞給接受 `&str` 的函式，極大地提高了靈活性和程式碼重用性。

#### 實際應用

```rust
fn main() {
    // 創建 String
    let mut s1 = String::new(); // 創建一個空字串
    s1.push_str("Hello"); //附加字串切片
    println!("s1: {}", s1);

    let data = " initial contents";
    let s2 = data.to_string(); // 從 &str 轉換
    println!("s2: {}", s2);

    let s3 = String::from("你好, 世界"); // 從字串字面量創建
    println!("s3: {}", s3);

    // 更新 String
    let mut s_grow = String::from("foo");
    s_grow.push_str("bar"); // 附加字串切片
    println!("s_grow after push_str: {}", s_grow);
    s_grow.push('!'); // 附加單個字元
    println!("s_grow after push: {}", s_grow);

    // 使用 + 運算子連接字串 (會取得 s4 的所有權)
    let s4 = String::from("Hello, ");
    let s5 = String::from("world!");
    // s4 在這裡被移動，之後不能再使用 s4
    // + 運算子類似 fn add(self, s: &str) -> String { ... }
    let s6 = s4 + &s5; // s5 需要是 &str，所以用了 &s5 (Deref Coercion)
    println!("s6 (concatenated with +): {}", s6);
    // println!("{}", s4); // 這行會報錯，s4 已被移動

    // 使用 format! 宏連接字串 (不會取得所有權)
    let s7 = String::from("tic");
    let s8 = String::from("tac");
    let s9 = String::from("toe");
    let s_format = format!("{}-{}-{}", s7, s8, s9);
    println!("s_format (concatenated with format!): {}", s_format);
    println!("s7, s8, s9 are still valid: {}, {}, {}", s7, s8, s9);
}
```

#### 練習題

1.  編寫一個函式 `reverse_string`，接收一個 `&String` 參數，返回一個新的 `String`，內容是原始字串的反轉。考慮到 Unicode 字元的正確反轉。
2.  編寫一個函式 `count_words`，接收一個 `&String` 參數，返回字串中單詞的數量。假設單詞之間由空格分隔。
3.  編寫一個函式 `replace_substring`，接收一個 `&String` 參數、一個要替換的子字串 `&str` 和一個替換為的子字串 `&str`，返回一個新的 `String`，其中所有匹配的子字串都被替換。

## 3. 字串切片 (`&str`)

#### 設計理念與重要性

字串切片 (`&str`) 是 Rust 中一種輕量級、不可變的字串視圖，它不擁有底層數據的所有權，而是對 `String` 或字串字面量中連續序列的引用。這種設計允許在不複製數據的情況下安全高效地操作字串的一部分，這對於性能優化和避免不必要的記憶體分配至關重要。`&str` 的靈活性使其成為函式參數的理想選擇，因為它既可以接受 `String` 的引用，也可以接受字串字面量。

#### 語法要點

- **定義切片**：使用 `&s[start..end]` 語法來創建字串切片，其中 `start` 是包含的起始位元組索引，`end` 是不包含的結束位元組索引。
- **索引限制**：由於 UTF-8 編碼的特性，Rust 不允許直接使用整數索引來存取 `String` 中的字元（例如 `s[0]`），因為這可能導致無效的字元片段或運行時錯誤。切片的索引必須位於有效的 UTF-8 字元邊界上，否則會導致運行時恐慌 (panic)。
- **字串字面量**：字串字面量本身就是 `&str` 類型，因此可以直接使用。
- **常用方法**：`&str` 提供了多種方法來處理字串內容，例如 `len()`（返回位元組長度）、`chars()`（迭代 Unicode 字元）、`bytes()`（迭代原始位元組）等。

#### 關鍵特性說明

1.  **不可變引用**：`&str` 是一個不可變的引用，它指向儲存在其他地方的 UTF-8 編碼的字串數據。這意味著你不能直接修改 `&str` 的內容。
2.  **借用語義**：`&str` 遵循 Rust 的借用規則，它只是「借用」數據，而不擁有數據。這使得在函式間傳遞字串數據時，可以避免不必要的複製，提高效率。
3.  **字串字面量**：所有字串字面量（例如 `"hello world"`）在 Rust 中都是 `&'static str` 類型，它們被直接編譯到程式的二進位檔案中，並在整個程式的生命週期中都可用。
4.  **切片操作**：你可以從 `String` 或其他 `&str` 創建 `&str` 切片。切片操作是零成本的，它只是創建了一個新的引用，指向原始數據的某個範圍。但需要注意的是，切片的索引必須落在有效的 UTF-8 字元邊界上，否則會導致運行時錯誤。
5.  **靈活性**：由於 `&str` 可以接受 `String` 的引用和字串字面量，因此在設計函式簽名時，通常建議使用 `&str` 作為字串參數類型，這樣可以使函式更具通用性。

#### 實際應用

```rust
fn main() {
    let s = String::from("hello world");

    // &s[開始索引..結束索引]
    // 索引必須位於有效的 UTF-8 字元邊界
    let hello = &s[0..5]; // 從索引 0 到 4 (不包含 5)
    let world = &s[6..11]; // 從索引 6 到 10 (不包含 11)

    println!("Slice 1: {}", hello); // 輸出 "hello"
    println!("Slice 2: {}", world); // 輸出 "world"

    // 字串字面量本身就是 &str 型別
    let literal_slice: &str = "I am a string literal.";
    println!("{}", literal_slice);

    // 對於多位元組字元，切片時要小心
    let unicode_string = String::from("你好世界");
    // "你" 佔 3 個位元組, "好" 佔 3 個位元組
    let ni_hao = &unicode_string[0..6]; // 正確：「你好」
    println!("Unicode slice: {}", ni_hao);

    // let invalid_slice = &unicode_string[0..1]; // 這會 panic! 1 不是一個字元邊界
    // println!("{}", invalid_slice);

    // 函式通常接受 &str 而不是 String，這樣更靈活
    fn first_word(s: &str) -> &str {
        let bytes = s.as_bytes();
        for (i, &item) in bytes.iter().enumerate() {
            if item == b' ' {
                return &s[0..i];
            }
        }
        &s[..] // 如果沒有空格，整個字串就是一個單詞
    }

    let my_string = String::from("hello beautiful world");
    let word = first_word(&my_string[..]); // 可以傳 String 的切片
    println!("First word of my_string: {}", word);

    let my_literal = "another example sentence";
    let word_literal = first_word(my_literal); // 也可以直接傳字串字面量
    println!("First word of my_literal: {}", word_literal);

    // UTF-8 編碼與內部表示
    let s = "नमस्ते"; // 印地語 "Namaste"

    println!("\nString: {}", s);
    println!("Length in bytes: {}", s.len()); // 每個字元 3 個位元組，總共 18 個位元組
    println!("Length in chars: {}", s.chars().count()); // 6 個 Unicode 純量值

    // 遍歷位元組
    print!("Bytes: ");
    for b in s.bytes() {
        print!("{} ", b);
    }
    println!();

    // 遍歷字元 (Unicode 純量值)
    print!("Chars: ");
    for c in s.chars() {
        print!("{} ", c);
    }
    println!();

    // 索引字串 (Indexing Strings) 的注意事項
    let s1 = String::from("hello");
    // let h = s1[0]; // 編譯錯誤! String 不能被整數索引

    let s2 = String::from("你好");
    // 如果允許 s2[0]，它會返回 '你' 的第一個位元組，這不是一個有效的字元。
}
```

#### 練習題

1.  編寫一個函式 `find_nth_word`，接收一個 `&str` 和一個整數 `n`，返回字串中第 `n` 個單詞的 `&str` 切片。如果不存在第 `n` 個單詞，則返回空字串切片。
2.  編寫一個函式 `trim_whitespace`，接收一個 `&str` 參數，返回一個新的 `&str` 切片，該切片移除了原始字串兩端的空白字元。
3.  編寫一個函式 `contains_substring`，接收兩個 `&str` 參數 `haystack` 和 `needle`，判斷 `haystack` 是否包含 `needle`。
    // 如果你確實需要基於位元組索引的切片，並且確定索引是有效的字元邊界：
    let hello = "hello";
    let slice = &hello[0..1]; // "h"
    println!("\nSlice of 'hello': {}", slice);

    let ni_hao = "你好"; // "你" 是 3 bytes, "好" 是 3 bytes
    let ni = &ni_hao[0..3]; // "你"
    println!("Slice of '你好': {}", ni);

    // 獲取第 N 個字元 (效率不高，因為需要遍歷)
    let text = "abcde你好";
    let third_char = text.chars().nth(2); // 'c' (0-indexed)
    let seventh_char = text.chars().nth(6); // '好'

    match third_char {
        Some(c) => println!("Third char: {}", c),
        None => println!("No third char."),
    }
    match seventh_char {
        Some(c) => println!("Seventh char: {}", c),
        None => println!("No seventh char."),
    }
}
```

### 6. 遍歷字串 (Iterating Over Strings)

遍歷字串的推薦方式是使用 `.chars()` (對於 Unicode 純量值) 或 `.bytes()` (對於原始位元組)。

```rust
fn main() {
    let s = String::from("Hello, 世界!");

    println!("Iterating over chars:");
    for c in s.chars() {
        println!("Char: {}", c);
    }
    // 輸出:
    // Char: H
    // Char: e
    // Char: l
    // Char: l
    // Char: o
    // Char: ,
    // Char:  
    // Char: 世
    // Char: 界
    // Char: !

    println!("\nIterating over bytes:");
    for b in s.bytes() {
        println!("Byte: {}", b);
    }
    // 輸出 (部分):
    // Byte: 72 (H)
    // ...
    // Byte: 228 (世 的第一個位元組)
    // Byte: 184 (世 的第二個位元組)
    // Byte: 150 (世 的第三個位元組)
    // ...
}
```

## 練習題

1.  **字元屬性探索:**
    *   創建一個 `char` 變數，賦值為 `'嗨'`。
    *   使用 `is_alphabetic()`, `is_numeric()`, `is_whitespace()`, `is_control()` 等方法檢查它的屬性並印出結果。
    *   將一個 `char` 轉換為小寫 (`to_lowercase()`) 和大寫 (`to_uppercase()`) (如果適用)，並印出結果。

2.  **`String` 的創建與組合:**
    *   創建三個 `String`：`first_name` (你的姓)，`last_name` (你的名)，`city` (你所在的城市)。
    *   使用 `format!` 宏將它們組合成一句話，例如："我是 [姓][名]，我來自 [城市]。" 並印出來。
    *   創建一個空的 `String`，然後使用 `push_str` 和 `push` 方法逐步構建同樣的句子。

3.  **字串切片與函式:**
    *   編寫一個函式 `get_domain_from_email(email: &str) -> &str`。
    *   此函式接受一個 email 字串 (例如 "user@example.com")，並返回 `@` 符號之後的域名部分 (例如 "example.com")。
    *   如果找不到 `@`，可以返回空字串或整個輸入字串 (自行決定)。
    *   在 `main` 中測試你的函式。

4.  **字串反轉:**
    *   編寫一個函式 `reverse_string(s: &str) -> String`。
    *   此函式接受一個字串切片，並返回一個新的 `String`，其內容是原字串的反轉。
    *   例如，輸入 "hello"，應返回 "olleh"。輸入 "你好世界"，應返回 "界世好你"。
    *   提示：`s.chars().rev().collect::<String>()` 是一個簡潔的方法。
    *   在 `main` 中測試你的函式。

5.  **計算字數:**
    *   編寫一個函式 `count_words(text: &str) -> usize`。
    *   此函式接受一個字串切片，並返回其中單詞的數量。假設單詞由空格分隔。
    *   例如，輸入 "Hello world from Rust"，應返回 4。
    *   提示：可以使用 `split_whitespace()` 方法。
    *   在 `main` 中測試你的函式。

6.  **UTF-8 字元邊界 (挑戰):**
    *   創建一個包含多位元組字元的 `String`，例如 `let s = String::from("你好 Rust");`。
    *   嘗試使用不同的位元組索引來切片這個字串，例如 `&s[0..1]`, `&s[0..3]`, `&s[3..4]`, `&s[7..8]`。
    *   觀察哪些會成功，哪些會 panic。解釋為什麼會 panic (因為切片不在有效的 UTF-8 字元邊界上)。
    *   (注意：執行此練習時，預期程式會 panic。在實際程式碼中，應避免這種不安全的切片。)
使用 `char`, `String`, 和 `&str` 來處理各種文字處理需求，同時避免常見的編碼和索引錯誤。
