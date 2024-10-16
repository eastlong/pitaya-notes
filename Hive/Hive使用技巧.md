# Hive使用技巧

## 判断字符串是否是数字

1、判断字符串是否数字

```sql
select * from dw_db.tb
where d = '2024-10-11'
and colume RLIKE '^-?[0-9]+(\\.[0-9]+)?$'
; 
```

2、判断字符串非数字

```
select * from dw_db.tb
where d = '2024-10-11'
and colume NOT RLIKE '^-?[0-9]+(\\.[0-9]+)?$'
; 
```

