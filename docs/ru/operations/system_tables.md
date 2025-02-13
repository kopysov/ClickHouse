# Системные таблицы

Системные таблицы используются для реализации части функциональности системы, а также предоставляют доступ к информации о работе системы.
Вы не можете удалить системную таблицу (хотя можете сделать DETACH).
Для системных таблиц нет файлов с данными на диске и файлов с метаданными. Сервер создаёт все системные таблицы при старте.
В системные таблицы нельзя записывать данные - можно только читать.
Системные таблицы расположены в базе данных system.

## system.asynchronous_metrics {#system_tables-asynchronous_metrics}

Содержит метрики, которые периодически вычисляются в фоновом режиме. Например, объем используемой оперативной памяти.

Столбцы:

- `metric` ([String](../data_types/string.md)) — название метрики.
- `value` ([Float64](../data_types/float.md)) — значение метрики.

**Пример**

```sql
SELECT * FROM system.asynchronous_metrics LIMIT 10
```

```text
┌─metric──────────────────────────────────┬──────value─┐
│ jemalloc.background_thread.run_interval │          0 │
│ jemalloc.background_thread.num_runs     │          0 │
│ jemalloc.background_thread.num_threads  │          0 │
│ jemalloc.retained                       │  422551552 │
│ jemalloc.mapped                         │ 1682989056 │
│ jemalloc.resident                       │ 1656446976 │
│ jemalloc.metadata_thp                   │          0 │
│ jemalloc.metadata                       │   10226856 │
│ UncompressedCacheCells                  │          0 │
│ MarkCacheFiles                          │          0 │
└─────────────────────────────────────────┴────────────┘
```

**Смотрите также**

