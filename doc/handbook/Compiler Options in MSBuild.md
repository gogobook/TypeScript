## 概述

編譯選項可以在使用MSBuild的項目裡通過MSBuild屬性指定。

## 例子

```XML
  <PropertyGroup Condition="'$(Configuration)' == 'Debug'">
    <TypeScriptRemoveComments>false</TypeScriptRemoveComments>
    <TypeScriptSourceMap>true</TypeScriptSourceMap>
  </PropertyGroup>
  <PropertyGroup Condition="'$(Configuration)' == 'Release'">
    <TypeScriptRemoveComments>true</TypeScriptRemoveComments>
    <TypeScriptSourceMap>false</TypeScriptSourceMap>
  </PropertyGroup>
  <Import Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets"
          Condition="Exists('$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets')" />
```

## 映射

編譯選項                                      | MSBuild屬性名稱                             | 可用值
---------------------------------------------|--------------------------------------------|-----------------
`--declaration`                              | TypeScriptGeneratesDeclarations            | 布林值
`--module`                                   | TypeScriptModuleKind                       | `AMD`, `CommonJs`, `UMD` 或 `System`
`--target`                                   | TypeScriptTarget                           | `ES3`, `ES5`, or `ES6`
`--charset`                                  | TypeScriptCharset                          |
`--emitBOM`                                  | TypeScriptEmitBOM                          | 布林值
`--emitDecoratorMetadata`                    | TypeScriptEmitDecoratorMetadata            | 布林值
`--experimentalDecorators`                   | TypeScriptExperimentalDecorators           | 布林值
`--inlineSourceMap`                          | TypeScriptInlineSourceMap                  | 布林值
`--inlineSources`                            | TypeScriptInlineSources                    | 布林值
`--locale`                                   | *自動的*                                    | 自動設置成PreferredUILang的值
`--mapRoot`                                  | TypeScriptMapRoot                          | 文件路徑
`--newLine`                                  | TypeScriptNewLine                          | `CRLF` 或 `LF`
`--noEmitOnError`                            | TypeScriptNoEmitOnError                    | 布林值
`--noEmitHelpers`                            | TypeScriptNoEmitHelpers                    | 布林值
`--noImplicitAny`                            | TypeScriptNoImplicitAny                    | 布林值
`--noLib`                                    | TypeScriptNoLib                            | 布林值
`--noResolve`                                | TypeScriptNoResolve                        | 布林值
`--out`                                      | TypeScriptOutFile                          | 文件路徑
`--outDir`                                   | TypeScriptOutDir                           | 文件路徑
`--preserveConstEnums`                       | TypeScriptPreserveConstEnums               | 布林值
`--removeComments`                           | TypeScriptRemoveComments                   | 布林值
`--rootDir`                                  | TypeScriptRootDir                          | 文件路徑
`--isolatedModules`                          | TypeScriptIsolatedModules                  | 布林值
`--sourceMap`                                | TypeScriptSourceMap                        | 文件路徑
`--sourceRoot`                               | TypeScriptSourceRoot                       | 文件路徑
`--suppressImplicitAnyIndexErrors`           | TypeScriptSuppressImplicitAnyIndexErrors   | 布林值
`--suppressExcessPropertyErrors`             |  TypeScriptSuppressExcessPropertyErrors    | 布林值
`--moduleResolution`                         | TypeScriptModuleResolution                 | `Classic` or `Node`
`--experimentalAsyncFunctions`               | TypeScriptExperimentalAsyncFunctions       | 布林值
`--jsx`                                      | TypeScriptJSXEmit                          | `React` or `Preserve`
`--reactNamespace`                           | TypeScriptReactNamespace                   | string
`--skipDefaultLibCheck`                      | TypeScriptSkipDefaultLibCheck              | 布林值
`--allowUnusedLabels`                        | TypeScriptAllowUnusedLabels                | 布林值
`--noImplicitReturns`                        | TypeScriptNoImplicitReturns                | 布林值
`--noFallthroughCasesInSwitch`               | TypeScriptNoFallthroughCasesInSwitch       | 布林值
`--allowUnreachableCode`                     | TypeScriptAllowUnreachableCode             | 布林值
`--forceConsistentCasingInFileNames`         | TypeScriptForceConsistentCasingInFileNames | 布林值
`--allowSyntheticDefaultImports`             | TypeScriptAllowSyntheticDefaultImports     | 布林值
`--noImplicitUseStrict`                      | TypeScriptNoImplicitUseStrict              | 布林值
`--project`                                  | *VS不支持*                                  |
`--watch`                                    | *VS不支持*                                  |
`--diagnostics`                              | *VS不支持*                                  |
`--listFiles`                                | *VS不支持*                                  |
`--noEmit`                                   | *VS不支持*                                  |
`--allowJs`                                  | *VS不支持*                                  |
*VS特有選項*                                  | TypeScriptAdditionalFlags                  | *任意編譯選項*


## 我使用的Visual Studio版本裡支持哪些選項?

查找 `C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets` 文件。
可用的MSBuild XML標籤與相應的`tsc`編譯選項的映射都在那裡。

## ToolsVersion

工程文件裡的`<TypeScriptToolsVersion>1.7</TypeScriptToolsVersion>`屬性值表明了構建時使用的編譯器的版本號（這個例子裡是1.7）
這樣就允許一個工程在不同的機器上使用固定的版本去編譯。

如果沒有指定`TypeScriptToolsVersion`，則會使用機器上安裝的最新版本的編譯器去構建。

如果用戶使用的是更新版本的TypeScript，則會在首次加載工程的時候看到一個提示升級工程的對話框。

## TypeScriptCompileBlocked

如果你使用其它的構建工具（比如，gulp， grunt等等）並且使用VS做為開發和調試工具，那麼在工程裡設置`<TypeScriptCompileBlocked>true</TypeScriptCompileBlocked>`。
這樣VS只會提供給你編輯的功能，而不會在你按F5的時候去構建。