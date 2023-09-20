[[CShape]]

1. 透過 SqlCommand 並設定 cmd.Transaction = _conn.BeginTransaction();
2. 需要透過 new出物件來使用, 因為物漸建立時會處理資料庫連線...
3. Transaction 缺點 : 相關資料表會被鎖住
4. 成功 : cmd.Transaction.Commit();
5. 失敗 : cmd.Transaction.Rollback();

        #region Transaction
        public void BeginTransaction()
        {
            //連接Open
            if (_conn.State != ConnectionState.Open)
                _conn.Open();
            cmd = new SqlCommand();
            cmd.Connection = _conn;
            cmd.Transaction = _conn.BeginTransaction();
        }
        public void Rollback()
        {
            cmd.Transaction.Rollback();
        }
        public void Commit()
        {
            cmd.Transaction.Commit();
        }
        public bool ExecSQL_T(string sSQL, params object[] sqlP)
        {
            try
            {
                cmd.CommandText = sSQL;
                cmd.CommandType = CommandType.Text;
                cmd.Parameters.Clear();

                addParams(cmd, sSQL, sqlP);
                cmd.ExecuteNonQuery();

                return true;
            }
            catch (SqlException ex)
            {
                Log.Info(ex.Message);
                Log.Info(sSQL);

                return false;
            }
        }
        #endregion
