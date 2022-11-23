# SPQR benchmarks

## Test #1

The main goal of the test was to show that adding one more shard is not increasing the query latency. On the one hand, it is very obvious statement but on the other hand we need make sure of this.

### Setup

In this test I benchmarked an SPQR installation:

- with only one spqr-router
- using [2,4,8,16,32,64,128] shards
- using sysbench and its out of box OLTP tests (see `results/` folder)
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

### Results

The query latency is not indeed increased. Test output are stored in `results/` folder.

## Test #2

The main goal of the test was to compare the query latency with and without using of spqr-router.

### Setup

- I created a Managed PostgreSQL 14 Cluster. 
- I use sysbench and its out of box OLTP tests (see `results/` folder)
- sysbench and spqr-router is running on the same host
- Hosts with the router and with postgres each have 8 vCPU, 100% vCPU rate, 16 GB RAM

I made two runs:

1. connecting directly to the cluster
2. connecting via router.

### Results

Raw postgres could process **591.77 transactions per second **, the router made **505.44 tps.**. The difference is 15%.


## Test #3

TODO tpc-c test
