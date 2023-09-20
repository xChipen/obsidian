#Iterator 迭代器模式
![[Pasted image 20230920091711.png]]
Iterator 可以用來巡訪一個**聚合器**裡的資料，並且可以把巡訪的邏輯跟聚合器分開，讓容器的職責保持單一。

Iterator pattern 有兩個主要的角色，一個是 Iterator 本身，另一個是容器（也稱作 Aggregate）。

-   **ConcreteAggregate**: `Integers`
    -   **Iterator()**: `GetEnumerator()`
-   **ConcreteIterator**: `IntegerEnum`
    -   **next()**: 傳回下一個元素，在C#中是以`Current`來抓出目前元素
    -   **hasNext()**: 確認是否有下一個元素，在C#中是由`MoveNext()`做確認

在`Iterator`裡有個跟C#上的實作差異:

-   在`Ierator Pattern`上是用`hasNext()`判斷是否有下一個元素，確定有了再Call `Next()`取得元素並更新`index`
-   c#裡是用`MoveNext()`判斷是否有下一個元素，確定有了之後去更新`Current`及`index`
---
-   **Aggregate**: IEnumerable(ConcreteAggregate的介面)
-   **Iterator**: IEnumerator(ConcreteIterator的介面)

```
繼承 IEnumerator, 實作 : Current, MoveNext(), Reset
public class IntegerEnum : IEnumerator
{
	private int _integers;
	private int _index;
	private int _maxDigit;
	public object Current { get; private set; }

	public IntegerEnum(int integers)
	{
		_integers = integers;
		_index = 0;
		_maxDigit = (int)Math.Log10(integers);
	}

	public bool MoveNext()
	{
		if (_maxDigit < _index) return false;

		Current = getCurrent();
		_index++;

		return true;
	}

	private int getCurrent()
	{
		int currentDigit = _maxDigit - _index;
		int result = (_integers / (int)Math.Pow(10, currentDigit)) % 10;  //Get first digit

		return result;
	}

	public void Reset()
	{
		_index = 0;
	}
}
    
使用模板
繼承 IEnumerable, 實作 : GetEnumerator
public class Integers : IEnumerable
{
	private int _integers;

	public Integers(int integers)
	{
		_integers = integers;
	}
給 foreach... 調用
	public IEnumerator GetEnumerator()
	{
		return new IntegerEnum(_integers);
	}
}

實際使用
static void intForeach()
{
	Integers integers = new Integers(123456789);

	foreach (int integer in integers)
	{
		Console.Write($"{integer} ");
	} // 1 2 3 4 5 6 7 8 9 

	Console.WriteLine();
}

```
會被擴充為列方式 :
{
    E e = ((C)(x)).GetEnumerator(); // 規則
    try {
        while (e.MoveNext()) { // 是否有下一筆資料
            V v = (V)(T)e.Current; // 取出正確資料
            embedded_statement // 運算處理
        }
    }
    finally {
        ... // Dispose e
    }
}
```






