title: mongodb的一些常用查询方法总结
date: 2016-12-27 15:58:25
tags: [mongodb]
categories: 学习笔记 

---
　　1. sql: select distinct(\`item\`) from \`table\` where \`condition\_start\` > start and \`condition\_end\` < end;  
　　　　mongo: db.table.distinct("item", {"condition\_start" : {"$gt" : start}, "condition\_end" : {"$lt" : end}});  
　　2. sql: select count(distinct(\`item\`)) from \`table\` where \`condition\_start\` > start and \`condition\_end\` < end;  
　　　　mongo: db.table.aggregate([{"$group": {"\_id":"$item", "count" : {"$sum" : 1}}}, {"$group":{"\_id":1,"totalCount":{"$sum": "$count"}, "distinctCount":{"$sum":1}}}])  
　　3. sql: select count(distinct(\`item\`)) from \`table\` where \`condition\_start\` > start and \`condition\_end\` < end group by \`another\` 
　　　　mongo: db.table.aggregate([{"$group" : {"\_id" : {"item\_id" : "$item", "another\_id" : "$another"}, "count" : {"$sum" : 1}}}, {"$group" : {"\_id" : "$\_id.another\_id", "distinctCount" : {"$sum" : 1}, "total" : {"$sum" : "$count"}}}])  

### 参考资料  
　　[SQL to Aggregation Mapping Chart](https://docs.mongodb.com/manual/reference/sql-aggregation-comparison/)