[[Helpers]]
```
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Reflection;

namespace Auto0800
{
    public static class DBExtension
    {
        //Converts DataTable To List
        public static List<TSource> ToList<TSource>(this DataTable dataTable) where TSource : new()
        {
            const BindingFlags flags = BindingFlags.Public | BindingFlags.Instance | BindingFlags.NonPublic;
            var fieldNames = (from PropertyInfo prop in typeof(TSource).GetProperties(flags)
                              select new { Name = prop.Name, Type = Nullable.GetUnderlyingType(prop.PropertyType) ?? prop.PropertyType }).ToList();
            var dataTblFieldNames = (from DataColumn header in dataTable.Columns
                                     select new { Name = header.ColumnName, Type = header.DataType }).ToList();
            var commonFields = fieldNames.Intersect(dataTblFieldNames).ToList();
            var dataList = new List<TSource>();

            foreach (DataRow row in dataTable.AsEnumerable().ToList())
            {
                var source = new TSource();
                foreach (var field in commonFields)
                {
                    PropertyInfo prop = source.GetType().GetProperty(field.Name);
                    prop.SetValue(source, row[field.Name] == DBNull.Value ? null : row[field.Name], null);
                }
                dataList.Add(source);
            }
            dataTable.Dispose();

            return dataList;
        }

        //Converts DataReader To List 
        public static List<TSource> ToList<TSource>(this IDataReader dataReader, bool autoClose = true) where TSource : new()
        {
            var dataList = new List<TSource>();

            if (dataReader != null)
            {
                while (dataReader.Read())
                {
                    var source = new TSource();
                    foreach (DataRow row in dataReader.GetSchemaTable().Rows)
                    {
                        PropertyInfo prop = source.GetType().GetProperty(row.ItemArray[0].ToString().ToUpper());
                        if (prop != null)
                        {
                            prop.SetValue(source, dataReader[row.ItemArray[0].ToString()] == DBNull.Value ? null : dataReader[row.ItemArray[0].ToString()], null);
                        }
                    }
                    dataList.Add(source);
                }
                if (autoClose)
                {
                    dataReader.Close();
                    dataReader.Dispose();
                }
            }

            return dataList;
        }

        //Converts DataTable To List<Dictionary<string,object>> Extension Method by fred
        public static List<dynamic> ToAnonymosuList(this DataTable dt)
        {
            List<dynamic> result = new List<dynamic>();
            foreach (DataRow row in dt.Rows)
            {
                dynamic dic = new Dictionary<string, object>();
                foreach (DataColumn column in dt.Columns)
                {
                    var colValue = (row[column.ColumnName] == DBNull.Value) ? null : row[column.ColumnName];
                    dic.Add(column.ColumnName, colValue);
                }
                result.Add(dic);
            }
            return result;
        } 
    }
}
```