> 這節假設你已經瞭解了模組的一些基本知識
請閱讀[模組](./Modules.md)文檔瞭解更多信息。

*模組解析*就是指編譯器所要依據的一個流程，用它來找出某個導入操作所引用的具體值。
假設有一個導入語句`import { a } from "moduleA"`;
為了去檢查任何對`a`的使用，編譯器需要準確的知道它表示什麼，並且會需要檢查它的定義`moduleA`。

這時候，編譯器會想知道「`moduleA`的shape是怎樣的？」
這聽上去很簡單，`moduleA`可能在你寫的某個`.ts`/`.tsx`文件裡或者在你的代碼所依賴的`.d.ts`裡。

首先，編譯器會嘗試定位表示導入模組的文件。
編譯會遵循下列二種策略之一：[Classic](#classic)或[Node](#node)。
這些策略會告訴編譯器到*哪裡*去查找`moduleA`。

如果它們失敗了並且如果模組名是非相對的（且是在`"moduleA"`的情況下），編譯器會嘗試定位一個[外部模組聲明](./Modules.md#ambient-modules)。
我們接下來會講到非相對導入。

最後，如果編譯器還是不能解析這個模組，它會記錄一個錯誤。
在這種情況下，錯誤可能為`error TS2307: Cannot find module 'moduleA'.`

## 相對 vs. 非相對模組導入

根據模組引用是相對的還是非相對的，模組導入會以不同的方式解析。

*相對導入*是以`/`，`./`或`../`開頭的。
下面是一些例子：

* `import Entry from "./components/Entry";`
* `import { DefaultHeaders } from "../constants/http";`
* `import "/mod";`

所有其它形式的導入被當作*非相對*的。
下面是一些例子：

* `import * as $ from "jQuery";`
* `import { Component } from "angular2/core";`

相對導入解析時是相對於導入它的文件來的，並且*不能*解析為一個外部模組聲明。
你應該為你自己寫的模組使用相對導入，這樣能確保它們在運行時的相對位置。

## 模組解析策略

共有兩種可用的模組解析策略：[Node](#node)和[Classic](#classic)。
你可以使用`--moduleResolution`標記為指定使用哪個。
默認值為[Node](#node)。

### Classic

這種策略以前是TypeScript默認的解析策略。
現在，它存在的理由主要是為了向後兼容。

相對導入的模組是相對於導入它的文件進行解析的。
因此`/root/src/folder/A.ts`文件裡的`import { b } from "./moduleB"`會使用下面的查找流程：

1. `/root/src/folder/moduleB.ts`
2. `/root/src/folder/moduleB.d.ts`

對了非相對模組的導入，編譯器則會從包含導入文件的目錄開始依次向上級目錄遍歷，嘗試定位匹配的聲明文件。

比如：

有一個對`moduleB`的非相對導入`import { b } from "moduleB"`，它是在`/root/src/folder/A.ts`文件裡，會以如下的方式來定位`"moduleB"`：

1. `/root/src/folder/moduleB.ts`
2. `/root/src/folder/moduleB.d.ts`
3. `/root/src/moduleB.ts`
4. `/root/src/moduleB.d.ts`
5. `/root/moduleB.ts`
6. `/root/moduleB.d.ts`
7. `/moduleB.ts`
8. `/moduleB.d.ts`

### Node

這個解析策略試圖在運行時模仿[Node.js](https://nodejs.org/)模組解析機制。
完整的Node.js解析算法可以在[Node.js module documentation](https://nodejs.org/api/modules.html#modules_all_together)找到。

#### Node.js如何解析模組

為了理解TypeScript編譯依照的解析步驟，先弄明白Node.js模組是非常重要的。
通常，在Node.js裡導入是通過`require`函數調用進行的。
Node.js會根據`require`的是相對路徑還是非相對路徑做出不同的行為。

相對路徑很簡單。
例如，假設有一個文件路徑為`/root/src/moduleA.js`，包含了一個導入`var x = require("./moduleB");`
Node.js以下面的順序解析這個導入：

1. 將`/root/src/moduleB.js`視為文件，檢查是否存在。

2. 將`/root/src/moduleB`視為目錄，檢查是否它包含`package.json`文件並且其指定了一個`"main"`模組。
   在我們的例子裡，如果Node.js發現文件`/root/src/moduleB/package.json`包含了`{ "main": "lib/mainModule.js" }`，那麼Node.js會引用`/root/src/moduleB/lib/mainModule.js`。

3. 將`/root/src/moduleB`視為目錄，檢查它是否包含`index.js`文件。
   這個文件會被隱式地當作那個文件夾下的"main"模組。

你可以閱讀Node.js文檔瞭解更多詳細信息：[file modules](https://nodejs.org/api/modules.html#modules_file_modules) 和 [folder modules](https://nodejs.org/api/modules.html#modules_folders_as_modules)。

但是，[非相對模組名](#relative-vs-non-relative-module-imports)的解析是個完全不同的過程。
Node會在一個特殊的文件夾`node_modules`裡查找你的模組。
`node_modules`可能與當前文件在同一級目錄下，或者在上層目錄裡。
Node會向上級目錄遍歷，查找每個`node_modules`直到它找到要加載的模組。

還是用上面例子，但假設`/root/src/moduleA.js`裡使用的是非相對路徑導入`var x = require("moduleB");`。
Node則會以下面的順序去解析`moduleB`，直到有一個匹配上。

1. `/root/src/node_modules/moduleB.js`
2. `/root/src/node_modules/moduleB/package.json` (如果指定了`"main"`屬性)
3. `/root/src/node_modules/moduleB/index.js`
   <br /><br />
4. `/root/node_modules/moduleB.js`
5. `/root/node_modules/moduleB/package.json` (如果指定了`"main"`屬性)
6. `/root/node_modules/moduleB/index.js`
   <br /><br />
7. `/node_modules/moduleB.js`
8. `/node_modules/moduleB/package.json` (如果指定了`"main"`屬性)
9. `/node_modules/moduleB/index.js`

注意Node.js在步驟（4）和（7）會向上跳一級目錄。

你可以閱讀Node.js文檔瞭解更多詳細信息：[loading modules from `node_modules`](https://nodejs.org/api/modules.html#modules_loading_from_node_modules_folders)。

#### TypeScript如何解析模組

TypeScript是模仿Node.js運行時的解析策略來在編譯階段定位模組定義文件。
因此，TypeScript在Node解析邏輯基礎上增加了TypeScript源文件的擴展名（`.ts`，`.tsx`和`.d.ts`）。
同時，TypeScript在`package.json`裡使用字段`"typings"`來表示類似`"main"`的意義 - 編譯器會使用它來找到要使用的"main"定義文件。

比如，有一個導入語句`import { b } from "./moduleB"`在`/root/src/moduleA.ts`裡，會以下面的流程來定位`"./moduleB"`：

1. `/root/src/moduleB.ts`
2. `/root/src/moduleB.tsx`
3. `/root/src/moduleB.d.ts`
4. `/root/src/moduleB/package.json` (如果指定了`"typings"`屬性)
5. `/root/src/moduleB/index.ts`
6. `/root/src/moduleB/index.tsx`
7. `/root/src/moduleB/index.d.ts`

回想一下Node.js先查找`moduleB.js`文件，然後是合適的`package.json`，再之後是`index.js`。

類似地，非相對的導入會遵循Node.js的解析邏輯，首先查找文件，然後是合適的文件夾。
因此`/src/moduleA.ts`文件裡的`import { b } from "moduleB"`會以下面的查找順序解析：

1. `/root/src/node_modules/moduleB.ts`
2. `/root/src/node_modules/moduleB.tsx`
3. `/root/src/node_modules/moduleB.d.ts`
4. `/root/src/node_modules/moduleB/package.json` (如果指定了`"typings"`屬性)
5. `/root/src/node_modules/moduleB/index.ts`
6. `/root/src/node_modules/moduleB/index.tsx`
7. `/root/src/node_modules/moduleB/index.d.ts`
   <br /><br />
8. `/root/node_modules/moduleB.ts`
9. `/root/node_modules/moduleB.tsx`
10. `/root/node_modules/moduleB.d.ts`
11. `/root/node_modules/moduleB/package.json` (如果指定了`"typings"`屬性)
12. `/root/node_modules/moduleB/index.ts`
13. `/root/node_modules/moduleB/index.tsx`
14. `/root/node_modules/moduleB/index.d.ts`
    <br /><br />
15. `/node_modules/moduleB.ts`
16. `/node_modules/moduleB.tsx`
17. `/node_modules/moduleB.d.ts`
18. `/node_modules/moduleB/package.json` (如果指定了`"typings"`屬性)
19. `/node_modules/moduleB/index.ts`
20. `/node_modules/moduleB/index.tsx`
21. `/node_modules/moduleB/index.d.ts`

不要被這裡步驟的數量嚇到 - TypeScript只是在步驟（8）和（15）向上跳了兩次目錄。
這並不比Node.js裡的流程複雜。

## 使用`--noResolve`

正常來講編譯器會在開始編譯之前解析模組導入。
每當它成功地解析了對一個文件`import`，這個文件被會加到一個文件列表裡，以供編譯器稍後處理。

`--noResolve`編譯選項告訴編譯器不要添加任何不是在命令行上傳入的文件到編譯列表。
編譯器仍然會嘗試解析模組，但是只要沒有指定這個文件，那麼它就不會被包含在內。

比如

#### app.ts

```ts
import * as A from "moduleA" // OK, moduleA passed on the command-line
import * as B from "moduleB" // Error TS2307: Cannot find module 'moduleB'.
```

```shell
tsc app.ts moduleA.ts --noResolve
```

使用`--noResolve`編譯`app.ts`：

* 可能正確找到`moduleA`，因為它在命令行上指定了。
* 找不到`moduleB`，因為沒有在命令行上傳遞。

## 常見問題

### 為什麼在`exclude`列表裡的模組還會被編譯器使用

`tsconfig.json`將文件夾轉變一個「工程」
如果不指定任何`「exclude」`或`「files」`，文件夾裡的所有文件包括`tsconfig.json`和所有的子目錄都會在編譯列表裡。
如果你想利用`「exclude」`排除某些文件，甚至你想指定所有要編譯的文件列表，請使用`「files」`。

有些是被`tsconfig.json`自動加入的。
它不會涉及到上面討論的模組解析。
如果編譯器識別出一個文件是模組導入目標，它就會加到編譯列表裡，不管它是否被排除了。

因此，要從編譯列表中排除一個文件，你需要在排除它的同時，還要排除所有對它進行`import`或使用了`/// <reference path="..." />`指令的文件。
