<!-- markdownlint-disable MD029 -->TypeScript編譯器處理Node模組名時使用的是[Node.js模組解析算法](https://nodejs.org/api/modules.html#modules_all_together)。
TypeScript也可以同時加載與npm包綁在一起的類型聲明文件。
編譯通過下面的規則來查找`"foo"`模組的類型信息：

1. 嘗試加載相應代碼包目錄下`package.json`文件（`node_modules/foo/`）。

如果存在，從`"typings"`字段裡讀取類型文件的路徑。比如，在下面的`package.json`裡，編譯器會認為類型文件位於`node_modules/foo/lib/foo.d.ts`。

```json
{
    "name": "foo",
    "author": "Vandelay Industries",
    "version": "1.0.0",
    "main": "./lib/foo.js",
    "typings": "./lib/foo.d.ts"
}
```

2. 嘗試加載在相應代碼包目錄下的名字為`index.d.ts`的文件（`node_modules/foo/`） - 這個文件應該包含了這個代碼包的類型信息。

解析模組的詳細算法可以在[這裡](https://github.com/Microsoft/TypeScript/issues/2338)找到。

### 你的定義文件應該

* 是`.d.ts`文件
* 寫做外部模組
* 不包含`///<reference>`引用

基本的原理是類型文件不能引入新的可編譯代碼；
否則真正的實現文件就可能會在編譯時被重蓋。
另外，**加載類型信息不應該污染全局空間**，當從同一個庫的不同版本中引入潛在衝突的實體的時候。
