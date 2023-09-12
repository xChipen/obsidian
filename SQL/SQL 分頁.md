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