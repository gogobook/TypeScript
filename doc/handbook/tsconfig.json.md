## 概述

如果一個目錄下存在一個`tsconfig.json`文件，那麼它意味著這個目錄是TypeScript項目的根目錄。
`tsconfig.json`文件中指定了用來編譯這個項目的根文件和編譯選項。
一個項目可以通過以下方式之一來編譯：

## 使用tsconfig.json

* 不帶任何輸入文件的情況下調用`tsc`，編譯器會從當前目錄開始去查找`tsconfig.json`文件，逐級向上搜索父目錄。
* 不帶任何輸入文件的情況下調用`tsc`，且使用命令行參數`--project`（或`-p`）指定一個包含`tsconfig.json`文件的目錄。

當命令行上指定了輸入文件時，`tsconfig.json`文件會被忽略。

## 示例

`tsconfig.json`示例文件:

* 使用`"files"`屬性

```json
{
    "compilerOptions": {
        "module": "commonjs",
        "noImplicitAny": true,
        "removeComments": true,
        "preserveConstEnums": true,
        "outFile": "../../built/local/tsc.js",
        "sourceMap": true
    },
    "files": [
        "core.ts",
        "sys.ts",
        "types.ts",
        "scanner.ts",
        "parser.ts",
        "utilities.ts",
        "binder.ts",
        "checker.ts",
        "emitter.ts",
        "program.ts",
        "commandLineParser.ts",
        "tsc.ts",
        "diagnosticInformationMap.generated.ts"
    ]
}
```

* 使用`"exclude"`屬性

```json
{
    "compilerOptions": {
        "module": "commonjs",
        "noImplicitAny": true,
        "removeComments": true,
        "preserveConstEnums": true,
        "outFile": "../../built/local/tsc.js",
        "sourceMap": true
    },
    "exclude": [
        "node_modules",
        "wwwroot"
    ]
}
```

## 細節

`"compilerOptions"`可以被忽略，這時編譯器會使用默認值。在這裡查看完整的[編譯器選項](./Compiler Options.md)列表。

如果`tsconfig.json`沒有提供`"files"`屬性，編譯器會默認包含當前目錄及子目錄下的所有TypeScript文件（`*.ts` 或 `*.tsx`）。
如果提供了`"files"`屬性值，只有指定的文件會被編譯。

如果指定了`"exclude"`選項，編譯器會包含當前目錄及子目錄下的所有TypeScript文件（`*.ts` 或 `*.tsx`），不包括這些指定要排除的文件。

`"files"`選項不能與`"exclude"`選項同時使用。如果同時指定了兩個選項的話，只有`"files"`會生效。

所有被`"files"`屬性裡的文件所引用的文件同樣會被包含進來。
就好比，`A.ts`引用了`B.ts`，因此`B.ts`不能被排除，除非引用它的`A.ts`在`"exclude"`列表中。

`tsconfig.json`可以是個空文件，那麼編譯器則使用默認編譯選項，編譯當前目錄及其子目錄下的所有文件。

命令行上提供的編譯選項會覆蓋`tsconfig.json`文件中的對應選項。

## `compileOnSave`

在最頂層設置`compileOnSave`標記，可以讓IDE在保存文件的時候根據`tsconfig.json`重新生成文件。

```json
{
    "compileOnSave": true,
    "compilerOptions": {
        "noImplicitAny" : true
    }
}
```

要想支持這個特性需要Visual Studio 2015， TypeScript1.8.4以上並且安裝[atom-typescript](https://github.com/TypeStrong/atom-typescript/blob/master/docs/tsconfig.md#compileonsave)插件。

## 模式

到這裡查看模式: [http://json.schemastore.org/tsconfig](http://json.schemastore.org/tsconfig).
