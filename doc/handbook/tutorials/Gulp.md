這個快速上手指南會告訴你如何結合使用TypeScript和[Knockout.js](http://knockoutjs.com/)。

這裡我們假設你已經會使用[Node.js](https://nodejs.org/)和[npm](https://www.npmjs.com/)

# 新建工程

首先，我們新建一個目錄。
暫時命名為`proj`，當然了你可以使用任何喜歡的名字。

```shell
mkdir proj
cd proj
```

接下來，我們按如下方式來組織這個工程：

```text
proj/
   +- src/
   +- built/
```

TypeScript源碼放在`src`目錄下，結過TypeScript編譯器編譯後，生成的文件放在`built`目錄裡。

下面創建目錄：

```shell
mkdir src
mkdir built
```

# 安裝構建依賴

首先確保TypeScript和Typings已經全局安裝。

```shell
npm install -g typescript typings
```

你肯定已經很瞭解TypeScript了，但你有可能還不太瞭解Typings.
[Typings](https://www.npmjs.com/package/typings)是一個包管理器用來獲取聲明文件。
我們將會使用它來獲取Knockout的聲明文件。

```shell
typings install --global --save dt~knockout
```

`--global`標記會告訴Typings從[DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped)去獲取聲明文件，這是由社區維護的`.d.ts`文件倉庫。
這個命令會在當前目錄下創建一個`typings.json`文件和一個`typings`文件夾。

# 獲取運行時依賴

我們需要Knockout和RequireJS。
[RequireJS](http://www.requirejs.org/)是一個庫，它可以讓我們在運行時異步地加載模組。

有以下幾種獲取方式：

1. 手動下載文件並維護它們。
2. 通過像[Bower](http://bower.io/)這樣的包管理下載並維護它們。
3. 使用內容分發網絡（CDN）來維護這兩個文件。

我們使用第一種方法，它會簡單一些，但是Knockout的官方文檔上有講解[如何使用CDN](http://knockoutjs.com/downloads/index.html)，更多像RequireJS一樣的代碼庫可以在[cdnjs](https://cdnjs.com/)上查找。

下面讓我們在工程根目錄下創建`externals`目錄。

```shell
mkdir externals
```

然後[下載Knockout](http://knockoutjs.com/downloads/index.html)和[下載RequireJS](http://www.requirejs.org/docs/download.html#latest)到這個目錄裡。
最新的壓縮後版本就可以。

# 添加TypeScript配置文件

下面我們想把所有的TypeScript文件整合到一起 - 包括自己寫的和必須的聲明文件。

我們需要創建一個`tsconfig.json`文件，包含了輸入文件列表和編譯選項。
在工程根目錄下創建一個新文件`tsconfig.json`，內容如下：

```json
{
    "compilerOptions": {
        "outDir": "./built/",
        "sourceMap": true,
        "noImplicitAny": true,
        "module": "amd",
        "target": "es5"
    },
    "files": [
        "./typings/index.d.ts",
        "./src/require-config.ts",
        "./src/hello.ts"
    ]
}
```

這裡引用了`typings/index.d.ts`，它是Typings幫我們創建的。
這個文件會自動地包含所有安裝的依賴。

你可能會對`typings`目錄下的`browser.d.ts`文件感到好奇，尤其因為我們將在瀏覽器裡運行代碼。
其實原因是這樣的，當目標為瀏覽器的時候，一些包會生成不同的版本。
通常來講，這些情況很少發生並且在這裡我們不會遇到這種情況，所以我們可以忽略`browser.d.ts`。

你可以在[這裡](../tsconfig.json.md)查看更多關於`tsconfig.json`文件的信息

# 寫些代碼

下面我們使用Knockout寫一段TypeScript代碼。
首先，在`src`目錄裡新建一個`hello.ts`文件。

```ts
import * as ko from "knockout";

class HelloViewModel {
    language: KnockoutObservable<string>
    framework: KnockoutObservable<string>

    constructor(language: string, framework: string) {
        this.language = ko.observable(language);
        this.framework = ko.observable(framework);
    }
}

ko.applyBindings(new HelloViewModel("TypeScript", "Knockout"));
```

接下來，在`src`目錄下再新建一個`require-config.ts`文件。

```ts
declare var require: any;
require.config({
    paths: {
        "knockout": "externals/knockout-3.4.0",
    }
});
```

這個文件會告訴RequireJS從哪裡導入Knockout，好比我們在`hello.ts`裡做的一樣。
你創建的所有頁面都應該在RequireJS之後和導入任何東西之前引入它。
為了更好地理解這個文件和如何配置RequireJS，可以查看[文檔](http://requirejs.org/docs/api.html#config)。

我們還需要一個視圖來顯示`HelloViewModel`。
在`proj`目錄的根上創建一個文件`index.html`，內容如下：

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Hello Knockout!</title>
    </head>
    <body>
        <p>
            Hello from
            <strong data-bind="text: language">todo</strong>
            and
            <strong data-bind="text: framework">todo</strong>!
        </p>

        <p>Language: <input data-bind="value: language" /></p>
        <p>Framework: <input data-bind="value: framework" /></p>

        <script src="./externals/require.js"></script>
        <script src="./built/require-config.js"></script>
        <script>
            require(["built/hello"]);
        </script>
    </body>
</html>
```

注意，有兩個script標籤。
首先，我們引入RequireJS。
然後我們再在`require-config.js`裡映射外部依賴，這樣RequireJS就能知道到哪裡去查找它們。
最後，使用我們要去加載的模組去調用`require`。

# 將所有部分整合在一起

運行

```shell
tsc
```

現在，在你喜歡的瀏覽器打開`index.html`，所有都應該好用了。
你應該可以看到頁面上顯示「Hello from TypeScript and Knockout!」。
在它下面，你還會看到兩個輸入框。
當你改變輸入和切換焦點時，就會看到原先顯示的信息改變了。
