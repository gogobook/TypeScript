# 變量聲明

`let`和`const`是JavaScript裡相對較新的變量聲明方式。
像我們之前提到過的，`let`在很多方面與`var`是相似的，但是可以幫助大家避免在JavaScript裡常見一些問題。
`const`是對`let`的一個增強，它能阻止對一個變量再次賦值。

因為TypeScript是JavaScript的超集，所以它本身就支持`let`和`const`。
下面我們會詳細說明這些新的聲明方式以及為什麼推薦使用它們來代替`var`。

如果你之前使用JavaScript時沒有特別在意，那麼這節內容會喚起你的回憶。
如果你已經對`var`聲明的怪異之處瞭如指掌，那麼你可以輕鬆地略過這節。

# `var` 聲明

一直以來我們都是通過`var`關鍵字定義JavaScript變量。

```ts
var a = 10;
```

大家都能理解，這裡定義了一個名為`a`值為`10`的變量。

我們也可以在函數內部定義變量：

```ts
function f() {
    var message = "Hello, world!";

    return message;
}
```

並且我們也可以在其它函數內部訪問相同的變量。

```ts
function f() {
    var a = 10;
    return function g() {
        var b = a + 1;
        return b;
    }
}

var g = f();
g(); // returns 11;
```

上面的例子裡，`g`可以獲取到`f`函數裡定義的`a`變量。
每當`g`被調用時，它都可以訪問到`f`裡的`a`變量。
即使當`g`在`f`已經執行完後才被調用，它仍然可以訪問及修改`a`。

```ts
function f() {
    var a = 1;

    a = 2;
    var b = g();
    a = 3;

    return b;

    function g() {
        return a;
    }
}

f(); // returns 2
```

## 作用域規則

對於熟悉其它語言的人來說，`var`聲明有些奇怪的作用域規則。
看下面的例子：

```ts
function f(shouldInitialize: boolean) {
    if (shouldInitialize) {
        var x = 10;
    }

    return x;
}

f(true);  // returns '10'
f(false); // returns 'undefined'
```

有些讀者可能要多看幾遍這個例子。
變量`x`是定義在*`if`語句裡面*，但是我們卻可以在語句的外面訪問它。
這是因為`var`聲明可以在包含它的函數，模組，命名空間或全局作用域內部任何位置被訪問（我們後面會詳細介紹），包含它的代碼塊對此沒有什麼影響。
有些人稱此為*`var`作用域*或*函數作用域*。
函數參數也使用函數作用域。

這些作用域規則可能會引發一些錯誤。
其中之一就是，多次聲明同一個變量並不會報錯：

```ts
function sumMatrix(matrix: number[][]) {
    var sum = 0;
    for (var i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (var i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```

這裡很容易看出一些問題，裡層的`for`循環會覆蓋變量`i`，因為所有`i`都引用相同的函數作用域內的變量。
有經驗的開發者們很清楚，這些問題可能在代碼審查時漏掉，引發無窮的麻煩。

## 變量獲取怪異之處

快速的猜一下下面的代碼會返回什麼：

```ts
for (var i = 0; i < 10; i++) {
    setTimeout(function() {console.log(i); }, 100 * i);
}
```

介紹一下，`setTimeout`會在若干毫秒的延時後執行一個函數（等待其它代碼執行完畢）。

好吧，看一下結果：

```text
10
10
10
10
10
10
10
10
10
10
```

很多JavaScript程序員對這種行為已經很熟悉了，但如果你很不解，你並不是一個人。
大多數人期望輸出結果是這樣：

```text
0
1
2
3
4
5
6
7
8
9
```

還記得我們上面講的變量獲取嗎？

> 每當`g`被調用時，它都可以訪問到`f`裡的`a`變量。

讓我們花點時間考慮在這個上下文裡的情況。
`setTimeout`在若干毫秒後執行一個函數，並且是在`for`循環結束後。
`for`循環結束後，`i`的值為`10`。
所以當函數被調用的時候，它會打印出`10`！

一個通常的解決方法是使用立即執行的函數表達式（IIFE）來捕獲每次迭代時`i`的值：

```ts
for (var i = 0; i < 10; i++) {
    // capture the current state of 'i'
    // by invoking a function with its current value
    (function(i) {
        setTimeout(function() { console.log(i); }, 100 * i);
    })(i);
}
```

這種奇怪的形式我們已經司空見慣了。
參數`i`會覆蓋`for`循環裡的`i`，但是因為我們起了同樣的名字，所以我們不用怎麼改`for`循環體裡的代碼。

# `let` 聲明

現在你已經知道了`var`存在一些問題，這恰好說明了為什麼用`let`語句來聲明變量。
除了名字不同外，`let`與`var`的寫法一致。

```ts
let hello = "Hello!";
```

主要的區別不在語法上，而是語義，我們接下來會深入研究。

## 塊作用域

當用`let`聲明一個變量，它使用的是*詞法作用域*或*塊作用域*。
不同於使用`var`聲明的變量那樣可以在包含它們的函數外訪問，塊作用域變量在包含它們的塊或`for`循環之外是不能訪問的。

