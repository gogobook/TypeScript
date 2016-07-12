# 介紹

傳統的JavaScript程序使用函數和基於原型的繼承來創建可重用的組件，但這對於熟悉使用面向物件方式的程序員來說有些棘手，因為他們用的是基於類的繼承並且物件是從類構建出來的。
從ECMAScript 2015，也就是ECMAScript 6，JavaScript程序將可以使用這種基於類的面向物件方法。
在TypeScript裡，我們允許開發者現在就使用這些特性，並且編譯後的JavaScript可以在所有主流瀏覽器和平台上運行，而不需要等到下個JavaScript版本。

# 類

下面看一個使用類的例子：

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
```

如果你使用過C#或Java，你會對這種語法非常熟悉。
我們聲明一個`Greeter`類。這個類有3個成員：一個叫做`greeting`的屬性，一個構造函數和一個`greet`方法。

你會注意到，我們在引用任何一個類成員的時候都用了`this`。
它表示我們訪問的是類的成員。

最後一行，我們使用`new`構造了Greeter類的一個實例。
它會調用之前定義的構造函數，創建一個`Greeter`類型的新物件，並執行構造函數初始化它。

# 繼承

在TypeScript裡，我們可以使用常用的面向物件模式。
當然，基於類的程序設計中最基本的模式是允許使用繼承來擴展一個類。

看下面的例子：

```ts
class Animal {
    name:string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 45) {
        console.log("Galloping...");
        super.move(distanceInMeters);
    }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

這個例子展示了TypeScript中繼承的一些特徵，與其它語言類似。
我們使用`extends`來創建子類。你可以看到`Horse`和`Snake`類是基類`Animal`的子類，並且可以訪問其屬性和方法。

包含constructor函數的派生類必須調用`super()`，它會執行基類的構造方法。

這個例子演示了如何在子類裡可以重寫父類的方法。
`Snake`類和`Horse`類都創建了`move`方法，重寫了從`Animal`繼承來的`move`方法，使得`move`方法根據不同的類而具有不同的功能。
注意，即使`tom`被聲明為`Animal`類型，因為它的值是`Horse`，`tom.move(34)`調用`Horse`裡的重寫方法：

```text
Slithering...
Sammy the Python moved 5m.
Galloping...
Tommy the Palomino moved 34m.
```

# 公共，私有與受保護的修飾符

## 默認為公有

在上面的例子裡，我們可以自由的訪問程序裡定義的成員。
如果你對其它語言中的類比較瞭解，就會注意到我們在之前的代碼裡並沒有使用`public`來做修飾；例如，C#要求必須明確地使用`public`指定成員是可見的。
在TypeScript裡，每個成員默認為`public`的。

你也可以明確的將一個成員標記成`public`。
我們可以用下面的方式來重寫上面的`Animal`類：

```ts
class Animal {
    public name: string;
    public constructor(theName: string) { this.name = theName; }
    public move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

## 理解`private`

當成員被標記成`private`時，它就不能在聲明它的類的外部訪問。比如：

```ts
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

new Animal("Cat").name; // Error: 'name' is private;
```

TypeScript使用的是結構性類型系統。
當我們比較兩種不同的類型時，並不在乎它們從哪兒來的，如果所有成員的類型都是兼容的，我們就認為它們的類型是兼容的。

然而，當我們比較帶有`private`或`protected`成員的類型的時候，情況就不同了。
如果其中一個類型裡包含一個`private`成員，那麼只有當另外一個類型中也存在這樣一個`private`成員， 並且它們是來自同一處聲明時，我們才認為這兩個類型是兼容的。
對於`protected`成員也使用這個規則。

下面來看一個例子，詳細的解釋了這點：

```ts
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
    constructor() { super("Rhino"); }
}

