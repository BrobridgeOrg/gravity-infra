

# NATS/JetStream

<!-- vscode-markdown-toc -->
* 1. [Pre-require](#Pre-require)
* 2. [Check Hardware Information](#CheckHardwareInformation)
* 3. [Stream Benchmark](#StreamBenchmark)
	* 3.1. [Replicated Stream Benchmark](#ReplicatedStreamBenchmark)
		* 3.1.1. [TestReplicatedStream Benchmark Result](#TestReplicatedStreamBenchmarkResult)
		* 3.1.2. [TestReplicatedStream Benchmark when scaling Consumer](#TestReplicatedStreamBenchmarkwhenscalingConsumer)
	* 3.2. [Single Stream Benchmark](#SingleStreamBenchmark)
		* 3.2.1. [Single Stream Benchmark Benchmark Result](#SingleStreamBenchmarkBenchmarkResult)
* 4. [KV Benchmark](#KVBenchmark)
	* 4.1. [KV Watch](#KVWatch)
		* 4.1.1. [KV Watch 結果](#KVWatch-1)
	* 4.2. [KV Put](#KVPut)
		* 4.2.1. [Key-Value Put Benchmark 結果](#Key-ValuePutBenchmark)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

##  1. <a name='Pre-require'></a>Pre-require
* Install goreman: 以便啟動 Local Cluster
* Install nats-cli tool: Benchmark Tool
* Download NATS binary: 自行下載官方網站適合自己的 CPU 和 OS 版本

##  2. <a name='CheckHardwareInformation'></a>Check Hardware Information
```
system_profiler SPHardwareDataType
```

```
Hardware:

    Hardware Overview:

      Model Name: MacBook Pro
      Model Identifier: MacBookPro18,1
      Chip: Apple M1 Pro
      Total Number of Cores: 10 (8 performance and 2 efficiency)
      Memory: 16 GB
      System Firmware Version: 7459.141.1
      OS Loader Version: 7459.141.1
      Activation Lock Status: Enabled
```
##  3. <a name='StreamBenchmark'></a>Stream Benchmark

###  3.1. <a name='ReplicatedStreamBenchmark'></a>Replicated Stream Benchmark
以下是 Stream Benchmark 的指令，建立一個 Replicated Stream Benchmark 以下 flags 來模擬特定場景。
* ```--js``` : 啟動 JetStream Client
* ```--dedup``` : 保證 Exactly-Once Deliver
* ```replicas=3```: HA 副本，底層用 Raft 實作
* ```--storage=file```: 持久化儲存
* ```--pub=1```: 模擬 CDC Adapter Publish Product Events
* ```--sub=2```: 一個用於 Gravity Event-Store, 一個用於 Gravity Snapshot
* ```--purge```: 執行完成 Benchamrk 移除 Stream 資料
* ```--push```: 以 Push 的方式傳送到訂閱端

```
nats bench TestReplicatedStream \
--js \
--dedup \
--replicas=3 \
--storage=file \
--pub=1 \
--sub=2 \
--push \
--purge
```
####  3.1.1. <a name='TestReplicatedStreamBenchmarkResult'></a>TestReplicatedStream Benchmark Result
```
17:17:25 JetStream durable push consumer mode, subscriber(s) will explicitly acknowledge the consumption of messages
17:17:25 JetStream ephemeral ordered push consumer mode, subscribers will not acknowledge the consumption of messages
17:17:25 Starting JetStream benchmark [subject=TestReplicatedStream, multisubject=false, multisubjectmax=0, js=true, msgs=100,000, msgsize=128 B, pubs=1, subs=2, stream=benchstream, maxbytes=1.0 GiB, storage=file, syncpub=false, pubbatch=100, jstimeout=30s, pull=false, consumerbatch=100, push=true, consumername=natscli-bench, replicas=3, purge=true, pubsleep=0s, subsleep=0s, dedup=true, dedupwindow=2m0s]
17:17:25 Purging the stream
17:17:26 Defined durable explicitly acked push consumer: natscli-bench
17:17:26 Starting subscriber, expecting 50,000 messages
17:17:26 Starting subscriber, expecting 50,000 messages
17:17:26 Starting publisher, publishing 100,000 messages

NATS Pub/Sub stats: 87,742 msgs/sec ~ 10.71 MB/sec
 Pub stats: 44,342 msgs/sec ~ 5.41 MB/sec
 Sub stats: 43,871 msgs/sec ~ 5.36 MB/sec
  [1] 21,982 msgs/sec ~ 2.68 MB/sec (50000 msgs)
  [2] 21,943 msgs/sec ~ 2.68 MB/sec (50000 msgs)
  min 21,943 | avg 21,962 | max 21,982 | stddev 19 msgs

17:17:28 Deleted durable consumer: natscli-bench
```

清除 Replicated Stream Benchmark 結果，以便測試其他 Stream Benchmark
```
nats stream delete benchstream
```

####  3.1.2. <a name='TestReplicatedStreamBenchmarkwhenscalingConsumer'></a>TestReplicatedStream Benchmark when scaling Consumer
| #Consumer | 1      | 2      | 3      | 4      | 5      | 6      | 7      | 8      |
|-----------|--------|--------|--------|--------|--------|--------|--------|--------|
| TPS (Sub) | 43,071 | 31,997 | 37,569 | 31,295 | 29,435 | 27,326 | 24,388 | 26,511 |


| #Consumer | 16     | 32     | 64     | 128    | 256    | 512    | 1024   | 2048   |
|-----------|--------|--------|--------|--------|--------|--------|--------|--------|
| TPS (Sub) | 23,131 | 32,535 | 35,041 | N/A    | N/A    | N/A    | N/A    | N/A    |


###  3.2. <a name='SingleStreamBenchmark'></a>Single Stream Benchmark
以下是 Stream Benchmark 的指令，建立一個 Single Stream Benchmark 以下 flags 來模擬特定場景。
* ```--js``` : 啟動 JetStream Client
* ```--dedup``` : 保證 Exactly-Once Deliver
* ```replicas=1```: 只選擇使用一個副本，可以增加 Thoughput 但是會有單點故障的風險
* ```--storage=file```: 持久化儲存
* ```--pub=1```: 模擬 CDC Adapter Publish Product Events
* ```--sub=2```: 一個用於 Gravity Event-Store, 一個用於 Gravity Snapshot
* ```--purge```: 執行完成 Benchamrk 移除 Stream 資料
* ```--push```: 以 Push 的方式傳送到訂閱端
```
nats bench TestReplicatedStream \
--js \
--dedup \
--replicas=1 \
--storage=file \
--pub=1 \
--sub=2 \
--push \
--purge
```
####  3.2.1. <a name='SingleStreamBenchmarkBenchmarkResult'></a>Single Stream Benchmark Benchmark Result
```
17:21:08 JetStream durable push consumer mode, subscriber(s) will explicitly acknowledge the consumption of messages
17:21:08 JetStream ephemeral ordered push consumer mode, subscribers will not acknowledge the consumption of messages
17:21:08 Starting JetStream benchmark [subject=TestReplicatedStream.*, multisubject=true, multisubjectmax=0, js=true, msgs=100,000, msgsize=128 B, pubs=1, subs=2, stream=benchstream, maxbytes=1.0 GiB, storage=file, syncpub=false, pubbatch=100, jstimeout=30s, pull=false, consumerbatch=100, push=true, consumername=natscli-bench, replicas=1, purge=true, pubsleep=0s, subsleep=0s, dedup=true, dedupwindow=2m0s]
17:21:08 Purging the stream
17:21:08 Defined durable explicitly acked push consumer: natscli-bench
17:21:08 Starting subscriber, expecting 50,000 messages
17:21:08 Starting subscriber, expecting 50,000 messages
17:21:08 Starting publisher, publishing 100,000 messages

NATS Pub/Sub stats: 115,989 msgs/sec ~ 14.16 MB/sec
 Pub stats: 58,034 msgs/sec ~ 7.08 MB/sec
 Sub stats: 57,996 msgs/sec ~ 7.08 MB/sec
  [1] 29,004 msgs/sec ~ 3.54 MB/sec (50000 msgs)
  [2] 28,998 msgs/sec ~ 3.54 MB/sec (50000 msgs)
  min 28,998 | avg 29,001 | max 29,004 | stddev 3 msgs

17:21:10 Deleted durable consumer: natscli-bench
```


##  4. <a name='KVBenchmark'></a>KV Benchmark
新增測試用的 Key-Value Bucket
```
nats kv add BenchKeyValueBucket
```

建立成功結果
```
Information for Key-Value Store Bucket BenchKeyValueBucket created 2023-05-12T17:37:03+08:00

Configuration:

          Bucket Name: BenchKeyValueBucket
         History Kept: 1
        Values Stored: 0
   Backing Store Kind: JetStream
          Bucket Size: 0 B
  Maximum Bucket Size: unlimited
   Maximum Value Size: unlimited
     JetStream Stream: KV_BenchKeyValueBucket
              Storage: File

  Cluster Information:

                Name: gravity-stream
              Leader: jetstream-1
```

###  4.1. <a name='KVWatch'></a>KV Watch
利用 Watch 來觀測 Key-Value 的 State 變化
```
nats kv watch BenchKeyValueBucket
```

####  4.1.1. <a name='KVWatch-1'></a>KV Watch 結果
```
[2023-05-12 17:55:22] PUT BenchKeyValueBucket > 99990:
[2023-05-12 17:55:22] PUT BenchKeyValueBucket > 99991:
[2023-05-12 17:55:22] PUT BenchKeyValueBucket > 99992:
[2023-05-12 17:55:22] PUT BenchKeyValueBucket > 99993:
[2023-05-12 17:55:22] PUT BenchKeyValueBucket > 99994:
[2023-05-12 17:55:22] PUT BenchKeyValueBucket > 99995:
[2023-05-12 17:55:22] PUT BenchKeyValueBucket > 99996:
[2023-05-12 17:55:22] PUT BenchKeyValueBucket > 99997:
[2023-05-12 17:55:22] PUT BenchKeyValueBucket > 99998:
[2023-05-12 17:55:22] PUT BenchKeyValueBucket > 99999:
```
* 特別注意: watch 十分吃記憶體，開太多 KeyWatcher 有可能會 影響整體效能。

###  4.2. <a name='KVPut'></a>KV Put
以下是建立一個 KV Benchmark 以下 flags 來模擬特定場景。
* ```--kv``` : 啟動 JetStream KV
* ```replicas=3```: HA Key-Value 本身
* ```--bucket="BenchKeyValueBucket"```: 目標 Bucket 名稱，上面指令建立的
* ```--storage=file```: 持久化儲存
* ```--pub=1```: 模擬 CDC Adapter Materialize Product Event State

```
nats bench BenchKeyValueSubject \
--kv \
--pub 1 \
--bucket="BenchKeyValueBucket" \
--storage=file \
--replicas=3
```
####  4.2.1. <a name='Key-ValuePutBenchmark'></a>Key-Value Put Benchmark 結果
```
17:51:04 KV mode, using the subject name as the KV bucket name. Publishers do puts, subscribers do gets
17:51:04 Starting KV benchmark [bucket=BenchKeyValueBucket, kv=true, msgs=100,000, msgsize=128 B, maxbytes=1.0 GiB, pubs=1, sub=0, storage=file, replicas=3, pubsleep=0s, subsleep=0s]
17:51:04 Starting KV putter, putting 100,000 messages
Putting       7s 

Pub stats: 14,257 msgs/sec ~ 1.74 MB/sec
```





