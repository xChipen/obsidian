[[Notion]]  #Notion 

取區間日期 - 起 : start(prop("日期"))
取區間日期 - 迄 : end(prop("日期"))

取今天是一週的第幾天 : 
formatDate(now(), "E")

日期減少 : 本週 - 起
dateSubtract(now(), toNumber(formatDate(now(), "E")), "days")

日期增加 : 本週 - 迄
dateSubtract(now(), toNumber(formatDate(now(), "E")) - 6, "days")

本週
(start(prop("日期"))<=dateSubtract(now(), toNumber(formatDate(now(), "E")), "days")) and (end(prop("日期"))>=dateSubtract(now(), toNumber(formatDate(now(), "E")) - 6, "days"))or(start(prop("日期"))>=dateSubtract(now(), toNumber(formatDate(now(), "E")), "days")) and (start(prop("日期"))<=dateSubtract(now(), toNumber(formatDate(now(), "E")) - 6, "days"))or(end(prop("日期"))>=dateSubtract(now(), toNumber(formatDate(now(), "E")), "days")) and (end(prop("日期"))<=dateSubtract(now(), toNumber(formatDate(now(), "E")) - 6, "days"))

下週
(start(prop("日期"))<=dateSubtract(now(), toNumber(formatDate(now(), "E"))-7, "days")) and (end(prop("日期"))>=dateSubtract(now(), toNumber(formatDate(now(), "E")) - 13, "days"))or(start(prop("日期"))>=dateSubtract(now(), toNumber(formatDate(now(), "E"))-7, "days")) and (start(prop("日期"))<=dateSubtract(now(), toNumber(formatDate(now(), "E")) - 13, "days"))or(end(prop("日期"))>=dateSubtract(now(), toNumber(formatDate(now(), "E"))-7, "days")) and (end(prop("日期"))<=dateSubtract(now(), toNumber(formatDate(now(), "E")) - 13, "days"))