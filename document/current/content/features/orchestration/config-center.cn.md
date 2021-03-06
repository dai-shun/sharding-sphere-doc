+++
pre = "<b>3.3.1. </b>"
toc = true
title = "配置中心"
weight = 1
+++

## 实现动机

- 配置集中化：越来越多的运行时实例，使得散落的配置难于管理，配置不同步导致的问题十分严重。将配置集中于配置中心，可以更加有效进行管理。

- 配置动态化：配置修改后的分发，是配置中心可以提供的另一个重要能力。它可支持数据源、表与分片及读写分离策略的动态切换。

## 配置中心数据结构

配置中心在定义的命名空间的config下，以YAML格式存储，包括数据源，分库分表，读写分离、ConfigMap及Properties配置，可通过修改节点来实现对于配置的动态管理。

```
config
    ├──datasource                                # 数据源配置
    ├──sharding                                  # 分库分表（包括分库分表+读写分离）配置根节点
    ├      ├──rule                               # 分库分表（包括分库分表+读写分离）规则
    ├      ├──configmap                          # 分库分表ConfigMap配置，以K/V形式存储，如：{"key1":"value1"}
    ├      ├──props                              # 属性配置
    ├──masterslave                               # 读写分离独立使用配置
    ├      ├──rule                               # 读写分离规则
    ├      ├──configmap                          # 读写分离ConfigMap配置，以K/V形式存储，如：{"key1":"value1"}
    ├      ├──props                              # 属性配置
```

### config/datasource

多个数据库连接池的集合，不同数据库连接池属性自适配（例如：DBCP，C3P0，Druid, HikariCP）。

```yaml
ds_0: !!org.apache.commons.dbcp.BasicDataSource
  defaultAutoCommit: true
  defaultCatalog:
  defaultReadOnly: false
  defaultTransactionIsolation: -1
  driverClassName: com.mysql.jdbc.Driver
  initialSize: 0
  logAbandoned: false
  maxActive: 8
  maxIdle: 8
  maxOpenPreparedStatements: -1
  maxWait: -1
  minEvictableIdleTimeMillis: 1800000
  minIdle: 0
  numTestsPerEvictionRun: 3
  password: ''
  removeAbandoned: false
  removeAbandonedTimeout: 300
  testOnBorrow: false
  testOnReturn: false
  testWhileIdle: false
  timeBetweenEvictionRunsMillis: -1
  url: jdbc:mysql://localhost:3306/demo_ds_0
  username: root
  validationQuery:
  validationQueryTimeout: -1
ds_1: !!org.apache.commons.dbcp.BasicDataSource
  defaultAutoCommit: true
  defaultCatalog:
  defaultReadOnly: false
  defaultTransactionIsolation: -1
  driverClassName: com.mysql.jdbc.Driver
  initialSize: 0
  logAbandoned: false
  maxActive: 8
  maxIdle: 8
  maxOpenPreparedStatements: -1
  maxWait: -1
  minEvictableIdleTimeMillis: 1800000
  minIdle: 0
  numTestsPerEvictionRun: 3
  password: ''
  removeAbandoned: false
  removeAbandonedTimeout: 300
  testOnBorrow: false
  testOnReturn: false
  testWhileIdle: false
  timeBetweenEvictionRunsMillis: -1
  url: jdbc:mysql://localhost:3306/demo_ds_1
  username: root
  validationQuery:
  validationQueryTimeout: -1
```

### config/sharding/rule

分库分表配置，包括分库分表 + 读写分离配置。

```yaml
tables:
  t_order:
    actualDataNodes: ds_$->{0..1}.t_order_$->{0..1}
    keyGeneratorColumnName: order_id
    logicTable: t_order
    tableStrategy:
      inline:
        algorithmExpression: t_order_$->{order_id % 2}
        shardingColumn: order_id
  t_order_item:
    actualDataNodes: ds_$->{0..1}.t_order_item_$->{0..1}
    keyGeneratorColumnName: order_item_id
    logicTable: t_order_item
    tableStrategy:
      inline:
        algorithmExpression: t_order_item_$->{order_id % 2}
        shardingColumn: order_id
defaultDatabaseStrategy:
  inline:
    algorithmExpression: ds_$->{user_id % 2}
    shardingColumn: user_id      
masterSlaveRules: {}
```

### config/sharding/configmap

分库分表ConfigMap配置，以K/V形式存储。

```yaml
key1: value1
```

### config/sharding/props

相对于sharding-sphere配置里面的Sharding Properties。

```yaml
executor.size: 20
sql.show: true
```

### config/masterslave/rule

读写分离独立使用时使用该配置。

```yaml
name: ds_ms
masterDataSourceName: ds_master 
slaveDataSourceNames:
  - ds_slave0
  - ds_slave1
loadBalanceAlgorithmType: ROUND_ROBIN
```

### config/masterslave/configmap

读写分离ConfigMap配置，以K/V形式存储。

```yaml
key2: value2
```
