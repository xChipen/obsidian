[[CShape]]
#Action #Func

**Action 無參數**
```
1. 宣告
public delegate void Action();
2. 實作
void MyMethod() 
{ 
  Console.WriteLine("Hello, world!"); 
} 
3. 定義變數
Action action = MyMethod; 
4. 使用
action();
```

```
List<T>.ForEach()

List<int> numbers = new List<int>() { 1, 2, 3, 4, 5 }; 
numbers.ForEach(number => { 
    Console.WriteLine(number); 
  }
);
```

```
匿名
Action action = delegate
{
    Console.WriteLine("Hello, world!");
};
action();

lambda
Action action = () => Console.WriteLine("Hello, world!");
```
