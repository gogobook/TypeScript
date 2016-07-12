# TypeScript 1.6

## JSX 支持

JSX 是一種可嵌入的類似 XML 的語法. 它將最終被轉換為合法的 JavaScript, 但轉換的語義和具體實現有關. JSX 隨著 React 流行起來, 也出現在其他應用中. TypeScript 1.6 支持 JavaScript 文件中 JSX 的嵌入, 類型檢查, 以及直接編譯為 JavaScript 的選項.

### 新的 `.tsx` 文件擴展名和 `as` 運算符

TypeScript 1.6 引入了新的 `.tsx` 文件擴展名. 這一擴展名一方面允許 TypeScript 文件中的 JSX 語法, 一方面將 `as` 運算符作為默認的類型轉換方式 (避免 JSX 表達式和 TypeScript 前置類型轉換運算符之間的歧義). 比如:

```ts
var x = <any> foo;
// 與如下等價:
var x = foo as any;
```

### 使用 React

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

### 使用其他 JSX 框架

JSX 元素的名稱和屬性是根據 `JSX` 命名空間來檢驗的. 請查看 [JSX](https://github.com/Microsoft/TypeScript/wiki/JSX) 頁面瞭解如何為自己的框架定義 `JSX` 命名空間.

### 編譯輸出

TypeScript 支持兩種 `JSX` 模式: `preserve` (保留) 和 `react`.

- `preserve` 模式將會在輸出中保留 JSX 表達式, 使之後的轉換步驟可以處理. *並且輸出的文件擴展名為 `.jsx`.*
- `react` 模式將會生成 `React.createElement`, 不再需要再通過 JSX 轉換即可運行, 輸出的文件擴展名為 `.js`.

查看 [JSX](https://github.com/Microsoft/TypeScript/wiki/JSX) 頁面瞭解更多 JSX 在 TypeScript 中的使用.

## 交叉類型 (intersection types)

TypeScript 1.6 引入了交叉類型作為聯合類型 (union types) 邏輯上的補充. 聯合類型 `A | B` 表示一個類型為 `A` 或 `B` 的實體, 而交叉類型 `A & B` 表示一個類型同時為 `A` 或 `B` 的實體.

### 例子

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

## 本地類型聲明

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

## 類表達式

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

## 繼承表達式

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

## `abstract` (抽象的) 類和方法

TypeScript 1.6 為類和它們的方法增加了 `abstract` 關鍵字. 一個抽象類允許沒有被實現的方法, 並且不能被構造.

### 例子

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

## 泛型別名

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

## 更嚴格的物件字面量賦值檢查

為了能發現多餘或者錯誤拼寫的屬性, TypeScript 1.6 使用了更嚴格的物件字面量檢查. 確切地說, 在將一個新的物件字面量賦值給一個變量, 或者傳遞給類型非空的參數時, 如果物件字面量的屬性在目標類型中不存在, 則會視為錯誤.

### 例子

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

## ES6 生成器 (generators)

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

## 對 `async` (異步) 函數的試驗性支持

TypeScript 1.6 增加了編譯到 ES6 時對 `async` 函數試驗性的支持. 異步函數會執行一個異步的操作, 在等待的同時不會阻塞程序的正常運行. 這是通過與 ES6 兼容的 `Promise` 實現完成的, 並且會將函數體轉換為支持在等待的異步操作完成時繼續的形式.

由 `async` 標記的函數或方法被稱作_異步函數_. 這個標記告訴了編譯器該函數體需要被轉換, 關鍵字 _await_ 則應該被當做一個一元運算符, 而不是標示符. 一個_異步函數_必須返回類型與 `Promise` 兼容的值. 返回值類型的推斷只能在有一個全局的, 與 ES6 兼容的 `Promise` 類型時使用.

### 例子

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

## 每天發佈新版本

由於並不算嚴格意義上的語言變化<sup>[2]</sup>, 每天的新版本可以使用如下命令安裝獲得:

```sh
npm install -g typescript@next
```

## 對模組解析邏輯的調整

從 1.6 開始, TypeScript 編譯器對於 "commonjs" 的模組解析會使用一套不同的規則. 這些[規則](https://github.com/Microsoft/TypeScript/issues/2338) 嘗試模仿 Node 查找模組的過程. 這就意味著 node 模組可以包含它的類型信息, 並且 TypeScript 編譯器可以找到這些信息. 不過用戶可以通過使用 `--moduleResolution` 命令行選項覆蓋模組解析規則. 支持的值有:

- 'classic' - TypeScript 1.6 以前的編譯器使用的模組解析規則
- 'node' - 與 node 相似的模組解析

## 合併外圍類和接口的聲明

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

## 用戶定義的類型收窄函數

TypeScript 1.6 增加了一個新的在 `if` 語句中收窄變量類型的方式, 作為對 `typeof` 和 `instanceof` 的補充. 用戶定義的類型收窄函數的返回值類型標註形式為 `x is T`, 這裡 `x` 是函數聲明中的形參, `T` 是任何類型. 當一個用戶定義的類型收窄函數在 `if` 語句中被傳入某個變量執行時, 該變量的類型會被收窄到 `T`.

### 例子

```ts
function isCat(a: any): a is Cat {
  return a.name === 'kitty';
}

var x: Cat | Dog;
if(isCat(x)) {
  x.meow(); // 那麼, x 在這個代碼塊內是 Cat 類型
}
```

## `tsconfig.json` 對 `exclude` 屬性的支持

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

## `--init` 命令行選項

在一個目錄中執行 `tsc --init` 可以在該目錄中創建一個包含了默認值的 `tsconfig.json`. 可以通過一併傳遞其他選項來生成初始的 `tsconfig.json`.
