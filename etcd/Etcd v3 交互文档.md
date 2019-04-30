

### Etcd v3 文档

etcd v3版本接口的常用方法,其它用法参见[Etcd官方文档中文版](<http://etcd.doczh.cn/>)

#### put 写入键值对

写入键来储存键到 etcd 中。每个存储的键通过 Raft 协议复制到 etcd 集群的所有成员来实现一致性和可靠性。

用法说明：

````
put [options] <key> <value> 
	options
	--prev-kv[=false] 返回修改前的键值对,默认值：false
	--lease=<租约ID> 给当前的key设置租约，租约到期key自动删除，默认值：0
````

示例：

```shell
向etcd中写入键为/cron/name1值为value1
# ./etcdctl put /cron/name1 value1
OK

更新时返回修改前的键值对
# ./etcdctl put /cron/name1 value1 --prev-kv
OK
/cron/name1
value1

新增键值对时设置一个租约，租约到期后，当前键值对自动删除
# ./etcdctl put /cron/name2 value2 --lease=694d6a6cd6c6e56c
OK

```

#### get 读取键值对

读取键的值。查询可以读取单个 key，或者某个范围的键。

用法：

```
get [options] <key> [range_end] 注：获取一段连续key时遵循左闭右开原则
	options
	--consistency="l" 数据一致性： 线性(i),串行(s)
	--from-key[=false] 使用字节比较获取大于或等于给定键的键，默认值：false
	--keys-only[=false] 只返回key不返回value
	--limit=0 指定返回的最大个数，默认值：0，不设置
	--order="" 排序，升序(ASCEND);降序(Descend),默认值：升序
	--prefix[=false] 根据key的前缀批量获取，（注：主要用于模拟获取目录),默认值：false
	--print-value-only[=false] 只返回value，仅在输出格式为“simple”时有效
	--rev=0 根据指定kv版本获取key
	--sort-by="" 指定排序目标：CREATE,KEY,MODIFY,VALUE,VERSION
```

实例：

```shell
读取键为/cron/name1的键值对
# ./etcdctl get /cron/name1
/cron/name1
value1

获取连续一段key，（注：左闭右开，即：结果包含左边，但不包含右边）
以下结果中包含/cron/name1,但不包含/cron/name4
# ./etcdctl get /cron/name1 /cron/name4
/cron/name1
value1
/cron/name2
value2
/cron/name3
value3

只返回key，不返回value
# ./etcdctl get /cron/name1 /cron/name3 --keys-only
/cron/name1
/cron/name2

指定返回的最大个数
# ./etcdctl get /cron/name1 /cron/name5 --limit=2
/cron/name1
value1
/cron/name2
value2

排序：默认是升序（ascend）
# ./etcdctl get /cron/name1 /cron/name5 --order="descend"
/cron/name4
value4
/cron/name3
value3
/cron/name2
value2
/cron/name1
value1

根据key前缀批量获取
# ./etcdctl get /cron/ --prefix
/cron/name1
value1
/cron/name2
value2
/cron/name3
value3
/cron/name4
value4
/cron/name5
value5

```



#### del 删除键值对

删除指定key或一段key

用法：

```
del [options] <key> [range_end] 注：删除一段连续key时遵循左闭右开原则
	options
	--from-key[=false] 使用字节比较获取大于或等于给定键的键，默认值：false
	--prefix[=false] 根据key的前缀批量操作，默认值：false
	--prev-kv[=false] 返回已删除的键值对,默认值：false
```

示例：

```shell
删除指定key
# ./etcdctl del /cron/name1
1

删除一段连续的key
# ./etcdctl del /cron/name2 /cron/name4
2  注：只删除了/cron/name2和/cron/name3

删除后返回删除的键值对
# ./etcdctl del /cron/name4 --prev-kv
1
/cron/name4
value4

根据key前缀删除
# ./etcdctl del /cron/name --prefix
3  注：删除了/cron/name5、/cron/name6、/cron/name7


```



#### lease 租约

租约用于设置键值对的声明周期，租约到期后键值对自动删除

用法：

```
lease grant <ttl> 创建一个租约，ttl 单位：秒
lease revoke <leaseID> 吊销指定租约，吊销租约将删除所有它附带的 key
lease timetolive <leaseID> 获取指定租约详情
lease list  列出所有活动的租约
lease keep-alive 租约到期自动续租
```

实例：

```
创建一个租约
# ./etcdctl lease grant 100
lease 694d6a6cd6c6e597 granted with TTL(100s)

吊销一个租约
# ./etcdctl lease revoke 694d6a6cd6c6e597
lease 694d6a6cd6c6e597 revoked

获取租约详情
# ./etcdctl lease timetolive 694d6a6cd6c6e597
lease 694d6a6cd6c6e597 already expired

列出所有活动中的租约
# ./etcdctl lease list
found 3 leases
694d6a6cd6c6e59a
694d6a6cd6c6e59c
694d6a6cd6c6e59e

租约到期自动续租
# ./etcdctl lease keep-alive 694d6a6cd6c6e5a0
lease 694d6a6cd6c6e5a0 keepalived with TTL(10)
lease 694d6a6cd6c6e5a0 keepalived with TTL(10)
lease 694d6a6cd6c6e5a0 keepalived with TTL(10)
lease 694d6a6cd6c6e5a0 keepalived with TTL(10)
^C

```

#### watch 监听

用于监听指定key的所有变化

用法：

```
watch [options] [key or prefix] [range_end]
    option
        -i, --interactive[=false]	交互模式
        --prefix[=false]		监听一个前缀
        --prev-kv[=false]		获取事件发生前的键值对
        --rev=0			从指定版本开始监听
```
示例：

```shell
监听以/cron开通的key值变化
# ./etcdctl watch /cron --prefix
PUT
/cron/name1
value1
DELETE
/cron/name1



```



