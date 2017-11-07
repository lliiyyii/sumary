## mysql更新数据时使用select查询
> mysql 在删除和更新数据时不能用select里面的一些方法，直接用select就出错，再包一层select就可以了

``` sql
update school set c='社会型' where id in (select id from (select id as id from school where name like '%师范%')as a);
```

## mysql删除重复数据保留id最小的一条时使用select查询
``` sql
delete from php_user where 
username in (select username from ( select username from php_user group by username having count(username)>1) a) 
and id not in ( select min(id) from (select min(id) as id from php_user group by username having count(username)>1 ) b)
```
