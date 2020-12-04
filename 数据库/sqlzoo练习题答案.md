

# sqlzoo练习题答案

[toc]

## 从[SELECT from world](https://sqlzoo.net/wiki/SELECT_from_WORLD_Tutorial)的第5题开始，前面的答案没存

5. `select name,population from world where name in('France','Germany','Italy');`

6. `select name from world where name like '%United%'`

7. `select name,population,area  from world where area>3000000 or population>250000000`

8. `select name,population,area from world  where (area > 3000000 or population>250000000) and name not in ('United States', 'India', 'China')`

9. `select name,round(population/1000000,2) ,round(gdp/1000000000,2)  from world where continent ='South America'`

10. `select name,round(gdp/population,-3) from world where gdp>1000000000000`

11. `select name,
    case when continent='Oceania' then 'Australasia'
    else continent
    end 
    from world where name like 'N%'`

12. `select name,case when continent='Europe' or continent='Asia' then 'Eurasia'
    when continent='North America' or continent='South America' or continent='Caribbean'  then 'America'
    else continent
    end
    from world where name like 'A%' or name like 'B%'`

13. ```sql
    -- 原网站好像炸了，这个是别人写的
    SELECT name,continent,
    CASE WHEN continent IN ('Eurasia', 'Turkey')
         THEN 'Europe/Asia'
    
         WHEN continent = 'Oceania'
         THEN 'Australasia'
    
         WHEN continent = 'Caribbean'
              THEN
              CASE
              WHEN name LIKE 'B%'
              THEN 'North America'
              ELSE 'South America'
              END
         ELSE continent
         END
    FROM world
    ORDER BY name ASC;
    ```

## [SELECT from nobel](https://sqlzoo.net/wiki/SELECT_from_Nobel_Tutorial)

1. ```sql
   SELECT yr, subject, winner
     FROM nobel
    WHERE yr = 1950
   ```

2. ```sql
   SELECT winner
     FROM nobel
    WHERE yr = 1962
      AND subject = 'Literature'
   ```

3. ```sql
   select yr,subject from nobel where winner = 'Albert Einstein'
   ```

4. ```sql
   select winner from nobel where yr>=2000 and subject='Peace'
   ```

5. ```sql
   select yr,subject,winner from nobel where yr between 1980 and 1989 and subject='Literature' 
   ```

6. ```sql
   SELECT * FROM nobel
    WHERE winner IN ('Theodore Roosevelt',
                     'Woodrow Wilson',
                     'Jimmy Carter')
   ```

7. ```sql
   select winner from nobel where winner like'John%'
   ```

8. ```sql
   select * from nobel where (yr=1980 and subject='physics') or  (yr=1984 and subject='chemistry')
   ```

9. ```sql
   select * from nobel where yr=1980 and subject not in('Chemistry','Medicine')
   ```

10. ```sql
    select * from nobel where (yr<1910 and subject='Medicine') or (yr>=2004 and subject='Literature')
    ```

11. ```sql
    select * from nobel where winner='PETER GRÜNBERG'
    ```

12. ```sql
    select * from nobel where winner='EUGENE O''NEILL'
    ```

13. ```sql
    select winner,yr,subject from nobel where winner like 'Sir%' order by yr desc, winner asc
    ```

14. ```sql
    SELECT winner, subject
      FROM nobel
     WHERE yr=1984
     ORDER BY subject in('Chemistry','Physics') asc,subject,winner
    ```

## [SELECT in SELECT](https://sqlzoo.net/wiki/SELECT_within_SELECT_Tutorial)

1. ```sql
   select name from world where population > (select population from world where name= 'Russia')
   ```

2. ```sql
   select name from world where gdp/population  > (select gdp/population  from world where name='United Kingdom') and continent='Europe'
   ```

3. ```sql
   select name,continent from world where continent in(select continent from world where name='Argentina ' or name='Australia') order by name 
   ```

4. ```sql
   select name,population from world where population between (select population from world where  name ='Canada')+1  and (select population from world where  name ='Poland')-1
   ```

5. ```sql
   select name,concat(round(100*population/(select population from world where name='Germany'),0),'%') from world where continent='Europe'
   ```

6. ```sql
   select name from world where gdp> ALL(select gdp from world where continent='Europe' and gdp>0)
   ```

