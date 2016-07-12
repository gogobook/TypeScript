# TypeScript 1.7

## 支持 `async`/`await` 編譯到 ES6 (Node v4+)

TypeScript 目前在已經原生支持 ES6 generator 的引擎 (比如 Node v4 及以上版本) 上支持異步函數. 異步函數前置 `async` 關鍵字; `await` 會暫停執行, 直到一個異步函數執行後返回的 promise 被 fulfill 後獲得它的值.

### 例子

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

## 支持同時使用 `--target ES6` 和 `--module`

TypeScript 1.7 將 `ES6` 添加到了 `--module` 選項支持的選項的列表, 當編譯到 `ES6` 時允許指定模組類型. 這讓使用具體運行時中你需要的特性更加靈活.

### 例子

```json
{
    "compilerOptions": {
        "module": "amd",
        "target": "es6"
    }
}
```

## `this` 類型

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

## ES7 冪運算符

TypeScript 1.7 支持將在 ES7/ES2016 中增加的[冪運算符](https://github.com/rwaldron/exponentiation-operator): `**` 和 `**=`. 這些運算符會被轉換為 ES3/ES5 中的 `Math.pow`.

### 舉例

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

## 改進物件字面量解構的檢查

TypeScript 1.7 使物件和陣列字面量解構初始值的檢查更加直觀和自然.

當一個物件字面量通過與之對應的物件解構綁定推斷類型時:

- 物件解構綁定中有默認值的屬性對於物件字面量來說可選.
- 物件解構綁定中的屬性如果在物件字面量中沒有匹配的值, 則該屬性必須有默認值, 並且會被添加到物件字面量的類型中.
- 物件字面量中的屬性必須在物件解構綁定中存在.

當一個陣列字面量通過與之對應的陣列解構綁定推斷類型時:

- 陣列解構綁定中的元素如果在陣列字面量中沒有匹配的值, 則該元素必須有默認值, 並且會被添加到陣列字面量的類型中.

### 舉例

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

## 裝飾器 (decorators) 支持的編譯目標版本增加 ES3

裝飾器現在可以編譯到 ES3. TypeScript 1.7 在 `__decorate` 函數中移除了 ES5 中增加的 `reduceRight`. 相關改動也單行內了對 `Object.getOwnPropertyDescriptor` 和 `Object.defineProperty` 的調用, 並向後兼容, 使 ES5 的輸出可以消除前面提到的 `Object` 方法的重複<sup>[1]</sup>.
