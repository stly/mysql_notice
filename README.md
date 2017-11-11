Mysql常见注意事项
========

举例
----

### sql的执行顺序。
1. where条件的执行顺序
    ```mysql
    SELECT
    Count(DISTINCT kidid , AttendanceDate) AS attendanceCount ,
    schoolID ,
    attendanceDate
    FROM
    jygy_attendance a
    WHERE
    SchoolID = 1
    AND DelFlag = 0
    AND kidID <> 0
    AND UserType IN(1 , 2)
    AND AttendanceType IN(1 , 2 , 3)
    AND KidID NOT IN(
      SELECT DISTINCT
      AT .KidID
      FROM
      jygy_attendancetrans AS AT
      WHERE
      AT .TransFlag = 0
      AND AT .SchoolID = 1
      AND AT .DelFlag = 0
      AND DATE_FORMAT(AT .AttendanceDate , '%Y-%m-%d') = a.AttendanceDate
      AND AT .PunchState = 3
    )
    AND DATE_FORMAT(AttendanceDate , '%Y-%m-%d') BETWEEN '2017-10-09' AND '2017-10-15'
    GROUP BY
    AttendanceDate

    /*
    注意 AND DATE_FORMAT(AttendanceDate , '%Y-%m-%d') BETWEEN '2017-10-09' AND '2017-10-15'（后面用t1表示）
    该条件会影响到 AND KidID NOT IN(...AND DATE_FORMAT(AT .AttendanceDate , '%Y-%m-%d') = a.AttendanceDate ) 
    (后面用t2表示)里面的条件关联查询，如果我们先将t1 放到t2前面，则t2里面的a.AttendanceDate变为常量值，
    将m*n的查询变为m*N(一个时间段，为一个具体数字常量)，查询效率提高很多
    */


    修改后sql：
    SELECT
    Count(DISTINCT kidid , AttendanceDate) AS attendanceCount ,
    schoolID ,
    attendanceDate
    FROM
    jygy_attendance a
    WHERE
    SchoolID = 1
    AND DelFlag = 0
    AND kidID <> 0
    AND UserType IN(1 , 2)
    AND AttendanceType IN(1 , 2 , 3)
    AND DATE_FORMAT(AttendanceDate , '%Y-%m-%d') BETWEEN '2017-10-09' AND '2017-10-15'        -- 此条件放到前面
    AND KidID NOT IN(
      SELECT DISTINCT
      AT .KidID
      FROM
      jygy_attendancetrans AS AT
      WHERE
      AT .TransFlag = 0
      AND AT .SchoolID = 1
      AND AT .DelFlag = 0
      AND DATE_FORMAT(AT .AttendanceDate , '%Y-%m-%d') = a.AttendanceDate
      AND AT .PunchState = 3
    )
    GROUP BY
    AttendanceDate

    /*
    以后优先把常量条件放到前面
    在本地测试优化前耗时160ms，优化后32ms
    */
    ```
### sql的嵌套
    ```mysql
    select * from t
    -- 防止sql无谓的嵌套
   select * from (
      select * from t
    )t1
    这样会增大一倍的查询时间,在实际编码中我们常常会犯类似的错误，
    这一点我们需要注意。
    ```
参考链接
--------

* [Mysql官网](https://www.mysql.com/)