7. ```sql
   SELECT continent, name, area FROM world x
     WHERE area >= ALL
       (SELECT area FROM world y
           WHERE y.continent=x.continent
             AND area>0)
   ```

8. ```sql
   select continent,name  from world x where name<=all(select name from world where x.continent=continent)
   ```

9. ```sql
   select name,continent,population from world where continent in(select continent from world x where 25000000>= all(select population from world y where x.continent=y.continent))
   ```

10. ```sql
    select name,continent from world x where population/3> all(select population from world y where x.continent=y.continent and x.name!=y.name)
    ```



## [SUM and COUNT](https://sqlzoo.net/wiki/SUM_and_COUNT)

1. ```sql
   SELECT sum(population) FROM world
   ```

2. ```sql
   select distinct continent from world 
   ```

3. ```sql
   select sum(gdp) from world where continent='Africa'
   ```

4. ```sql
   select count(name) from world where area > 1000000 
   ```

5. ```sql
   select sum(population) from world where name in('France','Germany','Spain')
   ```

6. ```sql
   select continent,count(name) from world group by continent
   ```

7. ```sql
   select continent,count(name) from world where population >10000000 group by continent
   ```

8. ```sql
   select continent from world  group by continent  having sum(population)>100000000
   ```

## [The nobel table can be used to practice more SUM and COUNT functions](https://sqlzoo.net/wiki/The_nobel_table_can_be_used_to_practice_more_SUM_and_COUNT_functions.)

1. ```sql
   SELECT COUNT(subject) FROM nobel
   ```

2. ```sql
   select distinct subject from nobel 
   ```

3. ```sql
   select count(winner) from nobel where subject='Physics' group by subject  
   ```

4. ```sql
   select subject,count(winner) from nobel group by subject	
   ```

5. ```sql
   select subject ,min(yr)  from nobel  group by subject
   ```

6. ```sql
   select subject, count(winner) from nobel where yr=2000 group by subject
   ```

7. ```sql
   select subject,count(distinct winner) from nobel group by subject
   ```

8. ```sql
   select subject,count(distinct yr) from nobel group by subject
   ```

9. ```sql
   select yr from nobel where subject='Physics' group by yr having count(winner)=3
   ```

10. ```sql
    select winner from nobel group by winner having count(winner)>1
    ```

11. ```sql
    select winner from nobel group by winner having count(distinct subject)>1
    ```

12. ```sql
    select yr,subject from nobel where yr>=2000 group by yr,subject having count(winner)=3
    ```

## [JOIN](https://sqlzoo.net/wiki/The_JOIN_operation)

1. ```sql
   select matchid,player from goal where teamid = 'GER'
   ```

2. ```sql
   SELECT id,stadium,team1,team2 FROM game where id=1012
   ```

3. ```sql
   SELECT player,teamid,stadium,mdate  FROM game JOIN goal ON (id=matchid) 
   where teamid='GER'
   ```

4. ```sql
   select team1,team2, player from game join goal on(id=matchid) where player LIKE 'Mario%'
   ```

5. ```sql
   SELECT player, teamid,coach, gtime
     FROM goal JOIN eteam on teamid=id
    WHERE gtime<=10
   ```

6. ```sql
   select mdate,teamname from game join eteam on(team1=eteam.id) where coach='Fernando Santos'
   ```

7. ```sql
   select player from game join goal on (id=matchid) where stadium in('National Stadium, Warsaw')
   ```

8. ```sql
   select distinct player from game join goal on(id=matchid) and (teamid!='GER') and (team1='GER' or team2='GER')
   ```

9. ```sql
   select teamname,count(*) from goal join eteam on(teamid=id)  GROUP BY teamname
   ```

10. ```sql
    select stadium,count(*) from game join goal on id=matchid group by stadium
    ```

11. ```sql
    select matchid,mdate,count(*) from game join goal on id=matchid and (team1='POL' or team2='POL') group by id 
    ```

12. ```sql
    select matchid,mdate,count(*) from game join goal on id=matchid and teamid='GER'
    group by id
    ```

