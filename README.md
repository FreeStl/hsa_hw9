First run docker-compose file to launch mySql db. Then follow steps:

1) Create table 'person_table' with columns:
    ```
   Field	    Type        Null    Key     Default
   id           int         NO              NULL
   name         char(255)   NO              NULL
   bd           date        NO      MUL     NULL
   ```
2) add 10 mln records:
   ```
   INSERT INTO person (id, name, bd)
   SELECT n, CONCAT('Person', n), CURDATE()
   FROM
   (
   select a.N + b.N * 10 + c.N * 100 + d.N * 1000 + e.N * 10000 + f.N * 100000 + g.N * 1000000 + 1 N
   from (select 0 as N union all select 1 union all select 2 union all select 3 union all select 4 union all select 5 union all select 6 union all select 7 union all select 8 union all select 9) a
   , (select 0 as N union all select 1 union all select 2 union all select 3 union all select 4 union all select 5 union all select 6 union all select 7 union all select 8 union all select 9) b
   , (select 0 as N union all select 1 union all select 2 union all select 3 union all select 4 union all select 5 union all select 6 union all select 7 union all select 8 union all select 9) c
   , (select 0 as N union all select 1 union all select 2 union all select 3 union all select 4 union all select 5 union all select 6 union all select 7 union all select 8 union all select 9) d
   , (select 0 as N union all select 1 union all select 2 union all select 3 union all select 4 union all select 5 union all select 6 union all select 7 union all select 8 union all select 9) e
   , (select 0 as N union all select 1 union all select 2 union all select 3 union all select 4 union all select 5 union all select 6 union all select 7 union all select 8 union all select 9) f
   , (select 0 as N union all select 1 union all select 2 union all select 3 union all select 4 union all select 5 union all select 6 union all select 7 union all select 8 union all select 9) g
   ) t
   ```
   add at least one record with unique date:
    ```
    INSERT INTO person_table(id, name, bd)
    VALUES (5, 'test1', '2022-01-24');
    ```
3) Test query without index:
    ```
    explain analyze SELECT * from person_table WHERE bd = '2022-01-24';
    EXPLAIN
    -> Filter: (person_table.bd = DATE'2022-01-24')  (cost=1.16e+6 rows=982645) (actual time=7829..7829 rows=3 loops=1)
    -> Table scan on person_table  (cost=1.16e+6 rows=9.83e+6) (actual time=0.0472..7366 rows=10e+6 loops=1)
    ```
   took about 8 sec
4) Add index on date column bd. MySQL currently supports only BTREE index.
5) Test again with index:
    ```
   explain analyze SELECT * from person_table WHERE bd = '2022-01-24';
    EXPLAIN
    -> Index lookup on person_table using bd (bd=DATE'2022-01-24')  (cost=3.3 rows=3) (actual time=0.0179..0.0232 rows=3 loops=1)
   ```
   took 0.01 sec. Conclusion indexed drastically improve performance;
6) test insert with different values of innodb_flush_log_at_trx_commit:
   -  ```
      SET GLOBAL innodb_flush_log_at_trx_commit = 2;
   
      INSERT INTO person_table(id, name, bd)
      VALUES (13, 'test13', '1996-11-13');
   
      0.004 s
      ```
   - ```
     SET GLOBAL innodb_flush_log_at_trx_commit = 2;
     0.005 s
     ```   
   - ```
     SET GLOBAL innodb_flush_log_at_trx_commit = 1;
     0.018 s
     ```     
   Conclusion: value 0 and 2 have the highest insertion speed;
