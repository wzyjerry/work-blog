### Web

```bash
atlas schema inspect -d "postgres://postgres:35197749-1a68-4558-87d6-f1c009f73f20@10.0.0.252:55432/auth-service?sslmode=disable" -w
```

### schema

```bash
atlas schema inspect -d "postgres://postgres:35197749-1a68-4558-87d6-f1c009f73f20@10.0.0.252:55432/auth-service?sslmode=disable" > atlas.hcl
```

### apply

```bash
atlas schema apply -d "postgres://data-center:E%7D%29GXlj%5Ed_x%3EN%3C+BhTyHOvMr@172.23.166.60:5432/data-center" -f datacenter.hcl -w
```

### 官网

https://atlasgo.io/ui/intro

### 发布订阅

```bash
create publication "data-center" for table patent,patent_applicant,patent_inventor,patent_assignee,category,venue_category;
CREATE SUBSCRIPTION "datacenter172" CONNECTION 'host=192.168.6.91 port=5432 dbname=datacenter user=postgres password=9f6ac1e0-b33f-47ad-ba72-a20d32f3646e' publication "data-center";
ALTER PUBLICATION "data-center" drop TABLE patent,patent_applicant,patent_inventor,patent_assignee;
ALTER PUBLICATION "data-center" add TABLE patent,patent_applicant,patent_inventor,patent_assignee;
ALTER SUBSCRIPTION "datacenter172" REFRESH PUBLICATION

select * from pg_publication;
select * from pg_replication_slots;
select * from pg_stat_replication;

select * from pg_subscription_rel;
select * from pg_stat_subscription;
select * from pg_replication_origin_status;
```

`kubectl create secret generic additional-configs --``from``-file=prometheus-additional.yaml -n monitoring`