13. ```sql
    -- left join 是因为要计算0:0比分
    -- group by 多个条件的意思是按照多个条件划分
    select mdate,team1,
    sum(case when team1=teamid then 1 else 0 end) score1,
    team2,
    sum(case when team2=teamid then 1 else 0 end) score2 
    from game left join goal on (id=matchid) 
    group by mdate,matchid,team1,team2
    ```

    # [More JOIN operations](https://sqlzoo.net/wiki/More_JOIN_operations/zh)

    1. ```sql
       select id,title from movie where yr=1962
       ```

    2. ```sql
       select yr from movie where title='Citizen Kane' 
       ```

    3. ```sql
       select id,title,yr from movie where title like 'Star Trek%'order by yr
       ```

    4. ```sql
       select title from movie where id in(11768, 11955, 21191)
       ```

    5. ```sql
       select id from actor where name='Glenn Close'
       ```

    6. ```sql
       select id from movie where title='Casablanca' 
       ```

    7. ```sql
       select name from actor join casting on (id=actorid) where movieid=11768
       ```

    8. ```sql
       select name from actor join casting on(id=actorid) join movie on(movieid=movie.id) where title='Alien'
       ```

    9. ```sql
       select title from movie join casting on(movieid=movie.id) join actor on(actor.id=actorid) where name= 'Harrison Ford' 
       ```

    10. ```sql
        select title from movie join casting on(movieid=movie.id) join actor on(actor.id=actorid) where name= 'Harrison Ford' and ord!=1
        ```

    11. ```sql
        select title,name from movie join casting on(movieid=movie.id) join actor on(actor.id=actorid) where yr=1962 and ord=1
        ```

    12. ```sql
        -- 这个语句比较复杂，意思是既然要选择最忙的一年，那么肯定要先选出最忙的一年，然后调用having语句选中最忙的那一年
        select yr,count(title) from movie join casting on(movieid=movie.id) join actor on(actor.id=actorid) 
        where name='John Travolta' group by yr 
        having count(title)=
        (select max(c) from(select count(title) c from movie join casting on(movieid=movie.id) join actor on(actor.id=actorid) where name='John Travolta' group by yr) t)
        ```

    13. ```sql
        select title,name  from movie join casting on(movieid=movie.id) join actor on(actor.id=actorid) where  movieid in(
        select movieid from movie join casting on(movieid=movie.id) join actor on(actor.id=actorid) where name='Julie Andrews'
        ) and  ord=1
        ```

    14. ```sql
        select name from  movie join casting on(movieid=movie.id) join actor on(actor.id=actorid) where ord=1  group by name having count(title)>=30
        order by name 
        ```

    15. ```sql
        -- 不知道为啥错的，翻了网上的答案，很多也是错的，怀疑数据有问题
        select title,count(actorid)  from  movie join casting on(movieid=movie.id)  where yr=1978  group by movieid order by count(actorid) desc,title
        ```

    16. ```sql
        select distinct name from  movie join casting on(movieid=movie.id) join actor on(actor.id=actorid) where movieid in
        (select movieid from  movie join casting on(movieid=movie.id) join actor on(actor.id=actorid) where name='Art Garfunkel')
        and name!='Art Garfunkel'	
        ```

 ## [Using Null](https://sqlzoo.net/wiki/Using_Null)

1. ```sql
   select name from teacher where dept is null
   ```

2. ```sql
   SELECT teacher.name, dept.name
    FROM teacher INNER JOIN dept
              ON (teacher.dept=dept.id)
   ```

3. ```sql
   select teacher.name,dept.name from teacher left join dept on (teacher.dept=dept.id)
   ```

4. ```sql
   select teacher.name,dept.name from teacher right join dept on (teacher.dept=dept.id)
   ```

5. ```sql
   select name,coalesce(mobile, '07986 444 2266') from teacher 
   ```

6. ```sql
   select teacher.name,COALESCE(dept.name,'None')from teacher left join dept on (teacher.dept=dept.id)
   ```

7. ```sql
   select count(name),count(mobile) from teacher
   ```

8. ```sql
   select dept.name,count(teacher.name) from teacher right join dept on(teacher.dept=dept.id) group by dept.name
   ```

9. ```sql
   select name,case when dept=1 or dept=2 then 'Sci'
   else 'Art'
   end
   from teacher 
   ```

10. ```sql
    select name,case when dept=1 or dept=2 then 'Sci'
    when dept=3 then 'Art'
    else 'None'
    end
    from teacher 
    ```

