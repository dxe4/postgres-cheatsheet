# backup of my brain

# Queries

buffers

```sql
SELECT c.relname 
   , pg_size_pretty(count(*) * 8192) as buffered 
   , round(100.0 * count(*) / ( SELECT setting FROM pg_settings WHERE name='shared_buffers')::integer,1) AS buffers_percent 
   , round(100.0 * count(*) * 8192 / pg_relation_size(c.oid),1) AS percent_of_relation 
  FROM pg_class c 
  INNER JOIN pg_buffercache b ON b.relfilenode = c.relfilenode 
  INNER JOIN pg_database d ON (b.reldatabase = d.oid AND d.datname = current_database()) 
  WHERE pg_relation_size(c.oid) > 0 
  GROUP BY c.oid, c.relname 
  ORDER BY 3 DESC 
  LIMIT 20;
```

cache hit ratio
```sql
SELECT calls, total_time, rows, 100.0 * shared_blks_hit / 
        nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent,
        query
    FROM pg_stat_statements where query != '<insufficient privilege>'
    ORDER BY hit_percent asc LIMIT 100;
```

query locks
```sql

WITH _locks AS  (
  SELECT
    PL.pid, unnest(pg_blocking_pids(PL.pid)) AS blocking_pid
  FROM pg_locks PL
  WHERE NOT granted
)
select
  PSA.pid AS query_pid,
  LEFT(query, 200) AS query,
  CASE
    WHEN PSA.pid = L.pid THEN 'blocked' 
    WHEN PSA.pid = L.blocking_pid THEN 'blocking'
  END AS status,
  L.pid AS blocked_query_pid,
  L.blocking_pid,
  PSA.application_name,
  PSA.state AS transaction_state,
  PSA.backend_xid,
  PSA.query_start
FROM _locks L
JOIN pg_stat_activity PSA ON (
  PSA.pid = L.pid OR
  PSA.pid = L.blocking_pid
)
ORDER BY PSA.query_start;
```


table sizes
```sql
SELECT nspname || '.' || relname AS "relation",
        pg_size_pretty(pg_total_relation_size(C.oid)) AS "total_size"
    FROM pg_class C
    LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
    WHERE nspname NOT IN ('pg_catalog', 'information_schema')
    
    ORDER BY pg_total_relation_size(C.oid) DESC
    LIMIT 10;
```

# Links

https://www.citusdata.com/blog/

https://postgresweekly.com/

https://blog.2ndquadrant.com/

--

https://www.postgresql.org/docs/9.4/static/pgstatstatements.html

https://www.postgresql.org/docs/9.6/static/auto-explain.html

https://www.postgresql.org/docs/9.6/functions-srf.html

https://www.postgresql.org/docs/9.6/functions-window.html

https://www.postgresql.org/docs/9.6/queries-table-expressions.html#QUERIES-GROUPING-SETS

--

https://github.com/dhamaniasad/awesome-postgres

-- 

https://www.pgcli.com/

https://www.postgresql.org/docs/10/static/pgbench.html

https://github.com/grayhemp/pgtoolkit

--

https://medium.com/@hakibenita/be-careful-with-cte-in-postgresql-fca5e24d2119

https://medium.com/carwow-product-engineering/sql-vs-pandas-how-to-balance-tasks-between-server-and-client-side-9e2f6c95677
