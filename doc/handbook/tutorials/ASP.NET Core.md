# ASP.NET Core

## 安裝 ASP.NET Core 和 TypeScript

首先，若有必要請[安裝 ASP.NET Core](https://get.asp.net)。
這個快速上手指南使用的是 Visual Studio ，若要使用 ASP.NET Core 你需要有 Visual Studio 2015。

其次，如果你的 Visual Studio 中沒有包含 TypeScript，你可以從這裡安裝[TypeScript for Visual Studio 2015](http://www.microsoft.com/en-us/download/details.aspx?id=48593)。

## 新建工程

1. 選擇 **File**
2. 選擇 **New Project** （Ctrl + Shift + N）
3. 選擇 **Visual C#**
4. 選擇 **ASP.NET Web Application**

   ![Create new ASP.NET project](../../assets/images/tutorials/aspnet/new-asp-project.png)

5. 選擇 **ASP.NET 5 Empty** 工程模板

   取消複選 "Host in the cloud" 本指南將使用一個本地示例。
   ![Use empty template](../../assets/images/tutorials/aspnet/new-asp-project-empty.png)

運行此應用以確保它能正常工作。

## 設置服務項

在 `project.json` 文件的 `"dependencies"` 字段裡添加:

```json
"Microsoft.AspNet.StaticFiles": "1.0.0-rc1-final"
```

最終的 dependencies 部分應該類似於下面這樣：

```json
"dependencies": {
  "Microsoft.AspNet.IISPlatformHandler": "1.0.0-rc1-final",
  "Microsoft.AspNet.Server.Kestrel": "1.0.0-rc1-final",
  "Microsoft.AspNet.StaticFiles": "1.0.0-rc1-final"
},
```

用以下內容替換 `Startup.cs` 文件裡的 `Configure` 函數：

```cs
public void Configure(IApplicationBuilder app)
{
    app.UseIISPlatformHandler();
    app.UseDefaultFiles();
    app.UseStaticFiles();
}
```

# 添加 TypeScript

下一步我們為 TypeScript 添加一個文件夾。

![Create new folder](../../assets/images/tutorials/aspnet/new-folder.png)

將文件夾命名為 `scripts`。

![scripts folder](../../assets/images/tutorials/aspnet/scripts-folder.png)

## 添加 TypeScript 代碼

在`scripts`上右擊並選擇**New Item**。
接著選擇**TypeScript File**（也可能 .NET Core 部分），並將此文件命名為`app.ts`。

![New item](../../assets/images/tutorials/aspnet/new-item.png)

## 添加示例代碼

將以下代碼寫入app.ts文件。

```ts
function sayHello() {
  const compiler = (document.getElementById("compiler") as HTMLInputElement).value;
  const framework = (document.getElementById("framework") as HTMLInputElement).value;
  return `Hello from ${compiler} and ${framework}!`;
}
```

## 構建設置

### 配置 TypeScript 編譯器

我們先來告訴TypeScript怎樣構建。
右擊scripts文件夾並選擇**New Item**。
接著選擇**TypeScript Configuration File**，保持文件的默認名字為`tsconfig.json`。

![Create tsconfig.json](../../assets/images/tutorials/aspnet/new-tsconfig.png)

將默認的`tsconfig.json`內容改為如下所示：

```json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "noEmitOnError": true,
    "sourceMap": true,
    "target": "es5",
  },
  "files": [
    "./app.ts"
  ],
  "compileOnSave": true
}
```

看起來和默認的設置差不多，但注意以下不同之處：

1. 設置`"noImplicitAny": true`。
2. 顯式列出了`"files"`而不是依據`"excludes"`。
3. 設置`"compileOnSave": true`。

當你寫新代碼時，設置`"noImplicitAny"`選項是個不錯的選擇 &mdash; 這可以確保你不會錯寫任何新的類型。
設置`"compileOnSave"`選項可以確保你在運行web程序前自動編譯保存變更後的代碼。

### 配置 NPM

現在，我們來配置NPM以使用我們能夠下載JavaScript包。
在工程上右擊並選擇**New Item**。
接著選擇**NPM Configuration File**，保持文件的默認名字為`package.json`。
在`"devDependencies"`部分添加"gulp"和"del"：

```json
"devDependencies": {
  "gulp": "3.9.0",
  "del": "2.2.0"
}
```

保存這個文件後，Visual Studio將開始安裝gulp和del。
若沒有自動開始，請右擊package.json文件選擇**Restore Packages**。

### 設置 gulp

最後，添加一個新JavaScript文件`gulpfile.js`。
鍵入以下內容：

```js
/// <binding AfterBuild='default' Clean='clean' />
/*
This file is the main entry point for defining Gulp tasks and using Gulp plugins.
Click here to learn more. http://go.microsoft.com/fwlink/?LinkId=518007
*/

var gulp = require('gulp');
var del = require('del');

var paths = {
  scripts: ['scripts/**/*.js', 'scripts/**/*.ts', 'scripts/**/*.map'],
};

gulp.task('clean', function () {
  return del(['wwwroot/scripts/**/*']);
});

gulp.task('default', function () {
  gulp.src(paths.scripts).pipe(gulp.dest('wwwroot/scripts'))
});
```

第一行是告訴Visual Studio構建完成後，立即運行'default'任務。
當你應答 Visual Studio 清除構建內容後，它也將運行'clean'任務。

現在，右擊`gulpfile.js`並選擇**Task Runner Explorer**。
若'default'和'clean'任務沒有顯示輸出內容的話，請刷新explorer：

![Refresh Task Runner Explorer](../../assets/images/tutorials/aspnet/task-runner-explorer.png)

## 編寫HTML頁

在`wwwroot`中添加一個新建項 `index.html`。
在`index.html`中寫入以下代碼：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <script src="scripts/app.js"></script>
    <title></title>
</head>
<body>
    <div id="message"></div>
    <div>
        Compiler: <input id="compiler" value="TypeScript" onkeyup="document.getElementById('message').innerText = sayHello()" /><br />
        Framework: <input id="framework" value="ASP.NET" onkeyup="document.getElementById('message').innerText = sayHello()" />
    </div>
</body>
</html>
```

## 測試

1. 運行項目。
2. 在輸入框中鍵入時，您應該看到一個消息：

![Picture of running demo](../../assets/images/tutorials/aspnet/running-demo.png)

## 調試

1. 在 Edge 瀏覽器中，按 F12 鍵並選擇 **Debugger** 標籤頁。
2. 展開 localhost 列表，選擇 scripts/app.ts
3. 在 `return` 那一行上打一個斷點。
4. 在輸入框中鍵入一些內容，確認TypeScript代碼命中斷點，觀察它是否能正確地工作。

![Demo paused on breakpoint](../../assets/images/tutorials/aspnet/paused-demo.png)

這就是你需要知道的在ASP.NET中使用TypeScript的基本知識了。接下來，我們引入Angular，寫一個簡單的Angular程序示例。

# 添加 Angular 2

## 使用 NPM 下載所需的包

在 `package.json` 文件的 `"dependencies"` 添加 Angular 2 和 SystemJS：

```json
  "dependencies": {
    "angular2": "2.0.0-beta.11",
    "systemjs": "0.19.24",
  },
```

## 安裝 typings

Angular 2 包含 es6-shim 以提供 Promise 支持，但 TypeScript 還需要它的類型文件。
打開一個 command prompt，切換到應用程序源文件目錄中：

```shell
cd C:\Users\<you>\Documents\Visual Studio 2015\Projects\<app>\src\<app>
npm install -g typings
typings install --global dt~es6-shim
```

## 更新 tsconfig.json

現在安裝好了 Angular 2 及其依賴項，我們還需要啟用 TypeScript 中實驗性的裝飾器支持並且引入 es6-shim 的類型文件。
將來的版本中，裝飾器和 ES6 選項將成為默認選項，我們就可以不做此設置了。添加
`"experimentalDecorators": true, "emitDecoratorMetadata": true` 選項到 `"compilerOptions"` 選項段，添加 `"./typings/index.d.ts"` 到 `"files"` 選項段。
最後，我們還將要創建新的代碼文件 `"./src/model.ts"`、`"./src/main.ts"` ，也將它們添加到 `"files"` 中，現在 tsconfig 看起來像這樣：

```json
{
  "compilerOptions": {
      "noImplicitAny": true,
      "noEmitOnError": true,
      "sourceMap": true,
      "experimentalDecorators": true,
      "emitDecoratorMetadata": true,
      "target": "es5"
  },
  "files": [
      "./app.ts",
      "./model.ts",
      "./main.ts",
      "../typings/main.d.ts"
  ],
  "compileOnSave": true
}
```

## 將 Angular 添加到 gulp 構建中

最後，我們需要確保 Angular 文件作為 build 的一部分複製進來。
我們需要添加：

1. 庫文件目錄。
2. 添加一個 `lib` 任務來輸送文件到 `wwwroot`。
3. 在 `default` 任務上添加 `lib` 任務依賴。

更新後的 `gulpfile.js` 像如下所示：

```xml
/// <binding AfterBuild='default' Clean='clean' />
/*
This file is the main entry point for defining Gulp tasks and using Gulp plugins.
Click here to learn more. http://go.microsoft.com/fwlink/?LinkId=518007
*/

var gulp = require('gulp');
var del = require('del');

var paths = {
    scripts: ['scripts/**/*.js', 'scripts/**/*.ts', 'scripts/**/*.map'],
    libs: ['node_modules/angular2/bundles/angular2.js',
           'node_modules/angular2/bundles/angular2-polyfills.js',
           'node_modules/systemjs/dist/system.src.js',
           'node_modules/rxjs/bundles/Rx.js']
};

gulp.task('lib', function () {
    gulp.src(paths.libs).pipe(gulp.dest('wwwroot/scripts/lib'))
});

gulp.task('clean', function () {
    return del(['wwwroot/scripts/**/*']);
});

gulp.task('default', ['lib'], function () {
    gulp.src(paths.scripts).pipe(gulp.dest('wwwroot/scripts'))
});
```

此外，保存了此gulpfile後，要確保 Task Runner Explorer 能看到 `lib` 任務。

## 用 TypeScript 寫一個簡單的 Angular 應用

首先，將 `app.ts` 改成：

```ts
import {Component} from "angular2/core"
import {MyModel} from "./model"

@Component({
    selector: `my-app`,
    template: `<div>Hello from {{getCompiler()}}</div>`
})
class MyApp {
    model = new MyModel();
    getCompiler() {
        return this.model.compiler;
    }
}
```

接著在 `scripts` 中添加 TypeScript 文件 `model.ts`:

```ts
export class MyModel {
    compiler = "TypeScript";
}
```

再在 `scripts` 中添加 `main.ts`：

```ts
import {bootstrap} from "angular2/platform/browser";
import {MyApp} from "./app";
bootstrap(MyApp);
```

最後，將 `index.html` 改成：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <script src="scripts/lib/angular2-polyfills.js"></script>
    <script src="scripts/lib/system.src.js"></script>
    <script src="scripts/lib/rx.js"></script>
    <script src="scripts/lib/angular2.js"></script>
    <script>
    System.config({
        packages: {
            'scripts': {
                format: 'cjs',
                defaultExtension: 'js'
            }
        }
    });
    System.import('scripts/main').then(null, console.error.bind(console));
    </script>
    <title></title>
</head>
<body>
    <my-app>Loading...</my-app>
</body>
</html>
```

這裡加載了此應用。
運行 ASP.NET 應用，你應該能看到一個 div 顯示 "Loading..." 緊接著更新成顯示 "Hello from TypeScript"。
