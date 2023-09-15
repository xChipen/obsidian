[[CShape]] [[LINQ]]

(parameters) => expression

(int number) => number * 2

```
void MyMethod(Action action)
{
    action();
}
MyMethod(() => Console.WriteLine("Hello, world!"));
```

```
void MyMethod()
{
    Console.WriteLine("Hello, world!");
}
(MyMethod)();
```

```
List<int> numbers = new List<int>() { 1, 2, 3, 4, 5 };

numbers.ForEach((number) =>
{
    Console.WriteLine(number);
});
```
---
```
Where 定義
1. 使用在 => IEnumerable<TSource> 
2. 參數 => Func<TSource, bool>
3. 來源 : TSource, 回傳條件 : bool
public static IEnumerable<TSource> Where<TSource>(
    this IEnumerable<TSource> source, Func<TSource, bool> predicate
)
```

```
Example : Where
List<int> numbers = new List<int>() { 1, 2, 3, 4, 5 };

var evenNumbers = numbers.Where((number) => number % 2 == 0);

foreach (var number in evenNumbers)
{
    Console.WriteLine(number);
}
ans : 2, 4
```
---
```
public static IEnumerable<TResult> Select<TSource, TResult>(
    this IEnumerable<TSource> source,
Func<TSource, TResult> selector
)
```

```
Example : Select
List<int> numbers = new List<int>() { 1, 2, 3, 4, 5 };

var squares = numbers.Select((number) => number * number);

foreach (var number in squares)
{
    Console.WriteLine(number);
}
ans : 1, 4, 9, 16, 25
```
---
```
public static IOrderedEnumerable<TSource> OrderBy<TSource, TKey>(
    this IEnumerable<TSource> source,
Func<TSource, TKey> keySelector
)
```

```
Example : OrderBy, OrderByDescending
List<string> names = new List<string>() { "Alice", "Bob", "Carol", "Dave", "Eve" };

var sortedNames = names.OrderBy((name) => name.Length);

foreach (var name in sortedNames)
{
    Console.WriteLine(name);
}
```
---
```
public static IEnumerable<IGrouping<TKey, TSource>> GroupBy<TSource, TKey>(
    this IEnumerable<TSource> source,
    Func<TSource, TKey> keySelector);

public static IEnumerable<IGrouping<TKey, TSource>> GroupBy<TSource, TKey>(
    this IEnumerable<TSource> source,
    Func<TSource, TKey> keySelector,
    IEqualityComparer<TKey> comparer);
```

```
Example : GroupBy
List<Customer> customers = new List<Customer>()
{
    new Customer { Name = "Alice", Age = 25 },
    new Customer { Name = "Bob", Age = 30 },
    new Customer { Name = "Carol", Age = 20 },
    new Customer { Name = "Dave", Age = 35 },
    new Customer { Name = "Eve", Age = 28 },
};

var groupedCustomers = customers.GroupBy((customer) => customer.Age);

foreach (var group in groupedCustomers)
{
    Console.WriteLine($"Customers of age {group.Key}:");
    foreach (var customer in group)
    {
        Console.WriteLine(customer.Name);
    }
}
---
MinBy, MaxBy, AverageBy
var youngestCustomer = customers.MinBy((customer) => customer.Age);
Console.WriteLine($"The youngest customer is {youngestCustomer.Name}."); Carol
---

```






