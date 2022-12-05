# Introduction to PolarDB-X

PolarDB-X is a cloud-native distributed database system designed for ultra-high concurrency, massive storage, and complex query scenarios. It adopts a shared-nothing and storage computing separation architecture, supports horizontal expansion, distributed transactions, mixed load and other capabilities, and has the characteristics of enterprise level, cloud native, high availability, and high compatibility with MySQL system and ecology.

PolarDB-X was originally born to solve the database scalability bottleneck of Alibaba's Tmall "Double Eleven" core transaction system, and has since grown along with Alibaba Cloud. It is a mature and stable database system that has been verified by multiple core business scenarios.
The core features of PolarDB-X are as follows:

- Horizontal expansion

PolarDB-X is designed with Shared-nothing architecture, supports multiple Hash and Range data splitting algorithms, realizes transparent horizontal expansion of the system through implicit primary key splitting and dynamic scheduling of data fragmentation.


- Distributed transactions

PolarDB-X uses the MVCC + TSO scheme and 2PC protocol to implement distributed transactions. Transactions meet ACID characteristics, support RC/RR isolation level, and achieve high performance of transactions through optimizations such as one-phase commit, read-only transaction, and asynchronous commit.


- mixed load

PolarDB-X supports analytical queries through native MPP capabilities, and achieves strong isolation between OLTP and OLAP traffic through CPU quota constraints, memory pooling, and storage resource separation.


- Enterprise

PolarDB-X has designed many core capabilities for enterprise scenarios, such as SQL current limiting, SQL Advisor, TDE, separation of powers, Flashback Query, etc.


- Cloud native

PolarDB-X has many years of cloud-native practice on Alibaba Cloud, supports management of cluster resources through K8S Operator, supports deployment in various forms such as public cloud, hybrid cloud, and proprietary cloud, and supports localized operating systems and chips.


- High availability

Strong data consistency is achieved through the majority Paxos protocol, and multiple disaster recovery methods such as three centers in two places and five replicas in three places are supported. At the same time, system availability is improved through Table Group and Geo-locality.


- Compatible with MySQL system and ecology

The goal of PolarDB-X is to be fully compatible with MySQL. Currently, the compatible content includes MySQL protocol, most of MySQL syntax, Collation, transaction isolation level, Binlog, etc.