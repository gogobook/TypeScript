# 介紹

當使用外部JavaScript庫或新的宿主API時，你需要一個聲明文件（.d.ts）定義程序庫的shape。
這個手冊包含了寫.d.ts文件的高級概念，並帶有一些例子，告訴你怎麼去寫一個聲明文件。

# 指導與說明

## 流程

最好從程序庫的文檔而不是代碼開始寫.d.ts文件。
這樣保證不會被具體實現所幹擾，而且相比於JS代碼更易讀。
下面的例子會假設你正在參照文檔寫聲明文件。

## 命名空間

當定義接口（例如：「options」物件），你會選擇是否將這些類型放進命名空間裡。
這主要是靠主觀判斷 -- 如果使用的人主要是用這些類型來聲明變量和參數，並且類型命名不會引起命名衝突，則放在全局命名空間裡更好。
如果類型不是被直接使用，或者沒法起一個唯一的名字的話，就使用命名空間來避免與其它類型發生衝突。

## 回調函數

許多JavaScript庫接收一個函數做為參數，之後傳入已知的參數來調用它。
當用這些類型為函數簽名的時候，不要把這些參數標記成可選參數。
正確的思考方式是「(調用者)會提供什麼樣的參數？」，不是「(函數)會使用到什麼樣的參數？」。
TypeScript 0.9.7+不會強制這種可選參數的使用，參數可選的雙向協變可以被外部的linter強制執行。

## 擴展與聲明合併

寫聲明文件的時候，要記住TypeScript擴展現有物件的方式。
你可以選擇用匿名類型或接口類型的方式聲明一個變量：

#### 匿名類型var

```ts
declare let MyPoint: { x: number; y: number; };
```

#### 接口類型var

```ts
interface SomePoint { x: number; y: number; }
declare let MyPoint: SomePoint;
```

從使用者角度來講，它們是相同的，但是SomePoint類型能夠通過接口合併來擴展：

```ts
interface SomePoint { z: number; }
MyPoint.z = 4; // OK
```

是否想讓你的聲明是可擴展的取決於主觀判斷。
通常來講，儘量符合library的意圖。

## 類的分解

TypeScript的類會創建出兩個類型：實例類型，定義了類型的實例具有哪些成員；構造函數類型，定義了類構造函數具有哪些類型。
構造函數類型也被稱做類的靜態部分類型，因為它包含了類的靜態成員。

你可以使用`typeof`關鍵字來拿到類靜態部分類型，在寫聲明文件時，想要把類明確的分解成實例類型和靜態類型時是有用且必要的。

下面是一個例子，從使用者的角度來看，這兩個聲明是等同的：

#### 標準版

```ts
class A {
    static st: string;
    inst: number;
    constructor(m: any) {}
}
```

#### 分解版

```ts
interface A_Static {
    new(m: any): A_Instance;
    st: string;
}
interface A_Instance {
    inst: number;
}
declare let A: A_Static;
```

這裡的利弊如下：

* 標準方式可以使用extends來繼承；分解的類不能。也可能會在未來版本的TypeScript裡做出改變：是否允許任意extends表達式
* 都允許之後為類添加靜態成員(通過合併聲明的方式)
* 分解的類允許增加實例成員，標準版不允許
* 使用分解類的時候，需要為多類型成員起合理的名字

## 命名規則

一般來講，不要給接口加I前綴（比如：IColor）。
因為TypeScript的接口類型概念比C#或Java裡的意義更為廣泛，IFoo命名不利於這個特點。

# 例子

下面進行例子部分。對於每個例子，首先使用*應用示例*，然後是類型聲明。
如果有多個好的聲明表示方法，會列出多個。

## 參數物件

#### 應用示例

```ts
animalFactory.create("dog");
animalFactory.create("giraffe", { name: "ronald" });
animalFactory.create("panda", { name: "bob", height: 400 });
// Invalid: name must be provided if options is given
animalFactory.create("cat", { height: 32 });
```

#### 類型聲明

```ts
namespace animalFactory {
    interface AnimalOptions {
        name: string;
        height?: number;
        weight?: number;
    }
    function create(name: string, animalOptions?: AnimalOptions): Animal;
}
```

## 帶屬性的函數

#### 應用示例

```ts
zooKeeper.workSchedule = "morning";
zooKeeper(giraffeCage);
```

#### 類型聲明

```ts
// Note: Function must precede namespace
function zooKeeper(cage: AnimalCage);
namespace zooKeeper {
    let workSchedule: string;
}
```

## 可以用new調用也可以直接調用的方法

#### 應用示例

```ts
let w = widget(32, 16);
let y = new widget("sprocket");
// w and y are both widgets
w.sprock();
y.sprock();
```

#### 類型聲明

```ts
interface Widget {
    sprock(): void;
}

interface WidgetFactory {
    new(name: string): Widget;
    (width: number, height: number): Widget;
}

declare let widget: WidgetFactory;
```

## 全局或外部的未知代碼庫

#### 應用示例

```ts
// Either
import x = require('zoo');
x.open();
// or
zoo.open();
```

#### 類型聲明

```ts
declare namespace zoo {
  function open(): void;
}

declare module "zoo" {
    export = zoo;
}
```

## 模組裡的單一複雜物件

#### 應用示例

```ts
// Super-chainable library for eagles
import Eagle = require('./eagle');

// Call directly
Eagle('bald').fly();

// Invoke with new
var eddie = new Eagle('Mille');

// Set properties
eddie.kind = 'golden';
```

#### 類型聲明

```ts
interface Eagle {
    (kind: string): Eagle;
    new (kind: string): Eagle;

    kind: string;
    fly(): void
}

declare var Eagle: Eagle;

export = Eagle;
```

## 將模組做為函數

#### 應用示例

```ts
// Common pattern for node modules (e.g. rimraf, debug, request, etc.)
import sayHello = require('say-hello');
sayHello('Travis');
```

#### 類型聲明

```ts
declare module 'say-hello' {
  function sayHello(name: string): void;
  export = sayHello;
}
```

## 回調函數

#### 應用示例

```ts
addLater(3, 4, x => console.log('x = ' + x));
```

#### 類型聲明

```ts
// Note: 'void' return type is preferred here
function addLater(x: number, y: number, callback: (sum: number) => void): void;
```

如果你想看其它模式的實現方式，請在[這裡](https://github.com/Microsoft/TypeScript-Handbook/issues)留言！
我們會儘可能地加到這裡來。
