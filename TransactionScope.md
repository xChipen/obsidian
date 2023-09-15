[[SQL]] [[CShape]]
#TransactionScope
#Complete

加入:
1. 參考 : System.Transactions
2. using System.Transactions;

使用方式 : 
1. 建立 TransactionScope 物件, 並使用 using
2. transactionScope.Complete(); 完成交易
3. 超過 using 的範圍, 未使用 Complete. 自動 rollback

public static void Main(string[] args)
{
    // 1.  建立物件 Create a transaction scope.
    using (TransactionScope transactionScope = new TransactionScope())
    {
        // Execute some SQL statements.
        SqlConnection connection = new SqlConnection("ConnectionString");
        connection.Open();

        SqlCommand command1 = new SqlCommand("UPDATE Products SET Price = Price * 1.10", connection);
        command1.ExecuteNonQuery();

        SqlCommand command2 = new SqlCommand("INSERT INTO Orders (ProductID, Quantity) VALUES (1, 10)", connection);
        command2.ExecuteNonQuery();

        // 2. 確認交易 Commit the transaction.
        transactionScope.Complete();
    }
}