```ts
function f(input: boolean) {
    let a = 100;

    if (input) {
        // Still okay to reference 'a'
        let b = a + 1;
        return b;
    }

    // Error: 'b' doesn't exist here
    return b;
}
```

這裡我們定義了2個變量`a`和`b`。
`a`的作用域是`f`函數體內，而`b`的作用域是`if`語句塊裡。

在`catch`語句裡聲明的變量也具有同樣的作用域規則。

```ts
try {
    throw "oh no!";
}
catch (e) {
    console.log("Oh well.");
}

// Error: 'e' doesn't exist here
console.log(e);
```

擁有塊級作用域的變量的另一個特點是，它們不能在被聲明之前讀或寫。
雖然這些變量始終「存在」於它們的作用域裡，但在直到聲明它的代碼之前的區域都屬於*時間死區*。
它只是用來說明我們不能在`let`語句之前訪問它們，幸運的是TypeScript可以告訴我們這些信息。

```ts
a++; // illegal to use 'a' before it's declared;
let a;
```

注意一點，我們仍然可以在一個擁有塊作用域變量被聲明前*獲取*它。
只是我們不能在變量聲明前去調用那個函數。
如果生成代碼目標為ES2015，現代的運行時會拋出一個錯誤；然而，現今TypeScript是不會報錯的。

```ts
function foo() {
    // okay to capture 'a'
    return a;
}

// 不能在'a'被聲明前調用'foo'
// 運行時應該拋出錯誤
foo();

let a;
```

關於*時間死區*的更多信息，查看這裡[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let).

## 重定義及屏蔽

我們提過使用`var`聲明時，它不在乎你聲明多少次；你只會得到1個。

```ts
function f(x) {
    var x;
    var x;

    if (true) {
        var x;
    }
}
```

在上面的例子裡，所有`x`的聲明實際上都引用一個*相同*的`x`，並且這是完全有效的代碼。
這經常會成為bug的來源。
好的是，`let`聲明就不會這麼寬鬆了。

```ts
let x = 10;
let x = 20; // 錯誤，不能在1個作用域裡多次聲明`x`
```

並不是要求兩個均是塊級作用域的聲明TypeScript才會給出一個錯誤的警告。

```ts
function f(x) {
    let x = 100; // error: interferes with parameter declaration
}

function g() {
    let x = 100;
    var x = 100; // error: can't have both declarations of 'x'
}
```

並不是說塊級作用域變量不能在函數作用域內聲明。
而是塊級作用域變量需要在不用的塊裡聲明。

```ts
function f(condition, x) {
    if (condition) {
        let x = 100;
        return x;
    }

    return x;
}

f(false, 0); // returns 0
f(true, 0);  // returns 100
```

在一個嵌套作用域裡引入一個新名字的行為稱做*屏蔽*。
它是一把雙刃劍，它可能會不小心地引入新問題，同時也可能會解決一些錯誤。
例如，假設我們現在用`let`重寫之前的`sumMatrix`函數。

```ts
function sumMatrix(matrix: number[][]) {
    let sum = 0;
    for (let i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (let i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```

這個版本的循環能得到正確的結果，因為內層循環的`i`可以屏蔽掉外層循環的`i`。

*通常*來講應該避免使用屏蔽，因為我們需要寫出清晰的代碼。
同時也有些場景適合利用它，你需要好好打算一下。

## 塊級作用域變量的獲取

在我們最初談及獲取用`var`聲明的變量時，我們簡略地探究了一下在獲取到了變量之後它的行為是怎樣的。
直觀地講，每次進入一個作用域時，它創建了一個變量的*環境*。
就算作用域內代碼已經執行完畢，這個環境與其捕獲的變量依然存在。

```ts
function theCityThatAlwaysSleeps() {
    let getCity;

    if (true) {
        let city = "Seattle";
        getCity = function() {
            return city;
        }
    }

    return getCity();
}
```

因為我們已經在`city`的環境裡獲取到了`city`，所以就算`if`語句執行結束後我們仍然可以訪問它。

回想一下前面`setTimeout`的例子，我們最後需要使用立即執行的函數表達式來獲取每次`for`循環迭代裡的狀態。
實際上，我們做的是為獲取到的變量創建了一個新的變量環境。
這樣做挺痛苦的，但是幸運的是，你不必在TypeScript裡這樣做了。

當`let`聲明出現在循環體裡時擁有完全不同的行為。
不僅是在循環裡引入了一個新的變量環境，而是針對*每次迭代*都會創建這樣一個新作用域。
這就是我們在使用立即執行的函數表達式時做的事，所以在`setTimeout`例子裡我們僅使用`let`聲明就可以了。

```ts
for (let i = 0; i < 10 ; i++) {
    setTimeout(function() {console.log(i); }, 100 * i);
}
```

會輸出與預料一致的結果：

```text
0
1
2
3
4
5
6
7
8
9
```

# `const` 聲明

`const` 聲明是聲明變量的另一種方式。

```ts
const numLivesForCat = 9;
```

它們與`let`聲明相似，但是就像它的名字所表達的，它們被賦值後不能再改變。
換句話說，它們擁有與`let`相同的作用域規則，但是不能對它們重新賦值。

