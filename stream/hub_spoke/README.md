# Hub & Spoke
![](./../../asserts/hub-spoke.jpg)
Hub & Spoke 架構為 NATS/JetStream Leaf Node 的一種模式，研究之後發現十分適合當成 Data Mesh 的基礎設施。

首先各個 Domain Team 都是 Leaf Node 模式，連接到中央的 Hub。特別注意，雖然 Leaf node 並沒有限制只能單一節點，事實上 Leaf Node 本身可以是 cluster。

以下為實驗範例，假設有一個中央的 Hub cluster，任何 Domain Team 都可以擁有自己的 Leaf Node Cluster，如果 Data Product Scale 擴大，可以直接把 Leaf node cluster 1:1 配一個給 Data Product，由於 NATS/JetStream 的設計，Hub 可以容納極高數量級的 Spoke 所以完全不需要擔心。

這樣的拓樸架構就形成了 Federated，如果需要消費任合 Domain 的 Stream 可以通過 Hub 做 Source 或是 Mirror。

下面的實驗模擬有五個 Data Product，或是五個 Domain Team 都擁有各自的 Leaf cluster，如果要交換資料則是連接到 Hub cluster。
## Start Hub & Spoke

### Start Hub cluster
```
cd hub-cluster
goreman start
```

### Start leaf domain-a cluster
```
cd leaf-cluster-a
goreman start
```

### Start leaf domain-b cluster
```
cd leaf-cluster-b
goreman start
```

### Start leaf domain-c cluster
```
cd leaf-cluster-c
goreman start
```

### Start leaf domain-d cluster
```
cd leaf-cluster-d
goreman start
```

### Start leaf domain-e cluster
```
cd leaf-cluster-e
goreman start
```


## Verify
```
nats  --server nats://admin:password@localhost:4222 server report jetstream
```

```
╭───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                       JetStream Summary                                                       │
├──────────────────┬───────────────┬───────────────┬─────────┬───────────┬──────────┬───────┬────────┬──────┬─────────┬─────────┤
│ Server           │ Cluster       │ Domain        │ Streams │ Consumers │ Messages │ Bytes │ Memory │ File │ API Req │ API Err │
├──────────────────┼───────────────┼───────────────┼─────────┼───────────┼──────────┼───────┼────────┼──────┼─────────┼─────────┤
│ hub-server-2     │ hub           │ hub           │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ hub-server-0     │ hub           │ hub           │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ hub-server-1*    │ hub           │ hub           │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-a-0  │ leaf-domain-a │ leaf-domain-a │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-a-1* │ leaf-domain-a │ leaf-domain-a │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-a-2  │ leaf-domain-a │ leaf-domain-a │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-b-2* │ leaf-domain-b │ leaf-domain-b │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-b-0  │ leaf-domain-b │ leaf-domain-b │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-b-1  │ leaf-domain-b │ leaf-domain-b │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-c-1  │ leaf-domain-c │ leaf-domain-c │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-c-0* │ leaf-domain-c │ leaf-domain-c │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-c-2  │ leaf-domain-c │ leaf-domain-c │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-d-0  │ leaf-domain-d │ leaf-domain-d │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-d-2* │ leaf-domain-d │ leaf-domain-d │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-d-1  │ leaf-domain-d │ leaf-domain-d │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-e-1* │ leaf-domain-e │ leaf-domain-e │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-e-2  │ leaf-domain-e │ leaf-domain-e │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
│ leaf-domain-e-0  │ leaf-domain-e │ leaf-domain-e │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
├──────────────────┼───────────────┼───────────────┼─────────┼───────────┼──────────┼───────┼────────┼──────┼─────────┼─────────┤
│                  │               │               │ 0       │ 0         │ 0        │ 0 B   │ 0 B    │ 0 B  │ 0       │ 0       │
╰──────────────────┴───────────────┴───────────────┴─────────┴───────────┴──────────┴───────┴────────┴──────┴─────────┴─────────╯

╭───────────────────────────────────────────────────────────────────────╮
│                      RAFT Meta Group Information                      │
├─────────────────┬──────────┬────────┬─────────┬────────┬────────┬─────┤
│ Name            │ ID       │ Leader │ Current │ Online │ Active │ Lag │
├─────────────────┼──────────┼────────┼─────────┼────────┼────────┼─────┤
│ leaf-domain-e-0 │ cBPwknw9 │        │ true    │ true   │ 0.17s  │ 0   │
│ leaf-domain-e-1 │ 7E2Wrrrd │ yes    │ true    │ true   │ 0.00s  │ 0   │
│ leaf-domain-e-2 │ 8Okt2M3s │        │ true    │ true   │ 0.17s  │ 0   │
╰─────────────────┴──────────┴────────┴─────────┴────────┴────────┴─────╯
```

