[[Helpers]]

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;
using System.Data;
using System.Data.SqlClient;
using Helpers;

namespace COMM
{
    public class commSQL
    {
        public static SqlConnection conn;

        public static string sConn(string sIP, string sDB, string sUser, string sPW)
        {
            return String.Format("Data Source={0};Initial Catalog = {1}; User ID = {2}; Password = {3}",
                      sIP, sDB, sUser, sPW);
        }

        public static bool openDB(SqlConnection conn, string sConn, ref string sError)
        {
            try
            {
                conn.Close();
                conn.ConnectionString = sConn;

                conn.Open();
                return true;
            }
            catch (SqlException ex)
            {
                sError = ex.Message;
                Log.Info("[openDB] 錯誤訊息:" + sError);
                return false;
            }
        }

        // sData 中取出 SQL 參數
        private static List<string> getParam(string sData)
        {
            List<string> sl = new List<string>();

            // 將 @ 帶頭視為參數
            string[] slData = sData.Split(new char[] {'@'});
            string[] sl_sub = { };

            string ss = "";
            for (int ii=1; ii < slData.Count(); ii++)
            {
                ss = slData[ii];
                // 逗號與空白視為 '結尾'
                sl_sub = slData[ii].Split(new char[] { ',', ' ' });
                if (sl_sub.Length != 0) ss = sl_sub[0];

                sl.Add("@" + ss.Replace('\r', ' ').Replace('\n', ' ').Trim());
            }
            return sl;
        }

        // 參數對應
        private static void addParams(SqlCommand cmd, string sSQL, object[] sqlP)
        {
            if (sqlP == null || sqlP.Count() == 0) return;

            List<string> sl = getParam(sSQL);

            if (sl.Count() != sqlP.Length)  // 參數與資料必須相符
            {
                Log.Info("參數與資料必須相符");
                return;
            }

            for (int ii = 0; ii < sl.Count(); ii++)
            {
                cmd.Parameters.AddWithValue(sl[ii], sqlP[ii] ?? DBNull.Value);
            }
        }

        // overload SQL: Select ; result DataSet
        public static bool RunSQL(SqlConnection conn, string sSQL, ref DataSet ds, params object[] sqlP)
        {
            try
            {
                using (SqlCommand cmd = new SqlCommand(sSQL, conn))
                {
                    addParams(cmd, sSQL, sqlP);

                    using (SqlDataAdapter da = new SqlDataAdapter(cmd))
                    {
                        da.Fill(ds);
                        return true;
                    }
                }
            }
            catch (SqlException ex)
            {
                Log.Info(ex.Message);
                Log.Info("RunSQL: " + sSQL);

                return false;
            }
        }

        // overload Result DataTable
        public static DataTable RunSQL(SqlConnection conn, string sSQL, params object[] sqlP)
        {
            DataTable dt = new DataTable();

            try
            {
                using (SqlCommand cmd = new SqlCommand(sSQL, conn))
                {
                    addParams(cmd, sSQL, sqlP);

                    using (SqlDataAdapter da = new SqlDataAdapter(cmd))
                    {
                        da.Fill(dt);
                        return dt;
                    }
                }
            }
            catch (SqlException ex)
            {
                Log.Info(ex.Message);
                Log.Info("RunSQL: " + sSQL);

                return dt;
            }
        }
        //
        public static bool ExecSQL(SqlConnection conn, string sSQL, params object[] sqlP)
        {
            try
            {
                using (SqlCommand cmd = new SqlCommand(sSQL, conn))
                {
                    addParams(cmd, sSQL, sqlP);

                    cmd.ExecuteNonQuery();
                    return true;
                }
            }
            catch (SqlException ex)
            {
                Log.Info(ex.Message);
                Log.Info(sSQL);

                return false;
            }
        }

        /// <summary>
        /// DataTable Filter Data
        /// </summary>
        /// <param name="oTable"></param>
        /// <param name="filterExpression"></param>
        /// <param name="sortExpression"></param>
        /// <returns></returns>
        public static DataTable DataTable_Filter(DataTable oTable, string filterExpression, string sortExpression = "")
        {
            DataView dv = new DataView();
            dv.Table = oTable;
            dv.RowFilter = filterExpression;
            if (sortExpression != string.Empty)
                dv.Sort = sortExpression;
            DataTable nTable = dv.ToTable();
            return nTable;
        }

        /// <summary>
        /// DataTable Select Data
        /// </summary>
        /// <param name="oTable"></param>
        /// <param name="filterExpression"></param>
        /// <param name="sortExpression"></param>
        /// <returns></returns>
        public static DataTable DataTable_Select(DataTable oTable, string filterExpression, string sortExpression = "")
        {
            DataTable nTable;
            if (sortExpression == string.Empty)
                nTable = oTable.Select(filterExpression).CopyToDataTable();
            else
                nTable = oTable.Select(filterExpression, sortExpression).CopyToDataTable();

            return nTable;
        }

