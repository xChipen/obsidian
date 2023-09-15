[[SQL]] [[CShape]]
#SqlTransaction #BeginTransaction
#Commit
#Rollback
```
public static void Main(string[] args)
{
    // Create a connection to the database.
    SqlConnection connection = new SqlConnection("ConnectionString");

    // 1. 建立交易 Create a transaction.
    SqlTransaction transaction = connection.BeginTransaction();

    try
    {
        // Execute some SQL statements.
        SqlCommand command1 = new SqlCommand("UPDATE Products SET Price = Price * 1.10", connection);
        command1.ExecuteNonQuery();

        SqlCommand command2 = new SqlCommand("INSERT INTO Orders (ProductID, Quantity) VALUES (1, 10)", connection);
        command2.ExecuteNonQuery();

        // 2. 交易確認 Commit the transaction.
        transaction.Commit();
    }
    catch (SqlException e)
    {
        // 3. 交易取消 Roll back the transaction.
        transaction.Rollback();

        // Handle the exception.
        Console.WriteLine(e.Message);
    }

    // Close the connection.
    connection.Close();
}
```

