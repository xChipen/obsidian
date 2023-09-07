### format : 轉字串 ; round : 四捨五入
[[Notion]] #Notion 

format( 
round(
( 
((prop("GMail") == true) ? 1 : 0) + ((prop("APP") == true) ? 1 : 0) + ((prop("業績") == true) ? 1 : 0)
) / 3 * 100 
) 
) + "%”
