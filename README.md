# SPQR benchmarks

## Test #1

The main goal of the test was to show that adding one more shard is not increasing the query latency. On the one hand, it is very obvious statement but on the other hand we need make sure of this.

### Setup

In this test I benchmarked an SPQR installation:

- with only one spqr-router
- using [2,4,8,16,32,64,128] shards
- using sysbench and its out of box OLTP tests
- sysbench and spqr-router is running on the same host
- each shard is PostgreSQL 14 with 8 vCPU, 100% vCPU rate Cascade Lake, 32 GB RAM, 100 GB local SSD disk
- host with router has 16 vCPU, 100% vCPU rate, 32 GB RAM, 100 GB local SSD disk

I used `config.py` script to generate the router config and `init.py` to generate the `init.sql`, SQL-like code that creating  key-ranges.

The router config was like:

```
log_level: ERROR
host: localhost
router_port: '6432'
admin_console_port: '7432'
grpc_api_port: '7000'
world_shard_fallback: false
show_notice_messages: false
init_sql: init.sql
router_mode: PROXY
frontend_rules:
  - db: denchick
    usr: denchick
    auth_rule:
      auth_method: ok
      password: ''
    pool_mode: SESSION
    pool_discard: false
    pool_rollback: false
    pool_prepared_statement: true
    pool_default: false
frontend_tls:
  sslmode: disable
backend_rules:
  - db: denchick
    usr: denchick
    auth_rule:
      auth_method: md5
      password: password
    pool_default: false
shards:
  shard01:
....

```

For creating shards I used [Yandex Managed Service for PostgreSQL](https://cloud.yandex.com/en/services/managed-postgresql) and its terraform provider, see `main.tf`.

```
sysbench 
  --threads=24 
  --range-selects=false 
  --table_size=100000 
  --auto_inc=false 
  --tables=10 
  --pgsql-host=localhost 
  --pgsql-port=6432 
  --db-driver=pgsql 
  --pgsql-user=denchick 
  --pgsql-password=password 
  --pgsql-db=denchick
  --pgsql-sslmode=disable
  --verbosity=5
  --db-debug=on  
  --db-ps-mode=disable 
oltp_read_write.lua run
```

### Results

The query latency is not indeed increased. Test output are stored in `results/` folder.

I used sysbench for benchmarking with following parameters:

## Test #2

The main goal of the test was to compare the query latency when with and without spqr-router.

### Setup

TODO

## Test #3

TODO tpc-c test
