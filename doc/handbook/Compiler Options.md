## 編譯選項

選項                                     | 類型      | 默認值                    | 描述
----------------------------------------|-----------|--------------------------|----------------------------------------------------------------------
`--allowJs`                             | `boolean` |  `true`                  | 允許編譯javascript文件。
`--allowSyntheticDefaultImports`        | `boolean` | `(module === "system")`  | 允許從沒有設置默認導出的模組中默認導入。這並不影響代碼的顯示，僅為了類型檢查。
`--allowUnreachableCode`                | `boolean` | `false`                  | 不報告執行不到的代碼錯誤。
`--allowUnusedLabels`                   | `boolean` | `false`                  | 不報告未使用的標籤錯誤。
`--charset`                             | `string`  | `"utf8"`                 | 輸入文件的字符集。
`--declaration`<br/>`-d`                | `boolean` | `false`                  | 生成相應的'.d.ts'文件。
`--diagnostics`                         | `boolean` | `false`                  | 顯示診斷信息。
`--emitBOM`                             | `boolean` | `false`                  | 在輸出文件的開頭加入BOM頭（UTF-8 Byte Order Mark）。
`--emitDecoratorMetadata`<sup>[1]</sup> | `boolean` | `false`                  | 給源碼裡的裝飾器聲明加上設計類型自觀照資料。查看[issue #2577](https://github.com/Microsoft/TypeScript/issues/2577)瞭解更多信息。
`--experimentalDecorators`<sup>[1]</sup>| `boolean` | `false`                  | 實驗性啟用ES7裝飾器支持。
`--forceConsistentCasingInFileNames`    | `boolean` | `false`                  | 不允許不一致包裝引用相同的文件。
`--help`<br/>`-h`                       |           |                          | 打印幫助信息。
`--init`                                |           |                          | 初始化TypeScript項目並創建一個`tsconfig.json`文件。
`--inlineSourceMap`                     | `boolean` | `false`                  | 生成單個sourcemaps文件，而不是將每sourcemaps生成不同的文件。
`--inlineSources`                       | `boolean` | `false`                  | 將代碼與sourcemaps生成到一個文件中，要求同時設置了`--inlineSourceMap`或`--sourceMap`屬性。
`--isolatedModules`                     | `boolean` | `false`                  | 無條件地給沒有解析的文件生成imports。
`--jsx`                                 | `string`  | `"Preserve"`             | 在'.tsx'文件裡支持JSX：'React' 或 'Preserve'。查看[JSX](./JSX.md)。
`--listFiles`                           | `boolean` | `false`                  | 編譯過程中打印文件名。
`--locale`                              | `string`  | *(platform specific)*    | 顯示錯誤信息時使用的語言，比如：en-us。
`--mapRoot`                             | `string`  | `null`                   | 為調試器指定指定sourcemap文件的路徑，而不是使用生成時的路徑。當`.map`文件是在運行時指定的，並不同於`js`文件的地址時使用這個標記。指定的路徑會嵌入到`sourceMap`裡告訴調試器到哪裡去找它們。
`--module`<br/>`-m`                     | `string`  | `(target === "ES6" ? "ES6" : "commonjs")`                                 | 指定生成哪個模組系統代碼：'commonjs'，'amd'，'system'，或 'umd'或'es2015'。只有'amd'和'system'能和`--outFile`一起使用。當目標是ES5或以下的時候不能使用'es2015'。
`--moduleResolution`                    | `string`  | `(module === 'amd' | 'system' | 'ES6' ? 'classic' : 'node')`              | 決定如何處理模組。或者是'node'對於Node.js/io.js，或者是'classic'（默認）。查看[模組解析](./Module Resolution.md)瞭解詳情。
`--newLine`                             | `string`  | *(platform specific)*    | 當生成文件時指定行結束符：'CRLF'（dos）或 'LF' （unix）。
`--noEmit`                              | `boolean` | `false`                  | 不生成輸出文件。
`--noEmitHelpers`                       | `boolean` | `false`                  | 不在輸出文件中生成用戶自定義的幫助函數代碼，如`__extends`。
`--noEmitOnError`                       | `boolean` | `false`                  | 報錯時不生成輸出文件。
`--noFallthroughCasesInSwitch`          | `boolean` | `false`                  | 報告switch語句的fallthrough錯誤。（即，不允許switch的case語句貫穿）
`--noImplicitAny`                       | `boolean` | `false`                  | 在表達式和聲明上有隱含的'any'類型時報錯。
`--noImplicitReturns`                   | `boolean` | `false`                  | 不是函數的所有返回路徑都有返回值時報錯。
`--noImplicitUseStrict`                 | `boolean` | `false`                  | 模組輸出中不包含'use strict'指令。
`--noLib`                               | `boolean` | `false`                  | 不包含默認的庫文件（lib.d.ts）。
`--noResolve`                           | `boolean` | `false`                  | 不把`/// <reference``>`或模組導入的文件加到編譯文件列表。
~~`--out`~~                             | `string`  | `null`                   | 棄用。使用 `--outFile` 代替。
`--outDir`                              | `string`  | `null`                   | 重定向輸出目錄。
`--outFile`                             | `string`  | `null`                   | 將輸出文件合併為一個文件。合併的順序是根據傳入編譯器的文件順序和`///<reference``>`和`import`的文件順序決定的。查看輸出文件順序文件瞭解詳情。
`--preserveConstEnums`                  | `boolean` | `false`                  | 保留`const`和`enum`聲明。查看[const enums documentation](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#94-constant-enum-declarations)瞭解詳情。
`--pretty`<sup>[1]</sup>                | `boolean` | `false`                  | 給錯誤和消息設置樣式，使用顏色和上下文。
`--project`<br/>`-p`                    | `string`  | `null`                   | 編譯指定目錄下的項目。這個目錄應該包含一個`tsconfig.json`文件來管理編譯。查看[tsconfig.json](./tsconfig.json.md)文檔瞭解更多信息。
`--reactNamespace`                      | `string`  | `"React"`                | 當目標為生成'react' JSX時，指定`createElement`和`__spread`的調用物件
`--removeComments`                      | `boolean` | `false`                  | 刪除所有註釋，除了以`/!*`開頭的版權信息。
`--rootDir`                             | `string`  | *(common root directory is computed from the list of input files)*   | 僅用來控制輸出的目錄結構`--outDir`。
`--skipDefaultLibCheck`                 | `boolean` | `false`                  |
`--sourceMap`                           | `boolean` | `false`                  | 生成相應的'.map'文件。
`--sourceRoot`                          | `string`  | `null`                   | 指定TypeScript源文件的路徑，以便調試器定位。當TypeScript文件的位置是在運行時指定時使用此標記。路徑信息會被加到`sourceMap`裡。
`--strictNullChecks`                    | `boolean` | `false`                  | 在嚴格的`null`檢查模式下，`null`和`undefined`值不包含在任何類型裡，只允許用它們自己和`any`來賦值（有個例外，`undefined`可以賦值到`void`）。
`--stripInternal`<sup>[1]</sup>         | `boolean` | `false`                  | 不對具有`/** @internal */` JSDoc註解的代碼生成代碼。
`--suppressExcessPropertyErrors`<sup>[1]</sup> | `boolean` | `false`           | 阻止對物件字面量的額外屬性檢查。
`--suppressImplicitAnyIndexErrors`      | `boolean` | `false`                  | 阻止`--noImplicitAny`對缺少索引簽名的索引物件報錯。查看[issue #1232](https://github.com/Microsoft/TypeScript/issues/1232#issuecomment-64510362)瞭解詳情。
`--target`<br/>`-t`                     | `string`  | `"ES5"`                  | 指定ECMAScript目標版本'ES3' (默認)，'ES5'，或'ES6'<sup>[1]</sup>
`--traceResolution`                     | `boolean` | `false`                  | 生成模組解析日誌信息
`--version`<br/>`-v`                    |           |                          | 打印編譯器版本號。
`--watch`<br/>`-w`                      |           |                          | 在監視模式下運行編譯器。會監視輸出文件，在它們改變時重新編譯。

<sup>[1]</sup> 這些選項是試驗性的。

## 相關信息

* 在[tsconfig.json](./tsconfig.json.md)文件裡設置編譯器選項。
* 在[MSBuild工程](./Compiler Options in MSBuild.md)裡設置編譯器選項。
