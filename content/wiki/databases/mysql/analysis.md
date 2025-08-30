---
{"publish":true,"title":"MySQL Analysis","created":"2025-08-29","modified":"2025-08-30T00:19:52.636-04:00","published":"2025-08-29","tags":["databases","mysql","analyze"],"cssclasses":""}
---

MySQL Analysis can be something that is tedious to optimize and help maintain health and growth. Data Usage

How much data do your database tables actually use and how much is growth, how much is unclaimed space?

~~~~~
SELECT IFNULL(B.engine,'Total') "Storage Engine",
  CONCAT(LPAD(REPLACE(FORMAT(B.DSize/POWER(1024,pw),3),',',''),17,' '),' ',SUBSTR(' KMGTP',pw+1,1),'B') "Data Size",
  CONCAT(LPAD(REPLACE(FORMAT(B.ISize/POWER(1024,pw),3),',',''),17,' '),' ',SUBSTR(' KMGTP',pw+1,1),'B') "Index Size",
  CONCAT(LPAD(REPLACE(FORMAT(B.TSize/POWER(1024,pw),3),',',''),17,' '),' ',SUBSTR(' KMGTP',pw+1,1),'B') "Table Size"
FROM (SELECT engine,SUM(data_length) DSize,SUM(index_length) ISize,SUM(data_length+index_length) TSize FROM information_schema.tables
WHERE table_schema NOT IN ('mysql','information_schema','performance_schema')
  AND engine IS NOT NULL GROUP BY engine WITH ROLLUP) B,(SELECT 3 pw) A
  ORDER BY TSize;
~~~~~

This will tell how much space is occupied by data and index for engine storage engine. Notice at the end of the query the clause: (SELECT 3 pw). Change the number to generate the report in different units.

~~~~~
(SELECT 0 pw) for Bytes
(SELECT 1 pw) for KiloBytes
(SELECT 2 pw) for MegaBytes
(SELECT 3 pw) for GigaBytes
(SELECT 4 pw) for TerraBytes
(SELECT 5 pw) for PetaBytes
~~~~~

To find out how much space usage each database is using:

~~~~~
SELECT table_schema "Data Base Name", SUM( data_length + index_length) / 1024 / 1024 "Data Base Size in MB" FROM information_schema.TABLES GROUP BY table_schema;
~~~~~

More extensive view of how much data each database is using:

~~~~~
SELECT
        count(*) tables,
        table_schema,concat(round(sum(table_rows)/1000000,2),'M') rows,
        concat(round(sum(data_length)/(1024*1024*1024),2),'G') data,
        concat(round(sum(index_length)/(1024*1024*1024),2),'G') idx,
        concat(round(sum(data_length+index_length)/(1024*1024*1024),2),'G') total_size,
        round(sum(index_length)/sum(data_length),2) idxfrac
        FROM information_schema.TABLES
        GROUP BY table_schema
        ORDER BY sum(data_length+index_length) DESC LIMIT 10;
~~~~~

By storage engine:

~~~~~
SELECT engine,
        count(*) tables,
        concat(round(sum(table_rows)/1000000,2),'M') rows,
        concat(round(sum(data_length)/(1024*1024*1024),2),'G') data,
        concat(round(sum(index_length)/(1024*1024*1024),2),'G') idx,
        concat(round(sum(data_length+index_length)/(1024*1024*1024),2),'G') total_size,
        round(sum(index_length)/sum(data_length),2) idxfrac
        FROM information_schema.TABLES
        GROUP BY engine
        ORDER BY sum(data_length+index_length) DESC LIMIT 10;
~~~~~

Largest tables:

~~~~~
SELECT CONCAT(table_schema, '.', table_name),
       CONCAT(ROUND(table_rows / 1000000, 2), 'M')                                    rows,
       CONCAT(ROUND(data_length / ( 1024 * 1024 * 1024 ), 2), 'G')                    DATA,
       CONCAT(ROUND(index_length / ( 1024 * 1024 * 1024 ), 2), 'G')                   idx,
       CONCAT(ROUND(( data_length + index_length ) / ( 1024 * 1024 * 1024 ), 2), 'G') total_size,
       ROUND(index_length / data_length, 2)                                           idxfrac
FROM   information_schema.TABLES
ORDER  BY data_length + index_length DESC
LIMIT  10;
~~~~~
