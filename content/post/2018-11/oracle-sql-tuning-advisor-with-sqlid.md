+++
banner = "oracle-sql-tuning-advisor-with-sqlid/oracle-sql-tuning-advisor-with-sqlid.png"
categories = ["work"]
date = "2018-11-05T21:36:13+01:00"
description = ""
images = []
menu = ""
tags = ["oracle","database"]
title = "SQL Tuning Advisor per uno specifico sql_id"

+++

Quando eseguiamo il tuning advisor di Oracle, specificando il sql_id di una query SQL, possiamo ottenere prezioni suggerimenti per migliorare le prestazioni delle istruzioni SQL.

In questo tutorial spiegherò come eseguire sql tuning advisor avendo a disposizione il sql_id di uno statement SQL.

Supponiamo che l'ID sql sia - 87s8z2zzpsg88

<!--more-->

* Creazione un tuning task

Per prima cosa si crea un task

```sql
DECLARE
  l_sql_tune_task_id  VARCHAR2(100);
BEGIN
  l_sql_tune_task_id := DBMS_SQLTUNE.create_tuning_task (
                          sql_id      => '87s8z2zzpsg88',
                          scope       => DBMS_SQLTUNE.scope_comprehensive,
                          time_limit  => 500,
                          task_name   => '87s8z2zzpsg88_tuning_task11',
                          description => 'Tuning task1 for statement 87s8z2zzpsg88');
  DBMS_OUTPUT.put_line('l_sql_tune_task_id: ' || l_sql_tune_task_id);
END;
/
```

* Esecuzione del tuning task

```sql
EXEC DBMS_SQLTUNE.execute_tuning_task(task_name => '87s8z2zzpsg88_tuning_task11');
```

* Recupero dei suggerimenti del tuning advisor

```sql
set long 65536
set longchunksize 65536
set linesize 100
select dbms_sqltune.report_tuning_task('87s8z2zzpsg88_tuning_task11') from dual;
```

* Vedere la lista dei task esistenti

```sql
SELECT TASK_NAME, STATUS FROM DBA_ADVISOR_LOG WHERE TASK_NAME = '87s8z2zzpsg88_tuning_task11';
```

* Eliminare un task esistente

```sql
execute dbms_sqltune.drop_tuning_task('87s8z2zzpsg88_tuning_task11');
```

## Se il sql_id non è presente nel cursore, ma lo è nello snap AWR

Per prima cosa dobbiamo trovare lo snapshot di inizio e fine in cui si trova il sql_id

```sql
select
  a.instance_number inst_id,
  a.snap_id,a.plan_hash_value,
  to_char(begin_interval_time,'dd-mon-yy hh24:mi') btime,
  abs(extract(minute from (end_interval_time-begin_interval_time)) + extract(hour from (end_interval_time-begin_interval_time))*60 + extract(day from (end_interval_time-begin_interval_time))*24*60) minutes,
  executions_delta executions,
  round(ELAPSED_TIME_delta/1000000/greatest(executions_delta,1),4) "avg duration (sec)"
from
  dba_hist_SQLSTAT a,
  dba_hist_snapshot b
where
  sql_id='&sql_id'
and a.snap_id=b.snap_id
and a.instance_number=b.instance_number
order by
  snap_id desc, a.instance_number;
```

Da cui otteniamo

1. begin_snap -> 235
2. end_snap -> 240

* Ora creiamo il task del tuning advisor

```sql
DECLARE
  l_sql_tune_task_id  VARCHAR2(100);
BEGIN
  l_sql_tune_task_id := DBMS_SQLTUNE.create_tuning_task (
                          begin_snap  => 235,
                          end_snap    => 240,
                          sql_id      => '24pzs2d6a6b13',
                          scope       => DBMS_SQLTUNE.scope_comprehensive,
                          time_limit  => 60,
                          task_name   => '24pzs2d6a6b13_AWR_tuning_task',
                          description => 'Tuning task for statement 24pzs2d6a6b13  in AWR');
  DBMS_OUTPUT.put_line('l_sql_tune_task_id: ' || l_sql_tune_task_id);
END;
/

```

* Eseguiamo il tuning task

```sql
EXEC DBMS_SQLTUNE.execute_tuning_task(task_name => '24pzs2d6a6b13_AWR_tuning_task');
```

* Recuperiamo i suggerimenti del tuning advisor

```sql
SET LONG 10000000;
SET PAGESIZE 100000000

SET LINESIZE 200
SELECT DBMS_SQLTUNE.report_tuning_task('24pzs2d6a6b13_AWR_tuning_task') AS recommendations FROM dual;
SET PAGESIZE 24
```
## Riferimenti

* http://dbaclass.com/article/how-to-run-sql-tuning-advisor-for-a-sql_id/
