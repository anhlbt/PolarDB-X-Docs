# Online experiment experience

Our Yunqi Lab in the Alibaba Cloud developer community provides a cloud-based experimental platform for quickly getting started with PolarDB-X with zero threshold. Through the online experiment platform, you can preset the experiment environment with one click, and refer to the detailed experiment manual to quickly experience the installation, deployment and functional application of PolarDB-X, and intuitively experience the capabilities and services provided by the product.

At present, you can use the practice series of [Learn PolarDB-X with me] (https://developer.aliyun.com/adc/scenarioSeries/cf58beaf1d3d4aafb127dfb3f7bfd549?spm=a2c6h.27088027.devcloud-scenarioSeriesList.22.427679a9YE067x) in Yunqi Lab class, learn relevant knowledge and complete the following experiments.

## Beginner PolarDB-X

### Experiment 1: How to install and deploy PolarDB-X with one click

[**Experience Now**](https://developer.aliyun.com/adc/scenario/b299867c133b46688912399149997335?spm=a2c6h.13858378.0.0.2d6c207cWBxcDO)

| Suggested experiment duration | Cloud product resource usage duration |
| ------------ | ------------------ |
| 1 hour | 2 hours |

**Introduction to the experiment**:

Before experiencing PolarDB-X horizontal expansion, distributed transactions, global secondary index, global Binlog and other features, you need to deploy the PolarDB-X environment locally. This experiment will introduce how to deploy PolarDB-X through a Docker image based on an ECS instance (cloud server) configured with the CentOS 8.5 operating system.

By completing this lab, you will be able to:

- Install Docker

- Deploy PolarDB-X

- Experience the distributed features of PolarDB-X

---

### Experiment 2: How to use PolarDB-X

[**Experience Now**](https://developer.aliyun.com/adc/scenario/6e7827274b004c7b9fad58ecf5404c6c)

| Suggested experiment duration | Cloud product resource usage duration |
| ------------ | ------------------ |
| 1 hour | 1 hour |

**Introduction to the experiment**:

This experiment will be based on an ECS instance (cloud server) configured with the CentOS 8.5 operating system. Taking Spring and WordPress official tutorials as examples, it will introduce the application development process and methods of Spring Boot+PolarDB-X and WordPress+PolarDB-X.

By completing this lab, you will be able to:

- Application development based on PolarDB-X

---
## PolarDB-X Advanced
### Experiment 1: How to communicate PolarDB-X with big data and other systems

[**Experience Now**](https://developer.aliyun.com/adc/scenario/a734d982339845f18baa71d3cd5a4387)

| Suggested experiment duration | Cloud product resource usage duration |
| ------------ | ------------------ |
| 1 hour | 1 hour |

**Introduction to the experiment**:

A new database must solve a problem: how to avoid becoming a data island. PolarDB-X provides an incremental data subscription capability that is fully compatible with MySQL Binlog, that is, global Binlog.

This experiment will be based on an ECS instance (cloud server) configured with the CentOS 8.5 operating system, and introduce the process and method of interworking PolarDB-X with ClickHouse through Canal to build a real-time analysis system.

By completing this lab, you will be able to:

- How to install PolarDB-X
- How to deploy Canal
- How to deploy ClickHouse
- Build a real-time analysis system based on PolarDB-X, Canal and ClickHouse

---

### Experiment 2: How to dynamically scale the PolarDB-X cluster

[**Experience Now**](https://developer.aliyun.com/adc/scenario/413a0f2ada9049a39e249414698385f1)

| Suggested experiment duration | Cloud product resource usage duration |
| ------------ | ------------------ |
| 1 hour | 1 hour |

**Introduction to the experiment**:

PolarDB-X is a cloud-native + distributed database system, and elastic scaling is one of the highlights of PolarDB-X.

This experiment will be based on an ECS instance (cloud server) configured with the CentOS 8.5 operating system, introduce how to use PolarDB-X Operator to install PolarDB-X, and take you to experience the expansion and contraction capabilities of PolarDB-X.

By completing this lab, you will be able to:

- Install PolarDB-X using PolarDB-X Operator
- Dynamic expansion and contraction of PolarDB-X cluster

---

### Experiment 3: How to perform Online DDL in PolarDB-X

[**Experience Now**](https://developer.aliyun.com/adc/scenario/32d8fd0efdb7414cba05f5f4cf88ac05)

| Suggested experiment duration | Cloud product resource usage duration |
| ------------ | ------------------ |
| 1 hour | 1 hour |

**Introduction to the experiment**:

Creating tables, adding columns, and adding indexes are the most common DDL operations in database systems. In distributed database systems, there are also unique DDL operations such as creating new global secondary indexes and changing splitting methods. Performing online DDL operations is one of the core functions of PolarDB-X.

This experiment will be based on an ECS instance (cloud server) configured with the CentOS 8.5 operating system and installed with a PolarDB-X cluster, and will take you to experience the Online DDL capability of PolarDB-X.

By completing this lab, you will be able to:

- How to use Sysbench OLTP scenario to simulate business load
- Experience the Online DDL capability of PolarDB-X

---

### Experiment 4: How to optimize slow SQL in PolarDB-X

[**Experience Now**](https://developer.aliyun.com/adc/scenario/e7cd646ddc4c43479cb51321d6607ff0)

| Suggested experiment duration | Cloud product resource usage duration |
| ------------ | ------------------ |
| 1 hour | 1 hour |

**Introduction to the experiment**:

Slow SQL is one of the most common problems in the development and operation and maintenance process. PolarDB-X provides native slow SQL analysis capabilities. This lab will demonstrate how to analyze and solve slow SQL in PolarDB-X around this scenario.

This experiment will be based on an ECS instance (cloud server) configured with the CentOS 8.5 operating system and installed with a PolarDB-X cluster, and will show you how to analyze and solve slow SQL in PolarDB-X.

By completing this lab, you will be able to:

- How to use Sysbench OLTP scenario to simulate business load
- Use SQL rate limiting
- Use SQL Advisor to optimize slow SQL

---

### Experiment 5: Use PolarDB-X+Flink to build a real-time data screen

[**Experience Now**](https://developer.aliyun.com/adc/scenario/884a36afe52d4850821b85f2c1e40691)

| Suggested experiment duration | Cloud product resource usage duration |
| ------------ | ------------------ |
| 1 hour | 1 hour |

**Introduction to the experiment**:

The big data screen plays an important role in business display and decision-making, and the real-time calculation behind it is one of the key technologies for the realization of the big data screen.

This experiment will be based on an ECS instance (cloud server) configured with the CentOS 8.5 operating system, and will show you how to use PolarDB-X and Flink to build a real-time data link, simulating the Alibaba Double Eleven GMV large screen.

By completing this lab, you will be able to:

- How to install PolarDB-X
- How to use Flink
- How to build a real-time data large screen based on PolarDB-X and Flink

------

### Experiment 6: Using PolarDB-X to build a highly available system

[**Experience Now**](https://developer.aliyun.com/adc/scenario/70d3ad96a23e4cfeabbd72fb9e729644)

| Suggested experiment duration | Cloud product resource usage duration |
| ------------ | ------------------ |
| 1 hour | 1 hour |

**Introduction to the experiment**:

Large-scale background systems often adopt a distributed architecture. One of the core issues to be considered in this architecture is how to design high availability of the system. PolarDB-X released version 2.1.0 on April 1. The X-Paxos multi-copy capability introduced in this version has made a qualitative improvement in the high availability of the system.

This experiment is based on an ECS instance (cloud server) configured with the CentOS 8.5 operating system and installed with a PolarDB-X cluster. It will show you how to use PolarDB-X to build a high-availability system, and simulate node failures by directly killing the container. Watch the automatic recovery of PolarDB-X.

By completing this lab, you will be able to:

- Use PolarDB-X+Sysbench OLTP to build a high availability system
- Experience the high availability of PolarDB-X

------

### Experiment 7: Experience PolarDB-X distributed transactions and data partitioning

[**Experience Now**](https://developer.aliyun.com/adc/scenario/49f35884e3b54bb8bf9a004d781ad446)

| Suggested experiment duration | Cloud product resource usage duration |
| ------------ | ------------------ |
| 1 hour | 1 hour |

**Introduction to the experiment**:

PolarDB-X is a distributed database based on the separation of storage and computing and the Shared-Nothing architecture. The separation of storage and computing means that the system contains computing nodes (CN) and data nodes (DN). Computing nodes are stateless nodes, which provide horizontal scalability of computing power by adding computing nodes. The data is divided into multiple partitions according to the partition key and stored on different data nodes. By migrating partitions to new data nodes, the horizontal expansion capability of storage capacity is provided.

This experiment is based on an ECS instance (cloud server) configured with the CentOS 8.5 operating system and installed with a PolarDB-X cluster. It will take you to experience PolarDB-X distributed transactions and data partitions, and demonstrate the related features of distributed transactions through examples, and Understand the implementation details of data partitioning through practical operation.

By completing this lab, you will be able to:

- Experience PolarDB-X distributed transactions
- Experience PolarDB-X data partitioning