```
nats  --server nats://admin:password@localhost:4222 server list --sort=name
```

```
╭───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                  Server Overview                                                                  │
├─────────────────┬───────────────┬──────┬─────────┬───────────────┬───────┬───────┬────────┬─────┬─────────┬───────┬───────┬──────┬──────────┬─────┤
│ Name            │ Cluster       │ Host │ Version │ JS            │ Conns │ Subs  │ Routes │ GWs │ Mem     │ CPU % │ Cores │ Slow │ Uptime   │ RTT │
├─────────────────┼───────────────┼──────┼─────────┼───────────────┼───────┼───────┼────────┼─────┼─────────┼───────┼───────┼──────┼──────────┼─────┤
│ hub-server-0    │ hub           │ 0    │ 2.9.15  │ hub           │ 1     │ 974   │ 2      │ 0   │ 24 MiB  │ 0     │ 10    │ 0    │ 1h28m3s  │ 1ms │
│ hub-server-1    │ hub           │ 0    │ 2.9.15  │ hub           │ 0     │ 430   │ 2      │ 0   │ 19 MiB  │ 0     │ 10    │ 0    │ 1h28m3s  │ 1ms │
│ hub-server-2    │ hub           │ 0    │ 2.9.15  │ hub           │ 0     │ 430   │ 2      │ 0   │ 19 MiB  │ 0     │ 10    │ 0    │ 1h28m3s  │ 1ms │
│ leaf-domain-a-0 │ leaf-domain-a │ 0    │ 2.9.15  │ leaf-domain-a │ 0     │ 457   │ 2      │ 0   │ 18 MiB  │ 0     │ 10    │ 0    │ 1h18m28s │ 1ms │
│ leaf-domain-a-1 │ leaf-domain-a │ 0    │ 2.9.15  │ leaf-domain-a │ 0     │ 456   │ 2      │ 0   │ 19 MiB  │ 0     │ 10    │ 0    │ 1h18m28s │ 1ms │
│ leaf-domain-a-2 │ leaf-domain-a │ 0    │ 2.9.15  │ leaf-domain-a │ 0     │ 456   │ 2      │ 0   │ 19 MiB  │ 0     │ 10    │ 0    │ 1h18m28s │ 1ms │
│ leaf-domain-b-0 │ leaf-domain-b │ 0    │ 2.9.15  │ leaf-domain-b │ 0     │ 457   │ 2      │ 0   │ 19 MiB  │ 0     │ 10    │ 0    │ 1h18m8s  │ 1ms │
│ leaf-domain-b-1 │ leaf-domain-b │ 0    │ 2.9.15  │ leaf-domain-b │ 0     │ 457   │ 2      │ 0   │ 19 MiB  │ 0     │ 10    │ 0    │ 1h18m8s  │ 1ms │
│ leaf-domain-b-2 │ leaf-domain-b │ 0    │ 2.9.15  │ leaf-domain-b │ 0     │ 457   │ 2      │ 0   │ 19 MiB  │ 0     │ 10    │ 0    │ 1h18m8s  │ 1ms │
│ leaf-domain-c-0 │ leaf-domain-c │ 0    │ 2.9.15  │ leaf-domain-c │ 0     │ 457   │ 2      │ 0   │ 18 MiB  │ 0     │ 10    │ 0    │ 3m7s     │ 1ms │
│ leaf-domain-c-1 │ leaf-domain-c │ 0    │ 2.9.15  │ leaf-domain-c │ 0     │ 457   │ 2      │ 0   │ 18 MiB  │ 0     │ 10    │ 0    │ 3m7s     │ 1ms │
│ leaf-domain-c-2 │ leaf-domain-c │ 0    │ 2.9.15  │ leaf-domain-c │ 0     │ 457   │ 2      │ 0   │ 18 MiB  │ 0     │ 10    │ 0    │ 3m7s     │ 2ms │
│ leaf-domain-d-0 │ leaf-domain-d │ 0    │ 2.9.15  │ leaf-domain-d │ 0     │ 457   │ 2      │ 0   │ 17 MiB  │ 0     │ 10    │ 0    │ 22.10s   │ 1ms │
│ leaf-domain-d-1 │ leaf-domain-d │ 0    │ 2.9.15  │ leaf-domain-d │ 0     │ 457   │ 2      │ 0   │ 17 MiB  │ 0     │ 10    │ 0    │ 22.10s   │ 2ms │
│ leaf-domain-d-2 │ leaf-domain-d │ 0    │ 2.9.15  │ leaf-domain-d │ 0     │ 457   │ 2      │ 0   │ 18 MiB  │ 0     │ 10    │ 0    │ 22.10s   │ 1ms │
│ leaf-domain-e-0 │ leaf-domain-e │ 0    │ 2.9.15  │ leaf-domain-e │ 0     │ 457   │ 2      │ 0   │ 17 MiB  │ 0     │ 10    │ 0    │ 11.98s   │ 2ms │
│ leaf-domain-e-1 │ leaf-domain-e │ 0    │ 2.9.15  │ leaf-domain-e │ 0     │ 457   │ 2      │ 0   │ 17 MiB  │ 0     │ 10    │ 0    │ 11.98s   │ 1ms │
│ leaf-domain-e-2 │ leaf-domain-e │ 0    │ 2.9.15  │ leaf-domain-e │ 0     │ 457   │ 2      │ 0   │ 16 MiB  │ 0     │ 10    │ 0    │ 11.98s   │ 1ms │
├─────────────────┼───────────────┼──────┼─────────┼───────────────┼───────┼───────┼────────┼─────┼─────────┼───────┼───────┼──────┼──────────┼─────┤
│                 │ 6             │ 18   │         │ 18            │ 1     │ 8,687 │        │     │ 331 MiB │       │       │ 0    │          │     │
╰─────────────────┴───────────────┴──────┴─────────┴───────────────┴───────┴───────┴────────┴─────┴─────────┴───────┴───────┴──────┴──────────┴─────╯

╭──────────────────────────────────────────────────────────────────────────────────╮
│                                 Cluster Overview                                 │
├───────────────┬────────────┬───────────────────┬───────────────────┬─────────────┤
│ Cluster       │ Node Count │ Outgoing Gateways │ Incoming Gateways │ Connections │
├───────────────┼────────────┼───────────────────┼───────────────────┼─────────────┤
│ leaf-domain-b │ 3          │ 0                 │ 0                 │ 0           │
│ leaf-domain-d │ 3          │ 0                 │ 0                 │ 0           │
│ leaf-domain-e │ 3          │ 0                 │ 0                 │ 0           │
│ leaf-domain-c │ 3          │ 0                 │ 0                 │ 0           │
│ leaf-domain-a │ 3          │ 0                 │ 0                 │ 0           │
│ hub           │ 3          │ 0                 │ 0                 │ 1           │
├───────────────┼────────────┼───────────────────┼───────────────────┼─────────────┤
│               │ 18         │ 0                 │ 0                 │ 1           │
╰───────────────┴────────────┴───────────────────┴───────────────────┴─────────────╯
```


