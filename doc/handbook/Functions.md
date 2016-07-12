# 介紹

函數是JavaScript應用程序的基礎。
它幫助你實現抽象層，模擬類，信息隱藏和模組。
在TypeScript裡，雖然已經支持類，命名空間和模組，但函數仍然是主要的定義*行為*的地方。
TypeScript為JavaScript函數添加了額外的功能，讓我們可以更容易地使用。

# 函數

和JavaScript一樣，TypeScript函數可以創建有名字的函數和匿名函數。
你可以隨意選擇適合應用程序的方式，不論是定義一系列API函數還是只使用一次的函數。

通過下面的例子可以迅速回想起這兩種JavaScript中的函數：

```ts
// Named function
function add(x, y) {
    return x + y;
}

// Anonymous function
let myAdd = function(x, y) { return x + y; };
```

在JavaScript裡，函數可以使用函數體外部的變量。
當函數這麼做時，我們說它『捕獲』了這些變量。
至於為什麼可以這樣做以及其中的利弊超出了本文的範圍，但是深刻理解這個機制對學習JavaScript和TypeScript會很有幫助。

```ts
let z = 100;

function addToZ(x, y) {
    return x + y + z;
}
```

# 函數類型

## 為函數定義類型

讓我們為上面那個函數添加類型：

```ts
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x+y; };
```

我們可以給每個參數添加類型之後再為函數本身添加返回值類型。
TypeScript能夠根據返回語句自動推斷出返回值類型，因此我們通常省略它。

## 書寫完整函數類型

現在我們已經為函數指定了類型，下面讓我們寫出函數的完整類型。

```ts
let myAdd: (x:number, y:number)=>number =
    function(x: number, y: number): number { return x+y; };
```

函數類型包含兩部分：參數類型和返回值類型。
當寫出完整函數類型的時候，這兩部分都是需要的。
我們以參數列表的形式寫出參數類型，為每個參數指定一個名字和類型。
這個名字只是為了增加可讀性。
我們也可以這麼寫：

```ts
let myAdd: (baseValue:number, increment:number) => number =
    function(x: number, y: number): number { return x + y; };
```

只要參數類型是匹配的，那麼就認為它是有效的函數類型，而不在乎參數名是否正確。

第二部分是返回值類型。
對於返回值，我們在函數和返回值類型之前使用(`=>`)符號，使之清晰明了。
如之前提到的，返回值類型是函數類型的必要部分，如果函數沒有返回任何值，你也必須指定返回值類型為`void`而不能留空。

函數的類型只是由參數類型和返回值組成的。
函數中使用的捕獲變量不會體現在類型裡。
實際上，這些變量是函數的隱藏狀態並不是組成API的一部分。

## 推斷類型

嘗試這個例子的時候，你會發現如果你在賦值語句的一邊指定了類型但是另一邊沒有類型的話，TypeScript編譯器會自動識別出類型：

```ts
// myAdd has the full function type
let myAdd = function(x: number, y: number): number { return x + y; };

// The parameters `x` and `y` have the type number
let myAdd: (baseValue:number, increment:number) => number =
    function(x, y) { return x + y; };
```

這叫做「按上下文歸類」，是類型推論的一種。
它幫助我們更好地為程序指定類型。

# 可選參數和默認參數

TypeScript裡的每個函數參數都是必須的。
這不是指不能傳遞`null`或`undefined`作為參數，而是說編譯器檢查用戶是否為每個參數都傳入了值。
編譯器還會假設只有這些參數會被傳遞進函數。
簡短地說，傳遞給一個函數的參數個數必須與函數期望的參數個數一致。

```ts
function buildName(firstName: string, lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // error, too few parameters
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // ah, just right
```

JavaScript裡，每個參數都是可選的，可傳可不傳。
沒傳參的時候，它的值就是undefined。
在TypeScript裡我們可以在參數名旁使用`?`實現可選參數的功能。
比如，我們想讓last name是可選的：

```ts
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");  // works correctly now
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");  // ah, just right
```

可選參數必須跟在必須參數後面。
如果上例我們想讓first name是可選的，那麼就必須調整它們的位置，把first name放在後面。

在TypeScript裡，我們也可以為參數提供一個默認值當用戶沒有傳遞這個參數或傳遞的值是`undefined`時。
它們叫做有默認初始化值的參數。
讓我們修改上例，把last name的默認值設置為`"Smith"`。

```ts
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // works correctly now, returns "Bob Smith"
let result2 = buildName("Bob", undefined);       // still works, also returns "Bob Smith"
let result3 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result4 = buildName("Bob", "Adams");         // ah, just right
```

