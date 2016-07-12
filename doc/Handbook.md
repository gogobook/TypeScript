# TypeScript手冊

翻譯：鐘勝平（[@zhongsp](https://github.com/zhongsp)）

2015年4月

TypeScript是微軟公司的註冊商標.

## 目錄

* [基本類型](#1)
  * [Boolean](#1.1)
  * [Number](#1.2)
  * [String](#1.3)
  * [Array](#1.4)
  * [Enum](#1.5)
  * [Any](#1.6)
  * [Void](#1.7)
* [接口](#2)
  * [接口初探](#2.1)
  * [可選屬性](#2.2)
  * [函數類型](#2.3)
  * [陣列類型](#2.4)
  * [類類型](#2.5)
  * [擴展接口](#2.6)
  * [混合類型](#2.7)
* [類](#3)
  * [類](#3.1)
  * [繼承](#3.2)
  * [公共，私有與保護的修飾符](#3.3)
  * [存取器](#3.4)
  * [靜態屬性](#3.5)
  * [高級技巧](#3.6)
* [命名空間和模組](#4)
  * [第一步](#第一步)
  * [使用命名空間](#使用命名空間)
  * [分割成多文件](#分割成多文件)
  * [模組](#4.2)
  * [Export =](#4.3)
  * [別名](#4.4)
  * [可選模組的加載與其它高級加載的場景](#4.5)
  * [使用其它JavaScript庫](#4.6)
  * [命名空間和模組的陷阱](#4.7)
* [函數](#5)
  * [函數](#5.1)
  * [函數類型](#5.2)
  * [可選參數和默認參數](#5.3)
  * [剩餘參數](#5.4)
  * [Lambda表達式和使用『this'](#5.5)
  * [重載](#5.6)
* [泛型](#6)
  * [Hello World泛型](#6.1)
  * [使用泛型變量](#6.2)
  * [泛型類型](#6.3)
  * [泛型類](#6.4)
  * [泛型約束](#6.5)
* [常見錯誤](#7)
  * [常見疑難問題](#7.1)
* [Mixins](#8)
  * [Mixin 例子](#8.1)
  * [理解這個例子](#8.2)
* [聲明合併](#9)
  * [基礎概念](#9.1)
  * [合併接口](#9.2)
  * [合併模組](#9.3)
  * [模組與類和函數和枚舉類型合併](#9.4)
  * [無效的合併](#9.5)
* [類型推論](#10)
  * [基礎](#10.1)
  * [最佳通用類型](#10.2)
  * [上下文類型](#10.3)
* [類型兼容性](#11)
  * [開始](#11.1)
  * [比較函數](#11.2)
  * [枚舉類型](#11.3)
  * [類](#11.4)
  * [泛型](#11.5)
  * [高級主題](#11.6)
* [書寫.d.ts文件](#12)
  * [指導與說明](#12.1)
  * [例子](#12.2)

## <a name="1"></a>基本類型

為了讓程序有價值，我們需要能夠處理最簡單的數據單元：數字，字符串，結構體，布林值等。TypeScript支持與JavaScript幾乎相同的數據類型，此外還提供了實用的枚舉類型方便我們使用。

### <a name="1.1"></a>布林值

最基本的數據類型就是簡單的true/false值，在JavaScript和TypeScript裡叫做`boolean`（其它語言中也一樣）。

```typescript
var isDone: boolean = false;
```

### <a name="1.2"></a>Number

和JavaScript一樣，TypeScript裡的所有數字都是浮點數。這些浮點數的類型是`number`。

```typescript
var height: number = 6;
```

### <a name="1.3"></a>字符串

JavaScript程序的另一項基本操作是處理網頁或服務器端的文本數據。
像其它語言裡一樣，我們使用`string`表示文本數據類型。
和JavaScript一樣，可以使用雙引號（`"`）或單引號（`'`）表示字符串。

```typescript
var name: string = "bob";
name = "smith";
```

你還可以使用*模版字符串*，它可以定義多行文本和內嵌表達式。

```TypeScript
var name: string = `Gene`;
var age: number = 37;
var sentence: string = `Hello, my name is ${ name }.

I'll be ${ age + 1 } years old next month`.;
```

這與下面定義`sentence`的方式效果相同：

```TypeScript
var sentence: string = "Hello, my name is " + ".\n\n" + "I'll be " + (age + 1) + " years old next month.";
```

### <a name="1.4"></a>Array

TypeScript像JavaScript一樣可以操作陣列元素。有兩種方式可以定義陣列。第一種，可以在元素類型後面接上`[]`，表示由此類型元素組成的一個陣列：

```typescript
var list: number[] = [1, 2 ,3];
```

第二種方式是使用陣列泛型，`Array<元素類型>`：

```typescript
var list: Array<number> = [1, 2, 3];
```

### <a name="1.5"></a>枚舉

`enum`類型是對JavaScript標準數據類型的一個補充。像C#等其它語言一樣，使用枚舉類型可以為一組數值賦予友好的名字。

```typescript
enum Color {Red, Green, Blue};
var c: Color = Color.Green;
```

默認情況下，從`0`開始為元素編號。你也可以手動的指定成員的數值。例如，我們將上面的例子改成從`1`開始編號：

```typescript
enum Color {Red = 1, Green, Blue};
var c: Color = Color.Green;
```

或者，全部都採用手動賦值：

```typescript
enum Color {Red = 1, Green = 2, Blue = 4};
var c: Color = Color.Green;
```

枚舉類型提供的一個便利是你可以由枚舉的值得到它的名字。例如，我們知道數值為2，但是不確定它映射到Color裡的哪個名字，我們可以查找相應的名字：

```typescript
enum Color {Red = 1, Green, Blue};
var colorName: string = Color[2];

alert(colorName);  // Green
```

### <a name="1.6"></a>Any

有時，我們可能會想要給在編寫程序時並不清楚的變量指定其類型。這些值可能來自於動態的內容，比如來自用戶或第三方代碼庫。這種情況下，我們不希望類型檢查器對這些值進行檢查或者說讓它們直接通過編譯階段的檢查。那麼我們可以使用`any`類型來標記這些變量：

```typescript
var notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay，definitely a boolean
```

在對現有代碼進行改寫的時候，`any`類型是十分有用的，它允許你在編譯時可選擇地包含或移除類型檢查。

當你只知道數據的類型的一部分時，`any`類型也是有用的。比如，你有一個陣列，它包含了不同的數據類型：

```typescript
var list: any[] = [1, true, "free"];

list[1] = 100;
```

### <a name="1.7"></a>Void

某種程度上來說，`void`類型與`any`類型是相反的，它表示沒有任何類型。當一個函數沒有返回值時，你通常會見到其返回值類型是`void`：

```typescript
function warnUser(): void {
    alert("This is my warning message");
}
```

## <a name="2"></a>接口

TypeScript的核心原則之一是對值所具有的`外形`進行類型檢查。它有時被稱做「鴨子類型」或「結構性子類型」。在TypeScript裡，接口的作用就是為這些類型命名和為你的代碼或第三方代碼定義契約。

### <a name="2.1"></a>接口初探

下面通過一個簡單示例來觀察接口是如何工作的：

```TypeScript
function printLabel(labelledObj: { label: string }) {
  console.log(labelledObj.label);
}

var myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```

類型檢查器會查看`printLabel`的調用。`printLabel`有一個參數，並要求這個物件參數有一個名為`label`類型為`string`的屬性。需要注意的是，我們傳入的物件參數實際上會包含很多屬性，但是編譯器只會檢查那些必需的屬性是否存在，並且其類型是否匹配。

下面我們重寫上面的例子，這次使用接口來描述：必須包含一個`label`屬性且類型為`string`：

```typescript
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

`LabelledValue`接口就好比一個名字，用來描述上面例子裡的要求。它代表了有一個`label`屬性且類型為`string`的物件。需要注意的是，我們在這裡並不能像在其它語言裡一樣，說傳給`printLabel`的物件實現了這個接口。我們只會去關注值的外形。只要傳入的物件滿足上面提到的必要條件，那麼它就是被允許的。

還有一點值得提的是，類型檢查器不會去檢查屬性的順序，只要相應的屬性存在並且類型也是對的就可以。

### <a name="2.2"></a>可選屬性

接口裡的屬性不全都是必需的。
有些是只在某些條件下存在，或者根本不存在。
可選屬性在應用「option bags」模式時很常用，即給函數傳入的參數物件中只有部分屬性賦值了。

下面是應用了「option bags」的例子：

```TypeScript
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});
```

帶有可選屬性的接口與普通的接口定義差不多，只是在可選屬性名字定義的後面加一個`?`符號。

可選屬性的好處之一是可以對可能存在的屬性進行預定義，好處之二是可以捕獲使用了不存在的屬性時的錯誤。
比如，我們故意將拼寫錯誤的屬性名傳入`createSquare`，就會得到一個錯誤提示。

```typescript
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.collor;  // 類型檢查器會查出這個拼寫錯誤
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});
```

### <a name="2.3"></a>函數類型

接口能夠描述JavaScript中物件擁有的各種各樣的外形。
除了描述帶有屬性的普通物件外，接口也可以描述函數類型。

為了使用接口表示函數類型，我們需要給接口定義一個調用簽名。它就像是一個只有參數列表和返回值類型的函數定義。

```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```

這樣定義後，我們可以像使用其它接口一樣使用這個函數類型的接口。下例展示了如何創建一個函數類型的變量，並將一個同類型的函數賦值給這個變量。

```typescript
var mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  var result = source.search(subString);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```

對於函數類型的類型檢查來說，函數的參數名不需要與接口裡定義的名字相匹配。
比如，我們使用下面的代碼重寫上面的例子：

```typescript
var mySearch: SearchFunc;
mySearch = function(src: string, sub: string) {
  var result = src.search(sub);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```

函數的參數會逐個進行檢查，要求對應位置上的參數類型是兼容的。
函數的返回值類型是通過其返回值推斷出來的（此例是`false`和`true`）。
如果讓這個函數返回數字或字符串，類型檢查器會警告我們函數的返回值類型與`SearchFunc`接口中的定義不匹配。

### <a name="2.4"></a>陣列類型

與使用接口描述函數類型差不多，我們也可以描述陣列類型。
陣列類型具有一個`index`類型表示索引的類型，還有一個相應的返回值類型表示通過索引得到的元素的類型。

```typescript
interface StringArray {
  [index: number]: string;
}

var myArray: StringArray;
myArray = ["Bob", "Fred"];
```

支持兩種索引類型：string和number。陣列可以同時使用這兩種索引類型，但是有一個限制，數字索引返回值的類型必須是字符串索引返回值的類型的子類型。

索引簽名能夠很好的描述陣列和`dictionary`模式，它們也要求所有屬性要與返回值類型相匹配。下面的例子裡，length屬性與一般的索引返回值類型不匹配，所以類型檢查器給出一個錯誤提示：

```typescript
interface Dictionary {
  [index: string]: string;
  length: number;    // error, the type of 'length' is not a subtype of the indexer
}
```

### <a name="2.5"></a>類類型

#### <a name="2.5.1"></a>實現接口

與C#或Java裡接口的基本作用一樣，TypeScript也能夠用它來明確的強制一個類去符合某種契約。

```typescript
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

你也可以在接口中描述一個方法，在類裡實現它，如同下面的`setTime`方法一樣：

```typescript
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface  {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

接口描述了類的公共部分，而不是公共和私有兩部分。它不會幫你檢查類是否具有某些私有成員。

#### <a name="2.5.2"></a>類靜態部分與實例部分的區別

當你操作類和接口的時候，你要知道類是具有兩個類型的：靜態部分的類型和實例的類型。你會注意到，當你用構造器簽名去定義一個接口並試圖定義一個類去實現這個接口時會得到一個錯誤：

```typescript
interface ClockInterface {
    new (hour: number, minute: number);
}

class Clock implements ClockInterface  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

這裡因為當一個類實現了一個接口時，只對其實例部分進行類型檢查。constructor存在於類的靜態部分，所以不在檢查的範圍內。

取而代之，我們應該直接操作類的`靜態`部分。看下面的例子：

```typescript
interface ClockStatic {
    new (hour: number, minute: number);
}

class Clock {
    currentTime: Date;
    constructor(h: number, m: number) { }
}

var cs: ClockStatic = Clock;
var newClock = new cs(7, 30);
```

### <a name="2.6"></a>擴展接口

和類一樣，接口也可以相互擴展。擴展接口時會將其它接口裡的屬性拷貝到這個接口裡，因此允許你把接口拆分成單獨的可重用的組件。

```typescript
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

var square = <Square>{};
square.color = "blue";
square.sideLength = 10;
```

一個接口可以繼承多個接口，創建出多個接口的合成接口。

```typescript
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

var square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

### <a name="2.7"></a>混合類型

先前我們提過，接口能夠描述JavaScript裡豐富的類型。因為JavaScript其動態靈活的特點，有時你會希望一個物件可以同時具有上面提到的多種類型。

一個例子就是，一個物件可以同時做為函數和物件使用，並帶有額外的屬性。

```typescript
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

var c: Counter;
c(10);
c.reset();
c.interval = 5.0;
```

使用第三方庫的時候，你可能會像上面那樣去定義完整的類型。

## <a name="3"></a>類

傳統的JavaScript程序使用函數和基於原型的繼承來創建可重用的組件，但這對於熟悉使用面向物件方式的程序員來說有些棘手，因為他們用的是基於類的繼承並且物件是從類構建出來的。從JavaScript的下個版本ECMAScript6開始，JavaScript程序將可以使用這種基於類的面向物件方法。在TypeScript裡，我們允許開發者現在就使用這些特性，並且編譯後的JavaScript可以在所有主流瀏覽器和平台上運行，而不需要等到下個JavaScript版本。

### <a name="3.1"></a>類

下面看一個類的例子：

```typescript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

var greeter = new Greeter("world");
```

如果你使用過C#或Java，你會對這種語法非常熟悉。
我們聲明一個`Greeter`類。這個類有3個成員：一個叫做`greeting`的屬性，一個構造函數和一個`greet`方法。

你會注意到，我們在引用任何一個類成員的時候都用了`this`。
它表示我們訪問的是類的成員。

最後一行，我們使用`new`構造了Greeter類的一個實例。
它會調用之前定義的構造函數，創建一個`Greeter`類型的新物件，並執行構造函數初始化它。

### <a name="3.2"></a>繼承

在TypeScript裡，我們可以使用常用的面向物件模式。
當然，基於類的程序設計中最基本的模式是允許使用繼承來擴展一個類。

看下面的例子：

```typescript
class Animal {
    name: string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        alert(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 5) {
        alert("Slithering...");
        super.move(distanceInMeters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 45) {
        alert("Galloping...");
        super.move(distanceInMeters);
    }
}

var sam = new Snake("Sammy the Python");
var tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

這個例子展示了TypeScript中繼承的一些特徵，與其它語言類似。
我們使用`extends`來創建子類。你可以看到`Horse`和`Snake`類是基類`Animal`的子類，並且可以訪問其屬性和方法。

這個例子也說明了，在子類裡可以重寫父類的方法。
`Snake`類和`Horse`類都創建了`move`方法，重寫了從`Animal`繼承來的`move`方法，使得`move`方法根據不同的類而具有不同的功能。

### <a name="3.3"></a>公共，私有與受保護的修飾符

#### <a name="3.3.1"></a>默認為公有

在上面的例子裡，我們可以自由的訪問程序裡定義的成員。
如果你對其它語言中的類比較瞭解，就會注意到我們在之前的代碼裡並沒有使用`public`來做修飾；例如，C#要求必須明確地使用`public`指定成員是可見的。
在TypeScript裡，每個成員默認為`public`的。

你仍然可以使用`public`明確地指定其訪問類型，並且這確實是一個最佳實踐。我們可以用下面的方式來重寫上面的`Animal`類：

```typescript
class Animal {
    public name: string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number) {
        alert(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

#### <a name="3.3.2"></a>理解`private`

當成員被標記成`private`時，它就不能在聲明它的類的外部訪問。比如：

```TypeScript
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

new Animal("Cat").name; // Error: 'name' is private;
```

TypeScript使用的是結構性類型系統。
當我們比較兩種不同的類型時，並不在乎它們從哪兒來的，如果它們中每個成員的類型都是兼容的，我們就認為它們的類型是兼容的。

然而，當我們比較帶有`private`或`protected`成員的類型的時候，情況就不同了。如果其中一個類型裡包含一個`private`成員，那麼只有當另外一個類型中也存在這樣一個`private`成員， 並且它們是來自同一處聲明時，我們才認為這兩個類型是兼容的。對於`protected`成員也使用這個規則。

下面來看一個例子，詳細的解釋了這點：

```typescript
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
    constructor() { super("Rhino"); }
}

class Employee {
    private name:string;
    constructor(theName: string) { this.name = theName; }
}

var animal = new Animal("Goat");
var rhino = new Rhino();
var employee = new Employee("Bob");

animal = rhino;
animal = employee; //error: Animal and Employee are not compatible
```

這個例子中有`Animal`和`Rhino`兩個類，`Rhino`是`Animal`類的子類。
還有一個`Employee`類，其類型看上去與`Animal`是相同的。
我們創建了幾個這些類的實例，並相互賦值來看看會發生什麼。
因為`Animal`和`Rhino`共享了來自`Animal`裡的私有成員定義`private name: string`，因此它們是兼容的。
然而`Employee`卻不是這樣。當把`Employee`賦值給`Animal`的時候，得到一個錯誤，說它們的類型不兼容。
儘管`Employee`裡也有一個私有成員`name`，但它明顯不是`Animal`裡面定義的那個。

#### <a name="3.3.3"></a>理解`protected`

`protected`修飾符與`private`修飾符的行為很相似，但有一點不同，`protected`成員在派生類中仍然可以訪問。例如：

```TypeScript
class Person {
    protected name: string;
    constructor(name: string) { this.name = name; }
}

class Employee extends Person {
    private department: string;
    
    constructor(name: string, department: string) {
        super(name);
        this.department = department;
    }
    
    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

var howard = new Employee("Howard", "Sales");
console.log(howard.getElevatorPitch());
```

注意，我們不能在`Person`類外使用`name`，但是我們仍然可以通過`Employee`類的實例方法訪問，因為`Employee`是由`Person`派生出來的。

#### <a name="3.3.4"></a>參數屬性

在上面的例子中，我們不得不定義一個私有成員`name`和一個構造函數參數`theName`，並且立刻給`name`和`theName`賦值。這種情況經常會遇到。*參數屬性*可以方便地讓我們在一個地方定義並初始化一個成員。下面的例子是對之前`Animal`類的修改版，使用了參數屬性：

```typescript
class Animal {
    constructor(private name: string) { }
    move(distanceInMeters: number) {
        alert(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

注意看我們是如何捨棄了`theName`，僅在構造函數裡使用`private name: string`參數來創建和初始化`name`成員。
我們把聲明和賦值合併至一處。

參數屬性通過給構造函數參數添加一個訪問限定符來聲明。
使用`private`限定一個參數屬性會聲明並初始化一個私有成員；對於`public`和`protected`來說也是一樣。

### <a name="3.4"></a>存取器

TypeScript支持getters/setters來截取對物件成員的訪問。它能幫助你有效的控制對物件成員的訪問。

下面來看如何把一類改寫成使用『get'和『set'。首先是沒用使用存取器的例子：

```typescript
class Employee {
    fullName: string;
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

我們可以隨意的設置fullName，這是非常方便的，但是這也可能會帶來麻煩。

下面這個版本裡，我們先檢查用戶密碼是否正確，然後再允許其修改employee。
我們把對fullName的直接訪問改成了可以檢查密碼的`set`方法。我們也加了一個`get`方法，讓上面的例子仍然可以工作。

```typescript
var passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }
  
    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            alert("Error: Unauthorized update of employee!");
        }
    }
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

我們可以修改一下密碼，來驗證一下存取器是否是工作的。當密碼不對時，會提示我們沒有權限去修改employee。

注意：若要使用存取器，要求設置編譯器輸出目標為ECMAScript 5。

### <a name="3.5"></a>靜態屬性

到目前為止，我們只討論了類的實例成員，那些僅當類被實例化的時候才會被初始化的屬性。
我們也可以創建類的靜態成員，這些屬性存在於類本身上面而不是類的實例上。
在這個例子裡，我們使用`static`定義`origin`，因為它是所有網格都會用到的屬性。
每個實例想要訪問這個屬性的時候，都要在origin前面加上類名。
如同在實例屬性上使用`this.`前綴來訪問屬性一樣，這裡我們使用`Grid.`來訪問靜態屬性。

```typescript
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        var xDist = (point.x - Grid.origin.x);
        var yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

var grid1 = new Grid(1.0);  // 1x scale
var grid2 = new Grid(5.0);  // 5x scale

alert(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
alert(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

### <a name="3.6"></a>高級技巧

#### <a name="3.6.1"></a>構造函數

當你在TypeScript裡定義類的時候，實際上同時定義了很多東西。首先是類的實例的類型。

```typescript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

var greeter: Greeter;
greeter = new Greeter("world");
alert(greeter.greet());
```

在這裡，我們寫了`var greeter: Greeter`，意思是`Greeter`類實例的類型是`Greeter`。
這對於用過其它面向物件語言的程序員來講已經是老習慣了。

我們也創建了一個叫做*構造函數*的值。
這個函數會在我們使用`new`創建類實例的時候被調用。
下面我們來看看，上面的代碼被編譯成JavaScript後是什麼樣子的：

```javascript
var Greeter = (function () {
    function Greeter(message) {
        this.greeting = message;
    }
    Greeter.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    return Greeter;
})();

var greeter;
greeter = new Greeter("world");
alert(greeter.greet());
```

上面的代碼裡，『var Greeter'將被賦值為構造函數。當我們使用『new'並執行這個函數後，便會得到一個類的實例。這個構造函數也包含了類的所有靜態屬性。換個角度說，我們可以認為類具有實例部分與靜態部分這兩個部分。

讓我們來改寫一下這個例子，看看它們之前的區別：

```typescript
class Greeter {
    static standardGreeting = "Hello, there";
    greeting: string;
    greet() {
        if (this.greeting) {
            return "Hello, " + this.greeting;
        }
        else {
            return Greeter.standardGreeting;
        }
    }
}

var greeter1: Greeter;
greeter1 = new Greeter();
alert(greeter1.greet());

var greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";
var greeter2:Greeter = new greeterMaker();
alert(greeter2.greet());
```

這個例子裡，`greeter1`與之前看到的一樣。
我們實例化`Greeter`類，並使用這個物件。
與我們之前看到的一樣。

再之後，我們直接使用類。
我們創建了一個叫做`greeterMaker`的變量。
這個變量保存了這個類或者說保存了類構造函數。
然後我們使用`typeof Greeter`，意思是取Greeter類的類型，而不是實例的類型。
或者理確切的說，"告訴我`Greeter`標識符的類型"，也就是構造函數的類型。
這個類型包含了類的所有靜態成員和構造函數。
之後，就和前面一樣，我們在`greeterMaker`上使用`new`，創建`Greeter`的實例。

#### <a name="3.6.2"></a>把類當做接口使用

如上一節裡所講的，類定義會創建兩個東西：類實例的類型和一個構造函數。因為類可以創建出類型，所以你能夠在可以使用接口的地方使用類。

```typescript
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

var point3d: Point3d = {x: 1, y: 2, z: 3};
```

## <a name="4"></a>命名空間和模組

這節會列出多種在TypeScript裡組織代碼的方法。
我們將介紹命名空間（之前叫做「內部模組」）和模組（之前叫做「外部模組」）並且會討論在什麼樣的場合下適合使用它們以及怎樣使用它們。
我們也會涉及到一些高級主題，如怎麼使用外部模組，當使用TypeScript模組時如何避免常見的陷阱。

### 一個關於術語的注意事項

我們剛剛提及了「內部模組」和「外部模組」。
如果你對這個術語感到似曾相識，那麼一定要注意在TypeScript1.5里，它們的命名發生了變化。
「內部模組」變成了「命名空間」。
「外部模組」變成了簡單的「模組」，為了與ECMAScript 6的術語保持一致。

並且，任何使用`module`關鍵字聲明內部模組的地方，都可以使用`namespace`關鍵字來代替。

這樣就避免了新用戶可能把它們搞混了。

### 第一步

我們先來寫一段程序並將在整個小節中都使用這個例子。
我們定義幾個簡單的字符串驗證器，好比你會使用它們來驗證表單裡的用戶輸入或驗證外部數據。

#### 所有的驗證器都放在一個文件裡

```typescript
interface StringValidator {
    isAcceptable(s: string): boolean;
}

var lettersRegexp = /^[A-Za-z]+$/;
var numberRegexp = /^[0-9]+$/;

class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}

class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: StringValidator; } = {};
validators['ZIP code'] = new ZipCodeValidator();
validators['Letters only'] = new LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

### 使用命名空間

隨著我們增加更多的驗證器，我們想要將它們組織在一起來保持對它們的追蹤記錄並且不用擔心與其它物件產生命名衝突。
我們把驗證器包裹到一個命名空間內，而不是把它們放在全局命名空間下。

這個例子裡，我們把所有驗證器相關的類型都放到一個叫做`Validation`的命名空間裡。
因為我們想讓這些接口和類在命名空間外也是可訪問的，所以我們需要使用`export`。
相反的，變量`lettersRegexp`和`numberRegexp`是具體實現，所以沒有導出，因此它們在命名空間外是不能訪問的。
在文件末尾的測試代碼裡，我們需要限制類型名稱，因為這是在命名空間外訪問，比如`Validation.LettersOnlyValidator`。

#### 使用命名空間的驗證器

```typescript
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    var lettersRegexp = /^[A-Za-z]+$/;
    var numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: Validation.StringValidator; } = {};
validators['ZIP code'] = new Validation.ZipCodeValidator();
validators['Letters only'] = new Validation.LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```
### 分割成多文件

當應用變得越來越大時，我們需要將代碼分散到不同的文件中以便於維護。

### <a name="4.1"></a>多文件中的命名空間

現在，我們把`Validation`命名空間分割成多個文件。
儘管是不同的文件，它們仍是同一個命名空間，並且在使用的時候就如同它們在一個文件中定義的一樣。
因為不同文件之間存在依賴關係，所以我們加入了引用標籤來告訴編譯器文件之間的關聯。
我們的測試代碼保持不變。

#### Validation.ts

```typescript
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }
}
```

#### LettersOnlyValidator.ts

```typescript
/// <reference path="Validation.ts" />
namespace Validation {
    var lettersRegexp = /^[A-Za-z]+$/;
    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }
}
```

#### ZipCodeValidator.ts

```typescript
/// <reference path="Validation.ts" />
namespace Validation {
    var numberRegexp = /^[0-9]+$/;
    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}
```

#### Test.ts

```typescript
/// <reference path="Validation.ts" />
/// <reference path="LettersOnlyValidator.ts" />
/// <reference path="ZipCodeValidator.ts" />

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: Validation.StringValidator; } = {};
validators['ZIP code'] = new Validation.ZipCodeValidator();
validators['Letters only'] = new Validation.LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

當涉及到多文件時，我們必須確保所有編譯後的代碼都被加載了。
我們有兩種方式。

第一種方式，把所有的輸入文件編譯為一個輸出文件，需要使用`--out`標記：

```Shell
tsc --out sample.js Test.ts
```

編譯器會根據源碼裡的引用標籤自動地對輸出進行排序。你也可以單獨地指定每個文件。

```Shell
tsc --out sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts
```

第二種方式，我們可以編譯每一個文件（默認方式），那麼每個源文件都會對應生成一個JavaScript文件。
然後，在頁面上通過`<script>`標籤把所有生成的JavaScript文件按正確的順序引進來，比如：

#### MyTestPage.html（摘錄部分）

```html
<script src="Validation.js" type="text/javascript" />
<script src="LettersOnlyValidator.js" type="text/javascript" />
<script src="ZipCodeValidator.js" type="text/javascript" />
<script src="Test.js" type="text/javascript" />
```

### <a name="4.2"></a>使用模組

TypeScript中同樣存在模組的概念。
模組會在兩種情況下被用到：Node.js或require.js。
對於沒有使用Node.js和require.js的應用來說是不需要使用外部模組的，最好使用上面介紹的命名空間的方式來組織代碼。

使用模組時，不同文件之間的關係是通過文件級別的導入和導出來指定的。
在TypeScript裡，任何具有頂級`import`和`export`的文件都會被視為模組。

下面，我們把之前的例子改寫成使用模組。
注意，我們不再使用`module`關鍵字 - 文件本身會被視為一個模組並以文件名來區分。

引用標籤用`import`語句來代替，指明了模組之前的依賴關係。
`import`語句有兩部分：模組在當前文件中的名字，`require`關鍵字指定了依賴模組的路徑：

```typescript
import someMod = require('someModule');
```

我們通過頂級的`export`關鍵字指出了哪些物件在模組外是可見的，如同使用`export`定義命名空間的公共接口一樣。

為了編譯，我們必須在命令行上指明生成模組的目標類型。對於Node.js，使用*--module commonjs*。對於require.js，使用`--module amd`。比如：

```Shell
ts --module commonjs Test.ts
```

編譯的時候，每個外部模組會變成一個單獨的文件。如同引用標籤，編譯器會按照*import*語句編譯相應的文件。

#### Validation.ts

```typescript
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

#### LettersOnlyValidator.ts

```typescript
import validation = require('./Validation');
var lettersRegexp = /^[A-Za-z]+$/;
export class LettersOnlyValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
```

#### ZipCodeValidator.ts

```typescript
import validation = require('./Validation');
var numberRegexp = /^[0-9]+$/;
export class ZipCodeValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

#### Test.ts

```typescript
import validation = require('./Validation');
import zip = require('./ZipCodeValidator');
import letters = require('./LettersOnlyValidator');

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: validation.StringValidator; } = {};
validators['ZIP code'] = new zip.ZipCodeValidator();
validators['Letters only'] = new letters.LettersOnlyValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

### 生成模組代碼

根據編譯時指定的目標模組類型，編譯器會生成相應的代碼，或者是適合Node.js（commonjs）或者是適合require.js（AMD）模組加載系統的代碼。
想要瞭解更多關於`define`和`require`函數的使用方法，請閱讀相應模組加載器的說明文檔。

這個例子展示了在導入導出階段使用的名字是怎麼轉換成模組加載代碼的。

#### SimpleModule.ts

```typescript
import m = require('mod');
export var t = m.something + 1;
```

#### AMD/RequireJS SimpleModule.js

```javascript
define(["require"，"exports"，"mod"]，function(require, exports, m) {
    exports.t = m.something + 1;
});
```

#### CommonJS / Node SimpleModule.js

```javascript
var m = require('mod');
exports.t = m.something + 1;
```

### <a name="4.3"></a>Export =

在上面的例子中，使用驗證器的時候，每個模組只導出一個值。
像這種情況，在驗證器物件前面再加上限定名就顯得累贅了，最好是直接使用一個標識符。

`export =`語法指定了模組導出的單個物件。
它可以是類，接口，模組，函數或枚舉類型。
當import的時候，直接使用模組導出的標識符，不再需要其它限定名。

下面，我們簡化驗證器的實現，使用`export =`語法使每個模組導出單一物件。
這會簡化對模組的使用 - 我們可以用`zipValidator`代替`zip.ZipCodeValidator`。

#### Validation.ts

```typescript
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

#### LettersOnlyValidator.ts

```typescript
import validation = require('./Validation');
var lettersRegexp = /^[A-Za-z]+$/;
class LettersOnlyValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
export = LettersOnlyValidator;
```

#### ZipCodeValidator.ts

```typescript
import validation = require('./Validation');
var numberRegexp = /^[0-9]+$/;
class ZipCodeValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;
```

#### Test.ts

```typescript
import validation = require('./Validation');
import zipValidator = require('./ZipCodeValidator');
import lettersValidator = require('./LettersOnlyValidator');

// Some samples to try
var strings = ['Hello', '98052', '101'];
// Validators to use
var validators: { [s: string]: validation.StringValidator; } = {};
validators['ZIP code'] = new zipValidator();
validators['Letters only'] = new lettersValidator();
// Show whether each string passed each validator
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

### <a name="4.4"></a>別名

另一種簡化模組操作的方法是使用`import q = x.y.z`給常用的模組起一個短的名字。
不要與`import x = require('name')`用來加載模組的語法弄混了，這裡的語法是為指定的符號創建一個別名。
你可以用這種方法為任意標識符創建別名，也包括導入的模組中的物件。

#### 創建別名基本方法

```typescript
namespace Shapes {
    export namespace Polygons {
        export class Triangle { }
        export class Square { }
    }
}

import polygons = Shapes.Polygons;
var sq = new polygons.Square(); // Same as 'new Shapes.Polygons.Square()'
```

注意，我們並沒有使用`require`關鍵字，而是直接使用導入符號的限定名賦值。
這與使用`var`相似，但它還適用於類型和導入的具有命名空間含義的符號。
重要的是，對於值來講，`import`會產生與原始符號不同的引用，所以改變別名的值並不會影響原始變量的值。

### <a name="4.5"></a>可選模組的加載與其它高級加載的場景

有些時候，你只想在某種條件下才去加載一個模組。
在TypeScript裡，我們可以使用下面的方式來實現它以及其它高級加載的場景，直接調用模組加載器而不必擔心類型安全問題。

編譯器能探測出一個模組是否在生成的JavaScript裡被使用到了。
對於那些只做為類型系統部分使用的模組來講，不會生成對應`require代碼`。
挑出未使用的引用有益於性能優化，同時也允許可選擇性的加載模組。

這種模式的核心是`import id = require('...')`讓我們可以訪問外部模組導出的類型。
模組加載是動態調用的（通過`require`），像下面`if`語句展示的那樣。
它利用了挑出對未使用引用的優化，模組只在需要的時候才去加載。
為了讓這種方法可行，通過`import`定義的符號只能在表示類型的位置使用（也就是說那段代碼永遠不會被編譯生成JavaScript）。

為了確保使用正確，我們可以使用`typeof`關鍵字。
在要求是類型的位置使用`typeof`關鍵字時，會得到類型值，在這個例子裡得到的是外部模組的類型。

#### Dynamic Module Loading in Node.js

```typescript
declare var require;
import Zip = require('./ZipCodeValidator');
if (needZipValidation) {
    var x: typeof Zip = require('./ZipCodeValidator');
    if (x.isAcceptable('.....')) { /* ... */ }
}
```

#### Sample: Dynamic Module Loading in require.js

```typescript
declare var require;
import Zip = require('./ZipCodeValidator');
if (needZipValidation) {
    require(['./ZipCodeValidator'], (x: typeof Zip) => {
        if (x.isAcceptable('...')) { /* ... */ }
    });
}
```

### <a name="4.6"></a>使用其它JavaScript庫

為了描述不是用TypeScript寫的程序庫的類型，我們需要對程序庫暴露的API進行聲明。
由於大部分程序庫只提供少數的頂級物件，命名空間和模組是用來表示它們是一個好辦法。
我們叫它聲明不是對執行環境的定義。
通常會在`.d.ts`裡寫這些定義。
如果你熟悉C/C++，你可以把它們當做`.h`文件。
讓我們看一些例子。

#### <a name="4.6.1"></a>外部命名空間

流行的程序庫D3在全局物件`d3`裡定義它的功能。
因為這個庫通過一個`<script>`標籤加載（不是通過模組加載器），它的聲明文件使用內部模組來定義它的類型。
為了讓TypeScript編譯器識別它的類型，我們使用外部命名空間聲明。
比如，我們像下面這樣寫：

#### D3.d.ts (部分摘錄)

```typescript
declare namespace d3 {
    export interface Selectors {
        select: {
            (selector: string): Selection;
            (element: EventTarget): Selection;
        };
    }

    export interface Event {
        x: number;
        y: number;
    }

    export interface Base extends Selectors {
        event: Event;
    }
}

declare var d3: D3.Base;
```

#### <a name="4.6.2"></a>外來的模組

在Node.js裡，大多數的任務可以通過加載一個或多個模組來完成。
我們可以使用頂級export聲明來為每個模組定義各自的`.d.ts`文件，但全部放在一個大的文件中會更方便。
為此，我們把模組名用引號括起來，方便之後的import。
例如：

#### node.d.ts (部分摘錄)

```typescript
declare module "url" {
    export interface Url {
        protocol?: string;
        hostname?: string;
        pathname?: string;
    }

    export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
}

declare module "path" {
    export function normalize(p: string): string;
    export function join(...paths: any[]): string;
    export var sep: string;
}
```

現在我們可以`///<reference path="node.d.ts"/>`, 然後使用`import url = require('url');`加載這個模組。

```typescript
///<reference path="node.d.ts"/>
import url = require("url");
var myUrl = url.parse("http://www.typescriptlang.org");
```

### <a name="4.7"></a>命名空間和模組的陷阱

這一節，將會介紹使用內部和外部模組時常見的陷阱和怎麼去避免它。

#### <a name="4.7.1"></a>對模組使用`/// <reference>`

一個常見的錯誤是使用`/// <reference>`引用模組文件，應該使用import。
要理解這之間的不同，我們首先應該弄清編譯器是怎麼找到模組的類型信息的。

首先，根據`import x = require(...);`聲明查找`.ts`文件。
這個文件應該是使用了頂級import或export聲明的執行文件。

其次，與前一步相似，去查找`.d.ts`文件，不同的是它不是執行文件而是聲明文件（同樣具有頂級的import或export聲明）。

最後，在`declare`的模組裡尋找名字匹配的「外來模組的聲明」。

#### myModules.d.ts

```typescript
// In a .d.ts file or .ts file that is not a module:
declare module "SomeModule" {
    export function fn(): string;
}
```

#### myOtherModule.ts

```typescript
/// <reference path="myModules.d.ts" />
import m = require("SomeModule");
```

這裡的引用標籤指定了外來模組的位置。
這就是一些Typescript例子中引用node.d.ts的方法。

#### <a name="4.7.2"></a>不必要的命名空間

如果你想把命名空間轉換為模組，它可能會像下面這個文件一件：

#### shapes.ts

```typescript
export namespace Shapes {
    export class Triangle { /* ... */ }
    export class Square { /* ... */ }
}
```

頂層的模組`Shapes`包裹了`Triangle`和`Square`。
這對於使用它的人來說是讓人迷惑和討厭的：

#### shapeConsumer.ts

```typescript
import shapes = require('./shapes');
var t = new shapes.Shapes.Triangle(); // shapes.Shapes?
```

TypeScript裡模組的一個特點是不同的模組永遠也不會在相同的作用域內使用相同的名字。
因為使用模組的人會為它們命名，所以完全沒有必要把導出的符號包裹在一個命名空間裡。

再次重申，不應該對模組使用命名空間，使用命名空間是為了提供邏輯分組和避免命名衝突。
模組文件本身已經是一個邏輯分組，並且它的名字是由導入這個模組的代碼指定，所以沒有必要為導出的物件增加額外的模組層。

下面是改進的例子：

#### shapes.ts

```typescript
export class Triangle { /* ... */ }
export class Square { /* ... */ }
```

#### shapeConsumer.ts

```typescript
import shapes = require('./shapes');
var t = new shapes.Triangle(); 
```

#### <a name="4.7.3"></a>模組的取捨

就像每個JS文件對應一個模組一樣，TypeScript裡模組文件與生成的JS文件也是一一對應的。
這會產生一個效果，就是無法使用`--out`來讓編譯器合併多個模組文件為一個JavaScript文件。

## <a name="5"></a>函數

函數是JavaScript應用程序的基礎。
它幫助你實現抽象層，模擬類，信息隱藏和模組。
在TypeScript裡，雖然已經支持類，命名空間和模組，但函數仍然是主要的定義*行為*的地方。
TypeScript為JavaScript函數添加了額外的功能，讓我們可以更容易的使用。

### <a name="5.1"></a>Functions

和JavaScript一樣，TypeScript函數可以創建有名字的函數和匿名函數。
你可以隨意選擇適合應用程序的方式，不論是定義一系列API函數還是只使用一次的函數。

通過下面的例子可以迅速回想起這兩種JavaScript中的函數：

```javascript
//Named function
function add(x, y) {
    return x + y;
}

//Anonymous function
var myAdd = function(x, y) { return x + y; };
```

在JavaScript裡，函數可以可以使用函數體外部的變量。當函數這麼做時，我們說它『捕獲』了這些變量。至於為什麼可以這樣做以及其中的利弊超出了本文的範圍，但是深刻理解這個機制對學習JavaScript和TypeScript會很有幫助。

```javascript
var z = 100;

function addToZ(x, y) {
    return x + y + z;
}
```

### <a name="5.2"></a>函數類型

#### <a name="5.2.1"></a>為函數定義類型

讓我們為上面那個函數添加類型：

```typescript
function add(x: number, y: number): number {
    return x + y;
}

var myAdd = function(x: number, y: number): number { return x + y; };
```

我們可以給每個參數添加類型之後再為函數本身添加返回值類型。TypeScript能夠根據返回語句自動推斷出返回值類型，因此我們通常省略它。

#### <a name="5.2.2"></a>書寫完整函數類型

現在我們已經為函數指定了類型，下面讓我們寫出函數的完整類型。

```typescript
var myAdd: (x:number, y:number) => number = 
    function(x: number, y: number): number { return x + y; };
```

函數類型包含兩部分：參數類型和返回值類型。當寫出完整函數類型的時候，這兩部分都是需要的。我們以參數列表的形式寫出參數類型，為每個參數指定一個名字和類型。這個名字只是為了增加可讀性。我們也可以這麼寫：

```typescript
var myAdd: (baseValue:number, increment:number)=>number = 
    function(x: number, y: number): number { return x+y; };
```

只要參數類型是匹配的，那麼就認為它是有效的函數類型，而不在乎參數名是否正確。

對於返回值，我們在函數和返回值類型之前使用(=>)符號，使之清晰明了。如之前提到的，返回值類型是函數類型的必要部分，如果函數沒有返回任何值，你也必須指定返回值類型為『void'而不能留空。

函數的類型只是由參數類型和返回值組成的。函數中使用的『捕獲』變量不會體現在類型裡。實際上，這些變量是函數的隱藏狀態並不是組成API的一部分。

#### <a name="5.2.3"></a>推斷類型

嘗試這個例子的時候，你會發現如果你在賦值語句的一邊指定了類型但是另一邊沒有類型的話，TypeScript編譯器會自動識別出類型：

```typescript
// myAdd has the full function type
var myAdd = function(x: number, y: number): number { return x + y; };

// The parameters 'x' and 'y' have the type number
var myAdd: (baseValue:number, increment:number) => number = 
    function(x, y) { return x + y; };
```

這叫做『按上下文歸類』，是類型推論的一種。
它幫助我們更好地為程序指定類型。

### <a name="5.3"></a>可選參數和默認參數

不同於JavaScript，TypeScript裡每個函數參數都是必須的。
這並不是指參數一定是個非`null`值，而是編譯器檢查用戶是否為每個參數都傳入了值。
編譯器還會假設只有這些參數會被傳遞進函數。
簡短地說，傳遞給函數的參數數量必須與函數期望的參數數量一致。

```typescript
function buildName(firstName: string, lastName: string) {
    return firstName + " " + lastName;
}

var result1 = buildName("Bob");  //error, too few parameters
var result2 = buildName("Bob", "Adams", "Sr.");  //error, too many parameters
var result3 = buildName("Bob", "Adams");  //ah, just right
```

JavaScript裡，每個參數都是可選的，可傳可不傳。沒傳參的時候，它的值就是undefined。在TypeScript裡我們可以在參數名旁使用『?』實現可選參數的功能。比如，我們想讓last name是可選的：

```typescript
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

var result1 = buildName("Bob");  //works correctly now
var result2 = buildName("Bob", "Adams", "Sr.");  //error, too many parameters
var result3 = buildName("Bob", "Adams");  //ah, just right
```

可選參數必須在必須跟在必須參數後面。如果上例我們想讓first name是可選的，那麼就必須調整它們的位置，把first name放在後面。

TypeScript裡，我們還可以為可選參數設置默認值。仍然修改上例，把last name的默認值設置為「Smith」。

```typescript
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

var result1 = buildName("Bob");  //works correctly now, also
var result2 = buildName("Bob", "Adams", "Sr.");  //error, too many parameters
var result3 = buildName("Bob", "Adams");  //ah, just right
```

和可選參數一樣，帶默認值的參數也要放在必須參數後面。

可選參數與默認值參數共享參數類型。

```typescript
function buildName(firstName: string, lastName?: string) {}
```

和

```typescript
function buildName(firstName: string, lastName = "Smith") {}
```

共享同樣的類型`(firstName: string, lastName?: string) => string`。默認參數的默認值消失了，只保留了它是一個可選參數的信息。

### <a name="5.4"></a>剩餘參數

必要參數，默認參數和可選參數有個共同點：它們表示某一個參數。有時，你想同時操作多個參數，或者你並不知道會有多少參數傳遞進來。在JavaScript裡，你可以使用arguments來訪問所有傳入的參數。

在TypeScript裡，你可以把所有參數收集到一個變量裡：

```typescript
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

var employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

剩餘參數會被當做個數不限的可選參數。可以一個都沒有，同樣也可以有任意個。編譯器創建參數陣列，名字是你在省略號（...）後面給定的名字，你可以在函數體內使用這個陣列。

這個省略號也會在帶有剩餘參數的函數類型定義上使用到。

```typescript
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

var buildNameFun: (fname: string, ...rest: string[]) => string = buildName;
```

### <a name="5.5"></a>Lambda表達式和使用『this'

JavaScript裡『this'的工作機制對JavaScript程序員來說已經是老生常談了。的確，學會如何使用它絕對是JavaScript編程中的一件大事。由於TypeScript是JavaScript的超集，TypeScript程序員也需要弄清『this'工作機制並且當有bug的時候能夠找出錯誤所在。『this'的工作機制可以單獨寫一本書了，並確已有人這麼做了。在這裡，我們只介紹一些基礎知識。

JavaScript裡，『this'的值在函數被調用的時候才會指定。這是個既強大又靈活的特點，但是你需要花點時間弄清楚函數調用的上下文是什麼。眾所周知這不是一件很簡單的事，特別是函數當做回調函數使用的時候。

下面看一個例子：

```javascript
var deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        return function() {
            var pickedCard = Math.floor(Math.random() * 52);
            var pickedSuit = Math.floor(pickedCard / 13);
      
            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

var cardPicker = deck.createCardPicker();
var pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

如果我們運行這個程序，會發現它並沒有彈出對話框而是報錯了。
因為`createCardPicker`返回的函數裡的`this`被設置成了`window`而不是`deck`物件。
當你調用`cardPicker()`時會發生這種情況。這裡沒有對`this`進行動態綁定因此為window。（注意在嚴格模式下，會是undefined而不是window）。

為瞭解決這個問題，我們可以在函數被返回時就綁好正確的`this`。
這樣的話，無論之後怎麼使用它，都會引用綁定的『deck'物件。

我們把函數表達式變為使用lambda表達式（ () => {} ）。這樣就會在函數創建的時候就指定了『this'值，而不是在函數調用的時候。

```typescript
var deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        // Notice: the line below is now a lambda, allowing us to capture 'this' earlier
        return () => {
            var pickedCard = Math.floor(Math.random() * 52);
            var pickedSuit = Math.floor(pickedCard / 13);
      
            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

var cardPicker = deck.createCardPicker();
var pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

為瞭解更多關於`this`的信息，請閱讀Yahuda Katz的[Understanding JavaScript Function Invocation and "this"](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)。

### <a name="5.6"></a>重載

JavaScript本身是個動態語言。
JavaScript裡函數根據傳入不同的參數而返回不同類型的數據是很常見的。

```typescript
var suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        var pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        var pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

var myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
var pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

var pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

『pickCard'方法根據傳入參數的不同會返回兩種不同的類型。如果傳入的是代表紙牌的物件，函數作用是從中抓一張牌。如果用戶想抓牌，我們告訴他抓到了什麼牌。但是這怎麼在類型系統裡表示呢。

方法是為同一個函數提供多個函數類型定義來進行函數重載。編譯器會根據這個列表去處理函數的調用。下面我們來重載『pickCard'函數。

```typescript
var suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        var pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        var pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

var myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
var pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

var pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

這樣改變後，重載的`pickCard`函數在調用的時候會進行正確的類型檢查。

為了讓編譯器能夠選擇正確的檢查類型，它與JavaScript裡的處理流程相似。
它查找重載列表，嘗試使用第一個重載定義。
如果匹配的話就使用這個。
因此，在定義重載的時候，一定要把最精確的定義放在最前面。

注意，`function pickCard(x): any`並不是重載列表的一部分，因此這裡只有兩個重載：一個是接收物件另一個接收數字。
以其它參數調用`pickCard`會產生錯誤。

## <a name="6"></a>泛型

軟件工程中，我們不僅要創建一致的定義良好的API，同時也要考慮可重用性。組件不僅能夠支持當前的數據類型，同時也能支持未來的數據類型，這在創建大型系統時為你提供了十分靈活的功能。

在像C#和Java這樣的語言中，可以使用『泛型』來創建可重用的組件，一個組件可以支持多種類型的數據。這樣用戶就可以以自己的數據類型來使用組件。

### <a name="6.1"></a>泛型之Hello World

下面來創建第一個使用泛型的例子：identity函數。這個函數會返回任何傳入它的值。你可以把這個函數當成是『echo'命令。

不用泛型的話，這個函數可能是下面這樣：

```typescript
function identity(arg: number): number {
    return arg;
}
```

或者，我們使用『any'類型來定義函數：

```typescript
function identity(arg: any): any {
    return arg;
}
```

雖然使用`any`類型後這個函數已經能接收任何類型的arg參數，但是卻丟失了一些信息：傳入的類型與返回的類型應該是相同的。
如果我們傳入一個數字，我們只知道任何類型的值都有可能被返回。

因此，我們需要一種方法使用返回值的類型與傳入參數的類型是相同的。
這裡，我們使用了*類型變量*，它是一種特殊的變量，只用於表示類型而不是值。

```typescript
function identity<T>(arg: T): T {
    return arg;
}
```

我們給identity添加了類型變量『T'。『T'幫助我們捕獲用戶傳入的類型（比如：number），之後我們就可以使用這個類型。之後我們再次使用了『T'當做返回值類型。現在我們可以知道參數類型與返回值類型是相同的了。

我們把這個版本的identity函數叫做泛型，它可以適用於多個類型。不像使用『any'，它不會丟失信息。同時也可以像第一個例子那像，傳入數值類型並返回數值類型。

我們定義了泛型函數後，可以用兩種方法使用。第一種是，傳入所有的參數，包含類型參數：

```typescript
var output = identity<string>("myString");  // type of output will be 'string'
```

這裡我們明確的指定了『T'是字符串類型，並做為一個參數傳給函數，使用了<>括起來。

第二種方法更普遍。利用了類型推論，編譯器會根據傳入的參數自動地幫助我們確定T的類型：

```typescript
var output = identity("myString");  // type of output will be 'string'
```

注意我們並沒用`<>`明確的指定類型，編譯器看到了`myString`，把`T`設置為此類型。
類型推論幫助我們保持代碼精簡和高可讀性。如果編譯器不能夠自動地推斷出類型的話，只能像上面那樣明確的傳入T的類型，在一些複雜的情況下，這是可能出現的。

### <a name="6.2"></a>使用泛型變量

使用泛型創建像`identity`這樣的泛型函數時，編譯器要求你在函數體必須正確的使用這個通用的類型。換句話說，你必須把這些參數當做是任意或所有類型。

看下之前`identity`例子：

```typescript
function identity<T>(arg: T): T {
    return arg;
}
```

如果我們想同時打印出arg的『length'屬性值，很可能會這樣做：

```typescript
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

如果這麼做，編譯器會報錯說我們使用了`arg`的`.length`屬性，但是沒有地方指明`arg`具有這個屬性。
記住，這些類型變量代表的是任意類型，所以使用這個函數的人可能傳入的是個數字，而數字是沒有`.length`屬性的。

現在假設我們想操作`T`類型的陣列而不直接是`T`。由於我們操作的是陣列，所以`.length`屬性是應該存在的。
我們可以像創建其它陣列一樣創建這個陣列：

```typescript
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

你可以這樣理解`loggingIdentity`的類型：泛型函數`loggingIdentity`，接收類型參數`T`，和函數`arg`，它是個元素類型是`T`的陣列，並返回元素類型是`T`的陣列。
如果我們傳入數字陣列，將返回一個數字陣列，因為此時`T`的的類型為`number`。
這可以讓我們把泛型變量T當做類型的一部分使用，而不是整個類型，增加了靈活性。

我們也可以這樣實現上面的例子：

```typescript
function loggingIdentity<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

使用過其它語言的話，你可能對這種語法已經很熟悉了。在下一節，會介紹如何創建自定義泛型像Array<T>一樣。

### <a name="6.3"></a>泛型類型

上一節，我們創建了identity通用函數，可以適用於不同的類型。在這節，我們研究一下函數本身的類型，以及如何創建泛型接口。

泛型函數的類型與非泛型函數的類型沒什麼不同，只是有一個類型參數在最前面，像函數聲明一樣：

```typescript
function identity<T>(arg: T): T {
    return arg;
}

var myIdentity: <T>(arg: T)=>T = identity;
```

我們也可以使用不同的泛型參數名，只要在數量上和使用方式上能對應上就可以。

```typescript
function identity<T>(arg: T): T {
    return arg;
}

var myIdentity: <U>(arg: U)=>U = identity;
```

我們還可以使用帶有調用簽名的物件字面量來定義泛型函數：

```typescript
function identity<T>(arg: T): T {
    return arg;
}

var myIdentity: { <T>(arg: T): T} = identity;
```

這引導我們去寫第一個泛型接口了。我們把上面例子裡的物件字面量拿出來做為一個接口：

```typescript
interface GenericIdentityFn {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

var myIdentity: GenericIdentityFn = identity;
```

一個相似的例子，我們可能想把泛型參數當作整個接口的一個參數。
這樣我們就能清楚的知道使用的具體是哪個泛型類型（比如：`Dictionary<string>而不只是Dictionary`）。
這樣接口裡的其它成員也能知道這個參數的類型了。

```typescript
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

var myIdentity: GenericIdentityFn<number> = identity;
```

我們並沒有描述泛型函數，而是使用一個非泛型函數簽名作為泛型類型一部分。當我們使用GenericIdentityFn的時候，我也得傳入一個類型參數來指定泛型類型（這個例子是：number），鎖定了之後代碼裡使用的類型。

除了泛型接口，我們還可以創建泛型類。
注意，無法創建枚舉泛型和命名空間泛型。

### <a name="6.4"></a>泛型類

泛型類看上去與泛型接口差不多。泛型類使用<>括起泛型類型，跟在類名後面。

```typescript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

var myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

『GenericNumber'類的使用是十分直觀的，並且你應該注意到了我們並不限制只能使用數字類型。也可以使用字符串或其它更複雜的類型。

```typescript
var stringNumeric = new GenericNumber<string>();
stringNumeric.zerValue = "";
stringNumeric.add = function(x, y) { return x + y; };

alert(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

與接口一樣，直接把泛型類型放在類後面，可以幫助我們確認類的所有屬性都在使用相同的類型。

我們在[類](#類)那節說過，類有兩部分：靜態部分和實例部分。泛型類指的是實例部分的類型，所以類的靜態屬性不能使用這個泛型類型。

### <a name="6.5"></a>泛型約束

你應該會記得之前的一個例子，我們有時候想操作某類型的一組值，並且我們知道這組值具有什麼樣的屬性。在『loggingIdentity'例子中，我們想訪問『arg'的『length'屬性，但是編譯器並不能證明每種類型都有『length'屬性，所以就報錯了。

```typescript
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

相比於操作任意類型，我們想要限制函數去處理任意帶有『length'屬性的類型。只要傳入的類型有這個屬性，就允許通過。所以我們必須對T定義約束。

為此，我們定義一個接口來描述約束條件。創建一個包含『length'屬性的接口，使用這個接口和extends關鍵字還實現約束。

```typescript
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

現在這個泛型函數被定義了約束，因此它不再是適用於任意類型：

```typescript
loggingIdentity(3);  // Error, number doesn't have a .length property
```

我們需要傳入符合約束類型的值，必須包含必須的屬性：

```typescript
loggintIdentity({ length: 10, value: 3 });
```

#### <a name="6.5.1"></a>在泛型約束中使用類型參數

有時候，我們需要使用類型參數去約束另一個類型參數。比如，

```typescript
function find<T, U extends Findable<T>>(n: T, s: U) {  // errors because type parameter used in contraint
  // ... 
}
find(giraffe, myAnimals);
```

可以通過下面的方法來實現，重寫上面的代碼，

```typescript
function find<T>(n: T, s: Findable<T>) {
  // ...
}
find(giraffe, myAnimals);
```

注意：上面兩種寫法並不完全等同，因為第一段程序的返回值可能是U，而第二段程序卻沒有這一限制。

#### <a name="6.5.2"></a>在泛型裡使用類類型

在TypeScript使用泛型創建工廠函數時，需要引用構造函數的類類型。比如，

```typescript
function create<T>(c: {new(): T;}): T {
    return new c();
}
```

一個更高級的例子，使用原型屬性推斷並約束構造函數與類實例的關係。

```typescript
class BeeKeeper {
    hasMask: boolean;
}

class ZooKeeper {
    nametag: string; 
}

class Animal {
    numLegs: number;
}

class Bee extends Animal {
    keeper: BeeKeeper;
}

class Lion extends Animal {
    keeper: ZooKeeper;
}

function findKeeper<A extends Animal, K> (a: {new(): A; 
    prototype: {keeper: K}}): K {

    return a.prototype.keeper;
}

findKeeper(Lion).nametag;  // typechecks!
```

## <a name="7"></a>常見錯誤

下面列出了一些在使用TypeScript和編譯器的時候常見的錯誤

### <a name="7.1"></a>常見疑難問題

#### <a name="7.1.1"></a>"tsc.exe" exited with error code 1.

**Fixes:**

檢查文件編碼是不是UTF-8 - [https://typescript.codeplex.com/workitem/1587](https://typescript.codeplex.com/workitem/1587)

#### <a name="7.1.2"></a>external module XYZ cannot be resolved

**Fixes:**

檢查模組路徑的大小寫 - [https://typescript.codeplex.com/workitem/2134](https://typescript.codeplex.com/workitem/2134)

## <a name="8"></a>Mixins

除了傳統的面向物件繼承方式，還有一種流行的從可重用組件中創建類的方式，就是通過聯合一個簡單類的代碼。你可能在Scala這樣的語言裡對mixins已經熟悉了，它在JavaScript中也是很流行的。

### <a name="8.1"></a>混入示例

下面的代碼演示了如何在TypeScript裡使用mixins。後面我們還會解釋這段代碼是怎麼工作的。

```typescript
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }
 
}
 
// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}
 
class SmartObject implements Disposable, Activatable {
    constructor() {
        setInterval(() => console.log(this.isActive + " : " + this.isDisposed), 500);
    }
 
    interact() {
        this.activate();
    }
 
    // Disposable
    isDisposed: boolean = false;
    dispose: () => void;
    // Activatable
    isActive: boolean = false;
    activate: () => void;
    deactivate: () => void;
}
applyMixins(SmartObject, [Disposable, Activatable])
 
var smartObj = new SmartObject();
setTimeout(() => smartObj.interact(), 1000);
 
////////////////////////////////////////
// In your runtime library somewhere
////////////////////////////////////////

function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        })
    }); 
}
```

### <a name="8.2"></a>理解這個例子

代碼裡首先定義了兩個類，它們將做為mixins。
可以看到每個類都只定義了一個特定的行為或功能。
稍後我們使用它們來創建一個新類，同時具有這兩種功能。

```typescript
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }
 
}
 
// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}
```

下面創建一個類，結合了這兩個mixins。下面來看一下具體是怎麼操作的。

```typescript
class SmartObject implements Disposable, Activatable {}
```

首先應該注意到的是，沒使用『extends'而是使用『implements'。把類當成了接口，僅使用Disposable和Activatable的類型而非其實現。這意味著我們需要在類裡面實現接口。但是這是我們在用mixin時想避免的。

我們可以這麼做來達到目的，為將要mixin進來的屬性方法創建出佔位屬性。這告訴編譯器這些成員在運行時是可用的。這樣就能使用mixin帶來的便利，雖說需要提前定義一些佔位屬性。

```typescript
// Disposable
isDisposed: boolean = false;
dispose: () => void;
// Activatable
isActive: boolean = false;
activate: () => void;
deactivate: () => void;
```

最後，把mixins混入定義的類，完成全部實現部分。

```typescript
applyMixins(SmartObjet, [Disposable, Activatable])
```

最後，創建這個幫助函數，幫我們做混入操作。它會遍歷mixins上的所有屬性，並複製到目標上去，把之前的佔位屬性替換成真正的實現代碼。

```typescript
function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        })
    }); 
}
```

## <a name="9"></a>聲明合併

TypeScript有一些獨特的概念，有的是因為我們需要描述JavaScript頂級物件的類型發生了什麼變化。這其中之一叫做『聲明合併』。理解了這個概念對於你使用TypeScript去操作現有的JavaScript是大有幫助的。同時，也開啟通往更多高級抽象概念的大門。

首先，在瞭解怎麼進行聲明合併之前，讓我們先看一下什麼叫做『聲明合併』。

在這個手冊裡，聲明合併是指編譯器會把兩個相同的名字的聲明合併成一個單獨的聲明。合併後的聲明同時具有那兩個被合併的聲明的特性。聲明合並不限於只合併兩個，任意數量都可以。

### <a name="9.1"></a>基礎概念

Typescript裡聲明可用於三個地方：命名空間/模組，類型或者值。創建命名空間/模組的聲明可以通過（.）標識符訪問其中的類型。創建類型的聲明就是用給定的名字創建相應類型。最後，創建值的聲明是在生成的JavaScript裡存在的那部分（比如：函數和變量）。

| Declaration Type | Namespace | Type | Value |
|------------------|:---------:|:----:|:-----:|
| Module           |     X     |      |   X   |
| Class            |           |   X  |   X   |
| Interface        |           |   X  |       |
| Function         |           |      |   X   |
| Variable         |           |      |   X   |

理解每個聲明創建了什麼，有助於理解當聲明合併時什麼東西被合併了。

### <a name="9.2"></a>合併接口

最簡單最常見的就是合併接口，聲明合併的種類是：接口合併。從根本上說，合併的機制是把各自聲明裡的成員放進一個同名的單一接口裡。

```typescript
interface Box {
    height: number;
    width: number;
}

interface Box {
    scale: number;
}

var box: Box = {height: 5, width: 6, scale: 10};
```

接口中非函數的成員必須是唯一的。如果多個接口中具有相同名字的非函數成員就會報錯。

對於函數成員，每個同名函數聲明都會被當成這個函數的一個重載。

需要注意的是，接口A與它後面的接口A（把這個接口叫做A'）合併時，A'中的重載函數具有更高的優先級。

如下例所示：

```typescript
interface Document {
    createElement(tagName: any): Element;
}
interface Document {
    createElement(tagName: string): HTMLElement;
}
interface Document {
    createElement(tagName: "div"): HTMLDivElement; 
    createElement(tagName: "span"): HTMLSpanElement;
    createElement(tagName: "canvas"): HTMLCanvasElement;
}
```

這三個接口合併成一個聲明。注意每組接口裡的聲明順序保持不變，只是靠後的接口會出現在它前面的接口聲明之前。

```typescript
interface Document {
    createElement(tagName: "div"): HTMLDivElement; 
    createElement(tagName: "span"): HTMLSpanElement;
    createElement(tagName: "canvas"): HTMLCanvasElement;
    createElement(tagName: string): HTMLElement;
    createElement(tagName: any): Element;
}
```

### <a name="9.3"></a>合併模組

與接口相似，同名的模組也會合併其成員。模組會創建出命名空間和值，我們需要知道這兩者都是怎麼合併的。

命名空間的合併，模組導出的同名接口進行合併，構成單一命名空間內含合併後的接口。

值的合併，如果當前已經存在給定名字的模組，那麼後來的模組的導出成員會被加到已經存在的那個模組裡。

『Animals'聲明合併示例：

```typescript
module Animals {
    export class Zebra { }
}

module Animals {
    export interface Legged { numberOfLegs: number; }
    export class Dog { }
}
```

等同於

```typescript
module Animals {
    export interface Legged { numberOfLegs: number; }
    
    export class Zebra { }
    export class Dog { }
}
```

除了這些合併外，你還需要瞭解非導出成員是如何處理的。非導出成員僅在其原始存在於的模組（未合併的）之內可見。這就是說合併之後，從其它模組合併進來的成員無法訪問非導出成員了。

下例提供了更清晰的說明：

```typescript
module Animal {
    var haveMuscles = true;

    export function animalsHaveMuscles() {
        return haveMuscles;
    }
}

module Animal {
    export function doAnimalsHaveMuscles() {
        return haveMuscles;  // <-- error, haveMuscles is not visible here
    }
}
```

因為haveMuscles並沒有導出，只有animalsHaveMuscles函數共享了原始未合併的模組可以訪問這個變量。doAnimalsHaveMuscles函數雖是合併模組的一部分，但是訪問不了未導出的成員。

### <a name="9.4"></a>模組與類和函數和枚舉類型合併

模組可以與其它類型的聲明進行合併。只要模組的定義符合將要合併類型的定義。合併結果包含兩者的聲明類型。Typescript使用這個功能去實現一些JavaScript裡的設計模式。

首先，嘗試將模組和類合併。這讓我們可以定義內部類。

```typescript
class Album {
    label: Album.AlbumLabel;
}
module Album {
    export class AlbumLabel { }
}
```

合併規則與上面『合併模組』小節裡講的規則一致，我們必須導出AlbumLabel類，好讓合併的類能訪問。合併結果是一個類並帶有一個內部類。你也可以使用模組為類增加一些靜態屬性。

除了內部類的模式，你在JavaScript裡，創建一個函數稍後擴展它增加一些屬性也是很常見的。Typescript使用聲明合併來達到這個目的並保證類型安全。

```typescript
function buildLabel(name: string): string {
    return buildLabel.prefix + name + buildLabel.suffix;
}

module buildLabel {
    export var suffix = "";
    export var prefix = "Hello, ";
}

alert(buildLabel("Sam Smith"));
```

相似的，模組可以用來擴展枚舉型：

```typescript
enum Color {
    red = 1,
    green = 2,
    blue = 4
}

module Color {
    export function mixColor(colorName: string) {
        if (colorName == "yellow") {
            return Color.red + Color.green;
        }
        else if (colorName == "white") {
            return Color.red + Color.green + Color.blue;
        }
        else if (colorName == "magenta") {
            return Color.red + Color.blue;
        }
        else if (colorName == "cyan") {
            return Color.green + Color.blue;
        }
    }
}
```

### <a name="9.5"></a>非法的合併

並不是所有的合併都被允許。現在，類不能與類合併，變量與類型不能合併，接口與類不能合併。想要模仿類的合併，請參考[Mixins in TypeScript](https://typescript.codeplex.com/wikipage?title=Mixins%20in%20TypeScript&referringTitle=Declaration%20Merging)。

## <a name="10"></a>類型推論

這節介紹TypeScript裡的類型推論。即，類型是在哪裡如何被推斷的。

### <a name="10.1"></a>基礎

TypeScript裡，在有些沒有明確指出類型的地方，類型推論會幫助提供類型。如下面的例子

```typescript
var x = 3;
```

變量x的類型被推斷為數字。這種推斷髮生在初始化變量和成員，設置默認參數值和決定函數返回值時。

大多數情況下，類型推論是直截了當地。後面的小節，我們會瀏覽類型推論時的細微差別。

### <a name="10.2"></a>最佳通用類型

當需要從幾個表達式中推斷類型時候，會使用這些表達式的類型來推斷出一個最合適的通用類型。例如，

```typescript
var x = [0, 1, null];
```

為了推斷x的類型，我們必須考慮所有元素的類型。這裡有兩種選擇：數字和null。計算通用類型算法會考慮所有的候選類型，並給出一個兼容所有候選類型的類型。

由於最終的通用類型取自候選類型，有些時候候選類型共享相同的通用類型，但是卻沒有一個類型能做為所有候選類型的類型。例如：

```typescript
var zoo = [new Rhino(), new Elephant(), new Snake()];
```

這裡，我們想讓zoo被推斷為`Animal[]`類型，但是這個陣列裡沒有物件是`Animal`類型的，因此不能推斷出這個結果。
為了更正，當候選類型不能使用的時候我們需要明確的指出類型：

```typescript
var zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()];
```

如果沒有找到最佳通用類型的話，類型推論的結果是空物件類型，`{}`。
因為這個類型沒有任何成員，所以訪問其成員的時候會報錯。

### <a name="10.3"></a>上下文類型

TypeScript類型推論也可能按照相反的方向進行。
這被叫做「按上下文歸類」。按上下文歸類會發生在表達式的類型與所處的位置相關時。比如：

```typescript
window.onmousedown = function(mouseEvent) { 
    console.log(mouseEvent.buton);  //<- Error  
};
```

這個例子會得到一個類型錯誤，TypeScript類型檢查器使用`Window.onmousedown`函數的類型來推斷右邊函數表達式的類型。
因此，就能推斷出`mouseEvent`參數的類型了。
如果函數表達式不是在上下文類型的位置，`mouseEvent`參數的類型需要指定為`any`，這樣也不會報錯了。

如果上下文類型表達式包含了明確的類型信息，上下文的類型被忽略。
重寫上面的例子：

```typescript
window.onmousedown = function(mouseEvent: any) { 
    console.log(mouseEvent.buton);  //<- Now, no error is given  
};
```

這個函數表達式有明確的參數類型註解，上下文類型被忽略。
這樣的話就不報錯了，因為這裡不會使用到上下文類型。

上下文歸類會在很多情況下使用到。
通常包含函數的參數，賦值表達式的右邊，類型斷言，物件成員和陣列字面量和返回值語句。
上下文類型也會做為最佳通用類型的候選類型。比如：

```typescript
function createZoo(): Animal[] {
    return [new Rhino(), new Elephant(), new Snake()];
}
```

這個例子裡，最佳通用類型有4個候選者：`Animal`，`Rhino`，`Elephant`和`Snake`。
當然，`Animal`會被做為最佳通用類型。

## <a name="11"></a>類型兼容性

TypeScript裡的類型兼容性是以結構性子類型來判斷的。結構性類型是完全根據成員關聯類型的一種方式。與正常的類型判斷不同。看下面的例子：

```typescript
interface Named {
    name: string;
}

class Person {
    name: string;
}

var p: Named;
// OK, because of structural typing
p = new Person();
```

在正常使用類型的語言像C#或Java中，這段代碼會報錯，因為Person類沒有明確說明其實現了Named接口。

TypeScript的結構性子類型是根據JavaScript代碼的通常寫法來設計的。因為JavaScript裡常用匿名物件像函數表達式或物件字面量，所以用結構性類型系統來描述這些類型比使用正常的類型系統更好。

### <a name="11.0"></a>關於穩定性的注意事項

TypeScript的類型系統允許那些在編譯階段無法否認其安全性的操作。當一個類型系統具有此屬性時，被當做是「不穩定」的。TypeScript裡允許這種不穩定行為發生的地方是經過仔細考慮的。通過這篇文章，我們會解釋什麼時候會發生這種情況和其背景。

### <a name="11.1"></a>開始

TypeScript結構化類型系統的基本規則是，如果x與y兼容，那麼y至少具有與x相同的屬性。比如：

```typescript
interface Named {
    name: string;
}

var x: Named;
// y's inferred type is { name: string; location: string; }
var y = { name: 'Alice', location: 'Seattle' };
x = y;
```

這裡要檢查y是否能賦值給x，編譯器x中的每個屬性，看是否能在y中也找到對應屬性。在這個例子中，y必須包含名字是『name'的string類型成員。y滿足條件，因此賦值正確。

檢查函數參數時使用相同的規則：

```typescript
function greet(n: Named) {
    alert('Hello, ' + n.name);
}
greet(y); // OK
```

注意，『y'有個額外的『location'屬性，但這不會引發錯誤。只有目標類型（這裡是『Named'）的成員會被一一檢查是否兼容。

這個比較過程是遞歸進行的，檢查每個成員及子成員。

### <a name="11.2"></a>比較兩個函數

比較原始類型和物件類型時是容易理解的，問題是如何判斷兩個函數是兼容的。讓我們以兩個函數開始，它們僅有參數列表不同：

```typescript
var x = (a: number) => 0;
var y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```

要查看x是否能賦值給y，首先看它們的參數列表。x的每個參數必須能在y裡找到對應類型的參數。注意的是參數的名字相同與否無所謂，只看它們的類型。這裡，x的每個參數在y中都能找到對應的參數，所以允許賦值。

第二個賦值錯誤，因為y有個必需的第二個參數，但是x並沒有，所以不允許賦值。

你可能會疑惑為什麼允許『忽略』參數，像例子y=x中那樣。原因是忽略額外的參數在JavaScript裡是很常見的。例如，Array#forEach給回調函數傳3個參數：陣列元素，索引和整個陣列。儘管如此，傳入一個只使用第一個參數的回調函數也是很有用的：

```typescript
var items = [1, 2, 3];

// Don't force these extra arguments
items.forEach((item, index, array) => console.log(item));

// Should be OK!
items.forEach((item) => console.log(item));
```

下面來看看如何處理返回值類型，創建兩個僅是返回值類型不同的函數：

```typescript
var x = () => ({name: 'Alice'});
var y = () => ({name: 'Alice', location: 'Seattle'});

x = y; // OK
y = x; // Error because x() lacks a location property
```

類型系統強制源函數的返回值類型必須是目標函數返回值類型的子類型。

#### <a name="11.2.1"></a>函數參數雙向協變

當比較函數參數類型時，只有源函數參數能夠賦值給目標函數或反過來才匹配成功。這是不穩定的，因為調用者可能會被給予一個函數，它接受一個更確切類型，但是調用函數使用不那麼確切的類型。實際上，這極少會發生錯誤，並且能夠實現很多JavaScript裡的常見模式。例如：

```typescript
enum EventType { Mouse, Keyboard }

interface Event { timestamp: number; }
interface MouseEvent extends Event { x: number; y: number }
interface KeyEvent extends Event { keyCode: number }

function listenEvent(eventType: EventType, handler: (n: Event) => void) {
    /* ... */
}

// Unsound, but useful and common
listenEvent(EventType.Mouse, (e: MouseEvent) => console.log(e.x + ',' + e.y));

// Undesirable alternatives in presence of soundness
listenEvent(EventType.Mouse, (e: Event) => console.log((<MouseEvent>e).x + ',' + (<MouseEvent>e).y));
listenEvent(EventType.Mouse, <(e: Event) => void>((e: MouseEvent) => console.log(e.x + ',' + e.y)));

// Still disallowed (clear error). Type safety enforced for wholly incompatible types
listenEvent(EventType.Mouse, (e: number) => console.log(e));
```

#### <a name="11.2.2"></a>可選參數及剩餘參數

比較函數兼容性的時候，可選參數與必須參數是可交換的。

當一個函數有剩餘參數時，它被當做無限個可選參數。

這對於類型系統來說是不穩定的，但從運行時的角度來看，可選參數一般來說是不強制的，因為對於大多數函數來說相當於傳遞了一些『undefinded'。

有一個好的例子，常見的函數接收一個回調函數並用對於程序員來說是可預知的參數但對類型系統來說是不確定的參數來調用：

```typescript
function invokeLater(args: any[], callback: (...args: any[]) => void) {
    /* ... Invoke callback with 'args' ... */
}

// Unsound - invokeLater "might" provide any number of arguments
invokeLater([1, 2], (x, y) => console.log(x + ', ' + y));

// Confusing (x and y are actually required) and undiscoverable
invokeLater([1, 2], (x?, y?) => console.log(x + ', ' + y));
```

#### <a name="11.2.3"></a>函數重載

對於有重載的函數，源函數的每個重載都要在目標函數上找到對應的函數簽名。這確保了目標函數可以在所有源函數可調用的地方調用。對於特殊的函數重載簽名不會用來做兼容性檢查。

### <a name="11.3"></a>枚舉

枚舉類型與數字類型兼容，並且數字類型與枚舉類型兼容。不同枚舉類型之間是不兼容的。比如，

```typescript
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

var status = Status.Ready;
status = Color.Green;  //error
```

### <a name="11.4"></a>類

類與物件字面量和接口差不多，但有一點不同：類有靜態部分和實例部分的類型。比較兩個類類型的物件時，只有實例的成員會被比較。靜態成員和構造函數不在比較的範圍內。

```typescript
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { }
}

class Size {
    feet: number;
    constructor(numFeet: number) { }
}

var a: Animal;
var s: Size;

a = s;  //OK
s = a;  //OK
```

#### <a name="11.4.1"></a>類的私有成員

私有成員會影響兼容性判斷。當類的實例用來檢查兼容時，如果它包含一個私有成員，那麼目標類型必須包含來自同一個類的這個私有成員。這允許子類賦值給父類，但是不能賦值給其它有同樣類型的類。

### <a name="11.5"></a>泛型

因為TypeScript是結構性的類型系統，類型參數隻影響使用其做為類型一部分的結果類型。比如，

```typescript
interface Empty<T> {
}
var x: Empty<number>;
var y: Empty<string>;

x = y;  // okay, y matches structure of x
```

上面代碼裡，x和y是兼容的，因為它們的結構使用類型參數時並沒有什麼不同。把這個例子改變一下，增加一個成員，就能看出是如何工作的了：

```typescript
interface NotEmpty<T> {
    data: T;
}
var x: NotEmpty<number>;
var y: NotEmpty<string>;

x = y;  // error, x and y are not compatible
```

在這裡，泛型類型在使用時就好比不是一個泛型類型。

對於沒指定泛型類型的泛型參數時，會把所有泛型參數當成『any'比較。然後用結果類型進行比較，就像上面第一個例子。

比如，

```typescript
var identity = function<T>(x: T): T { 
    // ...
}

var reverse = function<U>(y: U): U {
    // ...
}

identity = reverse;  // Okay because (x: any)=>any matches (y: any)=>any
```

### <a name="11.6"></a>高級主題

#### <a name="11.6.1"></a>子類型與賦值

目前為止，我們使用了`兼容性`，它在語言規範裡沒有定義。
在TypeScript裡，有兩種類型的兼容性：子類型與賦值。
它們的不同點在於，賦值擴展了子類型兼容，允許給`any`賦值或從`any`取值和允許數字賦值給枚舉類型或枚舉類型賦值給數字。

語言裡的不同地方分別使用了它們之中的機制。
實際上，類型兼容性是由賦值兼容性來控制的甚至在`implements`和`extends`語句裡。
更多信息，請參閱[TypeScript語言規範](http://go.microsoft.com/fwlink/?LinkId=267121).

## <a name="12"></a>書寫.d.ts文件

當使用外部JavaScript庫或新的宿主API時，你需要一個聲明文件（.d.ts）定義程序庫的shape。這個手冊包含了寫.d.ts文件的高級概念，並帶有一些例子，告訴你怎麼去寫一個聲明文件。

### <a name="12.1"></a>指導與說明

#### <a name="12.1.1"></a>流程

最好從程序庫的文檔開始寫.d.ts文件，而不是代碼。這樣保證不會被具體實現所幹擾，而且相比於JS代碼更易讀。下面的例子會假設你正在參照文檔寫聲明文件。

#### <a name="12.1.2"></a>命名空間

當定義接口（例如：「options」物件），你會選擇是否將這些類型放進命名空間裡。
這主要是靠主觀判斷 -- 使用的人主要是用這些類型聲明變量和參數，並且類型命名不會引起命名衝突，放在全局命名空間裡更好。
如果類型不是被直接使用，或者沒法起一個唯一的名字的話，就使用命名空間來避免與其它類型發生衝突。

#### <a name="12.1.3"></a>回調函數

許多JavaScript庫接收一個函數做為參數，之後傳入已知的參數來調用它。當為這些類型與函數簽名的時候，不要把這個參數標記成可選參數。正確的思考方式是「會提供什麼樣的參數？」，不是「會使用到什麼樣的參數？」。TypeScript 0.9.7+不會強制這種可選參數的使用，參數可選的雙向協變可以被外部的linter強制執行。

#### <a name="12.1.4"></a>擴展與聲明合併

寫聲明文件的時候，要記住TypeScript擴展現有物件的方式。你可以選擇用匿名類型或接口類型的方式聲明一個變量：

**匿名類型var**

```typescript
declare var MyPoint: { x: number; y: number; };
```

**接口類型var**

```typescript
interface SomePoint { x: number; y: number; }
declare var MyPoint: SomePoint;
```

從使用者角度來講，它們是相同的，但是SomePoint類型能夠通過接口合併來擴展：

```typescript
interface SomePoint { z: number; }
MyPoint.z = 4; // OK
```

是否想讓你的聲明是可擴展的取決於主觀判斷。通常來講，儘量符合library的意圖。

#### <a name="12.1.5"></a>類的分解

TypeScript的類會創建出兩個類型：實例類型，定義了類型的實例具有哪些成員；構造函數類型，定義了類構造函數具有哪些類型。構造函數類型也被稱做類的靜態部分類型，因為它包含了類的靜態成員。

你可以使用typeof關鍵字來拿到類靜態部分類型，在寫聲明文件時，想要把類明確的分解成實例類型和靜態類型時是有用且必要的。

下面是一個例子，從使用者的角度來看，這兩個聲明是等同的：

**標準版**

```typescript
class A {
    static st: string;
    inst: number;
    constructor(m: any) {}
}
```

**分解版**

```typescript
interface A_Static {
    new(m: any): A_Instance;
    st: string;
}
interface A_Instance {
    inst: number;
}
declare var A: A_Static;
```

這裡的利弊如下：

* 標準方式可以使用extends來繼承；分解的類不能。這可能會在未來版本的TypeScript裡改變：是否允許任何的extends表達式
* 都允許之後為類添加靜態成員
* 允許為分解的類再添加實例成員，標準版不允許
* 使用分解類的時候，為成員起合理的名字

#### <a name="12.1.6"></a>命名規則

一般來講，不要給接口加I前綴（比如：IColor）。類為TypeScript裡的接口類型比C#或Java裡的意義更為廣泛，IFoo命名不利於這個特點。

### <a name="12.2"></a>例子

下面進行例子部分。對於每個例子，先是使用使用方法，然後是類型聲明。如果有多個好的聲明表示方法，會列出多個。

#### <a name="12.2。1"></a>參數物件

**使用方法**

```javascript
animalFactory.create("dog");
animalFactory.create("giraffe", { name: "ronald" });
animalFactory.create("panda", { name: "bob", height: 400 });

// 錯誤：如果提供選項，那麼必須包含name
animalFactory.create("cat", { height: 32 });
```

**類型**

```typescript
namespace animalFactory {
    interface AnimalOptions {
        name: string;
        height?: number;
        weight?: number;
    }
    function create(name: string, animalOptions?: AnimalOptions): Animal;
}
```

#### <a name="12.2.2"></a>帶屬性的函數

**使用方法**

```javascript
zooKeeper.workSchedule = "morning";
zooKeeper(giraffeCage);
```

**類型**

```typescript
// 注意：函數必須在命名空間之前
function zooKeeper(cage: AnimalCage);
namespace zooKeeper {
    var workSchedule: string;
}
```

#### <a name="12.2.3"></a>可以用new調用也可以直接調用的方法

**使用方法**

```javascript
var w = widget(32, 16);
var y = new widget("sprocket");
// w and y are both widgets
w.sprock();
y.sprock();
```

**類型**

```typescript
interface Widget {
    sprock(): void;
}

interface WidgetFactory {
    new(name: string): Widget;
    (width: number, height: number): Widget;
}

declare var widget: WidgetFactory;
```

#### <a name="12.2.4"></a>全局的/不清楚的Libraries

**使用方法**

```javascript
// Either
import x = require('zoo');
x.open();
// or
zoo.open();
```

**類型**

```typescript
namespace zoo {
  function open(): void;
}

declare module "zoo" {
    export = zoo;
}
```

#### <a name="12.2.5"></a>外部模組的單個複雜物件

**使用方法**

```javascript
// Super-chainable library for eagles
import eagle = require('./eagle');
// Call directly
eagle('bald').fly();
// Invoke with new
var eddie = new eagle(1000);
// Set properties
eagle.favorite = 'golden';
```

**類型**

```typescript
// Note: can use any name here, but has to be the same throughout this file
declare function eagle(name: string): eagle;
declare namespace eagle {
    var favorite: string;
    function fly(): void;
}
interface eagle {
    new(awesomeness: number): eagle;
}

export = eagle;
```

#### <a name="12.2.6"></a>回調函數

**使用方法**

```javascript
addLater(3, 4, (x) => console.log('x = ' + x));
```

**類型**

```typescript
// Note: 'void' return type is preferred here
function addLater(x: number, y: number, (sum: number) => void): void;
```

如果你想看其它模式的實現方式，請在[這裡](https://github.com/Microsoft/TypeScript-Handbook/issues)留言！
我們會儘可能地加到這裡來。
