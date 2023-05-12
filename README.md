# gravity-infra
This repo test gravity's infrastructure such as Distributed Key-Value Database, Streaming Systems.

## Objective
Gravity 是一種位於 
* Infrastructure 之上
* Application 之下
* Domain Team 之間

的 Datamesh Distributed Platform 產品，主要客戶需求是企業版的 PaaS，以非侵入、替換式的方式來協助客戶達到轉型與加速的需求，除了 Gravity 本身的系統設計，我們需要更清楚的知道、選擇或自行設計 Gravity 的 Infrastructure，來讓產品本身更具由完善性的保證。

## Part: Streaming System
### SPEC
Gravity 經由多年經驗選擇 CNCF NATS/JetStream 做為基礎，所有 API 都基於這套 Messaging System 建立與設計，所以我們有必要知道 NATS/JetStream 的各種維運機制，在 Gravity 中 Streaming System 肩負 Event Store 的角色，以下是主要的務實目標。
* Clustered
    * Horizontal Scaling: 讓 Streaming System 可以擁有超過一個 Nodes 的能力
    * Single Point Failuare: Streaming System 必需要能夠承受故障，這點需要用到 Raft Consensus 來支撐 State Machine Replication
    * Throughput: 由於我們是只是使用單純使用 NATS/JetStream，所以知道 Cluster 物理上的極限在哪裡即可。 

## Part: Key-Value Database
### SPEC
Key-Value Database 肩負以下兩種任務
1. Gravity Snapshot
Gravity 本身未來會需要提供 Snapshot 的功能，讓客戶端可以 Query DataProduct 本身的資訊。

2. Cache/CQRS Pattern
多數企業都會擁有自己的 Database Vendor，但是換掉整套的 Database Infrastrutrue 是非常大的工程，幾乎是不可能採取這種方案，由於依賴的應用程式過於龐大，承受的風險過大，所以我們必須要提供一層 Platform 來專門做 Caching 的機制或是 CQRS 等功能，特別注意與傳統的 Memacached, Redis 的 In-memory HashMap Caching 不同，更像是輕量級的 Key-Value Cluster，需要擁有 Persist 和 High-Throughput 等特性。


### Candidate List 
* TiKV
* CockroachDB
* Dgraph
* ScyllaDB
* Graviton

### TiKV

### CockroachDB

### Dgraph

### ScyllaDB

### Graviton

