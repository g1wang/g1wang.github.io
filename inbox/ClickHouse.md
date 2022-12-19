ClickHouse 磁盘占用过多，手动清理，或者设置日志保留时间

- 手动清理
```
SELECT * FROM system.parts where database = 'system' and `table`= 'trace_log';
alter table system.trace_log drop partition '202211';

SELECT * FROM system.parts where database = 'system' and `table`= 'asynchronous_metric_log';
alter table system.asynchronous_metric_log drop partition '202112'; 

SELECT * FROM system.parts where database = 'system' and `table`= 'part_log';
alter table system.part_log drop partition '202211';

SELECT * FROM system.parts where database = 'system' and `table`= 'query_log';
alter table system.query_log drop partition '202211';

SELECT * FROM system.parts where database = 'system' and `table`= 'query_thread_log';
alter table system.query_thread_log drop partition '202211';

SELECT * FROM system.parts where database = 'system' and `table`= 'metric_log';
alter table system.metric_log drop partition '202211';

```

