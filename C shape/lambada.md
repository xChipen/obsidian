[[CShape]]

-   **Lambda**: `arguments => expression | block`
    -   arguments: 傳入參數的宣告，可以多個參數(以,隔開)。
        -   只有一個參數時可以不用括號，**複數個參數都要加上括號**
        -   可以不用明確指定型別
        -   可以明確指定型別，明確指定型別時一定要加上括號
        -   沒有傳入參數時用空括號`()`表示
    -   expression: 運算式，不括大括號`{}`，只能單行程式碼，代表回傳值
    -   block: `{ statements }`: 程式碼區塊，`statement`為此函式執行的程式碼片段

格式: `arguments => expression | block`

```
一個參數
x => x * x  //legal
(x) => x * x    //legal
```

```
加上型態
(string x) => x * x //legal
```

```
無參數
() => Console.WriteLine("Hello Lambda")
```

```
單行
x => x * x; == delegate (int x) { return x * x; }
```

```
區塊, 要加分號
x => { return x * x; }
```

