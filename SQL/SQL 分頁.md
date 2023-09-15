[[SQL]]

```
$@"DECLARE @PageIndex INT = {data.data.pagination.pageNumber}, @PageSize INT = {data.data.pagination.pageSize}

SELECT RTrim(SystemMenuId) as SystemMenuId, RoleName, AccessRights FROM
    SystemMenuRole
ORDER BY
    SystemMenuId 
OFFSET(@PageIndex) * @PageSize ROWS
 FETCH NEXT @PageSize ROWS ONLY; "
```

**加入總筆數**
**2012** 含之後的 SQL 版本, **CTE** **OFFSET 與 FETCH**
```
Select top 100 * From SystemMenu
where id='b5c2f573-2e84-4d4a-bc1f-234959454fff'

Select top 100 * From BackupLog
where id='b5c2f573-2e84-4d4a-bc1f-234959454fff'

select top 100 * From 


DECLARE @PageIndex INT = 1, @PageSize INT = 10

SELECT* FROM
    SystemMenuRole
ORDER BY
    SystemMenuId
OFFSET(@PageIndex - 1) * @PageSize ROWS
 FETCH NEXT @PageSize ROWS ONLY;
```

**2005**的版本, 利用 **ROW_NUMBER() + BETWEEN ... AND ...**
```
DECLARE @pageSize INT, @pageNo INT;
SET @pageSize = 10;
SET @pageNo = 1;
;WITH T
AS (
    SELECT ROW_NUMBER() OVER (ORDER BY ACNM_COMPID) AS RowNo,*
    FROM ACC_ACCNAME
--    WHERE ListPrice > 10
    ),
T2 AS (
    SELECT COUNT(1) TotalCount FROM T
)
SELECT *
FROM T2, T
WHERE RowNo BETWEEN (@pageNo - 1) * @pageSize  + 1 
AND @pageNo * @pageSize;
```

CTE 非必需
```
DECLARE @pageSize INT, @pageNo INT;
SET @pageSize = 10;
SET @pageNo = 1;

SELECT TotalCount = COUNT(1) OVER (), * 
FROM ACC_ACCNAME
--WHERE ListPrice > 10

ORDER BY ACNM_COMPID OFFSET(@pageNo - 1) * @pageSize ROWS
 
FETCH NEXT @pageSize ROWS ONLY;
```

```
SELECT * FROM products LIMIT 10;

返回 `products` 表中的第 11 到 20 個結果
SELECT * FROM products OFFSET 10 LIMIT 10;

SELECT * FROM products ORDER BY product_id DESC OFFSET 10 LIMIT 10;

使用 `ROW_NUMBER()` 函數來為每個結果分配一個唯一的編號，然後使用 `WHERE` 子句來選擇指定範圍的結果。
SELECT * FROM products ORDER BY product_id DESC WHERE ROW_NUMBER() OVER (ORDER BY product_id DESC) BETWEEN 11 AND 20;
```

