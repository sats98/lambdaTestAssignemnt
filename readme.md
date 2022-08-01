##Problem statement

A SQL table containing columns testCaseId , startDate, endDate and status of a Table Testcases is given.
A lot of data is streaming in continously. You need to find the maximum concurrency occured in a given period of time (lets say last hour).


I am trying to make a result using all the distinct timestamps in the table. (UNION strips duplicates.)

 ```
 SELECT startDate as time FROM Testcases
 UNION 
 SELECT endDate as time FROM Testcases
 ```
 
 After getting the result i can see how many testcases are active/running at these points in time. So I am grouping by 
 
```
SELECT COUNT(*) as concurrent, time.time
FROM Testcases 
JOIN (
    SELECT startDate as time FROM Testcases
    UNION 
    SELECT endDate as time FROM Testcases
) time ON Testcases.startDate <= time.time AND Testcases.endDate > time.time
GROUP BY time.time
```

We can compund index on (startDate,endDate) 
Now we can group the result set by the specified time here (hourly) to get the maximum cuncurrency for the last hour.

```
SELECT DATE_FORMAT(time, '%Y-%m-%d %H:00') hour_beginning,
       MAX(concurrent) concurrent
    FROM (
         SELECT COUNT(*) as concurrent, time.time
            FROM Testcases 
            JOIN (
                SELECT startDate as time FROM Testcases
                UNION 
                SELECT endDate as time FROM Testcases
                ) time ON Testcases.startDate <= time.time AND Testcases.endDate > time.time
                GROUP BY time.time
        ) q
        GROUP BY DATE_FORMAT(time, '%Y-%m-%d %H:00') 
 ```

This is the final query


// example input

CREATE TABLE Testcases (
  `id` INTEGER,
  `startDate` DATETIME,
  `endDate` DATETIME
  `status` Boolean
);

INSERT INTO Testcases
  (`id`, `startDate`, `endDate`)
VALUES
  ('1', '2011-12-19 06:00:00', '2011-12-19 08:45:00',true),
  ('2', '2011-12-19 06:15:00', '2011-12-19 06:30:00',true),
  ('3', '2011-12-19 06:30:00', '2011-12-19 06:45:00'true),
  ('4', '2011-12-19 06:40:00', '2011-12-19 07:15:00'true),
  ('5', '2011-12-19 07:15:00', '2011-12-19 08:45:00'true),
  ('6', '2011-12-19 07:30:00', '2011-12-19 07:50:00'true),
  ('7', '2011-12-19 08:00:00', '2011-12-19 08:30:00'true),










