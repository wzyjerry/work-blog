# Postgres jsonb 性能测试

## 测试目的

确定千万级数据下一对多关系Postgres独特数据类型jsonb与同构满足第三范式的关系表的查询性能。如果性能优秀，对于只和主实体关联的子表可以使用jsonb存为子文档，结构类似[{},{},{},...]

## 测试方案

1. 产生亿级一对多关系数据，主表master包含字段 (id: int, name: string, info: jsonb)，info结构为{url: string, count: int} ，多端关系表slave包含字段 ( id: int, fid: int, url: string, count: int ) 与info结构对应。

   1. 首先mock name uuid产生1千万条主表记录
   2. 对于每条主表记录，随机生成2～8条info和多端关系表记录，mock url和count，url在1000个候选中选1，count范围为0～65535的整数
   3. 记录统计信息

   ``` bash
   插入master表 1千万条数据 +212s
   ```

2. 对于常见场景设计查询，建立索引并比较查询时间

   1. 查询具有指定url的master id list

      ``` bash
      http://8c8aefe0-2e86-49fc-3a1e-10ae96274ff8
      SELECT DISTINCT master_slaves FROM slaves WHERE url = 'http://8c8aefe0-2e86-49fc-3a1e-10ae96274ff8'
      4383条 0.021s
      
      SELECT id FROM masters WHERE info @> '[{"url": "http://8c8aefe0-2e86-49fc-3a1e-10ae96274ff8"}]'::jsonb;
      4383条 0.140s
      
      SELECT DISTINCT(id) FROM (SELECT id, jsonb_array_elements(info) as info FROM masters) AS expandedtest WHERE info->>'url' = 'http://8c8aefe0-2e86-49fc-3a1e-10ae96274ff8'
      4383条 5.283s
      
      SELECT id FROM masters WHERE info @? '$[*].url ? (@ == "http://8c8aefe0-2e86-49fc-3a1e-10ae96274ff8")'
      4383条 0.103s
      ```

   2. 查询count<30000的master id list

      ``` bash
      SELECT DISTINCT master_slaves FROM slaves WHERE slaves."count" < 30000
      8957744条 26.975s
      
      SELECT id FROM masters WHERE info @? '$[*].count ? (@ < 30000)'
      8957744条 15.584s
      ```

   3. INNER JOIN查询

      ``` bash
      select count(*) from masters INNER JOIN slaves on masters.id=slaves.master_slaves
      45007873条 6.421s
      
      select count(*) from (SELECT id, jsonb_array_elements(info) as info FROM masters) AS expandedtest
      45007873条 4.313s
      ```

      

   