## [Self JOIN](https://sqlzoo.net/wiki/Self_join)

1. ```sql
   select count(distinct id) from stops
   ```

2. ```sql
   select id from stops where name='Craiglockhart' 
   ```

3. ```sql
   -- 没看懂，网上的大答案也是错的
   SELECT id,name FROM stops JOIN route ON id = stop WHERE num = 4 AND company = 'LRT'
   ```

4. ```sql
   SELECT company, num, COUNT(*) FROM route
   WHERE stop=149 OR stop=53
   GROUP BY company, num
   HAVING COUNT(*)=2
   ```

5. ```sql
   SELECT a.company, a.num, a.stop, b.stop
   FROM route a JOIN route b ON
     (a.company=b.company AND a.num=b.num)
   WHERE a.stop=53 and b.stop=149
   ```

6. ```sql
   SELECT a.company, a.num, stopa.name, stopb.name
   FROM route a JOIN route b ON
     (a.company=b.company AND a.num=b.num)
     JOIN stops stopa ON (a.stop=stopa.id)
     JOIN stops stopb ON (b.stop=stopb.id)
   WHERE stopa.name='Craiglockhart' and stopb.name='London Road'
   ```

7. ```sql
   select distinct a.company,a.num from route a inner join route b on a.num=b.num and a.company=b.company and a.stop=115 and b.stop=137
   ```

8. ```sql
   SELECT R1.company, R1.num
     FROM route R1, route R2, stops S1, stops S2
     WHERE R1.num=R2.num AND R1.company=R2.company
       AND R1.stop=S1.id AND R2.stop=S2.id
       AND S1.name='Craiglockhart'
       AND S2.name='Tollcross'
   ```

9. ```sql
   SELECT DISTINCT S2.name, R2.company, R2.num
   FROM stops S1, stops S2, route R1, route R2
   WHERE S1.name='Craiglockhart'
     AND S1.id=R1.stop
     AND R1.company=R2.company AND R1.num=R2.num
     AND R2.stop=S2.id
   ```

10. ```sql
    SELECT DISTINCT bus1.num, bus1.company, name, bus2.num, bus2.company FROM (
    SELECT start1.num, start1.company, stop1.stop FROM route AS start1 JOIN route AS stop1 
    ON start1.num = stop1.num 
    AND start1.company = stop1.company AND start1.stop != stop1.stop WHERE start1.stop = 
    (SELECT id FROM stops WHERE name = 'Craiglockhart')) AS bus1 JOIN (SELECT start2.num, start2.company, start2.stop FROM route AS start2 JOIN route AS stop2 ON start2.num = stop2.num AND start2.company = stop2.company AND start2.stop != stop2.stop WHERE stop2.stop = (SELECT id FROM stops WHERE name = 'Sighthill')) AS bus2 ON bus1.stop = bus2.stop JOIN stops ON bus1.stop = stops.id
    ```

11. 

# 一些关键字的用法

## ROUND

> ROUND(f,p) returns f rounded to p decimal places
>
> The number of decimal places may be negative, this will round to the nearest 10 (when p is -1) or 100 (when p is -2) or 1000 (when p is -3) etc..	

Round 意思是四舍五入，ROUND(f,p)将f保存小数点后p位，可以为负数

```
ROUND(7253.86, 0)    ->  7254
ROUND(7253.86, 1)    ->  7253.9
ROUND(7253.86,-3)    ->  7000
```

## CASE

>CASE allows you to return different values under different conditions.
>
>**If there no conditions match (and there is not ELSE) then NULL is returned.**

CASE 关键字允许按照不同的判断条件返回不同的值，如果不设置别名，字段名就是从CASE。。。到END

需要注意的是，如果不设置，返回值是NULL

```sql
  CASE WHEN condition1 THEN value1 
       WHEN condition2 THEN value2  
       ELSE def_value 
  END 
```

## COALESCE

> COALESCE takes any number of arguments and returns the first value that is not null.

CAOLASCE意思是返回第一个非NULL的值

```sql
COALESCE(x,y,z) = x if x is not NULL
COALESCE(x,y,z) = y if x is NULL and y is not NULL
COALESCE(x,y,z) = z if x and y are NULL but z is not NULL
COALESCE(x,y,z) = NULL if x and y and z are all NULL
```