- [Мониторинг](monitoring.md) — основы мониторинга в ClickHouse.
- [system.metrics](#system_tables-metrics) — таблица с мгновенно вычисляемыми метриками.
- [system.events](#system_tables-events) — таблица с количеством произошедших событий.

## system.clusters

Содержит информацию о доступных в конфигурационном файле кластерах и серверах, которые в них входят.
Столбцы:

```text
cluster String — имя кластера.
shard_num UInt32 — номер шарда в кластере, начиная с 1.
shard_weight UInt32 — относительный вес шарда при записи данных
replica_num UInt32  — номер реплики в шарде, начиная с 1.
host_name String — хост, указанный в конфигурации.
host_address String — IP-адрес хоста, полученный из DNS.
port UInt16 — порт, на который обращаться для соединения с сервером.
user String — имя пользователя, которого использовать для соединения с сервером.
```

## system.columns

Содержит информацию о столбцах всех таблиц.

С помощью этой таблицы можно получить информацию аналогично запросу [DESCRIBE TABLE](../query_language/misc.md#misc-describe-table), но для многих таблиц сразу.

Таблица `system.columns` содержит столбцы (тип столбца указан в скобках):

- `database` (String) — имя базы данных.
- `table` (String) — имя таблицы.
- `name` (String) — имя столбца.
- `type` (String) — тип столбца.
- `default_kind` (String) — тип выражения (`DEFAULT`, `MATERIALIZED`, `ALIAS`) значения по умолчанию, или пустая строка.
- `default_expression` (String) — выражение для значения по умолчанию или пустая строка.
- `data_compressed_bytes` (UInt64) — размер сжатых данных в байтах.
- `data_uncompressed_bytes` (UInt64) — размер распакованных данных в байтах.
- `marks_bytes` (UInt64) — размер засечек в байтах.
- `comment` (String) — комментарий к столбцу или пустая строка.
- `is_in_partition_key` (UInt8) — флаг, показывающий включение столбца в ключ партиционирования.
- `is_in_sorting_key` (UInt8) — флаг, показывающий включение столбца в ключ сортировки.
- `is_in_primary_key` (UInt8) — флаг, показывающий включение столбца в первичный ключ.
- `is_in_sampling_key` (UInt8) — флаг, показывающий включение столбца в ключ выборки.

## system.contributors {#system_contributors}

Содержит информацию о контрибьютерах. Контрибьютеры расположены в таблице в случайном порядке. Порядок определяется заново при каждом запросе.

Столбцы:

- `name` (String) — Имя контрибьютера (автора коммита) из git log.

**Пример**

```sql
SELECT * FROM system.contributors LIMIT 10
```
```text
┌─name─────────────┐
│ Olga Khvostikova │
│ Max Vetrov       │
│ LiuYangkuan      │
│ svladykin        │
│ zamulla          │
│ Šimon Podlipský  │
│ BayoNet          │
│ Ilya Khomutov    │
│ Amy Krishnevsky  │
│ Loud_Scream      │
└──────────────────┘
```

Чтобы найти себя в таблице, выполните запрос:

```sql
SELECT * FROM system.contributors WHERE name='Olga Khvostikova'
```
```text
┌─name─────────────┐
│ Olga Khvostikova │
└──────────────────┘
```

## system.databases

Таблица содержит один столбец name типа String - имя базы данных.
Для каждой базы данных, о которой знает сервер, будет присутствовать соответствующая запись в таблице.
Эта системная таблица используется для реализации запроса `SHOW DATABASES`.

## system.detached_parts {#system_tables-detached_parts}

Содержит информацию об отсоединённых кусках таблиц семейства [MergeTree](table_engines/mergetree.md). Столбец `reason` содержит причину, по которой кусок был отсоединён. Для кусов, отсоединённых пользователем, `reason` содержит пустую строку.
Такие куски могут быть присоединены с помощью [ALTER TABLE ATTACH PARTITION|PART](../query_language/query_language/alter/#alter_attach-partition). Остальные столбцы описаны в [system.parts](#system_tables-parts).
Если имя куска некорректно, значения некоторых столбцов могут быть `NULL`. Такие куски могут быть удалены с помощью [ALTER TABLE DROP DETACHED PART](../query_language/query_language/alter/#alter_drop-detached).

## system.dictionaries

Содержит информацию о внешних словарях.

Столбцы:

- `name String` — Имя словаря.
- `type String` — Тип словаря: Flat, Hashed, Cache.
- `origin String` — Путь к конфигурационному файлу, в котором описан словарь.
- `attribute.names Array(String)` — Массив имён атрибутов, предоставляемых словарём.
- `attribute.types Array(String)` — Соответствующий массив типов атрибутов, предоставляемых словарём.
- `has_hierarchy UInt8` — Является ли словарь иерархическим.
- `bytes_allocated UInt64` — Количество оперативной памяти, которое использует словарь.
- `hit_rate Float64` — Для cache-словарей - доля использований, для которых значение было в кэше.
- `element_count UInt64` — Количество хранящихся в словаре элементов.
- `load_factor Float64` — Доля заполненности словаря (для hashed словаря - доля заполнения хэш-таблицы).
- `creation_time DateTime` — Время создания или последней успешной перезагрузки словаря.
- `last_exception String` — Текст ошибки, возникшей при создании или перезагрузке словаря, если словарь не удалось создать.
- `source String` - Текст, описывающий источник данных для словаря.


Заметим, что количество оперативной памяти, которое использует словарь, не является пропорциональным количеству элементов, хранящихся в словаре. Так, для flat и cached словарей, все ячейки памяти выделяются заранее, независимо от реальной заполненности словаря.

## system.events {#system_tables-events}

Содержит информацию о количестве событий, произошедших в системе. Например, в таблице можно найти, сколько запросов `SELECT` обработано с момента запуска сервера ClickHouse.

Столбцы:

- `event` ([String](../data_types/string.md)) — имя события.
- `value` ([UInt64](../data_types/int_uint.md)) — количество произошедших событий.
- `description` ([String](../data_types/string.md)) — описание события.

**Пример**

```sql
SELECT * FROM system.events LIMIT 5
```

```text
┌─event─────────────────────────────────┬─value─┬─description────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Query                                 │    12 │ Number of queries to be interpreted and potentially executed. Does not include queries that failed to parse or were rejected due to AST size limits, quota limits or limits on the number of simultaneously running queries. May include internal queries initiated by ClickHouse itself. Does not count subqueries.                  │
│ SelectQuery                           │     8 │ Same as Query, but only for SELECT queries.                                                                                                                                                                                                                │
│ FileOpen                              │    73 │ Number of files opened.                                                                                                                                                                                                                                    │
│ ReadBufferFromFileDescriptorRead      │   155 │ Number of reads (read/pread) from a file descriptor. Does not include sockets.                                                                                                                                                                             │
│ ReadBufferFromFileDescriptorReadBytes │  9931 │ Number of bytes read from file descriptors. If the file is compressed, this will show the compressed data size.                                                                                                                                              │
└───────────────────────────────────────┴───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

**Смотрите также**

- [system.asynchronous_metrics](#system_tables-asynchronous_metrics) — таблица с периодически вычисляемыми метриками.
- [system.metrics](#system_tables-metrics) — таблица с мгновенно вычисляемыми метриками.
- [Мониторинг](monitoring.md) — основы мониторинга в ClickHouse.

## system.functions

Содержит информацию об обычных и агрегатных функциях.

Столбцы:

- `name` (`String`) – Имя функции.
- `is_aggregate` (`UInt8`) – Признак, является ли функция агрегатной.

## system.graphite_retentions

Содержит информацию о том, какие параметры [graphite_rollup](server_settings/settings.md#server_settings-graphite_rollup) используются в таблицах с движками [\*GraphiteMergeTree](table_engines/graphitemergetree.md).

Столбцы:
- `config_name`     (String) - Имя параметра, используемого для `graphite_rollup`.
- `regexp`          (String) - Шаблон имени метрики.
- `function`        (String) - Имя агрегирующей функции.
- `age`             (UInt64) - Минимальный возраст данных в секундах.
- `precision`       (UInt64) - Точность определения возраста данных в секундах.
- `priority`        (UInt16) - Приоритет раздела pattern.
- `is_default`      (UInt8) - Является ли раздел pattern дефолтным.
- `Tables.database` (Array(String)) - Массив имён баз данных таблиц, использующих параметр `config_name`.
- `Tables.table`    (Array(String)) - Массив имён таблиц, использующих параметр `config_name`.


## system.merges

Содержит информацию о производящихся прямо сейчас слияниях и мутациях кусков для таблиц семейства MergeTree.

Столбцы:

- `database String` — Имя базы данных, в которой находится таблица.
- `table String` — Имя таблицы.
- `elapsed Float64` — Время в секундах, прошедшее от начала выполнения слияния.
- `progress Float64` — Доля выполненной работы от 0 до 1.
- `num_parts UInt64` — Количество сливаемых кусков.
- `result_part_name String` — Имя куска, который будет образован в результате слияния.
- `is_mutation UInt8` - Является ли данный процесс мутацией куска.
- `total_size_bytes_compressed UInt64` — Суммарный размер сжатых данных сливаемых кусков.
- `total_size_marks UInt64` — Суммарное количество засечек в сливаемых кусках.
- `bytes_read_uncompressed UInt64` — Количество прочитанных байт, разжатых.
- `rows_read UInt64` — Количество прочитанных строк.
- `bytes_written_uncompressed UInt64` — Количество записанных байт, несжатых.
- `rows_written UInt64` — Количество записанных строк.

## system.metrics {#system_tables-metrics}

Содержит метрики, которые могут быть рассчитаны мгновенно или имеют текущее значение. Например, число одновременно обрабатываемых запросов или текущее значение задержки реплики. Эта таблица всегда актуальна.

Столбцы:

- `metric` ([String](../data_types/string.md)) — название метрики.
- `value` ([Int64](../data_types/int_uint.md)) — значение метрики.
- `description` ([String](../data_types/string.md)) — описание метрики.

**Пример**

```sql
SELECT * FROM system.metrics LIMIT 10
```

```text
┌─metric─────────────────────┬─value─┬─description──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Query                      │     1 │ Number of executing queries                                                                                                                                                                      │
│ Merge                      │     0 │ Number of executing background merges                                                                                                                                                            │
│ PartMutation               │     0 │ Number of mutations (ALTER DELETE/UPDATE)                                                                                                                                                        │
│ ReplicatedFetch            │     0 │ Number of data parts being fetched from replicas                                                                                                                                                │
│ ReplicatedSend             │     0 │ Number of data parts being sent to replicas                                                                                                                                                      │
│ ReplicatedChecks           │     0 │ Number of data parts checking for consistency                                                                                                                                                    │
│ BackgroundPoolTask         │     0 │ Number of active tasks in BackgroundProcessingPool (merges, mutations, fetches, or replication queue bookkeeping)                                                                                │
│ BackgroundSchedulePoolTask │     0 │ Number of active tasks in BackgroundSchedulePool. This pool is used for periodic ReplicatedMergeTree tasks, like cleaning old data parts, altering data parts, replica re-initialization, etc.   │
│ DiskSpaceReservedForMerge  │     0 │ Disk space reserved for currently running background merges. It is slightly more than the total size of currently merging parts.                                                                     │
│ DistributedSend            │     0 │ Number of connections to remote servers sending data that was INSERTed into Distributed tables. Both synchronous and asynchronous mode.                                                          │
└────────────────────────────┴───────┴──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

**Смотрите также**

- [system.asynchronous_metrics](#system_tables-asynchronous_metrics) — таблица с периодически вычисляемыми метриками.
- [system.events](#system_tables-events) — таблица с количеством произошедших событий.
- [Мониторинг](monitoring.md) — основы мониторинга в ClickHouse.

## system.numbers

Таблица содержит один столбец с именем number типа UInt64, содержащим почти все натуральные числа, начиная с нуля.
Эту таблицу можно использовать для тестов, а также если вам нужно сделать перебор.
Чтения из этой таблицы не распараллеливаются.

## system.numbers_mt

То же самое, что и system.numbers, но чтение распараллеливается. Числа могут возвращаться в произвольном порядке.
Используется для тестов.

## system.one

Таблица содержит одну строку с одним столбцом dummy типа UInt8, содержащим значение 0.
Эта таблица используется, если в SELECT запросе не указана секция FROM.
То есть, это - аналог таблицы DUAL, которую можно найти в других СУБД.

## system.parts {#system_tables-parts}

Содержит информацию о кусках данных таблиц семейства [MergeTree](table_engines/mergetree.md).

Каждая строка описывает один кусок данных.

Столбцы:

- `partition` (`String`) – Имя партиции. Что такое партиция можно узнать из описания запроса [ALTER](../query_language/alter.md#query_language_queries_alter).

	Форматы:

	- `YYYYMM` для автоматической схемы партиционирования по месяцам.
	- `any_string` при партиционировании вручную.

- `name` (`String`) – имя куска.
- `active` (`UInt8`) – признак активности. Если кусок активен, то он используется таблицей, в противном случает он будет удален. Неактивные куски остаются после слияний.
- `marks` (`UInt64`) – количество засечек. Чтобы получить примерное количество строк в куске, умножьте `marks` на гранулированность индекса (обычно 8192).
- `rows` (`UInt64`) – количество строк.
- `bytes_on_disk` (`UInt64`) – общий размер всех файлов кусков данных в байтах.
- `data_compressed_bytes` (`UInt64`) – общий размер сжатой информации в куске данных. Размер всех дополнительных файлов (например, файлов с засечками) не учитывается.
- `data_uncompressed_bytes` (`UInt64`) – общий размер распакованной информации куска данных. Размер всех дополнительных файлов (например, файлов с засечками) не учитывается.
- `marks_bytes` (`UInt64`) – размер файла с засечками.
- `modification_time` (`DateTime`) – время модификации директории с куском данных. Обычно соответствует времени создания куска.
- `remove_time` (`DateTime`) – время, когда кусок стал неактивным.
- `refcount` (`UInt32`) – количество мест, в котором кусок используется. Значение больше 2 говорит о том, что кусок участвует в запросах или в слияниях.
- `min_date` (`Date`) – минимальное значение ключа даты в куске данных.
- `max_date` (`Date`) – максимальное значение ключа даты в куске данных.
- `min_time` (`DateTime`) – минимальное значение даты и времени в куске данных.
- `max_time`(`DateTime`) – максимальное значение даты и времени в куске данных.
- `partition_id` (`String`) – ID партиции.
- `min_block_number` (`UInt64`) – минимальное число кусков, из которых состоит текущий после слияния.
- `max_block_number` (`UInt64`) – максимальное число кусков, из которых состоит текущий после слияния.
- `level` (`UInt32`) - глубина дерева слияний. Если слияний не было, то `level=0`.
- `data_version` (`UInt64`) – число, которое используется для определения того, какие мутации необходимо применить к куску данных (мутации с версией большей, чем `data_version`).
- `primary_key_bytes_in_memory` (`UInt64`) – объем памяти (в байтах), занимаемой значениями первичных ключей.
- `primary_key_bytes_in_memory_allocated` (`UInt64`) – объем памяти (в байтах) выделенный для размещения первичных ключей.
- `is_frozen` (`UInt8`) – Признак, показывающий существование бэкапа партиции. 1, бэкап есть. 0, бэкапа нет. Смотрите раздел [FREEZE PARTITION](../query_language/alter.md#alter_freeze-partition).
- `database` (`String`) – имя базы данных.
- `table` (`String`) – имя таблицы.
- `engine` (`String`) – имя движка таблицы, без параметров.
- `path` (`String`) – абсолютный путь к папке с файлами кусков данных..
- `hash_of_all_files` (`String`) – значение [sipHash128](../query_language/functions/hash_functions.md#hash_functions-siphash128) для сжатых файлов.
- `hash_of_uncompressed_files` (`String`) – значение [sipHash128](../query_language/functions/hash_functions.md#hash_functions-siphash128) несжатых файлов (файлы с засечками, первичным ключом и пр.)
- `uncompressed_hash_of_compressed_files` (`String`) – значение [sipHash128](../query_language/functions/hash_functions.md#hash_functions-siphash128) данных в сжатых файлах как если бы они были разжатыми.
- `bytes` (`UInt64`) – алиас для `bytes_on_disk`.
- `marks_size` (`UInt64`) – алиас для `marks_bytes`.


## system.part_log {#system_tables-part-log}

Системная таблица `system.part_log` создается только в том случае, если задана серверная настройка [part_log](server_settings/settings.md#server_settings-part-log).

Содержит информацию о всех событиях, произошедших с [кусками данных](table_engines/custom_partitioning_key.md) таблиц семейства [MergeTree](table_engines/mergetree.md) (например, события добавления, удаления или слияния данных).

Столбцы:

- `event_type` (Enum) — тип события. Столбец может содержать одно из следующих значений: `NEW_PART` — вставка нового куска; `MERGE_PARTS` — слияние кусков; `DOWNLOAD_PART` — загрузка с реплики; `REMOVE_PART` — удаление или отсоединение из таблицы с помощью [DETACH PARTITION](../query_language/alter.md#alter_detach-partition); `MUTATE_PART` — изменение куска; `MOVE_PART` — перемещение куска между дисками.
- `event_date` (Date) — дата события;
- `event_time` (DateTime) — время события;
- `duration_ms` (UInt64) — длительность;
- `database` (String) — имя базы данных, в которой находится кусок;
- `table` (String) — имя таблицы, в которой находится кусок;
- `part_name` (String) — имя куска;
- `partition_id` (String) — идентификатор партиции, в которую был добавлен кусок. В столбце будет значение 'all', если таблица партициируется по выражению `tuple()`;
- `rows` (UInt64) — число строк в куске;
- `size_in_bytes` (UInt64) — размер куска данных в байтах;
- `merged_from` (Array(String)) — массив имён кусков, из которых образован текущий кусок в результате слияния (также столбец заполняется в случае скачивания уже смерженного куска);
- `bytes_uncompressed` (UInt64) — количество прочитанных разжатых байт;
- `read_rows` (UInt64) — сколько было прочитано строк при слиянии кусков;
- `read_bytes` (UInt64) — сколько было прочитано байт при слиянии кусков;
- `error` (UInt16) — код ошибки, возникшей при текущем событии;
- `exception` (String) — текст ошибки.

Системная таблица `system.part_log` будет создана после первой вставки данных в таблицу `MergeTree`.

## system.processes

Эта системная таблица используется для реализации запроса `SHOW PROCESSLIST`.
Столбцы:

```text
user String              - имя пользователя, который задал запрос. При распределённой обработке запроса, относится к пользователю, с помощью которого сервер-инициатор запроса отправил запрос на данный сервер, а не к имени пользователя, который задал распределённый запрос на сервер-инициатор запроса.

address String           - IP-адрес, с которого задан запрос. При распределённой обработке запроса, аналогично.

elapsed Float64          - время в секундах, прошедшее от начала выполнения запроса.

rows_read UInt64         - количество прочитанных из таблиц строк. При распределённой обработке запроса, на сервере-инициаторе запроса, представляет собой сумму по всем удалённым серверам.

bytes_read UInt64        - количество прочитанных из таблиц байт, в несжатом виде. При распределённой обработке запроса, на сервере-инициаторе запроса, представляет собой сумму по всем удалённым серверам.

total_rows_approx UInt64 - приблизительная оценка общего количества строк, которые должны быть прочитаны. При распределённой обработке запроса, на сервере-инициаторе запроса, представляет собой сумму по всем удалённым серверам. Может обновляться в процессе выполнения запроса, когда становятся известны новые источники для обработки.

memory_usage UInt64      - потребление памяти запросом. Может не учитывать некоторые виды выделенной памяти.

query String             - текст запроса. В случае INSERT - без данных для INSERT-а.

query_id String          - идентификатор запроса, если был задан.
```

## system.query_log {#system_tables-query-log}

Содержит информацию о выполнении запросов. Для каждого запроса вы можете увидеть время начала обработки, продолжительность обработки, сообщения об ошибках и другую информацию.

!!! note "Внимание"
    Таблица не содержит входных данных для запросов `INSERT`.

ClickHouse создаёт таблицу только в том случае, когда установлен конфигурационный параметр сервера [query_log](server_settings/settings.md#server_settings-query-log). Параметр задаёт правила ведения лога, такие как интервал логирования или имя таблицы, в которую будут логгироваться запросы.

Чтобы включить логирование, задайте значение параметра [log_queries](settings/settings.md#settings-log-queries) равным 1. Подробности смотрите в разделе [Настройки](settings/settings.md).

Таблица `system.query_log` содержит информацию о двух видах запросов:

1. Первоначальные запросы, которые были выполнены непосредственно клиентом.
2. Дочерние запросы, инициированные другими запросами (для выполнения распределенных запросов). Для дочерних запросов информация о первоначальном запросе содержится в столбцах `initial_*`.

Столбцы:

- `type` (UInt8) — тип события, произошедшего при выполнении запроса. Возможные значения:
    - 1 — успешное начало выполнения запроса.
    - 2 — успешное завершение выполнения запроса.
    - 3 — исключение перед началом обработки запроса.
    - 4 — исключение во время обработки запроса.
- `event_date` (Date) — дата события.
- `event_time` (DateTime) — время события.
- `query_start_time` (DateTime) — время начала обработки запроса.
- `query_duration_ms` (UInt64) — длительность обработки запроса.
- `read_rows` (UInt64) — количество прочитанных строк.
- `read_bytes` (UInt64) — количество прочитанных байтов.
- `written_rows` (UInt64) — количество записанных строк для запросов `INSERT`. Для других запросов, значение столбца 0.
- `written_bytes` (UInt64) — объем записанных данных в байтах для запросов `INSERT`. Для других запросов, значение столбца 0.
- `result_rows` (UInt64) — количество строк в результате.
- `result_bytes` (UInt64) — объём результата в байтах.
- `memory_usage` (UInt64) — потребление RAM запросом.
- `query` (String) — строка запроса.
- `exception` (String) — сообщение исключения.
- `stack_trace` (String) — трассировка (список функций, последовательно вызванных перед ошибкой). Пустая строка, если запрос успешно завершен.
- `is_initial_query` (UInt8) — вид запроса. Возможные значения:
    - 1 — запрос был инициирован клиентом.
    - 0 — запрос был инициирован другим запросом при распределенном запросе.
- `user` (String) — пользователь, запустивший текущий запрос.
- `query_id` (String) — ID запроса.
- `address` (FixedString(16)) — IP адрес, с которого пришел запрос.
- `port` (UInt16) — порт, на котором сервер принял запрос.
- `initial_user` (String) —  пользователь, запустивший первоначальный запрос (для распределенных запросов).
- `initial_query_id` (String) — ID родительского запроса.
- `initial_address` (FixedString(16)) — IP адрес, с которого пришел родительский запрос.
- `initial_port` (UInt16) — порт, на котором сервер принял родительский запрос от клиента.
- `interface` (UInt8) — интерфейс, с которого ушёл запрос. Возможные значения:
    - 1 — TCP.
    - 2 — HTTP.
- `os_user` (String) — операционная система пользователя.
- `client_hostname` (String) — имя сервера, к которому присоединился [clickhouse-client](../interfaces/cli.md).
- `client_name` (String) — [clickhouse-client](../interfaces/cli.md).
- `client_revision` (UInt32) — ревизия [clickhouse-client](../interfaces/cli.md).
- `client_version_major` (UInt32) — старшая версия [clickhouse-client](../interfaces/cli.md).
- `client_version_minor` (UInt32) — младшая версия [clickhouse-client](../interfaces/cli.md).
- `client_version_patch` (UInt32) — патч [clickhouse-client](../interfaces/cli.md).
- `http_method` (UInt8) — HTTP метод, инициировавший запрос. Возможные значения:
    - 0 — запрос запущен с интерфейса TCP.
    - 1 — `GET`.
    - 2 — `POST`.
- `http_user_agent` (String) — HTTP заголовок `UserAgent`.
- `quota_key` (String) — идентификатор квоты из настроек [квот](quotas.md).
- `revision` (UInt32) — ревизия ClickHouse.
- `thread_numbers` (Array(UInt32)) — количество потоков, участвующих в обработке запросов.
- `ProfileEvents.Names` (Array(String)) — Счетчики для изменения метрик:
    - Время, потраченное на чтение и запись по сети.
    - Время, потраченное на чтение и запись на диск.
    - Количество сетевых ошибок.
    - Время, потраченное на ожидание, когда пропускная способность сети ограничена.
- `ProfileEvents.Values` (Array(UInt64)) — метрики, перечисленные в столбце `ProfileEvents.Names`.
- `Settings.Names` (Array(String)) — имена настроек, которые меняются, когда клиент выполняет запрос. Чтобы разрешить логирование изменений настроек, установите параметр `log_query_settings` равным 1.
- `Settings.Values` (Array(String)) — Значения настроек, которые перечислены в столбце `Settings.Names`.

Каждый запрос создаёт одну или две строки в таблице `query_log`, в зависимости от статуса запроса:

1. Если запрос выполнен успешно, создаются два события типа 1 и 2 (смотрите столбец `type`).
2. Если во время обработки запроса произошла ошибка, создаются два события с типами 1 и 4.
3. Если ошибка произошла до запуска запроса, создается одно событие с типом 3.

По умолчанию, строки добавляются в таблицу логирования с интервалом в 7,5 секунд. Можно задать интервал в конфигурационном параметре сервера [query_log](server_settings/settings.md#server_settings-query-log)  (смотрите параметр `flush_interval_milliseconds`). Чтобы принудительно записать логи из буффера памяти в таблицу, используйте запрос `SYSTEM FLUSH LOGS`.

Если таблицу удалить вручную, она пересоздастся автоматически "на лету". При этом все логи на момент удаления таблицы будут удалены.

!!! note "Примечание"
    Срок хранения логов не ограничен. Логи не удаляются из таблицы автоматически. Вам необходимо самостоятельно организовать удаление устаревших логов.

Можно указать произвольный ключ партиционирования для таблицы `system.query_log` в конфигурации [query_log](server_settings/settings.md#server_settings-query-log)  (параметр `partition_by`).

## system.replicas {#system_tables-replicas}

Содержит информацию и статус для реплицируемых таблиц, расположенных на локальном сервере.
Эту таблицу можно использовать для мониторинга. Таблица содержит по строчке для каждой Replicated\*-таблицы.

Пример:

```sql
SELECT *
FROM system.replicas
WHERE table = 'visits'
FORMAT Vertical
```

```text
Row 1:
──────
database:           merge
table:              visits
engine:             ReplicatedCollapsingMergeTree
is_leader:          1
is_readonly:        0
is_session_expired: 0
future_parts:       1
parts_to_check:     0
zookeeper_path:     /clickhouse/tables/01-06/visits
replica_name:       example01-06-1.yandex.ru
replica_path:       /clickhouse/tables/01-06/visits/replicas/example01-06-1.yandex.ru
columns_version:    9
queue_size:         1
inserts_in_queue:   0
merges_in_queue:    1
log_max_index:      596273
log_pointer:        596274
total_replicas:     2
active_replicas:    2
```

Столбцы:

```text
database:           имя БД
table:              имя таблицы
engine:             имя движка таблицы

is_leader:          является ли реплика лидером

В один момент времени, не более одной из реплик является лидером. Лидер отвечает за выбор фоновых слияний, которые следует произвести.
Замечу, что запись можно осуществлять на любую реплику (доступную и имеющую сессию в ZK), независимо от лидерства.

is_readonly:        находится ли реплика в режиме "только для чтения"
Этот режим включается, если в конфиге нет секции с ZK; если при переинициализации сессии в ZK произошла неизвестная ошибка; во время переинициализации сессии с ZK.

is_session_expired: истекла ли сессия с ZK.
В основном, то же самое, что и is_readonly.

future_parts:       количество кусков с данными, которые появятся в результате INSERT-ов или слияний, которых ещё предстоит сделать

parts_to_check:     количество кусков с данными в очереди на проверку
Кусок помещается в очередь на проверку, если есть подозрение, что он может быть битым.

zookeeper_path:     путь к данным таблицы в ZK
replica_name:       имя реплики в ZK; разные реплики одной таблицы имеют разное имя
replica_path:       путь к данным реплики в ZK. То же самое, что конкатенация zookeeper_path/replicas/replica_path.

columns_version:    номер версии структуры таблицы
Обозначает, сколько раз был сделан ALTER. Если на репликах разные версии, значит некоторые реплики сделали ещё не все ALTER-ы.

queue_size:         размер очереди действий, которых предстоит сделать
К действиям относятся вставки блоков данных, слияния, и некоторые другие действия.
Как правило, совпадает с future_parts.

inserts_in_queue:   количество вставок блоков данных, которых предстоит сделать
Обычно вставки должны быстро реплицироваться. Если величина большая - значит что-то не так.

merges_in_queue:    количество слияний, которых предстоит сделать
Бывают длинные слияния - то есть, это значение может быть больше нуля продолжительное время.

Следующие 4 столбца имеют ненулевое значение только если активна сессия с ZK.

log_max_index:      максимальный номер записи в общем логе действий
log_pointer:        максимальный номер записи из общего лога действий, которую реплика скопировала в свою очередь для выполнения, плюс единица
Если log_pointer сильно меньше log_max_index, значит что-то не так.

total_replicas:     общее число известных реплик этой таблицы
active_replicas:    число реплик этой таблицы, имеющих сессию в ZK; то есть, число работающих реплик
```

Если запрашивать все столбцы, то таблица может работать слегка медленно, так как на каждую строчку делается несколько чтений из ZK.
Если не запрашивать последние 4 столбца (log_max_index, log_pointer, total_replicas, active_replicas), то таблица работает быстро.

Например, так можно проверить, что всё хорошо:

```sql
SELECT
    database,
    table,
    is_leader,
    is_readonly,
    is_session_expired,
    future_parts,
    parts_to_check,
    columns_version,
    queue_size,
    inserts_in_queue,
    merges_in_queue,
    log_max_index,
    log_pointer,
    total_replicas,
    active_replicas
FROM system.replicas
WHERE
       is_readonly
    OR is_session_expired
    OR future_parts > 20
    OR parts_to_check > 10
    OR queue_size > 20
    OR inserts_in_queue > 10
    OR log_max_index - log_pointer > 10
    OR total_replicas < 2
    OR active_replicas < total_replicas
```

Если этот запрос ничего не возвращает - значит всё хорошо.

## system.settings

Содержит информацию о настройках, используемых в данный момент.
То есть, используемых для выполнения запроса, с помощью которого вы читаете из таблицы system.settings.

Столбцы:

```text
name String   - имя настройки
value String  - значение настройки
changed UInt8 - была ли настройка явно задана в конфиге или изменена явным образом
```

Пример:

```sql
SELECT *
FROM system.settings
WHERE changed
```

```text
┌─name───────────────────┬─value───────┬─changed─┐
│ max_threads            │ 8           │       1 │
│ use_uncompressed_cache │ 0           │       1 │
│ load_balancing         │ random      │       1 │
│ max_memory_usage       │ 10000000000 │       1 │
└────────────────────────┴─────────────┴─────────┘
```

## system.tables

Содержит метаданные каждой таблицы, о которой знает сервер. Отсоединённые таблицы не отображаются в `system.tables`.

Эта таблица содержит следующие столбцы (тип столбца показан в скобках):

- `database String` — имя базы данных, в которой находится таблица.
- `name` (String) — имя таблицы.
- `engine` (String) — движок таблицы (без параметров).
- `is_temporary` (UInt8) — флаг, указывающий на то, временная это таблица или нет.
- `data_path` (String) — путь к данным таблицы в файловой системе.
- `metadata_path` (String) — путь к табличным метаданным в файловой системе.
- `metadata_modification_time` (DateTime) — время последней модификации табличных метаданных.
- `dependencies_database` (Array(String)) — зависимости базы данных.
- `dependencies_table` (Array(String)) — табличные зависимости (таблицы [MaterializedView](table_engines/materializedview.md), созданные на базе текущей таблицы).
- `create_table_query` (String) — запрос, которым создавалась таблица.
- `engine_full` (String) — параметры табличного движка.
- `partition_key` (String) — ключ партиционирования таблицы.
- `sorting_key` (String) — ключ сортировки таблицы.
- `primary_key` (String) - первичный ключ таблицы.
- `sampling_key` (String) — ключ сэмплирования таблицы.

Таблица `system.tables` используется при выполнении запроса `SHOW TABLES`.

## system.zookeeper

Таблицы не существует, если ZooKeeper не сконфигурирован. Позволяет читать данные из ZooKeeper кластера, описанного в конфигурации.
В запросе обязательно в секции WHERE должно присутствовать условие на равенство path - путь в ZooKeeper, для детей которого вы хотите получить данные.

Запрос `SELECT * FROM system.zookeeper WHERE path = '/clickhouse'` выведет данные по всем детям узла `/clickhouse`.
Чтобы вывести данные по всем узлам в корне, напишите path = '/'.
Если узла, указанного в path не существует, то будет брошено исключение.

Столбцы:

- `name String` — Имя узла.
- `path String` — Путь к узлу.
- `value String` — Значение узла.
- `dataLength Int32` — Размер значения.
- `numChildren Int32` — Количество детей.
- `czxid Int64` — Идентификатор транзакции, в которой узел был создан.
- `mzxid Int64` — Идентификатор транзакции, в которой узел был последний раз изменён.
- `pzxid Int64` — Идентификатор транзакции, последний раз удаливший или добавивший детей.
- `ctime DateTime` — Время создания узла.
- `mtime DateTime` — Время последней модификации узла.
- `version Int32` — Версия узла - количество раз, когда узел был изменён.
- `cversion Int32` — Количество добавлений или удалений детей.
- `aversion Int32` — Количество изменений ACL.
- `ephemeralOwner Int64` — Для эфемерных узлов - идентификатор сессии, которая владеет этим узлом.


Пример:

```sql
SELECT *
FROM system.zookeeper
WHERE path = '/clickhouse/tables/01-08/visits/replicas'
FORMAT Vertical
```

```text
Row 1:
──────
name:           example01-08-1.yandex.ru
value:
czxid:          932998691229
mzxid:          932998691229
ctime:          2015-03-27 16:49:51
mtime:          2015-03-27 16:49:51
version:        0
cversion:       47
aversion:       0
ephemeralOwner: 0
dataLength:     0
numChildren:    7
pzxid:          987021031383
path:           /clickhouse/tables/01-08/visits/replicas

Row 2:
──────
name:           example01-08-2.yandex.ru
value:
czxid:          933002738135
mzxid:          933002738135
ctime:          2015-03-27 16:57:01
mtime:          2015-03-27 16:57:01
version:        0
cversion:       37
aversion:       0
ephemeralOwner: 0
dataLength:     0
numChildren:    7
pzxid:          987021252247
path:           /clickhouse/tables/01-08/visits/replicas
```

## system.mutations {#system_tables-mutations}

Таблица содержит информацию о ходе выполнения [мутаций](../query_language/alter.md#alter-mutations) MergeTree-таблиц. Каждой команде мутации соответствует одна строка. В таблице есть следующие столбцы:

**database**, **table** - имя БД и таблицы, к которой была применена мутация.

**mutation_id** - ID запроса. Для реплицированных таблиц эти ID соответствуют именам записей в директории `<table_path_in_zookeeper>/mutations/` в ZooKeeper, для нереплицированных - именам файлов в директории с данными таблицы.

**command** - Команда мутации (часть запроса после `ALTER TABLE [db.]table`).

**create_time** - Время создания мутации.

**block_numbers.partition_id**, **block_numbers.number** - Nested-столбец. Для мутаций реплицированных таблиц для каждой партиции содержит номер блока, полученный этой мутацией (в каждой партиции будут изменены только куски, содержащие блоки с номерами, меньшими номера, полученного мутацией в этой партиции). Для нереплицированных таблиц нумерация блоков сквозная по партициям, поэтому столбец содержит одну запись с единственным номером блока, полученным мутацией.

**parts_to_do** - Количество кусков таблицы, которые ещё предстоит изменить.

**is_done** - Завершена ли мутация. Замечание: даже если `parts_to_do = 0`, для реплицированной таблицы возможна ситуация, когда мутация ещё не завершена из-за долго выполняющейся вставки, которая добавляет данные, которые нужно будет мутировать.

Если во время мутации какого-либо куска возникли проблемы, заполняются следующие столбцы:

**latest_failed_part** - Имя последнего куска, мутация которого не удалась.

**latest_fail_time** — время последней ошибки мутации.

**latest_fail_reason** — причина последней ошибки мутации.

[Оригинальная статья](https://clickhouse.yandex/docs/ru/operations/system_tables/) <!--hide-->

## system.disks {#system_tables-disks}

Таблица содержит информацию о дисках, заданных в [конфигурации сервера](table_engines/mergetree.md#table_engine-mergetree-multiple-volumes_configure). Имеет следующие столбцы:

- `name String` — имя диска в конфигурации сервера.
- `path String` — путь к точке монтирования на файловой системе.
- `free_space UInt64` — свободное место на диске в данный момент времени в байтах.
- `total_space UInt64` — общее количество места на диске в данный момент времени в байтах.
- `keep_free_space UInt64` — количество байт, которое должно оставаться свободным (задается в конфигурации).


## system.storage_policies {#system_tables-storage_policies}

Таблица содержит информацию о политиках хранения и томах, заданных в [конфигурации сервера](table_engines/mergetree.md#table_engine-mergetree-multiple-volumes_configure). Данные в таблице денормализованны, имя одной политики хранения может содержаться несколько раз, по количеству томов в ней. Имеет следующие столбцы:

- `policy_name String` — имя политики хранения в конфигурации сервера.
- `volume_name String` — имя тома, который содержится в данной политике хранения.
- `volume_priority UInt64` — порядковый номер тома, согласно конфигурации.
- `disks Array(String)` — имена дисков, содержащихся в данной политике хранения.
- `max_data_part_size UInt64` — максимальный размер куска, который может храниться на дисках этого тома (0 — без ограничений).
- `move_factor Float64` — доля свободного места, при превышении которой данные начинают перемещаться на следующий том.
