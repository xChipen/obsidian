[[Helpers]]

```
using Newtonsoft.Json;
using System;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Threading.Tasks;

namespace Helpers
{
    public class PandaGoHelper
    {
        public static int getUnix_TimeStamp(int add = 1)
        {
            //                                        加一個月           
            int unixTimeSec = (int)DateTime.UtcNow.AddMonths(add)
                .Subtract(new DateTime(1970, 1, 1, 0, 0, 0, DateTimeKind.Utc))
                .TotalSeconds;
            return unixTimeSec;
        }

        // Int32 unixTimestamp = (Int32)(DateTime.Now.Subtract(new DateTime(1970, 1, 1))).TotalSeconds;
        public static DateTime UnixTimeStampToDateTime(double unixTimeStamp)
        {
            // Unix timestamp is seconds past epoch
            DateTime dateTime = new DateTime(1970, 1, 1, 0, 0, 0, 0, DateTimeKind.Utc);
            dateTime = dateTime.AddSeconds(unixTimeStamp).ToLocalTime();
            return dateTime;
        }

        // ContentType : application/json, application/x-www-form-urlencoded, ...
        // application/json = > JsonContent(postData);
        // application/x-www-form-urlencoded => new FormUrlEncodedContent(postData);
        public static bool postResult(string url, HttpContent HC, string ContentType, string Authorization, ref string rs)
        {
            // 回傳值
            HttpResponseMessage result = null;

            //使用加密憑證
            ServicePointManager.ServerCertificateValidationCallback = delegate { return true; };
            //提高預設認證等級
            ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;

            //建立 HttpClient
            HttpClient client = new HttpClient();

            // 指定 authorization header
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue(ContentType));
            if (!string.IsNullOrEmpty(Authorization))
                client.DefaultRequestHeaders.Add("Authorization", Authorization);

            // 準備寫入的 data
            result = client.PostAsync(url, HC).GetAwaiter().GetResult();

            if (result.IsSuccessStatusCode)
            {
                rs = result.Content.ReadAsStringAsync().GetAwaiter().GetResult();
                return true;
            }
            else
            {
                rs = result.Content.ReadAsStringAsync().Result;
                Log.Info($"postResult失敗：{rs}");
                return false;
            }
        }

        public static bool postResult2(string url, object postData, ref string rs, string Authorization = "")
        {
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
                client.DefaultRequestHeaders.Add("Authorization", Authorization);
            }

            // 準備寫入的 data
            HttpResponseMessage result = client.PostAsync(url, new JsonContent(postData)).GetAwaiter().GetResult();
            if (result.IsSuccessStatusCode)
            {
                rs = result.Content.ReadAsStringAsync().GetAwaiter().GetResult();
                return true;
            }
            else
            {
                rs = result.Content.ReadAsStringAsync().Result;
                Log.Info($"postResult失敗：{ rs }");
                return false;
            }
        }

        public static bool deleteResult(string url, object body, string Authorization, ref string rs)
        {
            //使用加密憑證
            ServicePointManager.ServerCertificateValidationCallback = delegate { return true; };
            //提高預設認證等級
            ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;

            var client = new HttpClient();
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            client.DefaultRequestHeaders.Add("Authorization", Authorization);

            string data = JsonConvert.SerializeObject(body);

            HttpResponseMessage result =
                client.SendAsync(
                new HttpRequestMessage(HttpMethod.Delete, url)
                {
                    Content = new StringContent(data, Encoding.UTF8, "application/json")
                }
                ).GetAwaiter().GetResult();

            if (result.IsSuccessStatusCode)
            {
                rs = result.Content.ReadAsStringAsync().GetAwaiter().GetResult();
                return true;
            }
            else
            {
                string error = result.Content.ReadAsStringAsync().Result;
                Log.Info($"postResult失敗：{error}");
                return false;
            }
        }

        public async static Task<string> deleteResult3(string baseUrl, string url, object body, string Authorization)
        {
            //使用加密憑證
            ServicePointManager.ServerCertificateValidationCallback = delegate { return true; };
            //提高預設認證等級
            ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;

            // 建立 HttpClient
            HttpClient client = new HttpClient();

            // 指定 base url 
//            client.BaseAddress = new Uri(baseUrl +);

            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

            // 指定 authorization header
            client.DefaultRequestHeaders.Add("authorization", Authorization);

            // 指定 request 的 method 與 detail url
            HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Delete, baseUrl + "/" +url);

            request.Content = new StringContent(JsonConvert.SerializeObject(body), Encoding.UTF8, "application/json");

            // 發出 post 並將回應內容轉為 string 再透過 linqpad 輸出
            await client.SendAsync(request)
                    .ContinueWith(responseTask =>
                {
                    var result = responseTask.GetAwaiter().GetResult();

                    if (result.IsSuccessStatusCode)
                    {
                        return result.Content.ReadAsStringAsync().GetAwaiter().GetResult();
                    }
                    else
                    {
                        Log.Info($"postResult失敗：{result.Content.ReadAsStringAsync().Result}");
                        return string.Empty;
                    }
                });

            return string.Empty;
        }

        public static bool putResult(string url, object postData, ref string rs, string Authorization = "")
        {
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
                client.DefaultRequestHeaders.Add("Authorization", Authorization);
            }

            // 準備寫入的 data
            HttpResponseMessage result = client.PutAsync(url, new JsonContent(postData)).GetAwaiter().GetResult();
            if (result.IsSuccessStatusCode)
            {
                rs = result.Content.ReadAsStringAsync().GetAwaiter().GetResult();
                return true;
            }
            else
            {
                rs = result.Content.ReadAsStringAsync().Result;
                Log.Info($"postResult失敗：{ rs }");
                return false;
            }
        }

        public static string getResult(string url, string Authorization = "")
        {
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
                client.DefaultRequestHeaders.Add("Authorization", Authorization);
            }
            HttpResponseMessage result = client.GetAsync(url).GetAwaiter().GetResult(); ;

            if (result.IsSuccessStatusCode)
            {
                return result.Content.ReadAsStringAsync().GetAwaiter().GetResult();
            }
            else
            {
                Log.Info($"getResult失敗：{result.Content.ReadAsStringAsync().Result}");
                return string.Empty;
            }
        }

        public static string deleteResult(string url, string Authorization = "")
        {
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
                client.DefaultRequestHeaders.Add("Authorization", Authorization);
            }
            HttpResponseMessage result = client.DeleteAsync(url).GetAwaiter().GetResult(); ;

            if (result.IsSuccessStatusCode)
            {
                return result.Content.ReadAsStringAsync().GetAwaiter().GetResult();
            }
            else
            {
                Log.Info($"getResult失敗：{result.Content.ReadAsStringAsync().Result}");
                return string.Empty;
            }
        }


    }
}
```