在所有必須參數後面的帶默認初始化的參數都是可選的，與可選參數一樣，在調用函數的時候可以省略。
也就是說可選參數與末尾的默認參數共享參數類型。

```ts
function buildName(firstName: string, lastName?: string) {
    // ...
}
```

和

```ts
function buildName(firstName: string, lastName = "Smith") {
    // ...
}
```

共享同樣的類型`(firstName: string, lastName?: string) => string`。
默認參數的默認值消失了，只保留了它是一個可選參數的信息。

與普通可選參數不同的是，帶默認值的參數不需要放在必須參數的後面。
如果帶默認值的參數出現在必須參數前面，用戶必須明確的傳入`undefined`值來獲得默認值。
例如，我們重寫最後一個例子，讓`firstName`是帶默認值的參數：

```ts
function buildName(firstName = "Will", lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // error, too few parameters
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // okay and returns "Bob Adams"
let result4 = buildName(undefined, "Adams");     // okay and returns "Will Adams"
```

# 剩餘參數

必要參數，默認參數和可選參數有個共同點：它們表示某一個參數。
有時，你想同時操作多個參數，或者你並不知道會有多少參數傳遞進來。
在JavaScript裡，你可以使用`arguments`來訪問所有傳入的參數。

在TypeScript裡，你可以把所有參數收集到一個變量裡：

```ts
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

剩餘參數會被當做個數不限的可選參數。
可以一個都沒有，同樣也可以有任意個。
編譯器創建參數陣列，名字是你在省略號（`...`）後面給定的名字，你可以在函數體內使用這個陣列。

這個省略號也會在帶有剩餘參數的函數類型定義上使用到：

```ts
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let buildNameFun: (fname: string, ...rest: string[]) => string = buildName;
```

# Lambda表達式和使用`this`

JavaScript裡`this`的工作機制對JavaScript程序員來說已經是老生常談了。
的確，學會如何使用它絕對是JavaScript編程中的一件大事。
由於TypeScript是JavaScript的超集，TypeScript程序員也需要弄清`this`工作機制並且當有bug的時候能夠找出錯誤所在。
`this`的工作機制可以單獨寫一本書了，並確已有人這麼做了。在這裡，我們只介紹一些基礎知識。

JavaScript裡，`this`的值在函數被調用的時候才會指定。
這是個既強大又靈活的特點，但是你需要花點時間弄清楚函數調用的上下文是什麼。
眾所周知這不是一件很簡單的事，特別是函數當做回調函數使用的時候。

下面看一個例子：

```ts
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        return function() {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

如果我們運行這個程序，會發現它並沒有彈出對話框而是報錯了。
因為`createCardPicker`返回的函數裡的`this`被設置成了`window`而不是`deck`物件。
當你調用`cardPicker()`時會發生這種情況。這裡沒有對`this`進行動態綁定因此為window。（注意在嚴格模式下，會是undefined而不是window）。

為瞭解決這個問題，我們可以在函數被返回時就綁好正確的`this`。
這樣的話，無論之後怎麼使用它，都會引用綁定的『deck'物件。

我們把函數表達式變為使用lambda表達式（ () => {} ）。
這樣就會在函數創建的時候就指定了『this'值，而不是在函數調用的時候。

```ts
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        // Notice: the line below is now a lambda, allowing us to capture `this` earlier
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

為瞭解更多關於`this`的信息，請閱讀Yahuda Katz的[Understanding JavaScript Function Invocation and "this"](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)。

# 重載

JavaScript本身是個動態語言。
JavaScript裡函數根據傳入不同的參數而返回不同類型的數據是很常見的。

```ts
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

`pickCard`方法根據傳入參數的不同會返回兩種不同的類型。
如果傳入的是代表紙牌的物件，函數作用是從中抓一張牌。
如果用戶想抓牌，我們告訴他抓到了什麼牌。
但是這怎麼在類型系統裡表示呢。

方法是為同一個函數提供多個函數類型定義來進行函數重載。
編譯器會根據這個列表去處理函數的調用。
下面我們來重載`pickCard`函數。

```ts
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

這樣改變後，重載的`pickCard`函數在調用的時候會進行正確的類型檢查。

為了讓編譯器能夠選擇正確的檢查類型，它與JavaScript裡的處理流程相似。
它查找重載列表，嘗試使用第一個重載定義。
如果匹配的話就使用這個。
因此，在定義重載的時候，一定要把最精確的定義放在最前面。

注意，`function pickCard(x): any`並不是重載列表的一部分，因此這裡只有兩個重載：一個是接收物件另一個接收數字。
以其它參數調用`pickCard`會產生錯誤。
