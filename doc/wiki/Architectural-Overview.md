<!-- markdownlint-disable MD007 -->
## 層次概述

![Architectural overview.](https://raw.githubusercontent.com/wiki/Microsoft/TypeScript/images/architecture.png)

* **核心TypeScript編譯器**

  * **語法分析器（Parser）：** 以一系列原文件開始, 根據語言的語法, 生成抽象語法樹（AST）

  * **聯合器（Binder）：** 使用一個`Symbol`將針對相同結構的聲明聯合在一起（例如：同一個接口或模組的不同聲明，或擁有相同名字的函數和模組）。這能幫助類型系統推導出這些具名的聲明。

  * **類型解析器與檢查器（Type resolver / Checker）：** 解析每種類型的構造，檢查讀寫語義並生成適當的診斷信息。

  * **生成器（Emitter）：** 從一系列輸入文件（.ts和.d.ts）生成輸出，它們可以是以下形式之一：JavaScript（.js），聲明（.d.ts），或者是source maps（.js.map）。

  * **預處理器（Pre-processor）：** 「編譯上下文」指的是某個「程序」裡涉及到的所有文件。上下文的創建是通過檢查所有從命令行上傳入編譯器的文件，按順序，然後再加入這些文件直接引用的其它文件或通過`import`語句和`/// <reference path=... />`標籤間接引用的其它文件。

沿著引用圖走下來你會發現它是一個有序的源文件列表，它們組成了整個程序。
當解析導出（import）的時候，會優先選擇「.ts」文件而不是「.d.ts」文件，以確保處理的是最新的文件。
編譯器會進行與Nodejs相似的流程來解析導入，沿著目錄鏈查找與將要導入相匹配的帶.ts或.d.ts擴展名的文件。
導入失敗不會報error，因為可能已經聲明了外部模組。

* **獨立編譯器（tsc）：** 批處理編譯命令行界面。主要處理針對不同支持的引擎讀寫文件（比如：Node.js）。

* **語言服務：** 「語言服務」在核心編譯器管道上暴露了額外的一層，非常適合類編輯器的應用。

語言服務支持一系列典型的編輯器操作比如語句自動補全，函數簽名提示，代碼格式化和突出高亮，著色等。
基本的重構功能比如重命名，調試接口輔助功能比如驗證斷點，還有TypeScript特有的功能比如支持增量編譯（在命令行上使用`--watch`）。
語言服務是被設計用來有效的處理在一個長期存在的編譯上下文中文件隨著時間改變的情況；在這樣的情況下，語言服務提供了與其它編譯器接口不同的角度來處理程序和源文件。
> 請參考 [[Using the Language Service API]] 以瞭解更多詳細內容。

## 數據結構

* **Node:** 抽象語法樹（AST）的基本組成塊。通常`Node`表示語言語法裡的非終結符；一些終結符保存在語法樹裡比如標識符和字面量。

* **SourceFile:** 給定源文件的AST。`SourceFile`本身是一個`Node`；它提供了額外的接口用來訪問文件的源碼，文件裡的引用，文件裡的標識符列表和文件裡的某個位置與它對應的行號與列號的映射。

* **Program:** `SourceFile`的集合和一系列編譯選項代表一個編譯單元。`Program`是類型系統和生成代碼的主入口。

* **Symbol:** 具名的聲明。`Symbols`是做為聯合的結果而創建。`Symbols`連接了樹裡的聲明節點和其它對同一個實體的聲明。`Symbols`是語義系統的基本構建塊。

* **Type:** `Type`是語義系統的其它部分。`Type`可能被命名（比如，類和接口），或匿名（比如，物件類型）。

* **Signature:** 一共有三種`Signature`類型：調用簽名（call），構造簽名（construct）和索引簽名（index）。

## 編譯過程概述

整個過程從預處理開始。
預處理器會算出哪些文件參與編譯，它會去查找如下引用（`/// <reference path=... />`標籤和`import`語句）。

語法分析器（Parser）生成抽象語法樹（AST）`Node`.
這些僅為用戶輸出的抽象表現，以樹的形式。
一個`SourceFile`物件表示一個給定文件的AST並且帶有一些額外的信息如文件名及源文件內容。

然後，聯合器（Binder）處理AST節點，結合併生成`Symbols`。
一個`Symbol`會對應到一個命名實體。
這裡有個一微妙的差別，幾個聲明節點可能會是名字相同的實體。
也就是說，有時候不同的`Node`具有相同的`Symbol`，並且每個`Symbol`保持跟蹤它的聲明節點。
比如，一個名字相同的`class`和`namespace`可以*合併*，並且擁有相同的`Symbol`。
聯合器也會處理作用域，以確保每個`Symbol`都在正確的封閉作用域裡創建。

生成`SourceFile`（還帶有它的`Symbols`們）是通過調用`createSourceFile` API。

到目前為止，`Symbol`代表的命名實體可以在單個文件裡看到，但是有些聲明可以從多文件合併，因此下一步就是構建一個全局的包含所有文件的視圖，也就是創建一個`Program`。

一個`Program`是`SourceFile`的集合併帶有一系列`CompilerOptions`。
通過調用`createProgram` API來創建`Program`。

通過一個`Program`實例創建`TypeChecker`。
`TypeChecker`是TypeScript類型系統的核心。
它負責計算出不同文件裡的`Symbols`之間的關係，將`Type`賦值給`Symbol`，並生成任何語義`Diagnostic`（比如：error）。

`TypeChecker`首先要做的是合並不同的`SourceFile`裡的`Symbol`到一個單獨的視圖，創建單一的`Symbol`表，合併所有普通的`Symbol`（比如：不同文件裡的`namespace`）。

在原始狀態初始化完成後，`TypeChecker`就可以解決關於這個程序的任何問題了。
這些「問題」可以是：

* 這個`Node`的`Symbol`是什麼？
* 這個`Symbol`的`Type`是什麼？
* 在AST的某個部分裡有哪些`Symbol`是可見的？
* 某個函數聲明的`Signature`都有哪些？
* 針對某個文件應該報哪些錯誤？

`TypeChecker`計算所有東西都是「懶惰的」；為了回答一個問題它僅「解決」必要的信息。
`TypeChecker`僅會檢測和這個問題有關的`Node`，`Symbol`或`Type`，不會檢測額外的實體。

對於一個`Program`同樣會生成一個`Emitter`。
`Emitter`負責生成給定`SourceFile`的輸出；它包括：`.js`，`.jsx`，`.d.ts`和`.js.map`。

## 術語

##### **完整開始/令牌開始（Full Start/Token Start）**

令牌本身就具有我們稱為一個「完整開始」和一個「令牌開始」。「令牌開始」是指更自然的版本，它表示在文件中令牌開始的位置。「完整開始」是指從上一個有意義的令牌之後掃瞄器開始掃瞄的起始位置。當關心瑣事時，我們往往更關心完整開始。

函數 | 描述
-----------------------|---------------------------------
`ts.Node.getStart`     | 取得某節點的第一個令牌起始位置。
`ts.Node.getFullStart` | 取得某節點擁有的第一個令牌的完整開始。

#### **瑣碎內容（Trivia）**

語法的瑣碎內容代表源碼裡那些對理解代碼無關緊要的內容，比如空白，註釋甚至一些衝突的標記。

因為瑣碎內容不是語言正常語法的一部分（不包括ECMAScript API規範）並且可能在任意2個令牌中的任意位置出現，它們不會包含在語法樹裡。但是，因為它們對於像重構和維護高保真源碼很重要，所以需要的時候還是能夠通過我們的APIs訪問。

因為`EndOfFileToken`後面可以沒有任何內容（令牌和瑣碎內容），所有瑣碎內容自然地在非瑣碎內容之前，而且存在於那個令牌的「完整開始」和「令牌開始」之間。

雖然這個一個方便的標記法來說明一個註釋「屬於」一個`Node`。比如，在下面的例子裡，可以明顯看出`genie`函數擁有兩個註釋：

```TypeScript
var x = 10; // This is x.

/**
 * Postcondition: Grants all three wishes.
 */
function genie([wish1, wish2, wish3]: [Wish, Wish, Wish]) {
    while (true) {
    }
} // End function
```

這是儘管事實上，函數聲明的完整開始是在`var x = 10;`後。

我們依據[Roslyn's notion of trivia ownership](https://github.com/dotnet/roslyn/wiki/Roslyn%20Overview#syntax-trivia)處理註釋所有權。通常來講，一個令牌擁有同一行上的所有的瑣碎內容直到下一個令牌開始。任何出現在這行之後的註釋都屬於下一個令牌。源文件的第一個令牌擁有所有的初始瑣碎內容，並且最後面的一系列瑣碎內容會添加到`end-of-file`令牌上。

對於大多數普通用戶，註釋是「有趣的」瑣碎內容。屬於一個節點的註釋內容可以通過下面的函數來獲取：

函數 | 描述
---------|------------
`ts.getLeadingCommentRanges`  | 提供源文件和一個指定位置，返回指定位置後的第一個換行與令牌之間的註釋的範圍（與`ts.Node.getFullStart`配合會更有用）。
`ts.getTrailingCommentRanges` | 提供源文件和一個指定位置，返回到指定位置後第一個換行為止的註釋的範圍（與`ts.Node.getEnd`配合會更有用）。

做為例子，假設有下面一部分源代碼：

```TypeScript
debugger;/*hello*/
    //bye
  /*hi*/    function
```

`function`關鍵字的完整開始是從`/*hello*/`註釋，但是`getLeadingCommentRanges`僅會返回後面2個註釋：

```plain
d e b u g g e r ; / * h e l l o * / _ _ _ _ _ [CR] [NL] _ _ _ _ / / b y e [CR] [NL] _ _ / * h i * / _ _ _ _ f u n c t i o n
                  ↑                                     ↑       ↑                       ↑                   ↑
                  完整開始                              查找      第一個註釋               第二個註釋     令牌開始
                                                       開始註釋
```

適當地，在`debugger`語句後調用`getTrailingCommentRanges`可以提取出`/*hello*/`註釋。

如果你關心令牌流的更多信息，`createScanner`也有一個`skipTrivia`標記，你可以設置成`false`，然後使用`setText`/`setTextPos`來掃瞄文件裡的不同位置。