這很好理解，它們引用的值是*不可變的*。

```ts
const numLivesForCat = 9;
const kitty = {
    name: "Aurora",
    numLives: numLivesForCat,
}

// Error
kitty = {
    name: "Danielle",
    numLives: numLivesForCat
};

// all "okay"
kitty.name = "Rory";
kitty.name = "Kitty";
kitty.name = "Cat";
kitty.numLives--;
```

除非你使用特殊的方法去避免，實際上`const`變量的內部狀態是可修改的。

# `let` vs. `const`

現在我們有兩種作用域相似的聲明方式，我們自然會問到底應該使用哪個。
與大多數泛泛的問題一樣，答案是：依情況而定。

使用[最小特權原則](https://en.wikipedia.org/wiki/Principle_of_least_privilege)，所有變量除了你計划去修改的都應該使用`const`。
基本原則就是如果一個變量不需要對它寫入，那麼其它使用這些代碼的人也不能夠寫入它們，並且要思考為什麼會需要對這些變量重新賦值。
使用`const`也可以讓我們更容易的推測數據的流動。

另一方面，用戶很喜歡`let`的簡潔性。
這個手冊大部分地方都使用了`let`。

跟據你的自己判斷，如果合適的話，與團隊成員商議一下。

# 解構

Another TypeScript已經可以解析其它 ECMAScript 2015 特性了。
完整列表請參見 [the article on the Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)。
本章，我們將給出一個簡短的概述。

## 解構陣列

最簡單的解構莫過於陣列的解構賦值了：

```ts
let input = [1, 2];
let [first, second] = input;
console.log(first); // outputs 1
console.log(second); // outputs 2
```

這創建了2個命名變量 `first` 和 `second`。
相當於使用了索引，但更為方便：

```ts
first = input[0];
second = input[1];
```

解構作用於已聲明的變量會更好：

```ts
// swap variables
[first, second] = [second, first];
```

作用於函數參數：

```ts
function f([first, second]: [number, number]) {
    console.log(first);
    console.log(second);
}
f(input);
```

你可以使用`...name`語法創建一個剩餘變量列表：

```ts
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]
```

當然，由於是JavaScript, 你可以忽略你不關心的尾隨元素：

```ts
let [first] = [1, 2, 3, 4];
console.log(first); // outputs 1
```

或其它元素：

```ts
let [, second, , fourth] = [1, 2, 3, 4];
```

## 物件解構

你也可以解構物件：

```ts
let o = {
    a: "foo",
    b: 12,
    c: "bar"
}
let {a, b} = o;
```

這通過 `o.a` and `o.b` 創建了 `a` 和 `b` 。
注意，如果你不需要 `c` 你可以忽略它。

就像陣列解構，你可以用沒有聲明的賦值：

```ts
({a, b} = {a: "baz", b: 101});
```

注意，我們需要用括號將它括起來，因為Javascript通常會將以 `{` 起始的語句解析為一個塊。

### 屬性重命名

你也可以給屬性以不同的名字：

```ts
let {a: newName1, b: newName2} = o;
```

這裡的語法開始變得混亂。
你可以將 `a: newName1` 讀做 "`a` 作為 `newName1`"。
方向是從左到右，好像你寫成了以下樣子：

```ts
let newName1 = o.a;
let newName2 = o.b;
```

令人困惑的是，這裡的冒號*不是*指示類型的。
如果你想指定它的類型， 仍然需要在其後寫上完整的模式。

```ts
let {a, b}: {a: string, b: number} = o;
```

### 默認值

默認值可以讓你在屬性為 undefined 時使用預設值：

```ts
function keepWholeObject(wholeObject: {a: string, b?: number}) {
    let {a, b = 1001} = wholeObject;
}
```

現在，即使 `b` 為 undefined ， `keepWholeObject` 函數的變量 `wholeObject` 的屬性 `a` 和 `b` 都會有值。

## 函數聲明

解構也能用於函數聲明。
看以下簡單的情況：

```ts
type C = {a: string, b?: number}
function f({a, b}: C): void {
    // ...
}
```

但是，通常情況下更多的是指定默認值，解構默認值有些棘手。
首先，你需要知道在設置默認值之前設置其類型。

```ts
function f({a, b} = {a: "", b: 0}): void {
    // ...
}
f(); // ok, default to {a: "", b: 0}
```

其次，你需要知道在解構屬性上給予一個默認或可選的屬性用來替換主初始化列表。
要知道 `C` 的定義有一個 `b` 可選屬性：

```ts
function f({a, b = 0} = {a: ""}): void {
    // ...
}
f({a: "yes"}) // ok, default b = 0
f() // ok, default to {a: ""}, which then defaults b = 0
f({}) // error, 'a' is required if you supply an argument
```

要小心使用解構。
從前面的例子可以看出，就算是最簡單的解構也會有很多問題。
尤其當存在深層嵌套解構的時候，就算這時沒有堆疊在一起的重命名，默認值和類型註解，也是令人難以理解的。
解構表達式要儘量保持小而簡單。
你自己也可以直接使用解構將會生成的賦值表達式。
