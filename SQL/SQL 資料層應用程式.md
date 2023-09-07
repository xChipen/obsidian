[[SQL]] #DAC #待確認
今天有同事來問我，他用 SQL Server 資料庫專案管理 SQL Server 資料庫的結構描述 (Schema)，在開發環境與測試環境的更新與比對都沒有問題，但偏偏正式機上的 SQL Server 資料庫因為防火牆的關係，通常無法直接從本機直接連接，這導致他沒有辦法比對**正式資料庫**與**開發資料庫**之間的差異，如果有人異動了正式資料庫，日後的資料庫更新就很容易出問題。這篇文章，我就來介紹這套 [SqlPackage](https://docs.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage?view=sql-server-ver15&WT.mc_id=DT-MVP-4015686) 命令列工具。

[使用 SqlPackage 對 SQL Server 資料庫進行匯出/匯入/擷取/發行/編寫/報告 | The Will Will Web (miniasp.com)](https://blog.miniasp.com/post/2021/10/14/Using-SqlPackage-to-Extract-DeployReport-DriftReport-Publish-Script-Export-Import)
