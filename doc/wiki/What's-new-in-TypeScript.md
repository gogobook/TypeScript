# TypeScript 新增特性一覽

<!-- https://github.com/Microsoft/TypeScript/wiki/What's-new-in-TypeScript/78a12d04d7ba25d5253bcb0bc4054976c9b628ac -->

## TypeScript 1.8

### 類型參數約束

在 TypeScript 1.8 中, 類型參數的限制可以引用自同一個類型參數列表中的類型參數. 在此之前這種做法會報錯. 這種特性通常被叫做 [F-Bounded Polymorphism](https://en.wikipedia.org/wiki/Bounded_quantification#F-bounded_quantification).

#### 例子

```ts
function assign<T extends U, U>(target: T, source: U): T {
    for (let id in source) {
        target[id] = source[id];
    }
    return target;
}

let x = { a: 1, b: 2, c: 3, d: 4 };
assign(x, { b: 10, d: 20 });
assign(x, { e: 0 });  // 錯誤
```

### 控制流錯誤分析

TypeScript 1.8 中引入了控制流分析來捕獲開發者通常會遇到的一些錯誤.

詳情見接下來的內容, 可以上手嘗試:

![cfa](https://cloud.githubusercontent.com/assets/8052307/5210657/c5ae0f28-7585-11e4-97d8-86169ef2a160.gif)

#### 不可及的代碼

一定無法在運行時被執行的語句現在會被標記上代碼不可及錯誤. 舉個例子, 在無條件限制的 `return`, `throw`, `break` 或者 `continue` 後的語句被認為是不可及的. 使用 `--allowUnreachableCode` 來禁用不可及代碼的檢測和報錯.

##### 例子

這裡是一個簡單的不可及錯誤的例子:

```ts
function f(x) {
    if (x) {
       return true;
    }
    else {
       return false;
    }

    x = 0; // 錯誤: 檢測到不可及的代碼.
}
```

這個特性能捕獲的一個更常見的錯誤是在 `return` 語句後添加換行:

```ts
function f() {
    return            // 換行導致自動插入的分號
    {
        x: "string"   // 錯誤: 檢測到不可及的代碼.
    }
}
```

因為 JavaScript 會自動在行末結束 `return` 語句, 下面的物件字面量變成了一個代碼塊.

#### 未使用的標籤

未使用的標籤也會被標記. 和不可及代碼檢查一樣, 被使用的標籤檢查也是默認開啟的. 使用 `--allowUnusedLabels` 來禁用未使用標籤的報錯.

##### 例子

```ts
loop: while (x > 0) {  // 錯誤: 未使用的標籤.
    x++;
}
```

#### 隱式返回

JS 中沒有返回值的代碼分支會隱式地返回 `undefined`. 現在編譯器可以將這種方式標記為隱式返回. 對於隱式返回的檢查默認是被禁用的, 可以使用 `--noImplicitReturns` 來啟用.

##### 例子

```ts
function f(x) { // 錯誤: 不是所有分支都返回了值.
    if (x) {
        return false;
    }

    // 隱式返回了 `undefined`
}
```

#### Case 語句貫穿

TypeScript 現在可以在 switch 語句中出現貫穿的幾個非空 case 時報錯.
這個檢測默認是關閉的, 可以使用 `--noFallthroughCasesInSwitch` 啟用.

##### 例子

```ts
switch (x % 2) {
    case 0: // 錯誤: switch 中出現了貫穿的 case.
        console.log("even");

    case 1:
        console.log("odd");
        break;
}
```

然而, 在下面的例子中, 由於貫穿的 case 是空的, 並不會報錯:

```ts
switch (x % 3) {
    case 0:
    case 1:
        console.log("Acceptable");
        break;

    case 2:
        console.log("This is *two much*!");
        break;
}
```

### React 無狀態的函數組件

TypeScript 現在支持[無狀態的函數組件](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions).
它是可以組合其他組件的輕量級組件.

```ts
// 使用參數解構和默認值輕鬆地定義 'props' 的類型
const Greeter = ({name = 'world'}) => <div>Hello, {name}!</div>;

// 參數可以被檢驗
let example = <Greeter name='TypeScript 1.8' />;
```

如果需要使用這一特性及簡化的 props, 請確認使用的是[最新的 react.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/react).

### 簡化的 React `props` 類型管理

在 TypeScript 1.8 配合最新的 react.d.ts (見上方) 大幅簡化了 `props` 的類型聲明.

具體的:

- 你不再需要顯式的聲明 `ref` 和 `key` 或者 `extend React.Props`
- `ref` 和 `key` 屬性會在所有組件上擁有正確的類型.
- `ref` 屬性在無狀態函數組件上會被正確地禁用.

### 在模組中擴充全局或者模組作用域

用戶現在可以為任何模組進行他們想要, 或者其他人已經對其作出的擴充.
模組擴充的形式和過去的包模組一致 (例如 `declare module "foo" { }` 這樣的語法), 並且可以直接嵌在你自己的模組內, 或者在另外的頂級外部包模組中.

除此之外, TypeScript 還以 `declare global { }` 的形式提供了對於_全局_聲明的擴充.
這能使模組對像 `Array` 這樣的全局類型在必要的時候進行擴充.

模組擴充的名稱解析規則與 `import` 和 `export` 聲明中的一致.
擴充的模組聲明合併方式與在同一個文件中聲明是相同的.

不論是模組擴充還是全局聲明擴充都不能向頂級作用域添加新的項目 - 它們只能為已經存在的聲明添加 "補丁".

#### 例子

這裡的 `map.ts` 可以聲明它會在內部修改在 `observable.ts` 中聲明的 `Observable` 類型, 添加 `map` 方法.

```ts
// observable.ts
export class Observable<T> {
    // ...
}
```

```ts
// map.ts
import { Observable } from "./observable";

// 擴充 "./observable"
declare module "./observable" {

    // 使用接口合併擴充 'Observable' 類的定義
    interface Observable<T> {
        map<U>(proj: (el: T) => U): Observable<U>;
    }

}

Observable.prototype.map = /*...*/;
```

```ts
// consumer.ts
import { Observable } from "./observable";
import "./map";

let o: Observable<number>;
o.map(x => x.toFixed());
```

相似的, 在模組中全局作用域可以使用 `declare global` 聲明被增強:

#### 例子

```ts
// 確保當前文件被當做一個模組.
export {};

declare global {
    interface Array<T> {
        mapToNumbers(): number[];
    }
}

Array.prototype.mapToNumbers = function () { /* ... */ }
```

### 字符串字面量類型

接受一個特定字符串集合作為某個值的 API 並不少見.
舉例來說, 考慮一個可以通過控制[動畫的漸變](https://en.wikipedia.org/wiki/Inbetweening)讓元素在屏幕中滑動的 UI 庫:

```ts
declare class UIElement {
    animate(options: AnimationOptions): void;
}

interface AnimationOptions {
    deltaX: number;
    deltaY: number;
    easing: string; // 可以是 "ease-in", "ease-out", "ease-in-out"
}
```

然而, 這容易產生錯誤 - 當用戶錯誤不小心錯誤拼寫了一個合法的值時, 並沒有任何提示:

```ts
// 沒有報錯
new UIElement().animate({ deltaX: 100, deltaY: 100, easing: "ease-inout" });
```

在 TypeScript 1.8 中, 我們新增了字符串字面量類型. 這些類型和字符串字面量的寫法一致, 只是寫在類型的位置.

用戶現在可以確保類型系統會捕獲這樣的錯誤.
這裡是我們使用了字符串字面量類型的新的 `AnimationOptions`:

```ts
interface AnimationOptions {
    deltaX: number;
    deltaY: number;
    easing: "ease-in" | "ease-out" | "ease-in-out";
}

// 錯誤: 類型 '"ease-inout"' 不能複製給類型 '"ease-in" | "ease-out" | "ease-in-out"'
new UIElement().animate({ deltaX: 100, deltaY: 100, easing: "ease-inout" });
```

### 更好的聯合/交叉類型接口

TypeScript 1.8 優化了源類型和目標類型都是聯合或者交叉類型的情況下的類型推導.
舉例來說, 當從 `string | string[]` 推導到 `string | T` 時, 我們將類型拆解為 `string[]` 和 `T`, 這樣就可以將 `string[]` 推導為 `T`.

#### 例子

```ts
type Maybe<T> = T | void;

function isDefined<T>(x: Maybe<T>): x is T {
    return x !== undefined && x !== null;
}

function isUndefined<T>(x: Maybe<T>): x is void {
    return x === undefined || x === null;
}

function getOrElse<T>(x: Maybe<T>, defaultValue: T): T {
    return isDefined(x) ? x : defaultValue;
}

function test1(x: Maybe<string>) {
    let x1 = getOrElse(x, "Undefined");         // string
    let x2 = isDefined(x) ? x : "Undefined";    // string
    let x3 = isUndefined(x) ? "Undefined" : x;  // string
}

function test2(x: Maybe<number>) {
    let x1 = getOrElse(x, -1);         // number
    let x2 = isDefined(x) ? x : -1;    // number
    let x3 = isUndefined(x) ? -1 : x;  // number
}
```

### 使用 `--outFile` 合併 `AMD` 和 `System` 模組

在使用 `--module amd` 或者 `--module system` 的同時制定 `--outFile` 將會把所有參與編譯的模組合併為單個包括了多個模組閉包的輸出文件.

每一個模組都會根據其相對於 `rootDir` 的位置被計算出自己的模組名稱.

#### 例子

```ts
// 文件 src/a.ts
import * as B from "./lib/b";
export function createA() {
    return B.createB();
}
```

```ts
// 文件 src/lib/b.ts
export function createB() {
    return { };
}
```

結果為:

```js
define("lib/b", ["require", "exports"], function (require, exports) {
    "use strict";
    function createB() {
        return {};
    }
    exports.createB = createB;
});
define("a", ["require", "exports", "lib/b"], function (require, exports, B) {
    "use strict";
    function createA() {
        return B.createB();
    }
    exports.createA = createA;
});
```

### 支持 SystemJS 使用 `default` 導入

像 SystemJS 這樣的模組加載器將 CommonJS 模組做了包裝並暴露為 `default` ES6 導入項. 這使得在 SystemJS 和 CommonJS 的實現由於不同加載器不同的模組導出方式不能共享定義.

設置新的編譯選項 `--allowSyntheticDefaultImports` 指明模組加載器會進行導入的 `.ts` 或 `.d.ts` 中未指定的某種類型的默認導入項構建. 編譯器會由此推斷存在一個 `default` 導出項和整個模組自己一致.

此選項在 System 模組默認開啟.

### 允許循環中被引用的 `let`/`const`

之前這樣會報錯, 現在由 TypeScript 1.8 支持.
循環中被函數引用的 `let`/`const` 聲明現在會被輸出為與 `let`/`const` 更新語義相符的代碼.

#### 例子

```ts
let list = [];
for (let i = 0; i < 5; i++) {
    list.push(() => i);
}

list.forEach(f => console.log(f()));
```

被編譯為:

```js
var list = [];
var _loop_1 = function(i) {
    list.push(function () { return i; });
};
for (var i = 0; i < 5; i++) {
    _loop_1(i);
}
list.forEach(function (f) { return console.log(f()); });
```

然後結果是:

```cmd
0
1
2
3
4
```

### 改進的 `for..in` 語句檢查

過去 `for..in` 變量的類型被推斷為 `any`, 這使得編譯器忽略了 `for..in` 語句內的一些不合法的使用.

從 TypeScript 1.8 開始:

- 在 `for..in` 語句中的變量隱含類型為 `string`.
- 當一個有數字索引簽名對應類型 `T` (比如一個陣列) 的物件被一個 `for..in` 索引*有*數字索引簽名並且*沒有*字符串索引簽名 (比如還是陣列) 的物件的變量索引, 產生的值的類型為 `T`.

#### 例子

```ts
var a: MyObject[];
for (var x in a) {   // x 的隱含類型為 string
    var obj = a[x];  // obj 的類型為 MyObject
}
```

### 模組現在輸出時會加上 `"use strict;"`

對於 ES6 來說模組始終以嚴格模式被解析, 但這一點過去對於非 ES6 目標在生成的代碼中並沒有遵循. 從 TypeScript 1.8 開始, 輸出的模組總會為嚴格模式. 由於多數嚴格模式下的錯誤也是 TS 編譯時的錯誤, 多數代碼並不會有可見的改動, 但是這也意味著有一些東西可能在運行時沒有徵兆地失敗, 比如賦值給 `NaN` 現在會有運行時錯誤. 你可以參考這篇 [MDN 上的文章](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mod) 查看詳細的嚴格模式與非嚴格模式的區別列表.

### 使用 `--allowJs` 加入 `.js` 文件

經常在項目中會有外部的非 TypeScript 編寫的源文件.
一種方式是將 JS 代碼轉換為 TS 代碼, 但這時又希望將所有 JS 代碼和新的 TS 代碼的輸出一起打包為一個文件.

`.js` 文件現在允許作為 `tsc` 的輸入文件. TypeScript 編譯器會檢查 `.js` 輸入文件的語法錯誤, 並根據 `--target` 和 `--module` 選項輸出對應的代碼.
輸出也會和其他 `.ts` 文件一起. `.js` 文件的 source maps 也會像 `.ts` 文件一樣被生成.

### 使用 `--reactNamespace` 自定義 JSX 工廠

在使用 `--jsx react` 的同時使用 `--reactNamespace <JSX 工廠名稱>` 可以允許使用一個不同的 JSX 工廠代替默認的 `React`.

新的工廠名稱會被用來調用 `createElement` 和 `__spread` 方法.

#### 例子

```ts
import {jsxFactory} from "jsxFactory";

var div = <div>Hello JSX!</div>
```

編譯參數:

```shell
tsc --jsx react --reactNamespace jsxFactory --m commonJS
```

結果:

```js
"use strict";
var jsxFactory_1 = require("jsxFactory");
var div = jsxFactory_1.jsxFactory.createElement("div", null, "Hello JSX!");
```

### 基於 `this` 的類型收窄

TypeScript 1.8 為類和接口方法擴展了[用戶定義的類型收窄函數](#用戶定義的類型收窄函數).

`this is T` 現在是類或接口方法的合法的返回值類型標註.
當在類型收窄的位置使用時 (比如 `if` 語句), 函數調用表達式的目標物件的類型會被收窄為 `T`.

#### 例子

```ts
class FileSystemObject {
    isFile(): this is File { return this instanceof File; }
    isDirectory(): this is Directory { return this instanceof Directory;}
    isNetworked(): this is (Networked & this) { return this.networked; }
    constructor(public path: string, private networked: boolean) {}
}

class File extends FileSystemObject {
    constructor(path: string, public content: string) { super(path, false); }
}
class Directory extends FileSystemObject {
    children: FileSystemObject[];
}
interface Networked {
    host: string;
}

let fso: FileSystemObject = new File("foo/bar.txt", "foo");
if (fso.isFile()) {
    fso.content; // fso 是 File
}
else if (fso.isDirectory()) {
    fso.children; // fso 是 Directory
}
else if (fso.isNetworked()) {
    fso.host; // fso 是 networked
}
```

### 官方的 TypeScript NuGet 包

從 TypeScript 1.8 開始, 將為 TypeScript 編譯器 (`tsc.exe`) 和 MSBuild 整合 (`Microsoft.TypeScript.targets` 和 `Microsoft.TypeScript.Tasks.dll`) 提供官方的 NuGet 包.

穩定版本可以在這裡下載:

- [Microsoft.TypeScript.Compiler](https://www.nuget.org/packages/Microsoft.TypeScript.Compiler/)
- [Microsoft.TypeScript.MSBuild](https://www.nuget.org/packages/Microsoft.TypeScript.MSBuild/)

與此同時, 和[每日 npm 包](http://blogs.msdn.com/b/typescript/archive/2015/07/27/introducing-typescript-nightlies.aspx)對應的每日 NuGet 包可以在 https://myget.org 下載:

- [TypeScript-Preview](https://www.myget.org/gallery/typescript-preview)

### `tsc` 錯誤信息更美觀

我們理解大量單色的輸出並不直觀. 顏色可以幫助識別信息的始末, 這些視覺上的線索在處理複雜的錯誤信息時非常重要.

通過傳遞 `--pretty` 命令行選項, TypeScript 會給出更豐富的輸出, 包含錯誤發生的上下文.

![展示在 ConEmu 中美化之後的錯誤信息](https://raw.githubusercontent.com/wiki/Microsoft/TypeScript/images/new-in-typescript/pretty01.png)

### 高亮 VS 2015 中的 JSX 代碼

在 TypeScript 1.8 中, JSX 標籤現在可以在 Visual Studio 2015 中被分別和高亮.

![jsx](https://cloud.githubusercontent.com/assets/8052307/12271404/b875c502-b90f-11e5-93d8-c6740be354d1.png)

通過 `工具`->`選項`->`環境`->`字體與顏色` 頁面在 `VB XML` 顏色和字體設置中還可以進一步改變字體和顏色來自定義.

### `--project` (`-p`) 選項現在接受任意文件路徑

`--project` 命令行選項過去只接受包含了 `tsconfig.json` 文件的文件夾.
考慮到不同的構建場景, 應該允許 `--project` 指向任何兼容的 JSON 文件.
比如說, 一個用戶可能會希望為 Node 5 編譯 CommonJS 的 ES 2015, 為瀏覽器編譯 AMD 的 ES5.
現在少了這項限制, 用戶可以更容易地直接使用 `tsc` 管理不同的構建目標, 無需再通過一些奇怪的方式, 比如將多個 `tsconfig.json` 文件放在不同的目錄中.

如果參數是一個路徑, 行為保持不變 - 編譯器會嘗試在該目錄下尋找名為 `tsconfig.json` 的文件.

### 允許 tsconfig.json 中的註釋

為配置添加文檔是很棒的! `tsconfig.json` 現在支持單行和多行註釋.

```json
{
    "compilerOptions": {
        "target": "ES2015", // 跑在 node v5 上, 呀!
        "sourceMap": true   // 讓調試輕鬆一些
    },
    /*
     * 排除的文件
     */
    "exclude": [
        "file.d.ts"
    ]
}
```

### 支持輸出到 IPC 驅動的文件

TypeScript 1.8 允許用戶將 `--outFile` 參數和一些特殊的文件系統物件一起使用, 比如命名的管道 (pipe), 設備 (devices) 等.

舉個例子, 在很多與 Unix 相似的系統上, 標準輸出流可以通過文件 `/dev/stdout` 訪問.

```sh
tsc foo.ts --outFile /dev/stdout
```

這一特性也允許輸出給其他命令.

比如說, 我們可以輸出生成的 JavaScript 給一個像 [pretty-js](https://www.npmjs.com/package/pretty-js) 這樣的格式美化工具:

```sh
tsc foo.ts --outFile /dev/stdout | pretty-js
```

### 改進了 Visual Studio 2015 中對 `tsconfig.json` 的支持

TypeScript 1.8 允許在任何種類的項目中使用 `tsconfig.json` 文件.
包括 ASP.NET v4 項目, *控制台應用*, 以及 *用 TypeScript 開發的 HTML 應用*.
與此同時, 你可以添加不止一個 `tsconfig.json` 文件, 其中每一個都會作為項目的一部分被構建.
這使得你可以在不使用多個不同項目的情況下為應用的不同部分使用不同的配置.

![展示 Visual Studio 中的 tsconfig.json](https://raw.githubusercontent.com/wiki/Microsoft/TypeScript/images/new-in-typescript/tsconfig-in-vs.png)

當項目中添加了 `tsconfig.json` 文件時, 我們還禁用了項目屬性頁面.
也就是說所有配置的改變必須在 `tsconfig.json` 文件中進行.

#### 一些限制

- 如果你添加了一個 `tsconfig.json` 文件, 不在其上下文中的 TypeScript 文件不會被編譯.
- Apache Cordova 應用依然有單個 `tsconfig.json` 文件的限制, 而這個文件必須在根目錄或者 `scripts` 文件夾.
- 多數項目類型中都沒有 `tsconfig.json` 的模板.

## TypeScript 1.7

### 支持 `async`/`await` 編譯到 ES6 (Node v4+)

TypeScript 目前在已經原生支持 ES6 generator 的引擎 (比如 Node v4 及以上版本) 上支持異步函數. 異步函數前置 `async` 關鍵字; `await` 會暫停執行, 直到一個異步函數執行後返回的 promise 被 fulfill 後獲得它的值.

#### 例子

在下面的例子中, 輸入的內容將會延時 200 毫秒逐個打印:

```ts
"use strict";

// printDelayed 返回值是一個 'Promise<void>'
async function printDelayed(elements: string[]) {
    for (const element of elements) {
        await delay(200);
        console.log(element);
    }
}

async function delay(milliseconds: number) {
    return new Promise<void>(resolve => {
        setTimeout(resolve, milliseconds);
    });
}

printDelayed(["Hello", "beautiful", "asynchronous", "world"]).then(() => {
    console.log();
    console.log("打印每一個內容!");
});
```

查看 [Async Functions](http://blogs.msdn.com/b/typescript/archive/2015/11/03/what-about-async-await.aspx) 一文瞭解更多.

### 支持同時使用 `--target ES6` 和 `--module`

TypeScript 1.7 將 `ES6` 添加到了 `--module` 選項支持的選項的列表, 當編譯到 `ES6` 時允許指定模組類型. 這讓使用具體運行時中你需要的特性更加靈活.

#### 例子

```json
{
    "compilerOptions": {
        "module": "amd",
        "target": "es6"
    }
}
```

### `this` 類型

在方法中返回當前物件 (也就是 `this`) 是一種創建鏈式 API 的常見方式. 比如, 考慮下面的 `BasicCalculator` 模組:

```ts
export default class BasicCalculator {
    public constructor(protected value: number = 0) { }

    public currentValue(): number {
        return this.value;
    }

    public add(operand: number) {
        this.value += operand;
        return this;
    }

    public subtract(operand: number) {
        this.value -= operand;
        return this;
    }

    public multiply(operand: number) {
        this.value *= operand;
        return this;
    }

    public divide(operand: number) {
        this.value /= operand;
        return this;
    }
}
```

使用者可以這樣表述 `2 * 5 + 1`:

```ts
import calc from "./BasicCalculator";

let v = new calc(2)
    .multiply(5)
    .add(1)
    .currentValue();
```

這使得這麼一種優雅的編碼方式成為可能; 然而, 對於想要去繼承 `BasicCalculator` 的類來說有一個問題. 想像使用者可能需要編寫一個 `ScientificCalculator`:

```ts
import BasicCalculator from "./BasicCalculator";

export default class ScientificCalculator extends BasicCalculator {
    public constructor(value = 0) {
        super(value);
    }

    public square() {
        this.value = this.value ** 2;
        return this;
    }

    public sin() {
        this.value = Math.sin(this.value);
        return this;
    }
}
```

因為 `BasicCalculator` 的方法返回了 `this`, TypeScript 過去推斷的類型是 `BasicCalculator`, 如果在 `ScientificCalculator` 的實例上調用屬於 `BasicCalculator` 的方法, 類型系統不能很好地處理.

舉例來說:

```ts
import calc from "./ScientificCalculator";

let v = new calc(0.5)
    .square()
    .divide(2)
    .sin()    // Error: 'BasicCalculator' 沒有 'sin' 方法.
    .currentValue();
```

這已經不再是問題 - TypeScript 現在在類的實例方法中, 會將 `this` 推斷為一個特殊的叫做 `this` 的類型. `this` 類型也就寫作 `this`, 可以大致理解為 "方法調用時點左邊的類型".

`this` 類型在描述一些使用了 mixin 風格繼承的庫 (比如 Ember.js) 的交叉類型:

```ts
interface MyType {
    extend<T>(other: T): this & T;
}
```

### ES7 冪運算符

TypeScript 1.7 支持將在 ES7/ES2016 中增加的[冪運算符](https://github.com/rwaldron/exponentiation-operator): `**` 和 `**=`. 這些運算符會被轉換為 ES3/ES5 中的 `Math.pow`.

#### 舉例

```ts
var x = 2 ** 3;
var y = 10;
y **= 2;
var z =  -(4 ** 3);
```

會生成下面的 JavaScript:

```ts
var x = Math.pow(2, 3);
var y = 10;
y = Math.pow(y, 2);
var z = -(Math.pow(4, 3));
```

### 改進物件字面量解構的檢查

TypeScript 1.7 使物件和陣列字面量解構初始值的檢查更加直觀和自然.

當一個物件字面量通過與之對應的物件解構綁定推斷類型時:

- 物件解構綁定中有默認值的屬性對於物件字面量來說可選.
- 物件解構綁定中的屬性如果在物件字面量中沒有匹配的值, 則該屬性必須有默認值, 並且會被添加到物件字面量的類型中.
- 物件字面量中的屬性必須在物件解構綁定中存在.

當一個陣列字面量通過與之對應的陣列解構綁定推斷類型時:

- 陣列解構綁定中的元素如果在陣列字面量中沒有匹配的值, 則該元素必須有默認值, 並且會被添加到陣列字面量的類型中.

#### 舉例

```ts
// f1 的類型為 (arg?: { x?: number, y?: number }) => void
function f1({ x = 0, y = 0 } = {}) { }

// And can be called as:
f1();
f1({});
f1({ x: 1 });
f1({ y: 1 });
f1({ x: 1, y: 1 });

// f2 的類型為 (arg?: (x: number, y?: number) => void
function f2({ x, y = 0 } = { x: 0 }) { }

f2();
f2({});        // 錯誤, x 非可選
f2({ x: 1 });
f2({ y: 1 });  // 錯誤, x 非可選
f2({ x: 1, y: 1 });
```

### 裝飾器 (decorators) 支持的編譯目標版本增加 ES3

裝飾器現在可以編譯到 ES3. TypeScript 1.7 在 `__decorate` 函數中移除了 ES5 中增加的 `reduceRight`. 相關改動也單行內了對 `Object.getOwnPropertyDescriptor` 和 `Object.defineProperty` 的調用, 並向後兼容, 使 ES5 的輸出可以消除前面提到的 `Object` 方法的重複<sup>[1]</sup>.

## TypeScript 1.6

### JSX 支持

JSX 是一種可嵌入的類似 XML 的語法. 它將最終被轉換為合法的 JavaScript, 但轉換的語義和具體實現有關. JSX 隨著 React 流行起來, 也出現在其他應用中. TypeScript 1.6 支持 JavaScript 文件中 JSX 的嵌入, 類型檢查, 以及直接編譯為 JavaScript 的選項.

#### 新的 `.tsx` 文件擴展名和 `as` 運算符

TypeScript 1.6 引入了新的 `.tsx` 文件擴展名. 這一擴展名一方面允許 TypeScript 文件中的 JSX 語法, 一方面將 `as` 運算符作為默認的類型轉換方式 (避免 JSX 表達式和 TypeScript 前置類型轉換運算符之間的歧義). 比如:

```ts
var x = <any> foo;
// 與如下等價:
var x = foo as any;
```

#### 使用 React

使用 React 及 JSX 支持, 你需要使用 [React 類型聲明](https://github.com/borisyankov/DefinitelyTyped/tree/master/react). 這些類型定義了 `JSX` 命名空間, 以便 TypeScript 能正確地檢查 React 的 JSX 表達式. 比如:

```ts
/// <reference path="react.d.ts" />

interface Props {
  name: string;
}

class MyComponent extends React.Component<Props, {}> {
  render() {
    return <span>{this.props.foo}</span>
  }
}

<MyComponent name="bar" />; // 沒問題
<MyComponent name={0} />; // 錯誤, `name` 不是一個數字
```

#### 使用其他 JSX 框架

JSX 元素的名稱和屬性是根據 `JSX` 命名空間來檢驗的. 請查看 [JSX](https://github.com/Microsoft/TypeScript/wiki/JSX) 頁面瞭解如何為自己的框架定義 `JSX` 命名空間.

#### 編譯輸出

TypeScript 支持兩種 `JSX` 模式: `preserve` (保留) 和 `react`.

- `preserve` 模式將會在輸出中保留 JSX 表達式, 使之後的轉換步驟可以處理. *並且輸出的文件擴展名為 `.jsx`.*
- `react` 模式將會生成 `React.createElement`, 不再需要再通過 JSX 轉換即可運行, 輸出的文件擴展名為 `.js`.

查看 [JSX](https://github.com/Microsoft/TypeScript/wiki/JSX) 頁面瞭解更多 JSX 在 TypeScript 中的使用.

### 交叉類型 (intersection types)

TypeScript 1.6 引入了交叉類型作為聯合類型 (union types) 邏輯上的補充. 聯合類型 `A | B` 表示一個類型為 `A` 或 `B` 的實體, 而交叉類型 `A & B` 表示一個類型同時為 `A` 或 `B` 的實體.

#### 例子

```ts
function extend<T, U>(first: T, second: U): T & U {
    let result = <T & U> {};
    for (let id in first) {
        result[id] = first[id];
    }
    for (let id in second) {
        if (!result.hasOwnProperty(id)) {
            result[id] = second[id];
        }
    }
    return result;
}

var x = extend({ a: "hello" }, { b: 42 });
var s = x.a;
var n = x.b;
```

```ts
type LinkedList<T> = T & { next: LinkedList<T> };

interface Person {
    name: string;
}

var people: LinkedList<Person>;
var s = people.name;
var s = people.next.name;
var s = people.next.next.name;
var s = people.next.next.next.name;
interface A { a: string }
interface B { b: string }
interface C { c: string }

var abc: A & B & C;
abc.a = "hello";
abc.b = "hello";
abc.c = "hello";
```

查看 [issue #1256](https://github.com/Microsoft/TypeScript/issues/1256) 瞭解更多.

### 本地類型聲明

本地的類, 接口, 枚舉和類型別名現在可以在函數聲明中出現. 本地類型為塊級作用域, 與 `let` 和 `const` 聲明的變量類似. 比如說:

```ts
function f() {
    if (true) {
        interface T { x: number }
        let v: T;
        v.x = 5;
    }
    else {
        interface T { x: string }
        let v: T;
        v.x = "hello";
    }
}
```

推導出的函數返回值類型可能在函數內部聲明的. 調用函數的地方無法引用到這樣的本地類型, 但是它當然能從類型結構上匹配. 比如:

```ts
interface Point {
    x: number;
    y: number;
}

function getPointFactory(x: number, y: number) {
    class P {
        x = x;
        y = y;
    }
    return P;
}

var PointZero = getPointFactory(0, 0);
var PointOne = getPointFactory(1, 1);
var p1 = new PointZero();
var p2 = new PointZero();
var p3 = new PointOne();
```

本地的類型可以引用類型參數, 本地的類和接口本身即可能是泛型. 比如:

```ts
function f3() {
    function f<X, Y>(x: X, y: Y) {
        class C {
            public x = x;
            public y = y;
        }
        return C;
    }
    let C = f(10, "hello");
    let v = new C();
    let x = v.x;  // number
    let y = v.y;  // string
}
```

### 類表達式

TypeScript 1.6 增加了對 ES6 類表達式的支持. 在一個類表達式中, 類的名稱是可選的, 如果指明, 作用域僅限於類表達式本身. 這和函數表達式可選的名稱類似. 在類表達式外無法引用其實例類型, 但是自然也能夠從類型結構上匹配. 比如:

```ts
let Point = class {
    constructor(public x: number, public y: number) { }
    public length() {
        return Math.sqrt(this.x * this.x + this.y * this.y);
    }
};
var p = new Point(3, 4);  // p has anonymous class type
console.log(p.length());
```

### 繼承表達式

TypeScript 1.6 增加了對類繼承任意值為一個構造函數的表達式的支持. 這樣一來內建的類型也可以在類的聲明中被繼承.

`extends` 語句過去需要指定一個類型引用, 現在接受一個可選類型參數的表達式. 表達式的類型必須為有至少一個構造函數簽名的構造函數, 並且需要和 `extends` 語句中類型參數數量一致. 匹配的構造函數簽名的返回值類型是類實例類型繼承的基類型. 如此一來, 這使得普通的類和與類相似的表達式可以在 `extends` 語句中使用.

一些例子:

```ts
// 繼承內建類

class MyArray extends Array<number> { }
class MyError extends Error { }

// 繼承表達式類

class ThingA {
    getGreeting() { return "Hello from A"; }
}

class ThingB {
    getGreeting() { return "Hello from B"; }
}

interface Greeter {
    getGreeting(): string;
}

interface GreeterConstructor {
    new (): Greeter;
}

function getGreeterBase(): GreeterConstructor {
    return Math.random() >= 0.5 ? ThingA : ThingB;
}

class Test extends getGreeterBase() {
    sayHello() {
        console.log(this.getGreeting());
    }
}
```

### `abstract` (抽象的) 類和方法

TypeScript 1.6 為類和它們的方法增加了 `abstract` 關鍵字. 一個抽象類允許沒有被實現的方法, 並且不能被構造.

#### 例子

```ts
abstract class Base {
    abstract getThing(): string;
    getOtherThing() { return 'hello'; }
}

let x = new Base(); // 錯誤, 'Base' 是抽象的

// 錯誤, 必須也為抽象類, 或者實現 'getThing' 方法
class Derived1 extends Base { }

class Derived2 extends Base {
    getThing() { return 'hello'; }
    foo() {
        super.getThing();// 錯誤: 不能調用 'super' 的抽象方法
    }
}

var x = new Derived2(); // 正確
var y: Base = new Derived2(); // 同樣正確
y.getThing(); // 正確
y.getOtherThing(); // 正確
```

### 泛型別名

TypeScript 1.6 中, 類型別名支持泛型. 比如:

```ts
type Lazy<T> = T | (() => T);

var s: Lazy<string>;
s = "eager";
s = () => "lazy";

interface Tuple<A, B> {
    a: A;
    b: B;
}

type Pair<T> = Tuple<T, T>;
```

### 更嚴格的物件字面量賦值檢查

為了能發現多餘或者錯誤拼寫的屬性, TypeScript 1.6 使用了更嚴格的物件字面量檢查. 確切地說, 在將一個新的物件字面量賦值給一個變量, 或者傳遞給類型非空的參數時, 如果物件字面量的屬性在目標類型中不存在, 則會視為錯誤.

#### 例子

```ts
var x: { foo: number };
x = { foo: 1, baz: 2 };  // 錯誤, 多餘的屬性 `baz`

var y: { foo: number, bar?: number };
y = { foo: 1, baz: 2 };  // 錯誤, 多餘或者拼錯的屬性 `baz`
```

一個類型可以通過包含一個索引簽名來現實指明未出現在類型中的屬性是被允許的.

```ts
var x: { foo: number, [x: string]: any };
x = { foo: 1, baz: 2 };  // 現在 `baz` 匹配了索引簽名
```

### ES6 生成器 (generators)

TypeScript 1.6 添加了對於 ES6 輸出的生成器支持.

一個生成器函數可以有返回值類型標註, 就像普通的函數. 標註表示生成器函數返回的生成器的類型. 這裡有個例子:

```ts
function *g(): Iterable<string> {
    for (var i = 0; i < 100; i++) {
        yield ""; // string 可以賦值給 string
    }
    yield * otherStringGenerator(); // otherStringGenerator 必須可遍歷, 並且元素類型需要可賦值給 string
}
```

沒有標註類型的生成器函數會有自動推演的類型. 在下面的例子中, 類型會由 yield 語句推演出來:

```ts
function *g() {
    for (var i = 0; i < 100; i++) {
        yield ""; // 推導出 string
    }
    yield * otherStringGenerator(); // 推導出 otherStringGenerator 的元素類型
}
```

### 對 `async` (異步) 函數的試驗性支持

TypeScript 1.6 增加了編譯到 ES6 時對 `async` 函數試驗性的支持. 異步函數會執行一個異步的操作, 在等待的同時不會阻塞程序的正常運行. 這是通過與 ES6 兼容的 `Promise` 實現完成的, 並且會將函數體轉換為支持在等待的異步操作完成時繼續的形式.

由 `async` 標記的函數或方法被稱作_異步函數_. 這個標記告訴了編譯器該函數體需要被轉換, 關鍵字 _await_ 則應該被當做一個一元運算符, 而不是標示符. 一個_異步函數_必須返回類型與 `Promise` 兼容的值. 返回值類型的推斷只能在有一個全局的, 與 ES6 兼容的 `Promise` 類型時使用.

#### 例子

```ts
var p: Promise<number> = /* ... */;
async function fn(): Promise<number> {
  var i = await p; // 暫停執行知道 'p' 得到結果. 'i' 的類型為 "number"
  return 1 + i;
}

var a = async (): Promise<number> => 1 + await p; // 暫停執行.
var a = async () => 1 + await p; // 暫停執行. 使用 --target ES6 選項編譯時返回值類型被推斷為 "Promise<number>"
var fe = async function(): Promise<number> {
  var i = await p; // 暫停執行知道 'p' 得到結果. 'i' 的類型為 "number"
  return 1 + i;
}

class C {
  async m(): Promise<number> {
    var i = await p; // 暫停執行知道 'p' 得到結果. 'i' 的類型為 "number"
    return 1 + i;
  }

  async get p(): Promise<number> {
    var i = await p; // 暫停執行知道 'p' 得到結果. 'i' 的類型為 "number"
    return 1 + i;
  }
}
```

### 每天發佈新版本

由於並不算嚴格意義上的語言變化<sup>[2]</sup>, 每天的新版本可以使用如下命令安裝獲得:

```sh
npm install -g typescript@next
```

### 對模組解析邏輯的調整

從 1.6 開始, TypeScript 編譯器對於 "commonjs" 的模組解析會使用一套不同的規則. 這些[規則](https://github.com/Microsoft/TypeScript/issues/2338) 嘗試模仿 Node 查找模組的過程. 這就意味著 node 模組可以包含它的類型信息, 並且 TypeScript 編譯器可以找到這些信息. 不過用戶可以通過使用 `--moduleResolution` 命令行選項覆蓋模組解析規則. 支持的值有:

- 'classic' - TypeScript 1.6 以前的編譯器使用的模組解析規則
- 'node' - 與 node 相似的模組解析

### 合併外圍類和接口的聲明

外圍類的實例類型可以通過接口聲明來擴展. 類構造函數物件不會被修改. 比如說:

```ts
declare class Foo {
    public x : number;
}

interface Foo {
    y : string;
}

function bar(foo : Foo)  {
    foo.x = 1; // 沒問題, 在類 Foo 中有聲明
    foo.y = "1"; // 沒問題, 在接口 Foo 中有聲明
}
```

### 用戶定義的類型收窄函數

TypeScript 1.6 增加了一個新的在 `if` 語句中收窄變量類型的方式, 作為對 `typeof` 和 `instanceof` 的補充. 用戶定義的類型收窄函數的返回值類型標註形式為 `x is T`, 這裡 `x` 是函數聲明中的形參, `T` 是任何類型. 當一個用戶定義的類型收窄函數在 `if` 語句中被傳入某個變量執行時, 該變量的類型會被收窄到 `T`.

#### 例子

```ts
function isCat(a: any): a is Cat {
  return a.name === 'kitty';
}

var x: Cat | Dog;
if(isCat(x)) {
  x.meow(); // 那麼, x 在這個代碼塊內是 Cat 類型
}
```

### `tsconfig.json` 對 `exclude` 屬性的支持

一個沒有寫明 `files` 屬性的 `tsconfig.json` 文件 (默認會引用所有子目錄下的 *.ts 文件) 現在可以包含一個 `exclude` 屬性, 指定需要在編譯中排除的文件或者目錄列表. `exclude` 屬性必須是一個字符串陣列, 其中每一個元素指定對應的一個文件或者文件夾名稱對於 `tsconfig.json` 文件所在位置的相對路徑. 舉例來說:

```json
{
    "compilerOptions": {
        "out": "test.js"
    },
    "exclude": [
        "node_modules",
        "test.ts",
        "utils/t2.ts"
    ]
}
```

`exclude` 列表不支持通配符. 僅僅可以是文件或者目錄的列表.

### `--init` 命令行選項

在一個目錄中執行 `tsc --init` 可以在該目錄中創建一個包含了默認值的 `tsconfig.json`. 可以通過一併傳遞其他選項來生成初始的 `tsconfig.json`.

## TypeScript 1.5

### ES6 模組

TypeScript 1.5 支持 ECMAScript 6 (ES6) 模組. ES6 模組可以看做之前 TypeScript 的外部模組換上了新的語法: ES6 模組是分開加載的源文件, 這些文件還可能引入其他模組, 並且導出部分供外部可訪問. ES6 模組新增了幾種導入和導出聲明. 我們建議使用 TypeScript 開發的庫和應用能夠更新到新的語法, 但不做強制要求. 新的 ES6 模組語法和 TypeScript 原來的內部和外部模組結構同時被支持, 如果需要也可以混合使用.

#### 導出聲明

作為 TypeScript 已有的 `export` 前綴支持, 模組成員也可以使用單獨導出的聲明導出, 如果需要, `as` 語句可以指定不同的導出名稱.

```ts
interface Stream { ... }
function writeToStream(stream: Stream, data: string) { ... }
export { Stream, writeToStream as write };  // writeToStream 導出為 write
```

引入聲明也可以使用 `as` 語句來指定一個不同的導入名稱. 比如:

```ts
import { read, write, standardOutput as stdout } from "./inout";
var s = read(stdout);
write(stdout, s);
```

作為單獨導入的候選項, 命名空間導入可以導入整個模組:

```ts
import * as io from "./inout";
var s = io.read(io.standardOutput);
io.write(io.standardOutput, s);
```

### 重新導出

使用 `from` 語句一個模組可以複製指定模組的導出項到當前模組, 而無需創建本地名稱.

```ts
export { read, write, standardOutput as stdout } from "./inout";
```

`export *` 可以用來重新導出另一個模組的所有導出項. 在創建一個聚合了其他幾個模組導出項的模組時很方便.

```ts
export function transform(s: string): string { ... }
export * from "./mod1";
export * from "./mod2";
```

#### 默認導出項

一個 export default 聲明表示一個表達式是這個模組的默認導出項.

```ts
export default class Greeter {
    sayHello() {
        console.log("Greetings!");
    }
}
```

對應的可以使用默認導入:

```ts
import Greeter from "./greeter";
var g = new Greeter();
g.sayHello();
```

#### 無導入加載

"無導入加載" 可以被用來加載某些只需要其副作用的模組.

```ts
import "./polyfills";
```

瞭解更多關於模組的信息, 請參見 [ES6 模組支持規範](https://github.com/Microsoft/TypeScript/issues/2242).

### 聲明與賦值的解構

TypeScript 1.5 添加了對 ES6 解構聲明與賦值的支持.

#### 解構

解構聲明會引入一個或多個命名變量, 並且初始化它們的值為物件的屬性或者陣列的元素對應的值.

比如說, 下面的例子聲明了變量 `x`, `y` 和 `z`, 並且分別將它們的值初始化為 `getSomeObject().x`, `getSomeObject().x` 和 `getSomeObject().x`:

```ts
var { x, y, z } = getSomeObject();
```

解構聲明也可以用於從陣列中得到值.

```ts
var [x, y, z = 10] = getSomeArray();
```

相似的, 解構可以用在函數的參數聲明中:

```ts
function drawText({ text = "", location: [x, y] = [0, 0], bold = false }) {
    // 畫出文本
}

// 以一個物件字面量為參數調用 drawText
var item = { text: "someText", location: [1,2,3], style: "italics" };
drawText(item);
```

#### 賦值

解構也可以被用於普通的賦值表達式. 舉例來講, 交換兩個變量的值可以被寫作一個解構賦值:

```ts
var x = 1;
var y = 2;
[x, y] = [y, x];
```

### `namespace` (命名空間) 關鍵字

過去 TypeScript 中 `module` 關鍵字既可以定義 "內部模組", 也可以定義 "外部模組"; 這讓剛剛接觸 TypeScript 的開發者有些困惑. "內部模組" 的概念更接近於大部分人眼中的命名空間; 而 "外部模組" 對於 JS 來講, 現在也就是模組了.

> 注意: 之前定義內部模組的語法依然被支持.

**之前**:

```ts
module Math {
    export function add(x, y) { ... }
}
```

**之後**:

```ts
namespace Math {
    export function add(x, y) { ... }
}
```

### `let` 和 `const` 的支持

ES6 的 `let` 和 `const` 聲明現在支持編譯到 ES3 和 ES5.

#### Const

```ts
const MAX = 100;

++MAX; // 錯誤: 自增/減運算符不能用於一個常量
```

#### 塊級作用域

```ts
if (true) {
  let a = 4;
  // 使用變量 a
}
else {
  let a = "string";
  // 使用變量 a
}

alert(a); // 錯誤: 變量 a 在當前作用域未定義
```

### `for...of` 的支持

TypeScript 1.5 增加了 ES6 `for...of` 循環編譯到 ES3/ES5 時對陣列的支持, 以及編譯到 ES6 時對滿足 `Iterator` 接口的全面支持.

#### 例子:

TypeScript 編譯器會轉譯 `for...of` 陣列到具有語義的 ES3/ES5 JavaScript (如果被設置為編譯到這些版本).

```ts
for (var v of expr) { }
```

會輸出為:

```js
for (var _i = 0, _a = expr; _i < _a.length; _i++) {
    var v = _a[_i];
}
```

### 裝飾器

> TypeScript 裝飾器是局域 [ES7 裝飾器](https://github.com/wycats/javascript-decorators) 提案的.

一個裝飾器是:

- 一個表達式
- 並且值為一個函數
- 接受 `target`, `name`, 以及屬性描述物件作為參數
- 可選返回一個會被應用到目標物件的屬性描述物件

> 瞭解更多, 請參見 [裝飾器](https://github.com/Microsoft/TypeScript/issues/2249) 提案.

#### 例子:

裝飾器 `readonly` 和 `enumerable(false)` 會在屬性 `method` 添加到類 `C` 上之前被應用. 這使得裝飾器可以修改其實現, 具體到這個例子, 設置了 `descriptor` 為 `writable: false` 以及 `enumerable: false`.

```ts
class C {
  @readonly
  @enumerable(false)
  method() { }
}

function readonly(target, key, descriptor) {
    descriptor.writable = false;
}

function enumerable(value) {
  return function (target, key, descriptor) {
     descriptor.enumerable = value;
  }
}
```

### 計算屬性

使用動態的屬性初始化一個物件可能會很麻煩. 參考下面的例子:

```ts
type NeighborMap = { [name: string]: Node };
type Node = { name: string; neighbors: NeighborMap;}

function makeNode(name: string, initialNeighbor: Node): Node {
    var neighbors: NeighborMap = {};
    neighbors[initialNeighbor.name] = initialNeighbor;
    return { name: name, neighbors: neighbors };
}
```

這裡我們需要創建一個包含了 neighbor-map 的變量, 便於我們初始化它. 使用 TypeScript 1.5, 我們可以讓編譯器來幹重活:

```ts
function makeNode(name: string, initialNeighbor: Node): Node {
    return {
        name: name,
        neighbors: {
            [initialNeighbor.name]: initialNeighbor
        }
    }
}
```

### 指出 `UMD` 和 `System` 模組輸出

作為 `AMD` 和 `CommonJS` 模組加載器的補充, TypeScript 現在支持輸出為 `UMD` ([Universal Module Definition](https://github.com/umdjs/umd)) 和 [`System`](https://github.com/systemjs/systemjs) 模組的格式.

**用法**:

> tsc --module umd

以及

> tsc --module system


### Unicode 字符串碼位轉義

ES6 中允許用戶使用單個轉義表示一個 Unicode 碼位.

舉個例子, 考慮我們需要轉義一個包含了字符 '𠮷' 的字符串. 在 UTF-16/USC2 中, '𠮷' 被表示為一個代理對, 意思就是它被編碼為一對 16 位值的代碼單元, 具體來說是 `0xD842` 和 `0xDFB7`. 之前這意味著你必須將該碼位轉義為 `"\uD842\uDFB7"`. 這樣做有一個重要的問題, 就事很難講兩個獨立的字符同一個代理對區分開來.

通過 ES6 的碼位轉義, 你可以在字符串或模板字符串中清晰地通過一個轉義表示一個確切的字符: `"\u{20bb7}"`. TypeScript 在編譯到 ES3/ES5 時會將該字符串輸出為 `"\uD842\uDFB7"`.

### 標籤模板字符串編譯到 ES3/ES5

TypeScript 1.4 中, 我們添加了模板字符串編譯到所有 ES 版本的支持, 並且支持標籤模板字符串編譯到 ES6. 得益於 [@ivogabe](https://github.com/ivogabe) 的大量付出, 我們填補了標籤模板字符串對編譯到 ES3/ES5 的支持.

當編譯到 ES3/ES5 時, 下面的代碼:

```ts
function oddRawStrings(strs: TemplateStringsArray, n1, n2) {
    return strs.raw.filter((raw, index) => index % 2 === 1);
}

oddRawStrings `Hello \n${123} \t ${456}\n world`
```

會被輸出為:

```ts
function oddRawStrings(strs, n1, n2) {
    return strs.raw.filter(function (raw, index) {
        return index % 2 === 1;
    });
}
(_a = ["Hello \n", " \t ", "\n world"], _a.raw = ["Hello \\n", " \\t ", "\\n world"], oddRawStrings(_a, 123, 456));
var _a;
```

### AMD 可選依賴名稱

`/// <amd-dependency path="x" />` 會告訴編譯器需要被注入到模組 `require` 方法中的非 TS 模組依賴; 然而在 TS 代碼中無法使用這個模組.

新的 `amd-dependency name` 屬性允許為 AMD 依賴傳遞一個可選的名稱.

```ts
/// <amd-dependency path="legacy/moduleA" name="moduleA"/>
declare var moduleA:MyType
moduleA.callStuff()
```

生成的 JS 代碼:

```ts
define(["require", "exports", "legacy/moduleA"], function (require, exports, moduleA) {
    moduleA.callStuff()
});
```

### 通過 `tsconfig.json` 指示一個項目

通過添加 `tsconfig.json` 到一個目錄指明這是一個 TypeScript 項目的根目錄. `tsconfig.json` 文件指定了根文件以及編譯項目需要的編譯器選項. 一個項目可以由以下方式編譯:

- 調用 tsc 並不指定輸入文件, 此時編譯器會從當前目錄開始往上級目錄尋找 `tsconfig.json` 文件.
- 調用 tsc 並不指定輸入文件, 使用 `-project` (或者 `-p`) 命令行選項指定包含了 `tsconfig.json` 文件的目錄.

#### 例子:
```json
{
    "compilerOptions": {
        "module": "commonjs",
        "noImplicitAny": true,
        "sourceMap": true,
    }
}
```

參見 [tsconfig.json wiki 頁面](https://github.com/Microsoft/TypeScript/wiki/tsconfig.json) 查看更多信息.

### `--rootDir` 命令行選項

選項 `--outDir` 在輸出中會保留輸入的層級關係. 編譯器將所有輸入文件共有的最長路徑作為根路徑; 並且在輸出中應用對應的子層級關係.

有的時候這並不是期望的結果, 比如輸入 `FolderA\FolderB\1.ts` 和 `FolderA\FolderB\2.ts`, 輸出結構會是 `FolderA\FolderB\` 對應的結構. 如果輸入中新增 `FolderA\3.ts` 文件, 輸出的結構將突然變為 `FolderA\` 對應的結構.

`--rootDir` 指定了會輸出對應結構的輸入目錄, 不再通過計算獲得.

### `--noEmitHelpers` 命令行選項

TypeScript 編譯器在需要的時候會輸出一些像 `__extends` 這樣的工具函數. 這些函數會在使用它們的所有文件中輸出. 如果你想要聚合所有的工具函數到同一個位置, 或者覆蓋默認的行為, 使用 `--noEmitHelpers` 來告知編譯器不要輸出它們.

### `--newLine` 命令行選項

默認輸出的換行符在 Windows 上是 `\r\n`, 在 *nix 上是 `\n`. `--newLine` 命令行標記可以覆蓋這個行為, 並指定輸出文件中使用的換行符.

### `--inlineSourceMap` and `inlineSources` 命令行選項

`--inlineSourceMap` 將內嵌源文件映射到 `.js` 文件, 而不是在單獨的 `.js.map` 文件中. `--inlineSources` 允許進一步將 `.ts` 文件內容包含到輸出文件中.

## TypeScript 1.4

### 聯合類型

#### 概覽

聯合類型是描述一個可能是幾個類型之一的值的有效方式. 舉例來說, 你可能會有一個 API 用於執行一個 `commandline` 為 `string`, `string[]` 或者是返回值為 `string` 的函數的程序. 現在可以這樣寫:

```ts
interface RunOptions {
   program: string;
   commandline: string[]|string|(() => string);
}
```

對聯合類型的賦值非常直觀 -- 任何可以賦值給聯合類型中任意一個類型的值都可以賦值給這個聯合類型:

```ts
var opts: RunOptions = /* ... */;
opts.commandline = '-hello world'; // 沒問題
opts.commandline = ['-hello', 'world']; // 沒問題
opts.commandline = [42]; // 錯誤, number 不是 string 或 string[]
```

當從聯合類型中讀取時, 你可以看到聯合類型中各類型共有的屬性:

```ts
if (opts.length === 0) { // 沒問題, string 和 string[] 都有 'length' 屬性
  console.log("it's empty");
}
```

使用類型收窄, 你可以方便的使用具有聯合類型的變量:

```ts
function formatCommandline(c: string|string[]) {
    if (typeof c === 'string') {
        return c.trim();
    } else {
        return c.join(' ');
    }
}
```

#### 更嚴格的泛型

結合聯合類型可以表示很多種類型場景, 我們決定讓某些泛型調用更加嚴格. 之前, 以下的代碼能出人意料地無錯通過編譯:

```ts
function equal<T>(lhs: T, rhs: T): boolean {
  return lhs === rhs;
}

// 過去: 無錯誤
// 現在: 錯誤, 'string' 和 'number' 間沒有最佳共有類型
var e = equal(42, 'hello');
```

而通過聯合類型, 你現在可以在函數聲明或者調用的時候指明想要的行為:

```ts
// 'choose' 函數的參數類型必須相同
function choose1<T>(a: T, b: T): T { return Math.random() > 0.5 ? a : b }
var a = choose1('hello', 42); // 錯誤
var b = choose1<string|number>('hello', 42); // 正確

// 'choose' 函數的參數類型不需要相同
function choose2<T, U>(a: T, b: U): T|U { return Math.random() > 0.5 ? a : b }
var c = choose2('bar', 'foo'); // 正確, c: string
var d = choose2('hello', 42); // 正確, d: string|number
```

#### 更好的類型接口

聯合類型也允許了陣列或者其他地方有更好的類型接口, 以便一個集合中可能有多重類型.

```ts
var x = [1, 'hello']; // x: Array<string|number>
x[0] = 'world'; // 正確
x[0] = false; // 錯誤, boolean 不是 string 或 number
```

### `let` 聲明

在 JavaScript 中, `var` 聲明會被 "提升" 到它們所在的作用域. 這可能會導致一些令人疑惑的問題:

```ts
console.log(x); // 本意是在這裡寫 'y'
/* 當前代碼塊靠後的位置 */
var x = 'hello';
```

ES6 的關鍵字 `let` 現在在 TypeScript 中得到支持, 聲明變量獲得了更直觀的塊級語義. 一個 `let` 變量只能在它聲明之後被引用, 其作用域被限定於它被聲明的句法塊:

```ts
if (foo) {
    console.log(x); // 錯誤, 在聲明前不能引用 x
    let x = 'hello';
} else {
    console.log(x); // 錯誤, x 在當前塊中沒有聲明
}
```

`let` 僅在編譯到 ECMAScript 6 時被支持 (`--target ES6`).

### `const` 聲明

另外一種在 TypeScript 中被支持的新的 ES6 聲明類型是 `const`. 一個 `const` 變量不能被賦值, 並且在聲明的時候必須被初始化. 這可以用在你聲明和初始化後不希望值被改變時:

```ts
const halfPi = Math.PI / 2;
halfPi = 2; // 錯誤, 不能賦值給一個 `const`
```

`const` 僅在編譯到 ECMAScript 6 時被支持 (`--target ES6`).

## 模板字符串

TypeScript 現在支持 ES6 模板字符串. 現在可以方便地在字符串中嵌入任何表達式:

```ts
var name = "TypeScript";
var greeting  = `Hello, ${name}! Your name has ${name.length} characters`;
```

當編譯到 ES6 以前的版本時, 字符串會被分解為:

```ts
var name = "TypeScript!";
var greeting = "Hello, " + name + "! Your name has " + name.length + " characters";
```

### 類型收窄

在 JavaScript 中常常用 `typeof` 或者 `instanceof` 在運行時檢查一個表達式的類型. TypeScript 現在理解這些條件, 並且在 `if` 語句中會據此改變類型接口.

使用 `typeof` 來檢查一個變量:

```ts
var x: any = /* ... */;
if(typeof x === 'string') {
    console.log(x.subtr(1)); // 錯誤, 'subtr' 在 'string' 上不存在
}
// 這裡 x 的類型依然是 any
x.unknown(); // 正確
```

與聯合類型和 `else` 一起使用 `typeof`:

```ts
var x: string | HTMLElement = /* ... */;
if (typeof x === 'string') {
    // x 如上所述是一個 string
} else {
    // x 在這裡是 HTMLElement
    console.log(x.innerHTML);
}
```

與類和聯合類型一起使用 `instanceof`:

```ts
class Dog { woof() { } }
class Cat { meow() { } }
var pet: Dog | Cat = /* ... */;
if (pet instanceof Dog) {
    pet.woof(); // 正確
} else {
    pet.woof(); // 錯誤
}
```

### 類型別名

現在你可以使用 `type` 關鍵字為類型定義一個_別名_:

```ts
type PrimitiveArray = Array<string | number | boolean>;
type MyNumber = number;
type NgScope = ng.IScope;
type Callback = () => void;
```

類型別名和它們原來的類型完全相同; 它們僅僅是另一種表述的名稱.

### `const enum` (完全單行內的枚舉)

枚舉非常有用, 但有的程序可能並不需要生成的代碼, 而簡單地將枚舉成員的數字值單行內能夠給這些程序帶來一定好處. 新的 `const enum` 聲明在類型安全上和 `enum` 一致, 但是編譯後會被完全抹去.

```ts
const enum Suit { Clubs, Diamonds, Hearts, Spades }
var d = Suit.Diamonds;
```

編譯為:

```js
var d = 1;
```

如果可能 TypeScript 現在會計算枚舉的值:

```ts
enum MyFlags {
  None = 0,
  Neat = 1,
  Cool = 2,
  Awesome = 4,
  Best = Neat | Cool | Awesome
}
var b = MyFlags.Best; // 輸出 var b = 7;
```

### `--noEmitOnError` 命令行選項

TypeScript 編譯器的默認行為會在出現類型錯誤 (比如, 嘗試賦值一個 `string` 給 `number`) 時依然輸出 .js 文件. 在構建服務器或者其他只希望有 "乾淨" 版本的場景可能並不是期望的結果. 新的 `noEmitOnError` 標記會使編譯器在有任何錯誤時不輸出 .js 代碼.

對於 MSBuild 的項目這是目前的默認設定; 這使 MSBuild 的增量編譯變得可行, 輸出僅在代碼沒有問題時產生.

### AMD 模組名稱

AMD 模組默認生成是匿名的. 對於一些像打包工具這樣的處理輸出模組的工具會帶來一些問題 (比如 r.js).

新的 `amd-module name` 標籤允許傳入一個可選的模組名稱給編譯器:

```ts
//// [amdModule.ts]
///<amd-module name='NamedModule'/>
export class C {
}
```

這會在調用 AMD 的 `define` 方法時傳入名稱 `NamedModule`:

```ts
//// [amdModule.js]
define("NamedModule", ["require", "exports"], function (require, exports) {
    var C = (function () {
        function C() {
        }
        return C;
    })();
    exports.C = C;
});
```

## TypeScript 1.3

### 受保護成員

在類中新的 `protected` 標示符就像它在其他一些像 C++, C# 與 Java 這樣的常見語言中的功能一致. 一個 `protected` (受保護的) 的成員僅在子類或者聲明它的類中可見:

```ts
class Thing {
  protected doSomething() { /* ... */ }
}

class MyThing extends Thing {
  public myMethod() {
    // 正確, 可以在子類中訪問受保護成員
    this.doSomething();
  }
}
var t = new MyThing();
t.doSomething(); // 錯誤, 不能在類外調用受保護成員
```

### 元組類型

元組類型可以表示一個陣列中部分元素的類型是已知, 但不一定相同的情況. 舉例來說, 你可能希望描述一個陣列, 在下標 0 處為 `string`, 在 1 處為 `number`:

```ts
// 聲明一個元組類型
var x: [string, number];
// 初始化
x = ['hello', 10]; // 正確
// 錯誤的初始化
x = [10, 'hello']; // 錯誤
```

當使用已知的下標訪問某個元素時, 能夠獲得正確的類型:

```ts
console.log(x[0].substr(1)); // 正確
console.log(x[1].substr(1)); // 錯誤, 'number' 類型沒有 'substr' 屬性
```

注意在 TypeScript 1.4 中, 當訪問某個下標不在已知範圍內的元素時, 獲得的是聯合類型:

```ts
x[3] = 'world'; // 正確
console.log(x[5].toString()); // 正確, 'string' 和 'number' 都有 toString 方法
x[6] = true; // 錯誤, boolean 不是 number 或 string
```

## TypeScript 1.1

### 性能優化

1.1 版編譯器大體比之前任何版本快 4 倍. 查看 [這篇文章裡令人印象深刻的對比](http://blogs.msdn.com/b/typescript/archive/2014/10/06/announcing-typescript-1-1-ctp.aspx).

### 更好的模組可見規則

TypeScript 現在僅在開啟了 `--declaration` 標記時嚴格要求模組類型的可見性. 對於 Angular 的場景來說非常有用, 比如:

```ts
module MyControllers {
  interface ZooScope extends ng.IScope {
    animals: Animal[];
  }
  export class ZooController {
    // 過去是錯誤的 (無法暴露 ZooScope), 而現在僅在需要生成 .d.ts 文件時報錯
    constructor(public $scope: ZooScope) { }
    /* 更多代碼 */
  }
}
```

---

**[1]** 原文為 "The changes also inline calls `Object.getOwnPropertyDescriptor` and `Object.defineProperty` in a backwards-compatible fashion that allows for a to clean up the emit for ES5 and later by removing various repetitive calls to the aforementioned `Object` methods."

**[2]** 原文為 "While not strictly a language change..."
