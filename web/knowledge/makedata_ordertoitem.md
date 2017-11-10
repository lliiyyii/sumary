## 利用数据库存储过程造数据以及将排序结果存入数据库
> 有很多坑，但是太累了，不想说了

```sql
DROP PROCEDURE IF EXISTS make_data;
CREATE PROCEDURE make_data()
  BEGIN
    DECLARE n, i,ch,ma,al INT;
    set n=0,i=20;
    WHILE (n<i) DO
      set ch=FLOOR(rand()*100),ma=floor(rand()*100),al=ch+ma;

      INSERT INTO grade(chinese, math, `all`) VALUES(ch,ma,al);
      SET n=n+1;
END WHILE;
    END ;
CALL make_data();


UPDATE grade SET all_rank = (SELECT i FROM (SELECT (@i:=@i+1) as i,grade.id FROM grade ,(select @i:=0) as it ORDER BY `all`
) as a WHERE a.id=grade.id);

SELECT (@i:=@i+1) as i,grade.* FROM grade ,(select @i:=0) as it ORDER BY `all`;


UPDATE grade SET chinese_rank = (SELECT i FROM (SELECT (@i:=@i+1) as i,grade.id FROM grade ,(select @i:=0) as it ORDER BY chinese
                                           ) as a WHERE a.id=grade.id);
UPDATE grade SET math_rank = (SELECT i FROM (SELECT (@i:=@i+1) as i,grade.id FROM grade ,(select @i:=0) as it ORDER BY math
                                               ) as a WHERE a.id=grade.id);
SELECT (@i:=@i+1) as i,grade.* FROM grade ,(select @i:=0) as it ORDER BY math;
```

