[[SQL]]  [[CShape]]
#byte #FileStream #buffer #image
image => byte[]

```C#
檔案 轉 byte[]
filename = "c:\demo.jpg";  
FileStream fs = new FileStream(filename, FileMode.Open);  
byte[] buffer = new byte[fs.Length]; // 用來儲存檔案的 byte 陣列，檔案有多大，陣列就有多大  
fs.Read(buffer, 0, buffer.Length);  
fs.Close();
```

```C#
資料表欄位 byte[] 轉 檔案
byte[] data = (byte[])dr["data"];

存成檔案
FileStream fs = new FileStream("c:\new.jpg");  
fs.Write(buffer, 0, buffer.Length);  
fs.Close();
```

其他資訊
https://goodlucky.pixnet.net/blog/post/27768104
