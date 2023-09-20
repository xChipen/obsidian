[[CShape]]
[[Iterator Pattern]]

yield來實作 IEnumerable
-   如果一個區塊(block)中有`yield`陳述式，此區塊就叫做 **Iterator Block**
-   一個方法的區塊如果是 **Iterator Block**，則它的回傳值會是 `IEnumerable`、`IEnumerator`。

###### 運作方式
A : 呼叫端 B : Iterator Block
1. A 呼叫 B
2. B執行至 yield return xxx; 會返回 A, 執行 A的邏輯運算
3. A 運算完, 在返回 B
4. 循環至結束

```
依序返回 1 - 9, 10 因為前面中斷, 所以不會執行
private static IEnumerable enumerable_yield2()
{
    yield return 1;
    yield return 2;
    yield return 3;
    yield return 4;
    yield return 5;
    yield return 6;
    yield return 7;
    yield return 8;
    yield return 9;
    yield break;
    yield return 10;

    //1 2 3 4 5 6 7 8 9
}
```

```
使用方式
回傳必須為 : IEnumerable
public static IEnumerable test(int MaxNum, int divide)
{
	for (int currentNum = 1; currentNum < MaxNum; currentNum++) {
		if (currentNum % divide != 0) continue; // 判斷條件
		yield return currentNum; // 回傳資料, 2, 4, 6, 8
	}
}

private void button3_Click(object sender, EventArgs e)
{
	foreach(int num in test(10, 2))
	{
		textBox1.Text += num.ToString();
	}
}
```