        /// <summary>
        /// get Table primary key
        /// </summary>
        /// <param name="conn"></param>
        /// <param name="sTableName"></param>
        /// <returns></returns>
        public static List<string> getPrimaryKey(SqlConnection conn, string sTableName)
        {
            string ss = @"SELECT ColumnName = col.column_name 
                        FROM information_schema.table_constraints tc 
                        INNER JOIN information_schema.key_column_usage col 
                              ON col.Constraint_Name = tc.Constraint_Name 
                              AND col.Constraint_schema = tc.Constraint_schema 
                        WHERE tc.Constraint_Type = 'Primary Key' AND col.Table_name = '{0}'";
            ss = string.Format(ss, sTableName);
            DataTable dt = commSQL.RunSQL(conn, ss);

            List<string> sl = new List<string>();

            if (dt != null)
            {
                foreach (DataRow dr in dt.Rows)
                {
                    sl.Add(dr["ColumnName"].ToString());
                }
            }
            return sl;
        }

        /// <summary>
        /// 取資料表資訊
        /// </summary>
        /// <param name="conn"></param>
        /// <param name="sTableName"></param>
        /// <returns></returns>
        public static DataTable getTableInfo_dr(SqlConnection conn, string sTableName)
        {
            string ss = string.Format("Select * from {0}", sTableName);
            SqlCommand cmd = new SqlCommand(ss, conn);
            System.Data.SqlClient.SqlDataReader dr = cmd.ExecuteReader(CommandBehavior.KeyInfo);

            DataTable dt = dr.GetSchemaTable();
            if (dr.HasRows)
            {
                for (int ii = dt.Columns.Count - 1; ii >= 0; ii--)
                {
                    if (!(dt.Columns[ii].ColumnName == "ColumnName" ||
                          dt.Columns[ii].ColumnName == "ColumnOrdinal" ||
                          dt.Columns[ii].ColumnName == "ColumnSize" ||
                          dt.Columns[ii].ColumnName == "NumericPrecision" ||
                          dt.Columns[ii].ColumnName == "NumericScale" ||
                          dt.Columns[ii].ColumnName == "IsKey" ||
                          dt.Columns[ii].ColumnName == "AllowDBNull" ||
                          dt.Columns[ii].ColumnName == "DataTypeName"
                          ))
                    {
                        dt.Columns.Remove(dt.Columns[ii]);
                    }
                }
            }
            return dt;
        }

        /// <summary>
        /// get Tbale primary key by DataReader
        /// </summary>
        /// <param name="conn"></param>
        /// <param name="sTableName"></param>
        /// <returns></returns>
        public static List<string> getPrimaryKey_dr(SqlConnection conn, string sTableName)
        {
            DataTable dt = getTableInfo_dr(conn, sTableName);

            List<string> sl = new List<string>();

            if (dt.Rows.Count != 0)
            {
                foreach (DataRow dr in dt.Rows)
                {
                    if ((bool)dr["IsKey"])
                        sl.Add(dr["ColumnName"].ToString());
                }
            }
            return sl;
        }

        /// <summary>
        /// get Table title list
        /// </summary>
        /// <param name="dt"></param>
        /// <returns></returns>
        public static List<string> getTable_Title(DataTable dt)
        {
            List<string> ls = new List<string>();
            foreach (DataColumn dc in dt.Columns)
            {
                ls.Add(dc.ColumnName);
            }
            return ls;
        }

        public static string getTable_Title_Str(DataTable dt)
        {
            List<string> sl = getTable_Title(dt);
            return string.Join(",", sl.ToArray());
        }

        public static void saveToFile(DataTable dt, string sPath, string sTableName)
        {
            if (File.Exists(sPath))
                File.Delete(sPath);

            string sTitle = commSQL.getTable_Title_Str(dt);

            using (StreamWriter sw = new StreamWriter(sPath, true))
            {
                string ss = string.Empty;

                sw.WriteLine(sTableName);       // 資料表名稱
                sw.WriteLine("");
                sw.WriteLine(dt.Rows.Count);    // 筆數     
                sw.WriteLine(sTitle);           // 欄位名稱

                if (dt.Rows.Count != 0)
                {
                    foreach (DataRow dr in dt.Rows)
                    {
                        ss = string.Empty;
                        foreach (DataColumn dc in dt.Columns)
                        {
                            ss += dr[dc].ToString() + ",";
                        }
                        ss = ss.Substring(0, ss.Length - 1);
                        sw.WriteLine(ss);
                    }
                }
            }
        }
    }
}

```