---
title: Nebula Guide
date: 2026-09-03 21:46:41
tags:
- note
categories:
- Digital Signal Processing
---

内容来源:

- **第一部分（基础：是什么/架构/语法）** 以[官方中文文档 v3.8.0 社区版](https://docs.nebula-graph.com.cn/3.8.0)为准。自 3.5.0 起官方文档只覆盖**社区版**功能，企业版能力（图算法、向量检索、更强 Studio/Dashboard 等）不在其中。
- **第二部分（进阶：执行/优化器/统计）** 以 [Github开源源码](https://github.com/vesoft-inc/nebula)（3.x 主线 master 快照，`git log -1 = cdef57e5f`）为**源码实证**，并标注哪些是推断。企业版 v5.4 与社区 3.8.0 的关系/差异。

三条主线：

| 主线 | 章节 | 内容 | 优先级 |
|---|---|---|---|
| 认知线 | Part 1–3 | NebulaGraph 是什么、核心概念、系统架构 | 必读 |
| 动手线 | Part 4–5 | 本机实操 + nGQL 语法 | 必读 |
| 课题线 ★ | Part 6–7 | 查询如何执行、优化器/统计现状、课题改动地图 | **核心**，反复精读 |

# Part 1 NebulaGraph 是什么

> NebulaGraph 是一款**开源的、分布式的、易扩展的原生图数据库**，能承载**数千亿点、数万亿边**的超大规模图数据，提供**毫秒级**查询。

普通数据库把"关系"拆进一行行表里，查“朋友的朋友”要多次 JOIN；NebulaGraph 把数据存成**点（Vertex）+ 边（Edge）**，让“关系”本身成为一等公民，查询时**沿着边走**（图遍历），天然适合社交、风控、推荐、知识图谱这类问题。

- 数据规模越大、图关系越复杂，它的相对优势越大。
- 内核由 **C++** 编写，开源协议 **Apache 2.0**（仓库 `vesoft-inc/nebula`）。

| 特性 | 含义 |
|---|---|
| 高性能 | C++ 原生内核，毫秒级查询，专为 SSD 设计 |
| 易扩展 | **shared-nothing 架构**，可**不停服**扩缩容 |
| 高可用 | 存储多副本 + Raft 一致性（见 Part 3） |
| 强 Schema 与灵活建模 | 点/边属性可自由增删改（有 schema 但可 ALTER） |
| 类 SQL 语言 nGQL | **部分兼容 openCypher**，学习成本相对低 |
| 生态丰富 | Console/Studio/Dashboard/Importer/Exchange/Operator/Bench 等官方工具 |
| 访问控制 | 严格 RBAC 角色权限，支持 LDAP 等外部认证 |

# Part 2 核心概念和数据模型

| 术语 | 一句话 | 类比 |
|---|---|---|
| 图空间 **Space** | 数据隔离/管理的单元 | $\approx$ 一个“数据库” |
| 点 **Vertex** | 一个实体 | 表里的一行 |
| **VID** | 点的唯一 ID（用户自定） | $\approx$ 主键 |
| 标签 **Tag** | 点的类型 + 属性模板 | $\approx$ “点表”的表结构 |
| 边 **Edge** | 两个点之间的关系，有向 | 关系表的一行 |
| 边类型 **Edge type** | 边的类型 + 属性模板 | $\approx$ “边表”的表结构 |
| 属性 **Property** | 键值对形式的信息 | 列/字段 |
| **Rank** | 同起终点同类型边的序号（int64，默认 0） | openCypher 没有 |
| 路径 **Path** | 点与边的序列 | 一条走出来的路 |

> NebulaGraph 使用有向属性图模型：点和边构成图，边有方向，点和边都可有属性。

六大要素：图空间 Space、点 Vertex、边 Edge、标签 Tag、边类型 Edge type、属性 Property。下面逐个展开。

## 图空间 Space

- 用于**隔离不同团队/项目的数据**：不同 Space 数据互不可见、物理隔离，可各自指定副本数、分片数、权限。
- 建 Space 时需定三个核心参数：
  - `partition_num`：**分片数**是指把整张图的数据切成多少个“分区”，是最小的分布/复制单位（默认 10；官方建议约为集群磁盘数的 20 倍，HDD 则约 2 倍）。
  - `replica_factor`：每分片副本数（默认 1，**必须为奇数**；生产建议 3，测试 1）。副本数=1 时无法做 balance 扩容。
  - `vid_type`：**必填**，`INT64` 或 `FIXED_STRING(N)`。
- **建后不可修改**：分区数、副本数、vid_type、comment，只能 DROP 重建 $\rightarrow$ **建 Space 前要想好规模**。

常用 NGQL语句示例如下：

```ngql
CREATE SPACE basketballplayer(partition_num=15, replica_factor=1, vid_type=fixed_string(30));
SHOW SPACES;                          -- 列出所有空间
USE basketballplayer;                 -- 切换当前工作空间（单条语句不能跨 Space）
SHOW CREATE SPACE basketballplayer;   -- 回看建空间语句
```

## 点 Vertex 与 VID

- **点 = 实体**（比如一个球员、一个账号）。一个点可以有 **0~多个 Tag**（3.x 起不再强制至少一个 Tag）。
- **VID = 点的唯一标识**，在同一 Space 内唯一，作用 $\approx$ 主键；底层把 VID 作为 KV 存储 key 的一部分，**按 VID 点查很快（无需额外索引）**，热数据还会被缓存在内存（见下方小注）。
- VID **只能用户自己指定**：`INT64` 或 `FIXED_STRING(N)`，在 `CREATE SPACE` 定死后**不可修改**，点插入后 VID 也不可改。
- VID 选择建议（性能从高到低）：
  1. 拿业务上唯一的属性直接当 VID（不依赖索引，按 VID 直接点查，最快）；
  2. 用唯一属性组合生成 VID（走属性索引）；
  3. snowflake 等算法生成 VID（也走属性索引）。
- 若用 hash 生成 int64 VID，注意冲突率文档口径：**10 亿个点时 hash 冲突概率约 1/10**（边的数量不影响冲突概率）。

> **底层存储与 LSM-tree**
>
> Nebula 每个分区的数据落在一个 RocksDB（**LSM-tree**，Log-Structured Merge Tree，日志结构合并树）KV 引擎里。LSM 面向**写密集**设计：写入先进内存 **MemTable**（有序表），刷成磁盘上多级有序 **SST** 文件，后台 **Compaction** 持续合并。读时按 key 查 MemTable → 各层 SST，靠 **Bloom Filter** 快速跳过不存在的 key，命中的块进入 **Block Cache**（内存缓存）。
> 所以"按 VID 快"并不是给 VID 单独建了索引，而是：① 点/边 key 以 VID 为核心（同一 VID 的点属性、出入边在 SST 里排在一起，可点查+范围扫）；② 热数据块留在内存缓存。
> 注意与**用户建的索引**区分：`CREATE TAG INDEX ON player(name)` 建的是**另一份独立的 KV 数据**（key=索引属性，指向 VID），同样存放在这套 LSM 引擎里。

## 标签 Tag

- Tag 是一组预定义属性的集合，作用类似关系型数据库"**点表**的表结构"。
- 例：给点贴 `player` 标签（带 `name`,`age`），一个点可同时贴多个 Tag（比如一个人既是球员又是教练）。

常用 NGQL语句示例如下：

```ngql
CREATE TAG player(name string, age int);
CREATE TAG team(name string);
```

## 边 Edge / 边类型 Edge type / Rank

- **边 = 两个点之间的关系**。Nebula **只有有向边**（`src -> dst`），不存在无向边。
- 每条边**有且仅有一个 Edge type**（类似"边表"的表结构）。
- 边的唯一标识 = **四元组 `Edge type + 起点 VID + rank + 终点 VID`**：
  - 四元组相同 ⇒ 同一条边，`INSERT EDGE` **覆盖式**（重复插入以最后一次为准）；
  - **rank 不同 ⇒ 是不同边** → 同一对起终点之间可以有多条同类型边（**平行边**），靠 rank 区分；rank 是 int64，默认 0，完全由用户指定（openCypher 无此概念）。
- **悬挂边（Dangling edge）**：3.8.0 允许先写边、后补点；用户需自行保证端点存在，官方不建议依赖悬挂边取点。
- 自环（起点=终点）在文档简介中**未作专门说明**，以官方最新文档为准。

常用 NGQL语句示例如下：

```ngql
CREATE EDGE follow(degree int);                 -- 关注：src 关注 dst
CREATE EDGE serve(start_year int, end_year int);-- 效力：球员 -> 球队

INSERT EDGE follow(degree) VALUES "player101" -> "player100":(95);
INSERT EDGE e1 () VALUES "10"->"11"@1:();        -- @1 = rank=1 的边
```

## 属性 Property

- 属性 = **键值对**。建 Tag/Edge type 时给每个属性定类型（`string`/`int`/`double`/`timestamp` 等）。
- 属性支持 `DEFAULT`、`NOT NULL`、`TTL`（见 5.x）。

## 路径 Path

| 类型 | 点可否重复 | 边可否重复 | 用到的语句 |
|---|---|---|---|
| **walk** | 可 | 可（可绕圈） | **`GO`** |
| **trail** | 可 | **不可** | **`MATCH`、`FIND PATH`、`GET SUBGRAPH`** |
| **path** | 不可 | 不可 |  |

在NGQL中，写 `GO` 允许绕圈（walk）；写 `MATCH`/`FIND PATH`/`GET SUBGRAPH` 检索的是 trail，不会重复走同一条边。理解这一点，才能预期多跳查询“会不会有环”。

## 索引

- **索引用于按属性定位点/边**（`LOOKUP` 依赖索引；`MATCH` 3.5.0 起可不建索引全表扫描，但慢且可能 OOM，建议带过滤或索引）。
- **索引会大幅降低写入性能** → 官方建议：**先灌数据、再建索引、最后 `REBUILD`**；不要在建索引后大规模写入。
- 存量数据必须 `REBUILD TAG/EDGE INDEX` 才查询得到；无唯一索引；复合索引不能跨 Tag/Edge type、遵循最左匹配。
- 具体语法与约束见 Part 5.7。

# Part 3 系统架构

NebulaGraph = **Graph 服务（计算）+ Meta 服务（元数据）+ Storage 服务（存储）**，每类服务独立二进制/进程，可部署在一台或多台机器。

| 服务 | 进程 | 职责 | 默认端口 |
|---|---|---|---|
| Graph | `nebula-graphd` | 计算：解析/校验/优化/执行查询 | **9669**（客户端） |
| Meta | `nebula-metad` | Schema、分片分布、权限、作业等元数据管理 | **9559** |
| Storage | `nebula-storaged` | 真正存点/边/属性，执行下推的过滤计算 | **9779** |

三服务还各有 Raft/Admin/HTTP 等内部端口，见[官方附录"产品端口全集"](https://docs.nebula-graph.com.cn/3.8.0/20.appendix/port-guide/)。

![alt text](nebula_architecture.svg)

## Graph 服务

graphd 处理一条 nGQL 的宏观四步（文档明示）：解析 -> 校验 -> 生成执行计划 -> 执行，对应模块：

| 阶段 | 模块 | 主要工作 / 要点 |
|---|---|---|
| 解析 | Parser | 词法(Flex) + 语法(Bison) → **AST**；语法错误在此拦截 |
| 校验 | Validator | 语义校验：查 Schema 确认 Tag/Edge 存在、校验变量/属性归属、类型推断、`*` 展开、管道前后一致 |
| 生成计划 | Planner | 为语句生成"默认可执行"的执行计划：计划节点 **PlanNode** 的 DAG（有向无环图） |
| 优化 | Optimizer | 受开关 `enable_optimizer` 控制：true -> 跑规则集做改写/下推 |
| 执行 | Executor | Scheduler 把 PlanNode 树转成执行算子并调度执行 |

- 执行计划是**一棵/一张 DAG**：叶子是 `Start`，逐级到根（如 `Project <- Filter <- GetNeighbors <- Start`）；每个计划节点与一个执行算子对应；算子的中间结果按"输出变量名"存进一张哈希表，供下游算子读取。
- **下推**：`GO` 等触发的取邻居算子（如 GetNeighbors）调用 Storage 接口时，Storage **在存储侧直接按条件过滤**边，只把结果传回 graphd，这样大幅减少网络传输。这要求底层 KV 引擎支持高效条件扫描。
- graphd 是**无状态计算层**，可多实例，前端负载均衡即可水平扩展。

## Meta 服务

- metad 集群本身是 **Raft 组**：1 Leader + 若干 Follower，**只有 Leader 对外服务**；Leader 故障自动重选，数据不丢（生产建议 3 个进程且不同机器）。
- 保存并管理：**用户账号与权限、分片(partition)位置与负载均衡、图空间元数据、Schema（强类型，带版本号以支持在线变更）、TTL 定义、作业 Job**（如 REBUILD INDEX / STATS / COMPACT）。
- **心跳与“两个心跳周期”**：graphd/storaged 通过心跳（默认 `heartbeat_interval_secs` = **10 秒**）从 Meta 刷新 schema/分片分布等元数据。因此 **数据定义语言（CREATE SPACE/TAG/EDGE/INDEX/ALTER）是异步生效的**，文档要求等约 **2 个心跳周期（≈20 秒）** 才能使用，这是新手最常见的坑。

> **Raft 是什么？**
>
> Raft 是一种**分布式一致性算法**，用来让一个集群里多台机器把同一份数据保持一致，并在部分机器故障时**不丢数据、继续可用**。Raft 的规则是：
>
> - **选主**：机器们投票选出一个 **Leader**，其余是 **Follower**；
> - **只有 Leader 能对外写**：写请求都交给 Leader，它把"这条记录"写进日志并同步给**超过半数的 Follower**，多数确认成功才告诉客户端"写好了"——所以哪怕挂 1 台，剩下的多数派仍保有最新数据；
> - **读也只走 Leader**：保证读到的总是最新一致的数据；
> - **故障自愈**：Leader 挂了，Follower 会发现并重新投票选出新 Leader，服务不中断。
>
> 因为要“超过半数”才算数，**副本数必须是奇数**：3 副本容忍挂 1 台，5 副本容忍挂 2 台；2 副本挂 1 台就失去多数，没有意义。在 Nebula 里，**Meta 服务的元数据**与 **Storage 里每个分区的数据**都各自构成 Raft 组来保证高可用——这也是建 Space 时 `replica_factor`（每分区副本数）被要求为奇数、生产建议 3 的原因。

## Storage 服务

Storage 面对的是一个远超单机容量的图空间，它需要解决三件事：**怎么切碎（分片）？怎么摆到多台机器（放置）？坏了怎么不丢、并发怎么不错（副本 + Raft）？**这三件事正好对应 Storage 的三层架构：

1. **Storage interface 层**：负责“翻译”，把 graphd 发来的图请求（`getNeighbors` 取邻居、`insert vertex/edge` 写点边、`getProps` 取属性）翻译成一组分片上的 KV 操作。这一层才是“真正的图存储”，底下两台都只是裸 KV。
2. **Consensus 层**：负责“一致性”，**Multi Group Raft**，每个分区各自构成一个独立的小 Raft 组，保证每个分区强一致与高可用。
3. **Store Engine 层**：负责“落实”，自研 **KVStore（基于 RocksDB）**，提供 get/put/scan，可插拔。

整个图空间（space）会被切成很多个分区（partition），每个分区还会有 `replica_factor` 个副本，每个分区分布在某个 Storage Service 节点上，落在那里的一块 RocksDB 实例里。一条数据去哪儿，要走两层映射，两层的规则完全不同：

1. 逻辑层：VID -> 分区。算出它在哪个分区。纯数学、确定、与机器无关、不可改。
2. 物理层：分区 -> 物理机。这个分区的 replica_factor 份副本，具体落在哪几台机器上。这层不由哈希决定，而是 Meta 管理，并且可以搬。

---

### 怎么切碎（分片）

Nebula 底层根本不存“图”。 Storage 的最底部只是一张大得多的、按 key 排好序的 KV（RocksDB）。图并不存在，它只是被“编码”进了这些 KV 的 key 里。具体做法是：**把一条数据的一切信息都塞进 key，而 key 的最高位就是分区号。** 点和边的 key 大致长这样：

```
vertex key :  partId | 点类型 | VID | tagId | ...
edge   key :  partId | VID | 边类型（带正负号） | rank | 另一顶点 VID | ...
```

其中分片算法如下：

$$
\text{partId} = (\text{hash}(\text{VID}) \bmod \text{partition\_num}) + 1
$$

注意取模与 `+1` 的**顺序**：先对 VID 取模，再加 1，这是因为：分区号是从 1 开始编号的（源码为 `vid % numParts + 1`，并断言 `pId > 0`）。对 `int64` 型 VID，"hash" 就是它本身、直接取模；``FIXED_STRING` 型才先做 `MurmurHash2`。

分区完全由 VID 决定，因此**同一顶点（同一 VID）的全部数据必然同属一个分区**：它各 Tag 的属性，以及以它为起点的出边、以它为终点的入边，key 前缀都是同一个 partId。于是“取某点所有邻居”只需访问这一个分区：在该分区 Leader 上做一次本地顺序扫描，无需跨分区汇聚。

---

**一条边为什么存两份（Edge cut）？**

一条有向边 $a \xrightarrow{\text{follow}} b$ 有两个端点，而 key 只能以一个“锚点 VID”来组织。为了让正向、反向遍历都只查本地，Nebula 把一条边**存两份**（Edge cut），两个端点的 VID 各当一次锚点、各算一次 partId。

- **出边副本** 以起点 $a$ 为锚，落在顶点 $a$ 所在的分区，key 形如 `a | +follow | rank | b`，服务正向遍历 `(a)-[:follow]->()`；
- **入边副本** 以终点 $b$ 为锚，落在顶点 $b$ 所在的分区，key 形如 `b | −follow | rank | a`，服务反向遍历 `()<-[:follow]-(b)`。

这也是边类型中**正负号的由来**：入边副本是为了支持反向遍历而人为构造的"反向边"，与原边方向相反。于是，围绕一个顶点的全部数据（自身 + 出边 + 入边）都在同一个分区，对它做任意方向的 1 跳遍历，都只是该分区 Leader 上的一次本地扫描。

---

### 怎么摆到多台机器（放置）。

- “分区 -> 物理机”的映射是随机的。一个分区的副本存放于哪些机器上，不是靠数学规则推导出来的，而是建 Space / 加机器时 Meta 随手分配的一个放置方案（只保证同一分区的副本不落在同一台机器）。
- Meta 里维护一张 “分区号 -> host 列表” 的映射表，是全集群唯一知道“每个分区住在哪”的地方。graphd / storaged 并不是靠自己猜，而是通过**心跳（默认 10s）**拉下来缓存用。所以加机器、`BALANCE` 搬分区之后，本质只是 Meta 改了这张表、客户端刷新而已。
- 负载均衡是手动的，不自动做（防止自动搬迁影响线上）。

## Part 4 Nebula 本地部署
