# TypeScript 1.8

## 類型參數約束

在 TypeScript 1.8 中, 類型參數的限制可以引用自同一個類型參數列表中的類型參數. 在此之前這種做法會報錯. 這種特性通常被叫做 [F-Bounded Polymorphism](https://en.wikipedia.org/wiki/Bounded_quantification#F-bounded_quantification).

### 例子

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

## 控制流錯誤分析

TypeScript 1.8 中引入了控制流分析來捕獲開發者通常會遇到的一些錯誤.

詳情見接下來的內容, 可以上手嘗試:

![cfa](https://cloud.githubusercontent.com/assets/8052307/5210657/c5ae0f28-7585-11e4-97d8-86169ef2a160.gif)

### 不可及的代碼

一定無法在運行時被執行的語句現在會被標記上代碼不可及錯誤. 舉個例子, 在無條件限制的 `return`, `throw`, `break` 或者 `continue` 後的語句被認為是不可及的. 使用 `--allowUnreachableCode` 來禁用不可及代碼的檢測和報錯.

#### 例子

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

### 未使用的標籤

未使用的標籤也會被標記. 和不可及代碼檢查一樣, 被使用的標籤檢查也是默認開啟的. 使用 `--allowUnusedLabels` 來禁用未使用標籤的報錯.

#### 例子

```ts
loop: while (x > 0) {  // 錯誤: 未使用的標籤.
    x++;
}
```

### 隱式返回

JS 中沒有返回值的代碼分支會隱式地返回 `undefined`. 現在編譯器可以將這種方式標記為隱式返回. 對於隱式返回的檢查默認是被禁用的, 可以使用 `--noImplicitReturns` 來啟用.

#### 例子

```ts
function f(x) { // 錯誤: 不是所有分支都返回了值.
    if (x) {
        return false;
    }

    // 隱式返回了 `undefined`
}
```

### Case 語句貫穿

TypeScript 現在可以在 switch 語句中出現貫穿的幾個非空 case 時報錯.
這個檢測默認是關閉的, 可以使用 `--noFallthroughCasesInSwitch` 啟用.

#### 例子

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

## React 無狀態的函數組件

TypeScript 現在支持[無狀態的函數組件](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions).
它是可以組合其他組件的輕量級組件.

```ts
// 使用參數解構和默認值輕鬆地定義 'props' 的類型
const Greeter = ({name = 'world'}) => <div>Hello, {name}!</div>;

// 參數可以被檢驗
let example = <Greeter name='TypeScript 1.8' />;
```

如果需要使用這一特性及簡化的 props, 請確認使用的是[最新的 react.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/react).

## 簡化的 React `props` 類型管理

在 TypeScript 1.8 配合最新的 react.d.ts (見上方) 大幅簡化了 `props` 的類型聲明.

具體的:

- 你不再需要顯式的聲明 `ref` 和 `key` 或者 `extend React.Props`
- `ref` 和 `key` 屬性會在所有組件上擁有正確的類型.
- `ref` 屬性在無狀態函數組件上會被正確地禁用.

## 在模組中擴充全局或者模組作用域

用戶現在可以為任何模組進行他們想要, 或者其他人已經對其作出的擴充.
模組擴充的形式和過去的包模組一致 (例如 `declare module "foo" { }` 這樣的語法), 並且可以直接嵌在你自己的模組內, 或者在另外的頂級外部包模組中.

除此之外, TypeScript 還以 `declare global { }` 的形式提供了對於_全局_聲明的擴充.
這能使模組對像 `Array` 這樣的全局類型在必要的時候進行擴充.

模組擴充的名稱解析規則與 `import` 和 `export` 聲明中的一致.
擴充的模組聲明合併方式與在同一個文件中聲明是相同的.

不論是模組擴充還是全局聲明擴充都不能向頂級作用域添加新的項目 - 它們只能為已經存在的聲明添加 "補丁".

### 例子

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

### 例子

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

## 字符串字面量類型

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

## 更好的聯合/交叉類型接口

TypeScript 1.8 優化了源類型和目標類型都是聯合或者交叉類型的情況下的類型推導.
舉例來說, 當從 `string | string[]` 推導到 `string | T` 時, 我們將類型拆解為 `string[]` 和 `T`, 這樣就可以將 `string[]` 推導為 `T`.

### 例子

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

## 使用 `--outFile` 合併 `AMD` 和 `System` 模組

在使用 `--module amd` 或者 `--module system` 的同時制定 `--outFile` 將會把所有參與編譯的模組合併為單個包括了多個模組閉包的輸出文件.

每一個模組都會根據其相對於 `rootDir` 的位置被計算出自己的模組名稱.

### 例子

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

## 支持 SystemJS 使用 `default` 導入

像 SystemJS 這樣的模組加載器將 CommonJS 模組做了包裝並暴露為 `default` ES6 導入項. 這使得在 SystemJS 和 CommonJS 的實現由於不同加載器不同的模組導出方式不能共享定義.

設置新的編譯選項 `--allowSyntheticDefaultImports` 指明模組加載器會進行導入的 `.ts` 或 `.d.ts` 中未指定的某種類型的默認導入項構建. 編譯器會由此推斷存在一個 `default` 導出項和整個模組自己一致.

此選項在 System 模組默認開啟.

## 允許循環中被引用的 `let`/`const`

之前這樣會報錯, 現在由 TypeScript 1.8 支持.
循環中被函數引用的 `let`/`const` 聲明現在會被輸出為與 `let`/`const` 更新語義相符的代碼.

### 例子

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

## 改進的 `for..in` 語句檢查

過去 `for..in` 變量的類型被推斷為 `any`, 這使得編譯器忽略了 `for..in` 語句內的一些不合法的使用.

從 TypeScript 1.8 開始:

- 在 `for..in` 語句中的變量隱含類型為 `string`.
- 當一個有數字索引簽名對應類型 `T` (比如一個陣列) 的物件被一個 `for..in` 索引*有*數字索引簽名並且*沒有*字符串索引簽名 (比如還是陣列) 的物件的變量索引, 產生的值的類型為 `T`.

### 例子

```ts
var a: MyObject[];
for (var x in a) {   // x 的隱含類型為 string
    var obj = a[x];  // obj 的類型為 MyObject
}
```

## 模組現在輸出時會加上 `"use strict;"`

對於 ES6 來說模組始終以嚴格模式被解析, 但這一點過去對於非 ES6 目標在生成的代碼中並沒有遵循. 從 TypeScript 1.8 開始, 輸出的模組總會為嚴格模式. 由於多數嚴格模式下的錯誤也是 TS 編譯時的錯誤, 多數代碼並不會有可見的改動, 但是這也意味著有一些東西可能在運行時沒有徵兆地失敗, 比如賦值給 `NaN` 現在會有運行時錯誤. 你可以參考這篇 [MDN 上的文章](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mod) 查看詳細的嚴格模式與非嚴格模式的區別列表.

## 使用 `--allowJs` 加入 `.js` 文件

經常在項目中會有外部的非 TypeScript 編寫的源文件.
一種方式是將 JS 代碼轉換為 TS 代碼, 但這時又希望將所有 JS 代碼和新的 TS 代碼的輸出一起打包為一個文件.

`.js` 文件現在允許作為 `tsc` 的輸入文件. TypeScript 編譯器會檢查 `.js` 輸入文件的語法錯誤, 並根據 `--target` 和 `--module` 選項輸出對應的代碼.
輸出也會和其他 `.ts` 文件一起. `.js` 文件的 source maps 也會像 `.ts` 文件一樣被生成.

## 使用 `--reactNamespace` 自定義 JSX 工廠

在使用 `--jsx react` 的同時使用 `--reactNamespace <JSX 工廠名稱>` 可以允許使用一個不同的 JSX 工廠代替默認的 `React`.

新的工廠名稱會被用來調用 `createElement` 和 `__spread` 方法.

### 例子

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

## 基於 `this` 的類型收窄

TypeScript 1.8 為類和接口方法擴展了[用戶定義的類型收窄函數](#用戶定義的類型收窄函數).

`this is T` 現在是類或接口方法的合法的返回值類型標註.
當在類型收窄的位置使用時 (比如 `if` 語句), 函數調用表達式的目標物件的類型會被收窄為 `T`.

### 例子

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

## 官方的 TypeScript NuGet 包

從 TypeScript 1.8 開始, 將為 TypeScript 編譯器 (`tsc.exe`) 和 MSBuild 整合 (`Microsoft.TypeScript.targets` 和 `Microsoft.TypeScript.Tasks.dll`) 提供官方的 NuGet 包.

穩定版本可以在這裡下載:

- [Microsoft.TypeScript.Compiler](https://www.nuget.org/packages/Microsoft.TypeScript.Compiler/)
- [Microsoft.TypeScript.MSBuild](https://www.nuget.org/packages/Microsoft.TypeScript.MSBuild/)

與此同時, 和[每日 npm 包](http://blogs.msdn.com/b/typescript/archive/2015/07/27/introducing-typescript-nightlies.aspx)對應的每日 NuGet 包可以在 https://myget.org 下載:

- [TypeScript-Preview](https://www.myget.org/gallery/typescript-preview)

## `tsc` 錯誤信息更美觀

我們理解大量單色的輸出並不直觀. 顏色可以幫助識別信息的始末, 這些視覺上的線索在處理複雜的錯誤信息時非常重要.

通過傳遞 `--pretty` 命令行選項, TypeScript 會給出更豐富的輸出, 包含錯誤發生的上下文.

![展示在 ConEmu 中美化之後的錯誤信息](https://raw.githubusercontent.com/wiki/Microsoft/TypeScript/images/new-in-typescript/pretty01.png)

## 高亮 VS 2015 中的 JSX 代碼

在 TypeScript 1.8 中, JSX 標籤現在可以在 Visual Studio 2015 中被分別和高亮.

![jsx](https://cloud.githubusercontent.com/assets/8052307/12271404/b875c502-b90f-11e5-93d8-c6740be354d1.png)

通過 `工具`->`選項`->`環境`->`字體與顏色` 頁面在 `VB XML` 顏色和字體設置中還可以進一步改變字體和顏色來自定義.

## `--project` (`-p`) 選項現在接受任意文件路徑

`--project` 命令行選項過去只接受包含了 `tsconfig.json` 文件的文件夾.
考慮到不同的構建場景, 應該允許 `--project` 指向任何兼容的 JSON 文件.
比如說, 一個用戶可能會希望為 Node 5 編譯 CommonJS 的 ES 2015, 為瀏覽器編譯 AMD 的 ES5.
現在少了這項限制, 用戶可以更容易地直接使用 `tsc` 管理不同的構建目標, 無需再通過一些奇怪的方式, 比如將多個 `tsconfig.json` 文件放在不同的目錄中.

如果參數是一個路徑, 行為保持不變 - 編譯器會嘗試在該目錄下尋找名為 `tsconfig.json` 的文件.

## 允許 tsconfig.json 中的註釋

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

## 支持輸出到 IPC 驅動的文件

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

## 改進了 Visual Studio 2015 中對 `tsconfig.json` 的支持

TypeScript 1.8 允許在任何種類的項目中使用 `tsconfig.json` 文件.
包括 ASP.NET v4 項目, *控制台應用*, 以及 *用 TypeScript 開發的 HTML 應用*.
與此同時, 你可以添加不止一個 `tsconfig.json` 文件, 其中每一個都會作為項目的一部分被構建.
這使得你可以在不使用多個不同項目的情況下為應用的不同部分使用不同的配置.

![展示 Visual Studio 中的 tsconfig.json](https://raw.githubusercontent.com/wiki/Microsoft/TypeScript/images/new-in-typescript/tsconfig-in-vs.png)

當項目中添加了 `tsconfig.json` 文件時, 我們還禁用了項目屬性頁面.
也就是說所有配置的改變必須在 `tsconfig.json` 文件中進行.

### 一些限制

- 如果你添加了一個 `tsconfig.json` 文件, 不在其上下文中的 TypeScript 文件不會被編譯.
- Apache Cordova 應用依然有單個 `tsconfig.json` 文件的限制, 而這個文件必須在根目錄或者 `scripts` 文件夾.
- 多數項目類型中都沒有 `tsconfig.json` 的模板.
