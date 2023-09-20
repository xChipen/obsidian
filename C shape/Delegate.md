[[CShape]]
[[LINQ]]
[[lambada]]
[[委派]]
委派的方式 : 有三種具名函式、匿名函式、Lambda運算式

```
public string nameMethod(string data)
{
  return data + ":DEF";
}
==
delegate(string data){ return data + ":DEF";}

--------------------------------------------------
//                    in      out 
public void test(Func<string, string> method, data)
{
  string rs = method(data);
  ...
}

使用
nameMethod("TEST");
邏輯在 function 外面處理, 降底偶合
nameMethod2(delegate (string outData) { return outData + ":DEF"; }, "ABC");
```
