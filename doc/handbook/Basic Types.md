# 介紹

為了讓程序有價值，我們需要能夠處理最簡單的數據單元：數字，字符串，結構體，布林值等。
TypeScript支持與JavaScript幾乎相同的數據類型，此外還提供了實用的枚舉類型方便我們使用。

# 布林值

最基本的數據類型就是簡單的true/false值，在JavaScript和TypeScript裡叫做`boolean`（其它語言中也一樣）。

```ts
let isDone: boolean = false;
```

# 數字

和JavaScript一樣，TypeScript裡的所有數字都是浮點數。
這些浮點數的類型是`number`。
除了支持十進制和十六進制字面量，Typescript還支持ECMAScript 2015中引入的二進制和八進制字面量。

```ts
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;
let binaryLiteral: number = 0b1010;
let octalLiteral: number = 0o744;
```

# 字符串

JavaScript程序的另一項基本操作是處理網頁或服務器端的文本數據。
像其它語言裡一樣，我們使用`string`表示文本數據類型。
和JavaScript一樣，可以使用雙引號（`"`）或單引號（`'`）表示字符串。

```ts
let name: string = "bob";
name = "smith";
```

你還可以使用*模版字符串*，它可以定義多行文本和內嵌表達式。
這種字符串是被反引號包圍（`` ` ``），並且以`${ expr }`這種形式嵌入表達式

```ts
let name: string = `Gene`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ name }.

I'll be ${ age + 1 } years old next month.`;
```

這與下面定義`sentence`的方式效果相同：

```ts
let sentence: string = "Hello, my name is " + name + ".\n\n" +
    "I'll be " + (age + 1) + " years old next month.";
```

# 陣列

TypeScript像JavaScript一樣可以操作陣列元素。
有兩種方式可以定義陣列。
第一種，可以在元素類型後面接上`[]`，表示由此類型元素組成的一個陣列：

```ts
let list: number[] = [1, 2, 3];
```

第二種方式是使用陣列泛型，`Array<元素類型>`：

```ts
let list: Array<number> = [1, 2, 3];
```

# 元組 Tuple

元組類型允許表示一個已知元素數量和類型的陣列，各元素的類型不必相同。
比如，你可以定義一對值分別為`string`和`number`類型的元組。

```ts
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ['hello', 10]; // OK
// Initialize it incorrectly
x = [10, 'hello']; // Error
```

當訪問一個已知索引的元素，會得到正確的類型：

```ts
console.log(x[0].substr(1)); // OK
console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'
```

當訪問一個越界的元素，會使用聯合類型替代：

```ts
x[3] = 'world'; // OK, 字符串可以賦值給(string | number)類型

console.log(x[5].toString()); // OK, 'string' 和 'number' 都有 toString

x[6] = true; // Error, 布爾不是(string | number)類型
```

聯合類型是高級主題，我們會在以後的章節裡討論它。

# 枚舉

`enum`類型是對JavaScript標準數據類型的一個補充。
像C#等其它語言一樣，使用枚舉類型可以為一組數值賦予友好的名字。

```ts
enum Color {Red, Green, Blue};
let c: Color = Color.Green;
```

默認情況下，從`0`開始為元素編號。
你也可以手動的指定成員的數值。
例如，我們將上面的例子改成從`1`開始編號：

```ts
enum Color {Red = 1, Green, Blue};
let c: Color = Color.Green;
```

或者，全部都採用手動賦值：

```ts
enum Color {Red = 1, Green = 2, Blue = 4};
let c: Color = Color.Green;
```

枚舉類型提供的一個便利是你可以由枚舉的值得到它的名字。
例如，我們知道數值為2，但是不確定它映射到Color裡的哪個名字，我們可以查找相應的名字：

```ts
enum Color {Red = 1, Green, Blue};
let colorName: string = Color[2];

alert(colorName);
```

# 任意值

有時候，我們會想要為那些在編程階段還不清楚類型的變量指定一個類型。
這些值可能來自於動態的內容，比如來自用戶輸入或第三方代碼庫。
這種情況下，我們不希望類型檢查器對這些值進行檢查而是直接讓它們通過編譯階段的檢查。
那麼我們可以使用`any`類型來標記這些變量：

```ts
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

在對現有代碼進行改寫的時候，`any`類型是十分有用的，它允許你在編譯時可選擇地包含或移除類型檢查。
你可能認為`Object`有相似的作用，就像它在其它語言中那樣。
但是`Object`類型的變量只是允許你給它賦任意值 -- 但是卻不能夠在它上面調用任意的方法，即便它真的有這些方法：

```ts
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
```

當你只知道一部分數據的類型時，`any`類型也是有用的。
比如，你有一個陣列，它包含了不同的類型的數據：

```ts
let list: any[] = [1, true, "free"];

list[1] = 100;
```

# 空值

某種程度上來說，`void`類型像是與`any`類型相反，它表示沒有任何類型。
當一個函數沒有返回值時，你通常會見到其返回值類型是`void`：

```ts
function warnUser(): void {
    alert("This is my warning message");
}
```

聲明一個`void`類型的變量沒有什麼大用，因為你只能為它賦予`undefined`和`null`：

```ts
let unusable: void = undefined;
```

# 類型斷言

有時候你會遇到這樣的情況，你會比TypeScript更瞭解某個值的詳細信息。
通常這會發生在你清楚地知道一個實體具有比它現有類型更確切的類型。

通過*類型斷言*這種方式可以告訴編譯器，「相信我，我知道自己在幹什麼」。
類型斷言好比其它語言裡的類型轉換，但是不進行特殊的數據檢查和解構。
它沒有運行時的影響，只是在編譯階段起作用。
TypeScript會假設你，程序員，已經進行了必須的檢查。

類型斷言有兩種形式。
其一是「尖括號」語法：

```ts
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;
```

另一個為`as`語法：

```ts
let someValue: any = "this is a string";

let strLength: number = (someValue as string).length;
```

兩種形式是等價的。
至於使用哪個大多數情況下是憑個人喜好；然而，當你在TypeScript裡使用JSX時，只有`as`語法斷言是被允許地。

# 關於`let`

你可能已經注意到了，我們使用`let`關鍵字來代替大家所熟悉的JavaScript關鍵字`var`。
`let`關鍵字是JavaScript的一個新概念，TypeScript實現了它。
我們會在以後詳細介紹它，很多常見的問題都可以通過使用`let`來解決，所以儘可能地使用`let`來代替`var`吧。
