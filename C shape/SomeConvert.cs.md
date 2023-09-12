[[Helpers]]

```
using Newtonsoft.Json;
using System;
using System.Collections;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Reflection;
using System.Text;
using System.Web;

namespace Helpers
{
    public static class SomeConvert
    {
        //
        /// <summary>
        /// 伺服器API調用回傳List
        /// </summary>
        /// <typeparam name="T">Model</typeparam>
        /// <param name="url">API網址</param>
        /// <returns>ListModel</returns>
        public static List<T> getLists<T>(string url)
        {
            string error = "";

            string json = getWebData(url, out error);
            if (error == "")
                return jsonToList<T>(json);

            return new List<T>();
        }

        // url => JSON
        public static string getWebData(string url, out string error)
        {
            string Data = "";
            error = "";
            try
            {
                WebClient httpClient = new WebClient();
                httpClient.Encoding = Encoding.UTF8;

                Data = httpClient.DownloadString(url);
            }
            catch (Exception ex)
            {
                error = ex.Message;
            }
            return Data;
        }

        // josn => List<T>
        public static List<T> jsonToList<T>(string json)
        {
            List<T> getData = new List<T>();
            try
            {
                if (!string.IsNullOrEmpty(json))
                    getData = JsonConvert.DeserializeObject<List<T>>(json);
            }
            catch
            {
            }
            return getData;
        }

        /// <summary>
        /// List => DataTable
        /// </summary>
        /// <typeparam name="T">Model</typeparam>
        /// <param name="ListM">ListModel</param>
        /// <returns>DataTable</returns>
        public static DataTable ConvertDataTable<T>(List<T> ListM)
        {
            DataTable dtM = new DataTable(typeof(T).Name);
            //父抬頭
            PropertyInfo[] Mcols = typeof(T).GetProperties(BindingFlags.Public | BindingFlags.Instance);
            if (Mcols != null)
            {
                for (int i = 0; i < Mcols.Length; i++)
                {
                    PropertyInfo col = Mcols[i];

                    if (col.PropertyType.FullName.Contains("System.DateTime"))
                        dtM.Columns.Add(new DataColumn(col.Name, typeof(DateTime)));
                    else
                        dtM.Columns.Add(new DataColumn(col.Name));
                }
            }
            //內容
            foreach (T rowM in ListM)
            //for (int r = 0; r < ListM.Count; r++)
            {
                //T rowM = ListM[r];
                DataRow drM = dtM.NewRow();
                for (int i = 0; i < Mcols.Length; i++)
                {
                    PropertyInfo col = Mcols[i];
                    if (col.GetValue(rowM, null) == null)
                        drM[col.Name] = DBNull.Value;
                    else
                        drM[col.Name] = col.GetValue(rowM, null);
                }

                dtM.Rows.Add(drM);
            }

            return dtM;
        }

        /// <summary>
        /// DataTable => SQL Command 獲取寫入資料庫的SQL語句
        /// </summary>
        /// <param name="dt">寫入資料來源</param>
        /// <param name="strWhere">清除條件</param>
        /// <returns></returns>
        public static string SaveData(DataTable dt, String strWhere, string frontKey = "")
        {
            StringBuilder strCmd = new StringBuilder();
            try
            {
                DateTime TT = DateTime.Now;
                String strTable = dt.TableName;
                strTable = frontKey.Trim() != "" ? frontKey + "." + strTable : strTable;

                string sql = "";

                //清除
                if (strWhere != "")
                {
                    sql = $"DELETE FROM {strTable} WHERE {strWhere} ;";
                    strCmd.AppendLine(sql);
                }
                //加入
                foreach (DataRow dr in dt.Rows)
                {
                    Hashtable ht = new Hashtable();
                    List<string> cols = new List<string>();
                    List<string> vals = new List<string>();
                    foreach (DataColumn dc in dt.Columns)
                    {
                        if (dr.IsNull(dc))
                        {
                            ht.Add(dc.ColumnName, "NULL");
                        }
                        else if (dc.DataType.ToString() == TT.GetType().ToString())
                        {
                            if (TT < DateTime.Parse("1973/07/26"))
                                TT = DateTime.Parse("1973/07/26");
                            else if (TT > DateTime.Parse("2079/06/06"))
                                TT = DateTime.Parse("2079/06/06");
                            else
                                TT = (DateTime)dr[dc];
                            //加上時區小時
                            //ht.Add(dc.ColumnName, "'" + TT.AddHours(8).ToString("yyyy-MM-dd HH:mm:ss") + "'");

                            ht.Add(dc.ColumnName, "'" + TT.ToString("yyyy-MM-dd HH:mm:ss") + "'");
                            //ht.Add(dc.ColumnName, $"to_date('{TT.ToString("yyyy-MM-dd HH:mm:ss")}', 'YYYY-MM-DD HH24:MI:SS')");
                        }
                        else
                            //ht.Add(dc.ColumnName, "N'" + dr[dc].ToString().Trim().Replace("'", "' + char(39) + '").Replace(";", "' + CHAR(59) + '") + "'");
                            ht.Add(dc.ColumnName, "'" + dr[dc].ToString().Trim().Replace("'", "' + char(39) + '").Replace(";", "' + CHAR(59) + '") + "'");
                    }
                    foreach (var item in ht.Keys)
                    {
                        cols.Add(item.ToString());
                        vals.Add(ht[item].ToString());
                    }
                    sql = $"INSERT INTO {strTable} ({string.Join(",", cols.ToArray())}) VALUES ({string.Join(",", vals.ToArray())}) ; ";
                    strCmd.AppendLine(sql);
                }

            }
            catch //(Exception ex)
            {
                //setLog(ex.ToString());
            }
            return strCmd.ToString();
        }

        //public static HttpResponseMessage toJson(Object obj)
        //{
        //    String str;
        //    if (obj is String || obj is Char)
        //    {
        //        str = obj.ToString();
        //    }
        //    else
        //    {
        //        JavaScriptSerializer serializer = new JavaScriptSerializer();
        //        str = serializer.Serialize(obj);
        //    }
        //    HttpResponseMessage result = new HttpResponseMessage { Content = new StringContent(str, Encoding.GetEncoding(“UTF - 8”), “application / json”) };
        //    return result;
        //}

    }
}
```