class Employee {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

let animal = new Animal("Goat");
let rhino = new Rhino();
let employee = new Employee("Bob");

animal = rhino;
animal = employee; // Error: Animal and Employee are not compatible
```

這個例子中有`Animal`和`Rhino`兩個類，`Rhino`是`Animal`類的子類。
還有一個`Employee`類，其類型看上去與`Animal`是相同的。
我們創建了幾個這些類的實例，並相互賦值來看看會發生什麼。
因為`Animal`和`Rhino`共享了來自`Animal`裡的私有成員定義`private name: string`，因此它們是兼容的。
然而`Employee`卻不是這樣。當把`Employee`賦值給`Animal`的時候，得到一個錯誤，說它們的類型不兼容。
儘管`Employee`裡也有一個私有成員`name`，但它明顯不是`Animal`裡面定義的那個。

## 理解`protected`

`protected`修飾符與`private`修飾符的行為很相似，但有一點不同，`protected`成員在派生類中仍然可以訪問。例如：

```ts
class Person {
    protected name: string;
    constructor(name: string) { this.name = name; }
}

class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name)
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
console.log(howard.getElevatorPitch());
console.log(howard.name); // error
```

注意，我們不能在`Person`類外使用`name`，但是我們仍然可以通過`Employee`類的實例方法訪問，因為`Employee`是由`Person`派生出來的。

## 參數屬性

在上面的例子中，我們不得不定義一個受保護的成員`name`和一個構造函數參數`theName`在`Person`類裡，並且立刻給`name`和`theName`賦值。
這種情況經常會遇到。*參數屬性*可以方便地讓我們在一個地方定義並初始化一個成員。
下面的例子是對之前`Animal`類的修改版，使用了參數屬性：

```ts
class Animal {
    constructor(private name: string) { }
    move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

注意看我們是如何捨棄了`theName`，僅在構造函數裡使用`private name: string`參數來創建和初始化`name`成員。
我們把聲明和賦值合併至一處。

參數屬性通過給構造函數參數添加一個訪問限定符來聲明。
使用`private`限定一個參數屬性會聲明並初始化一個私有成員；對於`public`和`protected`來說也是一樣。

# 存取器

TypeScript支持getters/setters來截取對物件成員的訪問。
它能幫助你有效的控制對物件成員的訪問。

下面來看如何把一類改寫成使用`get`和`set`。
首先是一個沒用使用存取器的例子。

```ts
class Employee {
    fullName: string;
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    console.log(employee.fullName);
}
```

我們可以隨意的設置`fullName`，這是非常方便的，但是這也可能會帶來麻煩。

下面這個版本裡，我們先檢查用戶密碼是否正確，然後再允許其修改employee。
我們把對`fullName`的直接訪問改成了可以檢查密碼的`set`方法。
我們也加了一個`get`方法，讓上面的例子仍然可以工作。

```ts
let passcode = "secret passcode";

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
            console.log("Error: Unauthorized update of employee!");
        }
    }
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

我們可以修改一下密碼，來驗證一下存取器是否是工作的。當密碼不對時，會提示我們沒有權限去修改employee。

注意：若要使用存取器，要求設置編譯器輸出目標為ECMAScript 5或更高。

# 靜態屬性

到目前為止，我們只討論了類的實例成員，那些僅當類被實例化的時候才會被初始化的屬性。
我們也可以創建類的靜態成員，這些屬性存在於類本身上面而不是類的實例上。
在這個例子裡，我們使用`static`定義`origin`，因為它是所有網格都會用到的屬性。
每個實例想要訪問這個屬性的時候，都要在origin前面加上類名。
如同在實例屬性上使用`this.`前綴來訪問屬性一樣，這裡我們使用`Grid.`來訪問靜態屬性。

```ts
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

# 抽象類

抽象類是供其它類繼承的基類。
他們一般不會直接被實例化。
不同於接口，抽象類可以包含成員的實現細節。
`abstract`關鍵字是用於定義抽象類和在抽象類內部定義抽象方法。

```ts
abstract class Animal {
    abstract makeSound(): void;
    move(): void {
        console.log('roaming the earch...');
    }
}
```

抽象類中的抽象方法不包含具體實現並且必須在派生類中實現。
抽象方法的語法與接口方法相似。
兩者都是定義方法簽名不包含方法體。
然而，抽象方法必須使用`abstract`關鍵字並且可以包含訪問符。

```ts
abstract class Department {

    constructor(public name: string) {
    }

    printName(): void {
        console.log('Department name: ' + this.name);
    }

    abstract printMeeting(): void; // 必須在派生類中實現
}

class AccountingDepartment extends Department {

    constructor() {
        super('Accounting and Auditing'); // constructors in derived classes must call super()
    }

    printMeeting(): void {
        console.log('The Accounting Department meets each Monday at 10am.');
    }

    generateReports(): void {
        console.log('Generating accounting reports...');
    }
}

let department: Department; // ok to create a reference to an abstract type
department = new Department(); // error: cannot create an instance of an abstract class
department = new AccountingDepartment(); // ok to create and assign a non-abstract subclass
department.printName();
department.printMeeting();
department.generateReports(); // error: method doesn't exist on declared abstract type
```

# 高級技巧

## 構造函數

當你在TypeScript裡定義類的時候，實際上同時定義了很多東西。
首先是類的*實例*的類型。

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter: Greeter;
greeter = new Greeter("world");
console.log(greeter.greet());
```

在這裡，我們寫了`let greeter: Greeter`，意思是`Greeter`類實例的類型是`Greeter`。
這對於用過其它面向物件語言的程序員來講已經是老習慣了。

我們也創建了一個叫做*構造函數*的值。
這個函數會在我們使用`new`創建類實例的時候被調用。
下面我們來看看，上面的代碼被編譯成JavaScript後是什麼樣子的：

```ts
let Greeter = (function () {
    function Greeter(message) {
        this.greeting = message;
    }
    Greeter.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    return Greeter;
})();

let greeter;
greeter = new Greeter("world");
console.log(greeter.greet());
```

上面的代碼裡，`let Greeter`將被賦值為構造函數。
當我們使用`new`並執行這個函數後，便會得到一個類的實例。
這個構造函數也包含了類的所有靜態屬性。
換個角度說，我們可以認為類具有實例部分與靜態部分這兩個部分。

讓我們來改寫一下這個例子，看看它們之前的區別：

```ts
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

let greeter1: Greeter;
greeter1 = new Greeter();
console.log(greeter1.greet());

let greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";
let greeter2:Greeter = new greeterMaker();
console.log(greeter2.greet());
```

這個例子裡，`greeter1`與之前看到的一樣。
我們實例化`Greeter`類，並使用這個物件。
與我們之前看到的一樣。

再之後，我們直接使用類。
我們創建了一個叫做`greeterMaker`的變量。
這個變量保存了這個類或者說保存了類構造函數。
然後我們使用`typeof Greeter`，意思是取Greeter類的類型，而不是實例的類型。
或者更確切的說，"告訴我`Greeter`標識符的類型"，也就是構造函數的類型。
這個類型包含了類的所有靜態成員和構造函數。
之後，就和前面一樣，我們在`greeterMaker`上使用`new`，創建`Greeter`的實例。

## 把類當做接口使用

如上一節裡所講的，類定義會創建兩個東西：類實例的類型和一個構造函數。
因為類可以創建出類型，所以你能夠在可以使用接口的地方使用類。

```ts
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3};
```
