這個快速上手指南將會教你如何將TypeScript和[React](http://facebook.github.io/react/)還有[webpack](http://webpack.github.io/)連結在一起使用。

我們假設已經在使用[Node.js](https://nodejs.org/)和[npm](https://www.npmjs.com/)。

# 初始化項目結構

讓我們新建一個目錄。
將會命名為`proj`，但是你可以改成任何你喜歡的名字。

```shell
mkdir proj
cd proj
```

我們會像下面的結構組織我們的工程：

```text
proj/
   +- src/
   |    +- components/
   |
   +- dist/
```

TypeScript文件會放在`src`文件夾裡，通過TypeScript編譯器編譯，然後經webpack處理，最後生成一個`bundle.js`文件放在`dist`目錄下。
我們自定義的組件將會放在`src/components`文件夾下。

下面來創建基本結構：

```shell
mkdir src
cd src
mkdir components
cd ..
mkdir dist
```

# 初始化工程

現在把這個目錄變成npm包。

```shell
npm init
```

你會看到一些提示。
你可以使用默認項除了開始腳本。
使用`./dist/bundle.js`做為開始腳本。
當然，你也可以隨時到生成的`package.json`文件裡修改。

# 安裝依賴

首先確保TypeScript，typings和webpack已經全局安裝了。

```shell
npm install -g typescript typings webpack
```

Webpack這個工具可以將你的所有代碼和可選擇地將依賴捆綁成一個單獨的`.js`文件。
[Typings](https://www.npmjs.com/package/typings)是一個包管理器，它是用來獲取定義文件的。

現在我們添加React和React-DOM依賴到`package.json`文件裡：

```shell
npm install --save react react-dom
```

接下來，我們要添加開發時依賴[ts-loader](https://www.npmjs.com/package/ts-loader)和[source-map-loader](https://www.npmjs.com/package/source-map-loader)。

```shell
npm install --save-dev ts-loader source-map-loader
npm link typescript
```

這些依賴會讓TypeScript和webpack在一起良好地工作。
ts-loader可以讓webpack使用TypeScript的標準配置文件`tsconfig.json`編譯TypeScript代碼。
source-map-loader使用TypeScript輸出的sourcemap文件來告訴webpack何時生成*自己的*sourcemaps。
這就允許你在調試最終生成的文件時就好像在調試TypeScript源碼一樣。

連結TypeScript，允許ts-loader使用全局安裝的TypeScript，而不需要單獨的本地拷貝。
如果你想要一個本地的拷貝，執行`npm install typescript`。

最後，我們使用`typings`工具來獲取React的聲明文件：

```shell
typings install --global --save "dt~react"
typings install --global --save "dt~react-dom"
```

`--global`標記，還有`dt~`前綴，告訴Typings從[DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped)獲取聲明文件，它是一個由社區維護的`.d.ts`文件倉庫。
這個命令會創建一個名為`typings.json`的文件和一個`typings`目錄在當前目錄下。

# 寫一些代碼

下面使用React寫一段TypeScript代碼。
首先，在`src/components`目錄下創建一個名為`Hello.tsx`的文件，代碼如下：

```ts
import * as React from "react";
import * as ReactDOM from "react-dom";

export class HelloComponent extends React.Component<any, any> {
    render() {
        return <h1>Hello from {this.props.compiler} and {this.props.framework}!</h1>;
    }
}

```

注意一點這個例子已經很像類了，我們不再需要使用類。
使用React的其它方式（比如[無狀態的功能組件](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions)）。

接下來，在`src`下創建`index.tsx`文件，源碼如下：

```ts
import * as React from "react";
import * as ReactDOM from "react-dom";

import { HelloComponent } from "./components/Hello";

ReactDOM.render(
    <HelloComponent compiler="TypeScript" framework="React" />,
    document.getElementById("example")
);
```

我們僅僅將`Hello`組件導入`index.tsx`。
注意，不同於`"react"`或`"react-dom"`，我們使用`index.tsx`的*相對路徑* - 這很重要。
如果不這樣做，TypeScript只會嘗試在`node_modules`文件夾裡查找。
其它使用React的方法也應該可以。

我們還需要一個頁面來顯示`Hello`組件。
在根目錄`proj`創建一個名為`index.html`的文件，如下：

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Hello React!</title>
    </head>
    <body>
        <div id="example"></div>

        <!-- Dependencies -->
        <script src="./node_modules/react/dist/react.js"></script>
        <script src="./node_modules/react-dom/dist/react-dom.js"></script>

        <!-- Main -->
        <script src="./dist/bundle.js"></script>
    </body>
</html>
```

需要注意一點我們是從`node_modules`引入的文件。
React和React-DOM的npm包裡包含了獨立的`.js`文件，你可以在頁面上引入它們，這裡我們為了快捷就直接引用了。
可以隨意地將它們拷貝到其它目錄下，或者從CDN上引用。
Facebook在CND上提供了一系列可用的React版本，你可以在這裡查看[更多內容](http://facebook.github.io/react/downloads.html#development-vs.-production-builds)。

# 添加TypeScript配置文件

現在，可以把所有TypeScript文件放在一起 - 包括我們編寫的代碼和必要的typings文件。

現在需要創建`tsconfig.json`文件，它包含輸入文件的列表和編譯選項。
在根目錄下執行下在命令：

```shell
tsc --init ./typings/main.d.ts ./src/index.tsx --jsx react --outDir ./dist --sourceMap --noImplicitAny
```

你可以在[這裡](../tsconfig.json.md)學習到更多關於`tsconfig.json`。

# 創建webpack配置文件

新建一個`webpack.config.js`文件在工程根目錄下。

```js
module.exports = {
    entry: "./src/index.tsx",
    output: {
        filename: "./dist/bundle.js",
    },

    // Enable sourcemaps for debugging webpack's output.
    devtool: "source-map",

    resolve: {
        // Add '.ts' and '.tsx' as resolvable extensions.
        extensions: ["", ".webpack.js", ".web.js", ".ts", ".tsx", ".js"]
    },

    module: {
        loaders: [
            // All files with a '.ts' or '.tsx' extension will be handled by 'ts-loader'.
            { test: /\.tsx?$/, loader: "ts-loader" }
        ],

        preLoaders: [
            // All output '.js' files will have any sourcemaps re-processed by 'source-map-loader'.
            { test: /\.js$/, loader: "source-map-loader" }
        ]
    }，

    // When importing a module whose path matches one of the following, just
    // assume a corresponding global variable exists and use that instead.
    // This is important because it allows us to avoid bundling all of our
    // dependencies, which allows browsers to cache those libraries between builds.
    externals: {
        "react": "React",
        "react-dom": "ReactDOM"
    },
};
```

大家可能對`externals`字段有所疑惑。
我們想要避免把所有的React都放到一個文件裡，因為會增加編譯時間並且瀏覽器還能夠緩存沒有發生改變的庫文件。
理想情況下，我們只需要在瀏覽器裡引入React模組，但是大部分瀏覽器還沒有支持模組。
因此大部分代碼庫會把自己包裹在一個單獨的全局變量內，比如：`jQuery`或`_`。
這叫做「命名空間」模式，webpack也允許我們繼續使用通過這種方式寫的代碼庫。
通過我們的設置`"react": "React"`，webpack會神奇地將所有對`"react"`的導入轉換成從`React`全局變量中加載。

你可以在[這裡](http://webpack.github.io/docs/configuration.html)瞭解更多如何配置webpack。

# 整合在一起

執行：

```shell
webpack
```

在瀏覽器裡打開`index.html`，工程應該已經可以用了！
你可以看到頁面上顯示著「Hello from TypeScript and React!」
