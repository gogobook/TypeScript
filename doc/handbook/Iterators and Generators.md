# 介紹

[JSX](https://facebook.github.io/jsx/)是一種嵌入式的類似XML的語法。
它可以被轉換成合法的JavaScript，儘管轉換的語義是依據不同的實現而定的。
JSX因[React](http://facebook.github.io/react/)框架而流行，但是也被其它應用所使用。
TypeScript支持內嵌，類型檢查和將JSX直接編譯為JavaScript。

# 基本用法

想要使用JSX必須做兩件事：

1. 給文件一個`.tsx`擴展名
2. 啟用`jsx`選項

TypeScript具有兩種JSX模式：`preserve`和`react`。
這些模式只在代碼生成階段起作用 - 類型檢查並不受影響。
在`preserve`模式下生成代碼中會保留JSX以供後續的轉換操作使用（比如：[Babel](https://babeljs.io/)）。
另外，輸出文件會帶有`.jsx`擴展名。
`react`模式會生成`React.createElement`，在使用前不需要再進行轉換操作了，輸出文件的擴展名為`.js`。

模式        | 輸入      | 輸出                          | 輸出文件擴展名
-----------|-----------|------------------------------|----------------------
`preserve` | `<div />` | `<div />`                    | `.jsx`
`react`    | `<div />` | `React.createElement("div")` | `.js`

你可以通過在命令行裡使用`--jsx`標記或[tsconfig.json](./tsconfig.json.md)裡的選項來指定模式。

> *注意：`React`標識符是寫死的硬代碼，所以你必須保證React（大寫的R）是可用的。*
> *Note: The identifier `React` is hard-coded, so you must make React available with an uppercase R.*

# `as`操作符

回想一下怎麼寫類型斷言：

```ts
var foo = <foo>bar;
```

這裡我們斷言`bar`變量是`foo`類型的。
因為TypeScript也使用尖括號來表示類型斷言，JSX的語法帶來瞭解析的困難。因此，TypeScript在`.tsx`文件裡禁用了使用尖括號的類型斷言。

為了彌補`.tsx`裡的這個功能，新加入了一個類型斷言符號：`as`。
上面的例子可以很容易地使用`as`操作符改寫：

```ts
var foo = bar as foo;
```

`as`操作符在`.ts`和`.tsx`裡都可用，並且與其它類型斷言行為是等價的。

# 類型檢查

為了理解JSX的類型檢查，你必須首先理解固有元素與基於值的元素之間的區別。
假設有這樣一個JSX表達式`<expr />`，`expr`可能引用環境自帶的某些東西（比如，在DOM環境裡的`div`或`span`）或者是你自定義的組件。
這是非常重要的，原因有如下兩點：

1. 對於React，固有元素會生成字符串（`React.createElement("div")`），然而由你自定義的組件卻不會生成（`React.createElement(MyComponent)`）。
2. 傳入JSX元素裡的屬性類型的查找方式不同。
  固有元素屬性*本身*就支持，然而自定義的組件會自己去指定它們具有哪個屬性。

TypeScript使用[與React相同的規範](http://facebook.github.io/react/docs/jsx-in-depth.html#html-tags-vs.-react-components) 來區別它們。
固有元素總是以一個小寫字母開頭，基於值的元素總是以一個大寫字母開頭。

## 固有元素

固有元素使用特殊的接口`JSX.IntrinsicElements`來查找。
默認地，如果這個接口沒有指定，會全部通過，不對固有元素進行類型檢查。
然而，如果接口存在，那麼固有元素的名字需要在`JSX.IntrinsicElements`接口的屬性裡查找。
例如：

```ts
declare namespace JSX {
    interface IntrinsicElements {
        foo: any
    }
}

<foo />; // 正確
<bar />; // 錯誤
```

在上例中，`<foo />`沒有問題，但是`<bar />`會報錯，因為它沒在`JSX.IntrinsicElements`裡指定。

> 注意：你也可以在`JSX.IntrinsicElements`上指定一個用來捕獲所有字符串索引：
>```ts
>declare namespace JSX {
>    interface IntrinsicElements {
>        [elemName: string]: any;
>    }
>}
>```

## 基於值的元素

基於值的元素會簡單的在它所在的作用域裡按標識符查找。

```ts
import MyComponent from "./myComponent";

<MyComponent />; // 正確
<SomeOtherComponent />; // 錯誤
```

可以限制基於值的元素的類型。
然而，為了這麼做我們需要引入兩個新的術語：*元素類的類型*和*元素實例的類型*。

現在有`<Expr />`，*元素類的類型*為`Expr`的類型。
所以在上面的例子裡，如果`MyComponent`是ES6的類，那麼它的類類型就是這個類。
如果`MyComponent`是個工廠函數，類類型為這個函數。

一旦建立起了類類型，實例類型就確定了，為類類型調用簽名的返回值與構造簽名的聯合類型。
再次說明，在ES6類的情況下，實例類型為這個類的實例的類型，並且如果是工廠函數，實例類型為這個函數返回值類型。

```ts
class MyComponent {
  render() {}
}

// 使用構造簽名
var myComponent = new MyComponent();

// 元素類的類型 => MyComponent
// 元素實例的類型 => { render: () => void }

function MyFactoryFunction() {
  return {
    render: () => {
    }
  }
}

// 使用調用簽名
var myComponent = MyFactoryFunction();

// 元素類的類型 => FactoryFunction
// 元素實例的類型 => { render: () => void }
```

元素的實例類型很有趣，因為它必須賦值給`JSX.ElementClass`或拋出一個錯誤。
默認的`JSX.ElementClass`為`{}`，但是它可以被擴展用來限制JSX的類型以符合相應的接口。

```ts
declare namespace JSX JSX {
  interface ElementClass {
    render: any;
  }
}

class MyComponent {
  render() {}
}
function MyFactoryFunction() {
  return { render: () => {} }
}

<MyComponent />; // 正確
<MyFactoryFunction />; // 正確

class NotAValidComponent {}
function NotAValidFactoryFunction() {
  return {};
}

<NotAValidComponent />; // 錯誤
<NotAValidFactoryFunction />; // 錯誤
```

## 屬性類型檢查

屬性類型檢查的第一步是確定*元素屬性類型*。
這在固有元素和基於值的元素之間稍有不同。

對於固有元素，這是`JSX.IntrinsicElements`屬性的類型。

```ts
declare namespace JSX {
  interface IntrinsicElements {
    foo: { bar?: boolean }
  }
}

// `foo`的元素屬性類型為`{bar?: boolean}`
<foo bar />;
```

對於基於值的元素，就稍微複雜些。
它取決於先前確定的在元素實例類型上的某個屬性的類型。
至於該使用哪個屬性來確定類型取決於`JSX.ElementAttributesProperty`。
它應該使用單一的屬性來定義。
這個屬性名之後會被使用。

```ts
declare namespace JSX {
  interface ElementAttributesProperty {
    props; // 指定用來使用的屬性名
  }
}

class MyComponent {
  // 在元素實例類型上指定屬性
  props: {
    foo?: string;
  }
}

// `MyComponent`的元素屬性類型為`{foo?: string}`
<MyComponent foo="bar" />
```

元素屬性類型用於的JSX裡進行屬性的類型檢查。
支持可選屬性和必須屬性。

```ts
declare namespace JSX {
  interface IntrinsicElements {
    foo: { requiredProp: string; optionalProp?: number }
  }
}

<foo requiredProp="bar" />; // 正確
<foo requiredProp="bar" optionalProp={0} />; // 正確
<foo />; // 錯誤, 缺少 requiredProp
<foo requiredProp={0} />; // 錯誤, requiredProp 應該是字符串
<foo requiredProp="bar" unknownProp />; // 錯誤, unknownProp 不存在
<foo requiredProp="bar" some-unknown-prop />; // 正確, `some-unknown-prop`不是個合法的標識符
```

> 注意：如果一個屬性名不是個合法的JS標識符（像`data-*`屬性），並且它沒出現在元素屬性類型裡時不會當做一個錯誤。

延展操作符也可以使用：

```JSX
var props = { requiredProp: 'bar' };
<foo {...props} />; // 正確

var badProps = {};
<foo {...badProps} />; // 錯誤
```

# JSX結果類型

默認地JSX表達式結果的類型為`any。
你可以自定義這個類型，通過指定`JSX.Element`接口。
然而，不能夠從接口裡檢索元素，屬性或JSX的子元素的類型信息。
它是一個黑盒。

# 嵌入的表達式

JSX允許你使用`{ }`標籤來內嵌表達式。

```JSX
var a = <div>
  {['foo', 'bar'].map(i => <span>{i / 2}</span>)}
</div>
```

上面的代碼產生一個錯誤，因為你不能用數字來除以一個字符串。
輸出如下，若你使用了`preserve`選項：

```JSX
var a = <div>
  {['foo', 'bar'].map(function (i) { return <span>{i / 2}</span>; })}
</div>
```

# React整合

要想一起使用JSX和React，你應該使用[React類型定義](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/react)。
這些類型聲明定義了`JSX`合適命名空間來使用React。

```ts
/// <reference path="react.d.ts" />

interface Props {
  foo: string;
}

class MyComponent extends React.Component<Props, {}> {
  render() {
    return <span>{this.props.foo}</span>
  }
}

<MyComponent foo="bar" />; // 正確
<MyComponent foo={0} />; // 錯誤
```