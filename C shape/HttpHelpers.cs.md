[[Helpers]]
```
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using System;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Net.NetworkInformation;
using System.Security.Cryptography;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace Helpers
{
    public class HttpHelper
    {
        private static readonly HttpHelper _instance = new HttpHelper();
        private HttpHelper()
        {
            //使用加密憑證
            ServicePointManager.ServerCertificateValidationCallback = delegate { return true; };
            //提高預設認證等級
            ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;
        }

        public static string HMACSHA256(string message, string key)
        {
            var encoding = new System.Text.UTF8Encoding();
            byte[] keyByte = encoding.GetBytes(key);
            byte[] messageBytes = encoding.GetBytes(message);
            using (var hmacSHA256 = new HMACSHA256(keyByte))
            {
                byte[] hashMessage = hmacSHA256.ComputeHash(messageBytes);
                return BitConverter.ToString(hashMessage).Replace("-", "").ToLower();
            }
        }

        public static string GetBase64Encode(string cTemp)
        { //注意！下面這個正規表示式不一定可以正確的將JSON Minify（驗證用而已），若正式用途建議使用JSON套件
            cTemp = System.Text.RegularExpressions.Regex.Replace(
              cTemp,
              @"\s(?=([^""]*""[^""]*"")*[^""]*$)|\r\n+|\r+|\n+",
              string.Empty
            );
            //消除空白與換行，並進行編碼
            return System.Convert.ToBase64String(
              System.Text.Encoding.UTF8.GetBytes(cTemp)
            )
            .Replace("=", string.Empty)
            .Replace("+", "-")
            .Replace("/", "_")
            ;
        }

        public static string GetBase64Decode(string cTemp)
        { //處理可能被消失或替換掉的=、+、/等符號
            cTemp = cTemp
              .PadRight(cTemp.Length + ((cTemp.Length % 4 != 0) ? (4 - cTemp.Length % 4) : 0), '=')
              .Replace("-", "+")
              .Replace("_", "/");
            return System.Text.Encoding.UTF8.GetString(
              System.Convert.FromBase64String(cTemp)
            );
        }

        public static string GetHmacSha256(string cTemp, string cKey = "your-256-bit-secret")
        {
            using (var oHash = new System.Security.Cryptography.HMACSHA256(
              System.Text.Encoding.UTF8.GetBytes(cKey)
            ))
            {
                byte[] bytMessage = oHash.ComputeHash(System.Text.Encoding.UTF8.GetBytes(cTemp));
                return System.Convert.ToBase64String(bytMessage)
                  .Replace("=", string.Empty)
                  .Replace("+", "-")
                  .Replace("/", "_");
            }
        }



        //public static string HMACSHA256(string message, string key)
        //{
        //    var encoding = new UTF8Encoding();
        //    byte[] keyByte = HexToByte(key);
        //    byte[] messageBytes = encoding.GetBytes(message);
        //    using (var hmacSHA256 = new HMACSHA256(keyByte))
        //    {
        //        byte[] hashMessage = hmacSHA256.ComputeHash(messageBytes);
        //        return BitConverter.ToString(hashMessage).Replace("-", "").ToLower();
        //    }
        //}

        public static byte[] HexToByte(string hexString)
        {
            //運算後的位元組長度:16進位數字字串長/2
            byte[] byteOUT = new byte[hexString.Length / 2];
            for (int i = 0; i < hexString.Length; i = i + 2)
            {
                //每2位16進位數字轉換為一個10進位整數
                byteOUT[i / 2] = Convert.ToByte(hexString.Substring(i, 2), 16);
            }
            return byteOUT;
        }
        public static string MD5Hash(string K)
        {
            byte[] O = Encoding.Default.GetBytes(K);
            MD5 M = MD5.Create(); // 使用 MD5 
            byte[] C = M.ComputeHash(O);// 進行加密 
            string Temp = "";
            foreach (byte ibyte in C)
            {
                Temp += ibyte.ToString("x2");
            }

            Temp = Temp.Replace("-", "");
            return Temp;
        }
        public static string SHA256Hash(string text)
        {
            SHA256 sha256 = new SHA256CryptoServiceProvider();//建立一個SHA256
            byte[] source = Encoding.UTF8.GetBytes(text);//將字串轉為Byte[]
            byte[] crypto = sha256.ComputeHash(source);//進行SHA256加密
            string result = Convert.ToBase64String(crypto);//把加密後的字串從Byte[]轉為字串
            return result;
        }
        public static string getResult(string url)
        {
            string responseString = string.Empty;
            HttpClient httpClient = new HttpClient();
            httpClient.DefaultRequestHeaders.Accept.Add(
               new MediaTypeWithQualityHeaderValue("application/json"));
            HttpResponseMessage result = httpClient.GetAsync(url).Result;

            if (result.IsSuccessStatusCode)
            {
                responseString = result.Content.ReadAsStringAsync().Result;
            }
            else
            {
                responseString = result.Content.ReadAsStringAsync().Result;
                Log.Info($"postResult失敗，StatusCode：{result.StatusCode}，RequestMessage：{result.RequestMessage.ToString()}");
            }
            Log.Info(responseString);
            return responseString;
        }
        public static object postResult(string url, object postData, string Authorization = "")
        {
            object responseJson = string.Empty;
            //使用加密憑證
            ServicePointManager.ServerCertificateValidationCallback = delegate { return true; };
            //提高預設認證等級
            ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;
            //建立 HttpClient
            HttpClient client = new HttpClient();
            // 指定 authorization header
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            if (!string.IsNullOrEmpty(Authorization))
            {
                //client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue(Authorization);
                client.DefaultRequestHeaders.Add("Authorization", Authorization);
            }

            // 準備寫入的 data
            //PostData postData = new PostData() { userId = 123422, title = "yowko 中文", body = "yowko test body 中文" };
            HttpResponseMessage result = client.PostAsync(url, new JsonContent(postData)).GetAwaiter().GetResult();
            if (result.IsSuccessStatusCode)
            {
                responseJson = result.Content.ReadAsStringAsync().GetAwaiter().GetResult();
                // 將回應結果轉換為JObject
                try
                {
                    responseJson = JObject.Parse(responseJson.ToString());
                }
                catch
                {
                }
            }
            else
            {
                //LogHelper.Info($"postResult失敗，StatusCode：{result.StatusCode}，RequestMessage：{result.RequestMessage.ToString()}");
                responseJson = result.Content.ReadAsStringAsync().Result;
                // 將回應結果轉換為JObject
                try
                {
                    responseJson = JObject.Parse(responseJson.ToString());
                }
                catch
                {
                }
            }
            //LogHelper.Info($"postResult：{responseJson}");
            // 將回應結果內容取出並轉為 string 再透過 linqpad 輸出
            return responseJson;
        }
        public static string postResultDecoder(string url, string postData)
        {
            string responseJson = string.Empty;
            //建立 HttpClient
            HttpClient client = new HttpClient();
            //client.DefaultRequestHeaders.Add("Content-Type", "application/json; charset=UTF-8");
            // 指定 authorization header
            //client.DefaultRequestHeaders.Add("authorization", "token {api token}");

            // 準備寫入的 data
            //PostData postData = new PostData() { userId = 123422, title = "yowko 中文", body = "yowko test body 中文" };
            HttpResponseMessage result = client.PostAsync(url, new StringContent(postData)).GetAwaiter().GetResult();
            if (result.IsSuccessStatusCode)
            {
                responseJson = result.Content.ReadAsStringAsync().GetAwaiter().GetResult();
                //byte[] bb = result.Content.ReadAsByteArrayAsync().GetAwaiter().GetResult();
                //Decoder decoder = Encoding.GetEncoding("big5").GetString(bb).GetDecoder();
                responseJson = System.Web.HttpUtility.UrlDecode(responseJson, Encoding.GetEncoding("big5"));
            }
            else
            {
                responseJson = result.Content.ReadAsStringAsync().Result;
                Log.Info($"postResult失敗，StatusCode：{result.StatusCode}，RequestMessage：{result.RequestMessage.ToString()}");
            }
            Log.Info($"postResult：{responseJson}");
            // 將回應結果內容取出並轉為 string 再透過 linqpad 輸出
            return responseJson;
        }
        public static bool PingIP(string IP)
        {
            // Ping's the local machine.
            Ping pingSender = new Ping();
            IPAddress address = IPAddress.Parse(IP);
            PingReply reply = pingSender.Send(address);
            Log.Info($"PingIP{IP} reply.Status:{reply.Status.ToString()}！");

            return reply.Status == IPStatus.Success;
        }
    }
    public class JsonContent : StringContent
    {
        public JsonContent(object obj) :
            base(JsonConvert.SerializeObject(obj), Encoding.UTF8, "application/json")
        { }
    }
    public static class HttpClientExtensions
    {
        public async static Task<HttpResponseMessage> PatchAsync(this HttpClient client, string requestUri, HttpContent content)
        {
            var method = new HttpMethod("PATCH");

            var request = new HttpRequestMessage(method, requestUri)
            {
                Content = content
            };

            return await client.SendAsync(request);
        }

        public async static Task<HttpResponseMessage> PatchAsync(this HttpClient client, Uri requestUri, HttpContent content)
        {
            var method = new HttpMethod("PATCH");

            var request = new HttpRequestMessage(method, requestUri)
            {
                Content = content
            };

            return await client.SendAsync(request);
        }

        public async static Task<HttpResponseMessage> PatchAsync(this HttpClient client, string requestUri, HttpContent content, CancellationToken cancellationToken)
        {
            var method = new HttpMethod("PATCH");

            var request = new HttpRequestMessage(method, requestUri)
            {
                Content = content
            };

            return await client.SendAsync(request, cancellationToken);
        }

        public async static Task<HttpResponseMessage> PatchAsync(this HttpClient client, Uri requestUri, HttpContent content, CancellationToken cancellationToken)
        {
            var method = new HttpMethod("PATCH");

            var request = new HttpRequestMessage(method, requestUri)
            {
                Content = content
            };

            return await client.SendAsync(request, cancellationToken);
        }
    }
}
```