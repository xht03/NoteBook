---
title: Nebula Guide
date: 2026-09-03 21:46:41
tags:
- note
categories:
- Digital Signal Processing
---

本篇内容来源：

- **第一部分（基础：是什么/架构/语法）** 以[官方中文文档 v3.8.0 社区版](https://docs.nebula-graph.com.cn/3.8.0)为准。自 3.5.0 起官方文档只覆盖**社区版**功能，企业版能力（图算法、向量检索、更强 Studio/Dashboard 等）不在其中。
- **第二部分（进阶：执行/优化器/统计）** 以 [Github开源源码](https://github.com/vesoft-inc/nebula)（3.x 主线 master 快照，`git log -1 = cdef57e5f`）为**源码实证**，并标注哪些是推断。企业版 v5.4 与社区 3.8.0 的关系/差异。

三条主线：

| 主线 | 章节 | 内容 | 优先级 |
|---|---|---|---|
| 认知线 | 1–3 | NebulaGraph 是什么、核心概念、系统架构 | 必读 |
| 动手线 | 4–5 | 本机实操 + nGQL 语法 | 必读 |
| 课题线 ★ | 6–7 | 查询如何执行、优化器/统计现状、课题改动地图 | **核心**，反复精读 |

## 1 NebulaGraph 是什么

> NebulaGraph 是一款**开源的、分布式的、易扩展的原生图数据库**，能承载**数千亿点、数万亿边**的超大规模图数据，提供**毫秒级**查询。

普通数据库把“关系”拆进一行行表里，查“朋友的朋友”要多次 JOIN；NebulaGraph 把数据存成**点（Vertex）+ 边（Edge）**，让“关系”本身成为一等公民，查询时**沿着边走**（图遍历），天然适合社交、风控、推荐、知识图谱这类问题。

- 数据规模越大、图关系越复杂，它的相对优势越大。

- 内核由 **C++** 编写，开源协议 **Apache 2.0**（仓库 `vesoft-inc/nebula`）。

| 特性 | 含义 |
|---|---|
| 高性能 | C++ 原生内核，毫秒级查询，专为 SSD 设计 |
| 易扩展 | **shared-nothing 架构**，可**不停服**扩缩容 |
| 高可用 | 存储多副本 + Raft 一致性（见第3节） |
| 强 Schema 与灵活建模 | 点/边属性可自由增删改（有 schema 但可 ALTER） |
| 类 SQL 语言 nGQL | **部分兼容 openCypher**，学习成本相对低 |
| 生态丰富 | Console/Studio/Dashboard/Importer/Exchange/Operator/Bench 等官方工具 |
| 访问控制 | 严格 RBAC 角色权限，支持 LDAP 等外部认证 |

## 2 核心概念和数据模型

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

---

### 2.1 图空间 Space

- 用于**隔离不同团队/项目的数据**：不同 Space 数据互不可见、物理隔离，可各自指定副本数、分片数、权限。

- 建 Space 时需定三个核心参数：
  - `partition_num`：**分片数**是指把整张图的数据切成多少个“分区”，是最小的分布/复制单位（默认 10；官方建议约为集群磁盘数的 20 倍，HDD 则约 2 倍）。
  - `replica_factor`：每分片副本数（默认 1，**必须为奇数**；生产建议 3，测试 1）。副本数=1 时无法做 balance 扩容。
  - `vid_type`：**必填**，`INT64` 或 `FIXED_STRING(N)`。

- **建后不可修改**：分区数、副本数、vid_type、comment，只能 DROP 重建 $\rightarrow$ **建 Space 前要想好规模**。

常用 NGQL语句示例如下：

```ngql
CREATE SPACE basketballplayer(partition_num=15, replica_factor=1, vid_type=fixed_string(30));
SHOW SPACES;                          # 列出所有空间
USE basketballplayer;                 # 切换当前工作空间（单条语句不能跨 Space）
SHOW CREATE SPACE basketballplayer;   # 回看建空间语句
```

---

### 2.2 点 Vertex 与 VID

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
> 所以“按 VID 快”并不是给 VID 单独建了索引，而是：① 点/边 key 以 VID 为核心（同一 VID 的点属性、出入边在 SST 里排在一起，可点查+范围扫）；② 热数据块留在内存缓存。
> 注意与**用户建的索引**区分：`CREATE TAG INDEX ON player(name)` 建的是**另一份独立的 KV 数据**（key=索引属性，指向 VID），同样存放在这套 LSM 引擎里。

---

### 2.3 标签 Tag

- Tag 是一组预定义属性的集合，作用类似关系型数据库“**点表**的表结构”。

- 例：给点贴 `player` 标签（带 `name`,`age`），一个点可同时贴多个 Tag（比如一个人既是球员又是教练）。

常用 NGQL语句示例如下：

```ngql
CREATE TAG player(name string, age int);
CREATE TAG team(name string);
```

---

### 2.4 边 Edge / 边类型 Edge type / Rank

- **边 = 两个点之间的关系**。Nebula **只有有向边**（`src -> dst`），不存在无向边。

- 每条边**有且仅有一个 Edge type**（类似“边表”的表结构）。

- 边的唯一标识 = **四元组 `Edge type + 起点 VID + rank + 终点 VID`**：
  - 四元组相同 ⇒ 同一条边，`INSERT EDGE` **覆盖式**（重复插入以最后一次为准）；
  - **rank 不同 ⇒ 是不同边** → 同一对起终点之间可以有多条同类型边（**平行边**），靠 rank 区分；rank 是 int64，默认 0，完全由用户指定（openCypher 无此概念）。

- **悬挂边（Dangling edge）**：3.8.0 允许先写边、后补点；用户需自行保证端点存在，官方不建议依赖悬挂边取点。

- 自环（起点=终点）在文档简介中**未作专门说明**，以官方最新文档为准。

常用 NGQL语句示例如下：

```ngql
CREATE EDGE follow(degree int);                 # 关注：src 关注 dst
CREATE EDGE serve(start_year int, end_year int);# 效力：球员 -> 球队

INSERT EDGE follow(degree) VALUES "player101" -> "player100":(95);
INSERT EDGE e1 () VALUES "10"->"11"@1:();        # @1 = rank=1 的边
```

---

### 2.5 属性 Property

- 属性 = **键值对**。建 Tag/Edge type 时给每个属性定类型（`string`/`int`/`double`/`timestamp` 等）。

- 属性支持 `DEFAULT`、`NOT NULL`、`TTL`（见 5.x）。

### 2.6 路径 Path

| 类型 | 点可否重复 | 边可否重复 | 用到的语句 |
|---|---|---|---|
| **walk** | 可 | 可（可绕圈） | **`GO`** |
| **trail** | 可 | **不可** | **`MATCH`、`FIND PATH`、`GET SUBGRAPH`** |
| **path** | 不可 | 不可 |  |

在NGQL中，写 `GO` 允许绕圈（walk）；写 `MATCH`/`FIND PATH`/`GET SUBGRAPH` 检索的是 trail，不会重复走同一条边。理解这一点，才能预期多跳查询“会不会有环”。

---

### 2.7 索引

- **索引用于按属性定位点/边**（`LOOKUP` 依赖索引；`MATCH` 3.5.0 起可不建索引全表扫描，但慢且可能 OOM，建议带过滤或索引）。

- **索引会大幅降低写入性能** → 官方建议：**先灌数据、再建索引、最后 `REBUILD`**；不要在建索引后大规模写入。

- 存量数据必须 `REBUILD TAG/EDGE INDEX` 才查询得到；无唯一索引；复合索引不能跨 Tag/Edge type、遵循最左匹配。

- 具体语法与约束见 5.7。

---

## 3 系统架构

NebulaGraph = **Graph 服务（计算）+ Meta 服务（元数据）+ Storage 服务（存储）**，每类服务独立二进制/进程，可部署在一台或多台机器。

| 服务 | 进程 | 职责 | 默认端口 |
|---|---|---|---|
| Graph | `nebula-graphd` | 计算：解析/校验/优化/执行查询 | **9669**（客户端） |
| Meta | `nebula-metad` | Schema、分片分布、权限、作业等元数据管理 | **9559** |
| Storage | `nebula-storaged` | 真正存点/边/属性，执行下推的过滤计算 | **9779** |

三服务还各有 Raft/Admin/HTTP 等内部端口，见[官方附录“产品端口全集”](https://docs.nebula-graph.com.cn/3.8.0/20.appendix/port-guide/)。

![](https://ref.xht03.online/202609072308632.svg)

### 3.1 Graph 服务

graphd 处理一条 nGQL 的宏观四步（文档明示）：解析 -> 校验 -> 生成执行计划 -> 执行，对应模块：

| 阶段 | 模块 | 主要工作 / 要点 |
|---|---|---|
| 解析 | Parser | 词法（Flex） + 语法（Bison） → **AST**；语法错误在此拦截 |
| 校验 | Validator | 语义校验：查 Schema 确认 Tag/Edge 存在、校验变量/属性归属、类型推断、`*` 展开、管道前后一致 |
| 生成计划 | Planner | 为语句生成“默认可执行”的执行计划：计划节点 **PlanNode** 的 DAG（有向无环图） |
| 优化 | Optimizer | 受开关 `enable_optimizer` 控制：true -> 跑规则集做改写/下推 |
| 执行 | Executor | Scheduler 把 PlanNode 树转成执行算子并调度执行 |

- 执行计划是**一棵/一张 DAG**：叶子是 `Start`，逐级到根（如 `Project <- Filter <- GetNeighbors <- Start`）；每个计划节点与一个执行算子对应；算子的中间结果按“输出变量名”存进一张哈希表，供下游算子读取。

- **下推**：`GO` 等触发的取邻居算子（如 GetNeighbors）调用 Storage 接口时，Storage **在存储侧直接按条件过滤**边，只把结果传回 graphd，这样大幅减少网络传输。这要求底层 KV 引擎支持高效条件扫描。

- graphd 是**无状态计算层**，可多实例，前端负载均衡即可水平扩展。

---

### 3.2 Meta 服务

- metad 集群本身是 **Raft 组**：1 Leader + 若干 Follower，**只有 Leader 对外服务**；Leader 故障自动重选，数据不丢（生产建议 3 个进程且不同机器）。

- 保存并管理：**用户账号与权限、分片（partition）位置与负载均衡、图空间元数据、Schema（强类型，带版本号以支持在线变更）、TTL 定义、作业 Job**（如 REBUILD INDEX / STATS / COMPACT）。

- **心跳与“两个心跳周期”**：graphd/storaged 通过心跳（默认 `heartbeat_interval_secs` = **10 秒**）从 Meta 刷新 schema/分片分布等元数据。因此 **数据定义语言（CREATE SPACE/TAG/EDGE/INDEX/ALTER）是异步生效的**，文档要求等约 **2 个心跳周期（≈20 秒）** 才能使用，这是新手最常见的坑。

> **Raft 是什么？**
>
> Raft 是一种**分布式一致性算法**，用来让一个集群里多台机器把同一份数据保持一致，并在部分机器故障时**不丢数据、继续可用**。Raft 的规则是：
>
> - **选主**：机器们投票选出一个 **Leader**，其余是 **Follower**；
> - **只有 Leader 能对外写**：写请求都交给 Leader，它把“这条记录”写进日志并同步给**超过半数的 Follower**，多数确认成功才告诉客户端“写好了”——所以哪怕挂 1 台，剩下的多数派仍保有最新数据；
> - **读也只走 Leader**：保证读到的总是最新一致的数据；
> - **故障自愈**：Leader 挂了，Follower 会发现并重新投票选出新 Leader，服务不中断。
>
> 因为要“超过半数”才算数，**副本数必须是奇数**：3 副本容忍挂 1 台，5 副本容忍挂 2 台；2 副本挂 1 台就失去多数，没有意义。在 Nebula 里，**Meta 服务的元数据**与 **Storage 里每个分区的数据**都各自构成 Raft 组来保证高可用——这也是建 Space 时 `replica_factor`（每分区副本数）被要求为奇数、生产建议 3 的原因。

---

### 3.3 Storage 服务

Storage 面对的是一个远超单机容量的图空间，它需要解决三件事：**怎么切碎（分片）？怎么摆到多台机器（放置）？坏了怎么不丢、并发怎么不错（副本 + Raft）？**这三件事正好对应 Storage 的三层架构：

1. **Storage interface 层**：负责“翻译”，把 graphd 发来的图请求（`getNeighbors` 取邻居、`insert vertex/edge` 写点边、`getProps` 取属性）翻译成一组分片上的 KV 操作。这一层才是“真正的图存储”，底下两台都只是裸 KV。

2. **Consensus 层**：负责“一致性”，**Multi Group Raft**，每个分区各自构成一个独立的小 Raft 组，保证每个分区强一致与高可用。

3. **Store Engine 层**：负责“落实”，自研 **KVStore（基于 RocksDB）**，提供 get/put/scan，可插拔。

整个图空间（space）会被切成很多个分区（partition），每个分区还会有 `replica_factor` 个副本，每个分区分布在某个 Storage Service 节点上，落在那里的一块 RocksDB 实例里。一条数据去哪儿，要走两层映射，两层的规则完全不同：

1. 逻辑层：VID -> 分区。算出它在哪个分区。纯数学、确定、与机器无关、不可改。

2. 物理层：分区 -> 物理机。这个分区的 replica_factor 份副本，具体落在哪几台机器上。这层不由哈希决定，而是 Meta 管理，并且可以搬。

---

#### 怎么切碎（分片）

Nebula 底层根本不存“图”。 Storage 的最底部只是一张大得多的、按 key 排好序的 KV（RocksDB）。图并不存在，它只是被“编码”进了这些 KV 的 key 里。具体做法是：**把一条数据的一切信息都塞进 key，而 key 的最高位就是分区号。** 点和边的 key 大致长这样：

```
vertex key :  partId | 点类型 | VID | tagId | ...
edge   key :  partId | VID | 边类型（带正负号） | rank | 另一顶点 VID | ...
```

其中分片算法如下：

$$
partId = \left(hash(VID) \bmod partition\_num\right) + 1
$$

注意取模与 `+1` 的**顺序**：先对 VID 取模，再加 1，这是因为：分区号是从 1 开始编号的（源码为 `vid % numParts + 1`，并断言 `pId > 0`）。对 `int64` 型 VID，“hash” 就是它本身、直接取模；``FIXED_STRING` 型才先做 `MurmurHash2`。

分区完全由 VID 决定，因此**同一顶点（同一 VID）的全部数据必然同属一个分区**：它各 Tag 的属性，以及以它为起点的出边、以它为终点的入边，key 前缀都是同一个 partId。于是“取某点所有邻居”只需访问这一个分区：在该分区 Leader 上做一次本地顺序扫描，无需跨分区汇聚。

---

**一条边为什么存两份（Edge cut）？**

一条有向边 $a \xrightarrow{\text{follow}} b$ 有两个端点，而 key 只能以一个“锚点 VID”来组织。为了让正向、反向遍历都只查本地，Nebula 把一条边**存两份**（Edge cut），两个端点的 VID 各当一次锚点、各算一次 partId。

- **出边副本** 以起点 $a$ 为锚，落在顶点 $a$ 所在的分区，key 形如 `a | +follow | rank | b`，服务正向遍历 `(a)-[:follow]->()`；
- **入边副本** 以终点 $b$ 为锚，落在顶点 $b$ 所在的分区，key 形如 `b | −follow | rank | a`，服务反向遍历 `()<-[:follow]-(b)`。

这也是边类型中**正负号的由来**：入边副本是为了支持反向遍历而人为构造的“反向边”，与原边方向相反。于是，围绕一个顶点的全部数据（自身 + 出边 + 入边）都在同一个分区，对它做任意方向的 1 跳遍历，都只是该分区 Leader 上的一次本地扫描。

---

#### 怎么摆到多台机器（放置）。

- “分区 -> 物理机”的映射是随机的。一个分区的副本存放于哪些机器上，不是靠数学规则推导出来的，而是建 Space / 加机器时 Meta 随手分配的一个放置方案（只保证同一分区的副本不落在同一台机器）。

- Meta 里维护一张 “分区号 -> host 列表” 的映射表，是全集群唯一知道“每个分区住在哪”的地方。graphd / storaged 并不是靠自己猜，而是通过**心跳（默认 10s）**拉下来缓存用。所以加机器、`BALANCE` 搬分区之后，本质只是 Meta 改了这张表、客户端刷新而已。

- 负载均衡是手动的，不自动做（防止自动搬迁影响线上）。

---

## 4 Nebula 本地部署

### 4.1 源码编译

Nebula Graph 提供单机和分布式两种版本，因为它由**开关** `ENABLE_STANDALONE_VERSION` 决定：

| 形态 | 开关取值 | 编译产物 | 部署形态 |
|---|---|---|---|
| 单机版 standalone | `=ON` | 单个可执行 `nebula-standalone` | **一个进程**内同时跑 Graph + Meta + Storage |
| 分布式（集群） | `=OFF`（默认） | `nebula-graphd` / `nebula-metad` / `nebula-storaged` | 三组进程，可跨机器部署 |

首先安装相关依赖：

```bash
sudo apt install git cmake ninja-build g++ flex bison autoconf automake libtool wget
```

然后编译，以**单机版 + 关单测**为例（最常用组合），在仓库根目录执行：

```bash
INSTALL_DIR=~/nebula/build/install    # 安装目录，按需修改

cmake -B build -GNinja \
    -DCMAKE_BUILD_TYPE=RelWithDebInfo \
    -DENABLE_STANDALONE_VERSION=ON \
    -DENABLE_TESTING=OFF \
    -DENABLE_WERROR=OFF

cmake --build build -j8                     # 等价于 ninja -C build -j8
cmake --install build --prefix "$INSTALL_DIR"
```

各开关含义：

| 开关 | 例子 | 默认 | 说明 |
|---|---|---|---|
| `ENABLE_STANDALONE_VERSION` | ON | OFF | ON = 单机版；OFF（默认）= 编 graphd/metad/storaged 三进程 |
| `ENABLE_TESTING` | OFF | ON | 关掉单测编译，能省大量时间；要跑测试再开 ON |
| `ENABLE_WERROR` | OFF | ON | 默认把编译告警当错误；用新编译器常触发旧代码告警，开发期建议先关 |
| `CMAKE_BUILD_TYPE` | RelWithDebInfo | — | 带调试信息的发布版，兼顾性能与可调式；纯调试用 `Debug` |
| `-GNinja` | — | Unix Makefiles | 构建工具选 ninja，更快 |

- **首次全量编译会很久**：几十分钟到数小时都正常。中途 Ctrl-C 没关系，下次再跑会从断点增量继续。

- 成功后到安装目录检查产物：单机版应看到 `$INSTALL_DIR/bin/nebula-standalone`，配置为 `$INSTALL_DIR/etc/nebula-standalone.conf.default`（分布式版才会出现三个 daemon 和各自的 `.conf`）。

---

### 4.2 启停与连接

```bash
# 启动/状态/停止
build/install/scripts/nebula.service start all
build/install/scripts/nebula.service status all
build/install/scripts/nebula.service stop all

# 连接（独立客户端）
build/install/bin/nebula-console -addr 127.0.0.1 -port 9669 -u root -p nebula
```

```ngql
SHOW HOSTS;      # 集群状态
SHOW SPACES;     # 现有图空间
```

---

### 4.3 创建 Space

```ngql
CREATE SPACE IF NOT EXISTS demo(
    partition_num = 1, replica_factor = 1, vid_type = INT64
);
USE demo;
SHOW CREATE SPACE demo;
```

---

### 4.4 创建 Schema

```ngql
CREATE TAG IF NOT EXISTS person(name STRING, age INT);
CREATE EDGE IF NOT EXISTS knows(since INT);
SHOW TAGS; SHOW EDGES;
```

> 若立即插入报 `TagNotFound/EdgeNotFound`：DDL 异步生效，稍等重试即可。

---

### 4.5 插入数据

```ngql
INSERT VERTEX person(name, age) VALUES
    1:("Alice", 30), 2:("Bob", 20), 3:("Carol", 35), 4:("David", 28), 5:("Eve", 22);
INSERT EDGE knows(since) VALUES
    1->2:(2020), 1->3:(2021), 1->4:(2022), 2->4:(2023), 3->5:(2024);
```

对应的图如下：

```
Alice(1) ──→ Bob(2) ──→ David(4)
    │
    ├────→ Carol(3) ──→ Eve(5)
    └────→ David(4)
```

---

### 4.6 查询

```ngql
FETCH PROP ON person 1, 2, 3 YIELD id(vertex), properties(vertex);   # 按 VID 取属性
GO FROM 1 OVER knows YIELD dst(edge) AS friend_id;                    # 一跳
GO 2 STEPS FROM 1 OVER knows YIELD dst(edge) AS friend_id;            # 两跳
MATCH (a:person)-[:knows]->(b:person) WHERE id(a) == 1
RETURN a.person.name, b.person.name;                                  # 声明式匹配
```

---

### 4.7 查看执行计划

以同一条查询为例，用**三种方式**执行，对照输出差别：

```ngql
# 普通查询：真执行，只返回结果集
GO FROM 1 OVER knows YIELD dst(edge) AS friend_id;

# EXPLAIN：只生成执行计划、不真执行
EXPLAIN FORMAT="row" GO FROM 1 OVER knows YIELD dst(edge) AS friend_id;

# PROFILE：真执行，并额外回传每个算子的行数 / 耗时
PROFILE FORMAT="row" GO FROM 1 OVER knows YIELD dst(edge) AS friend_id;
```

---

## 5 nGQL 语法

### 5.1 语言风格

**nGQL = 原生 nGQL（命令式）+ openCypher 兼容语句（声明式）**。

| 风格 | 语句 | 定义输出 | 如何串联 |
|---|---|---|---|
| 原生（命令式） | `GO`、`FETCH PROP`、`LOOKUP`、`FIND PATH`、`GET SUBGRAPH`、`INSERT/UPDATE/DELETE` | `YIELD` | 管道 `\|`、分号 `;`、变量 `$var=` |
| openCypher（声明式） | `MATCH`、`OPTIONAL MATCH`、`WITH`、`UNWIND` | `RETURN` | 子句串联 |

注意：

1. 两种风格**不要混用**（`MATCH … | GO …` 不支持）。

2. 判等用 `==`，`=` 是赋值（openCypher 用 `=`，这是兼容性差异）。

3. 引用点属性**必须带 Tag**（`v.player.name` 而非 `v.name`）；边属性可直接 `e.degree`（边只有一个 Edge type）。

### 5.2 书写规范

- 关键字/函数**不区分**大小写（`SHOW SPACES`= `show spaces`）；**标识符**区分大小写（空间名/Tag/Edge type/属性名/变量）。

- 语句以 `;` 结束；多语句以分号分隔（返回最后一个结果）；续行在行尾加 `\`。

### 5.3 数据类型

| 类别 | 说明 |
|---|---|
| 整数 | `INT8/16/32/64`；读出统一 `INT64`；仅 `INT64` 可作 VID |
| 浮点 | `FLOAT`（单）/`DOUBLE`（双）；读出统一 `DOUBLE`；不支持 DECIMAL |
| 字符串 | `STRING` 变长 / `FIXED_STRING(N)` 定长；作 VID 超长报错，作属性超长截断 |
| 布尔 | `BOOL` |
| 日期时间 | `DATE/TIME/DATETIME/TIMESTAMP/DURATION`；按 `timezone_name` 转 UTC 存储 |
| NULL | 属性默认允许 NULL，可用 `NOT NULL` + `DEFAULT` |
| 复合类型 | `List []`、`Set {}`、`Map {}` **不能作为点/边属性存储**（仅表达式/中间结果可用） |
| 地理空间 | `GEOGRAPHY`（点/线/面），插入需经 `ST_GeogFromText` 等函数 |

### 5.4 DDL

#### 创建

整体流程：**CREATE SPACE** -> `USE` -> 建 Tag/Edge

```ngql
# 建图空间，分片数、副本数、vid_type 三者建后不可改
CREATE SPACE basketballplayer(partition_num=15, replica_factor=1, vid_type=fixed_string(30));

# 切换到该空间
USE basketballplayer;

# 建结构：Tag 定义“点类型”，EDGE 定义“边类型”
CREATE TAG player(name string, age int DEFAULT 20);
CREATE EDGE follow(degree int);
```

创建 Tag 和 Edge type 用到的是这条**语法模板**：

```ngql
# Edge type 语法相同，关键字换成 EDGE。
CREATE TAG [IF NOT EXISTS] <tag_name> (
    <prop> <type> [NULL|NOT NULL] [DEFAULT <v>] [, ...]
) [TTL_DURATION=<sec>] [TTL_COL=<prop>];
```

---

#### 查看

查看当前 Space 里有哪些 Tag / Edge type（只列名字）：

```ngql
SHOW TAGS; SHOW EDGES;
```

查看某个 Tag 的具体字段定义：

```ngql
DESCRIBE TAG player
```

把当初建它的那条 CREATE 语句原样回显：

```ngql
SHOW CREATE TAG player
```

---

#### 修改 / 删除

统一语法模板如下（`TAG` 换成 `EDGE` 即作用于边类型）：

```ngql
ALTER {TAG|EDGE} <name> ADD (<prop> <type> [, ...]);     # 加属性（可一次多个）
ALTER {TAG|EDGE} <name> DROP (<prop> [, ...]);           # 删属性（可一次多个）
ALTER {TAG|EDGE} <name> CHANGE (<prop> <新类型>);         # 改属性类型（一次一个，且只可放宽）
DROP {TAG|EDGE} [IF EXISTS] <name>;                      # 删整个类型（IF EXISTS = 不存在也静默通过）
DROP SPACE [IF EXISTS] <space_name>;                     # 删整个图空间（最重，慎用）
```

照抄可跑的例子（沿用上面的 `player` / `follow`）：

```ngql
ALTER TAG player ADD (height int, weight int);   # 加两个属性
ALTER TAG player DROP (height);                  # 删一个属性
ALTER TAG player ADD (level int32);              # 先建个低精度属性
ALTER TAG player CHANGE (level int64);           # int32 → int64：放宽 ✓
ALTER TAG player CHANGE (level int32);           # int64 → int32：收窄 ✗ 会报错
ALTER EDGE follow ADD (since int);               # Edge type 同样操作
SHOW CREATE TAG player;                          # 改完回看定义，确认已生效
```

> **“只可放宽”的边界**：数值类型只能往**更高精度 / 更大范围**改（如 `int32 → int64`、`float → double`），反向收窄会报错；字符串只能**加长**、不能缩短。原因：降精度有溢出/截断风险，底层要重排整列数据，官方直接禁止。

两个约束：

- 已建索引的 Tag/Edge 或其字段做 ALTER/DROP 会报 `Conflict!`——须**先 DROP 相关索引**，改完结构再重建索引并 `REBUILD`（见下方 CREATE INDEX）。

- `DROP TAG` 会把该 Tag 的属性数据一起删掉：**只挂这一个 Tag 的点会失去全部属性**（变“空点”，只剩 VID），但点本身和连到它的边仍在：删前先想清依赖。

---

#### 索引

NebulaGraph 里“点/边”本身是按 VID 组织存储的，所以“我知道 VID，取这个点的属性”不需要任何索引，直接按点查（`FETCH PROP` 就是干这个的）。但现实里更多查询是“我不知道 VID，只知道条件”，比如查找“名字是 'Tim' 的球员”。这需要从一堆点里按属性筛：那就必须有一份“属性 -> VID”的倒排 KV。这份 KV 不会自动存在，得靠额外 `CREATE INDEX` 建。所以：

- `LOOKUP` 是按条件查找点/边，依赖索引；

- `MATCH` 里 `where` 用属性当起点时也依赖索引，否则不知道该从哪些点开始走。

创建索引的语法模板如下：给哪个类型（Tag 或 Edge type）、索引名、建在哪个 Tag/Edge 的哪些属性上。

```ngql
CREATE {TAG|EDGE} INDEX [IF NOT EXISTS] <name> ON <tag|edge> (<prop>[(长度)], ...);
```

比如一个可运行的例子：

```ngql
CREATE TAG INDEX player_index ON player(name(20), age); 
REBUILD TAG INDEX player_index; 
SHOW TAG INDEX STATUS;
```

- `CREATE INDEX` 只登记索引，不扫存量。索引建成后，**之后新写入的数据**会自动顺带维护索引条目；但**建之前就已存在的数据**不会自己进索引。所以必须 `REBUILD` 触发一次全量回填，否则按属性查会漏掉老数据。

- `REBUILD` 是异步任务，立即返回一个 Job Id，不阻塞。

- `SHOW ... STATUS` 就是去查看所有索引的最近一次的 rebuild 任务的状态：QUEUE（排队）-> RUNNING（在跑）-> FINISHED（成功）/ FAILED（失败，可查日志重试）。

---

### 5.5 DML

#### INSERT VERTEX

语法模板和样例如下：

```ngql
INSERT VERTEX [IF NOT EXISTS] <tag>(<props>) VALUES <vid>:(<vals>), ...;
INSERT VERTEX player(name, age) VALUES "player100":("Tim", 42), "player101":("Tony", 36);
```

注意：

- `INSERT` 不带 `IF NOT EXISTS` 时，存储层根本不读旧值，直接做一次盲写（Put），最后写进去的那条把整行顶掉。所以没有主键冲突、不会报错，最后一次写入生效。因为是无读盲写，引擎不可能知道旧行还有哪些属性，所以它会用这次给的列重新编码整行，你没写到的列会被填回 schema 默认值，而不是保留旧值。

- `IF NOT EXISTS` 只判断 VID+Tag 是否存在，存在则跳过（会先读，影响性能）。

---

#### INSERT EDGE

```ngql
INSERT EDGE [IF NOT EXISTS] <edge_type>(<props>) VALUES <src>-><dst>[@rank]:(<vals>), ...;
INSERT EDGE follow(degree) VALUES "player100"->"player101":(95);
```

- 同 （type, src, dst, rank） -> 覆盖；rank 不同 -> 平行边。

- 3.x 允许悬挂边（先写边后补点）。

---

#### UPDATE / UPSERT / DELETE

| 语句 | 目标不存在时 | 目标存在时 | 是否读旧值 | 改的是“整行”还是“指定列” |
|---|---|---|---|---|
| `INSERT` | 写入 | **盲写覆盖整行**，漏写的列归默认 | ✗ 不读 | 整行（你给的列） |
| `UPDATE` | 什么都不做 | WHEN 满足才改，**只动 SET 的列** | ✓ 读 | 指定列（其余保留） |
| `UPSERT` | **无条件插入** | 同 UPDATE（WHEN 满足才改） | ✓ 读 | 指定列 |
| `DELETE` | 什么都不做 | 删掉整行 | ✓ |  |

语法模板和示例如下：

```ngql
UPDATE VERTEX ON player "player101" SET age = age + 2 WHEN name == "Tony Parker";
UPDATE EDGE ON serve "player100"->"team204"@0 SET ... WHEN ... YIELD ...;

UPSERT VERTEX ON player "player101" SET age = age + 1 ...;      # 不存在则插入

DELETE VERTEX <vid> [WITH EDGE];            # 3.x 默认不删边(WITH EDGE 才连带删)
DELETE EDGE follow <src>-><dst>[@rank];     # 不带 rank 只删 rank=0
```

- `UPSERT` 是一串“读-改-写”，要保证不错，引擎必须在“读到写”之间独占这行。也就是：先拿锁，再“读-改-写”；拿不到锁则读都不读，直接报冲突。于是高并发场景（比如大家都来给同一个点 +1）会变成：
  1. 大家争同一把锁 -> 大部分请求拿不到锁 -> 报冲突；
  2. 客户端重试 -> 重试风暴 -> 有效吞吐暴跌。

- 可用 `GO … YIELD dst(edge) AS id | DELETE VERTEX $-.id` 管道删符合条件的。

> **注：拿不到锁为何“立刻失败”，而不是“排队等待”？**
>
> 操作系统里的信号量/互斥量（“登记 + 等待队列”）划算，是因为临界区只有微秒级、等待者是可以随时睡下的本机线程。而 UPSERT 的临界区要从拿锁一直持有到 Raft 提交确认（毫秒级网络往返），等待者又是**远端客户端**——服务端若让工作线程睡在队列里等一个热点键，会拖垮整个节点。况且：
>
> - 排队**不提升吞吐**：同一行“读-改-写”一次仍只能做一个，队列只是把“立刻报错”换成“无限等待”，最终客户端照样超时，还白占服务端线程/连接；
>
> - “记录队列”本身会成为**第二个热点**；
>
> - 若阻塞等待 + 一个操作要拿多把锁（如相关索引锁），有死锁风险；try-lock“全拿或全不拿”从结构上排除死锁。
>
> 所以 Nebula 选快速失败：拿不到锁直接返回 `E_DATA_CONFLICT_ERROR`，把重试/退避/攒批的节奏**踢回客户端**。高并发下真正该做的不是把队列排得更优雅，而是**别让每个请求都做一次分片内读改写**（应用层先合并）。

---

### 5.6 DQL ★

#### FETCH PROP

已知 VID，取属性，不需要索引。语法示例如下：

```ngql
FETCH PROP ON player "player100" YIELD properties(vertex);      # 取 Tag player 的全部属性
FETCH PROP ON * "player100" YIELD vertex AS v;                  # 取所有 Tag,打包成整点对象 v
FETCH PROP ON serve "player100"->"team204" YIELD properties(edge).start_year;   # 取 serve 这条边的属性,只要 start_year 字段
```

---

#### GO

- 命令式图遍历：告诉它“从这几个点出发、沿着哪类边、走几步、怎么筛、吐出什么”，它一步一步物理地沿边走。

- **不需要索引**：因为起点是已知 VID，引擎从这些点出发按“邻接”找边。

- 路径语义是 walk：点和边可以重复经过，允许绕圈。

- 执行方式接近“每一跳展开前沿（frontier）”，逐跳推进——所以它快、可控，性能敏感路径首选（相比 `MATCH` 更贴近底层）。

语法模板如下：

```
GO [<M> TO] <N> STEP|STEPS FROM <vid_list>
OVER <edge_type>[, ...] [REVERSELY|BIDIRECT]   # 默认出边；REVERSELY=入边；BIDIRECT=双向
[WHERE ...] YIELD [DISTINCT] <cols> [AS alias];
```

各片段拆解：

| 片段 | 含义 |
|---|---|
| `FROM <vid_list>` | **起点**集合：逗号分隔的 VID，或管道上游的 `$-.id` |
| `<N> STEPS` | **精确 N 跳**；不写默认 1 跳（`GO ... OVER ...` 就是 1 跳） |
| `<M> TO <N> STEPS` | **M 到 N 跳（含边界）**：走 1 步能到的、走 2 步能到的都算 |
| `OVER <edge_type>[, ...]` | 沿哪些边类型走，可逗号多类，也可 `*` 任意边 |
| 方向：默认 / `REVERSELY` / `BIDIRECT` | 默认**出边**（沿 src→dst）；`REVERSELY` 只走**入边**；`BIDIRECT` 双向都走 |
| `WHERE` | 遍历中对**起点/边/终点**逐行过滤 |
| `YIELD [DISTINCT] <cols> [AS alias]` | 输出列；`DISTINCT` 去重；不写 YIELD 默认返回每条边的目标点 vid |

语法样例如下：

```ngql
GO 2 STEPS FROM "player102" OVER follow YIELD dst(edge);                    # 两跳
GO 1 TO 2 STEPS FROM "player100" OVER follow YIELD dst(edge) AS destination;
GO FROM "player100" OVER follow REVERSELY YIELD src(edge) AS who_follows_me;
GO FROM "player100" OVER follow, serve YIELD properties(edge).degree, properties(edge).start_year; -- 多边类型
```

---

#### MATCH

- 声明式模式匹配：只写“我要找长得什么样的子图”，至于先扫谁、怎么走，引擎说了算。

- 匹配到的每条路径是 trail，边不重复。

语法模板如下：

```
MATCH <pattern> [WHERE ...] RETURN <cols> [ORDER BY][LIMIT];
```

其中 pattern 主体 = 一串“节点 + 关系”的骨架：`(节点) -[关系]-> (节点) -[关系]-> ...`。各元素写法：

| 元素 | 写法 | 含义 |
|---|---|---|
| 节点 | `(v)` / `(v:player)` | 匿名点 / 给点起名 `v` 并限定 Tag=`player` |
| 节点属性过滤 | `(v:player{name:"Tim"})` | 直接嵌在花括号里的**等值条件**（等价于 WHERE 里 `v.player.name == "Tim"`） |
| 关系方向 | `-->` 出边 / `<--` 入边 / `--` 无向（任一端） | 边本身有向，`--` 表示“哪头都行” |
| 关系类型 | `-[e:follow]->` | 只走 follow 边，边变量叫 `e` |
| 变长 | `-[e:follow*1..3]->` | 长度 1~3 跳都算；`*` 不带范围 = 任意长度（1..∞）；`*..5` = 1..5 跳 |
| 整条路径赋值 | `p = (a)-[e*..5]-(b)` | 把匹配到的整条路径存进变量 `p`（如最短路径场景） |

```ngql
MATCH (v:player) RETURN v.player.name AS Name LIMIT 5;                      # 找所有 player 点
MATCH (v:player{name:"Tim Duncan"}) RETURN v;
MATCH (v:player{name:"Tim Duncan"})-->(v2:player) RETURN v2.player.name;    # 出边一跳
MATCH (v:player{name:"Tim Duncan"})-[e:follow*1..3]->(v2)                   # 变长 1~3 跳
      RETURN DISTINCT v2, count(v2);
MATCH p = allShortestPaths((a:player{name:"Tim"})-[e*..5]-(b:player{name:"Tony"}))
      RETURN p;                                                             # 所有最短路径
```

注意：

- 比较用 `==`，赋值用 `=`；

- 点属性必须注明 Tag（比如 `v.player.name`）；

- `RETURN v` 返回整点，`id(v)` 取 VID。

- `GO` 靠 `|` 把上一步结果喂下一步，`MATCH` 则通过 `WITH ... AS ...` 把中间结果传给后续 `MATCH/WHERE/RETURN`；

- `MATCH` 匹配的是“端到端完整的边”：它需要把边两头的点都解析出来才能构成模式。而悬挂边（终点还没建点）缺了一头 -> 无法出现在任何匹配结果里。

- 3.5.0 起 MATCH 可不建索引（全表扫描）执行，但应避免——大图上会慢/OOM；建议带过滤或索引。

---

#### OPTIONAL MATCH

只是多了个 `OPTIONAL` 修饰词，而这个词把整条子句的语义从 INNER JOIN 换成了 LEFT JOIN。区别就一点：匹配不上时，行保留，缺的字段给 NULL。

```ngql
MATCH (m)-[]->(n) WHERE id(m)=="player100"
OPTIONAL MATCH (n)-[]->(l)      # 找不到 l 也不丢 (m,n)
RETURN id(m), id(n), id(l);     # 找不到 l 返回 __NULL__
```

---

#### LOOKUP

- 按属性条件找点/边：LOOKUP = “我只有属性条件，不知道 VID，把符合条件的 VID/边捞出来。”

- **依赖索引。**


语法模板如下：

```ngql
LOOKUP ON {tag|edge_type} [WHERE expr [AND expr]] YIELD [DISTINCT] cols [AS alias];
```

语法样例如下：

```ngql
LOOKUP ON player WHERE player.name == "Tony Parker" YIELD id(vertex);
LOOKUP ON player WHERE player.age > 45 YIELD id(vertex);
LOOKUP ON player WHERE player.name STARTS WITH "B" AND player.age IN [22,30]
        YIELD properties(vertex).name, properties(vertex).age;
LOOKUP ON player YIELD id(vertex) | LIMIT 4;
```

注意：

- 常与 GO/FETCH 管道连用（先反查 VID 再遍历）。

- 因为它不是“全查回来再过滤”，而是“索引直接定位”。普通数据库的 WHERE 可以随便写，大不了全扫；而 LOOKUP 的 WHERE 是喂给索引扫描用的，所以凡是“索引没法直接回答的谓词”都不支持：
  1. `$-`/`$^`/`$$` 这些管道/遍历上下文的变量。LOOKUP 阶段根本不存在那个上下文。
  2. 字段间的比较，比如 `a.p1 > a.p2`。要比较同一条记录的两个属性，而索引只存了定位列，还需逐行取值才能判断。这个 LOOKUP 做不了。
  3. `XOR`、除 `STARTS WITH` 外的字符串运算（如正则、`ENDS WITH`、`CONTAINS`）：前缀匹配能靠索引的有序性范围扫，其余字符串运算没法只用索引完成。
  4. rank 过滤：rank 是边的定位坐标之一，查边时按 src/dst/rank 定位逻辑和“属性过滤”不是一回事。

---

#### FIND PATH / GET SUBGRAPH —— 路径与子图

`FIND PATH`：把 FROM 和 TO 之间的路径找出来，返回给路径变量 `p`。语法模板如下：

```ngql
FIND {SHORTEST | SINGLE SHORTEST | ALL | NOLOOP} PATH
[WITH PROP] FROM <起点们> TO <终点们>
OVER <edge_type|*> [UPTO <N> STEPS] YIELD path AS p;
```

| 关键字 | 语义 | 可能返回 |
|---|---|---|
| `SHORTEST` | 长度**都最短**的路径 | 可能**多条**（等长的不同走法全给） |
| `SINGLE SHORTEST` | 只要最短路径**一条** | 1 条（挑第一条） |
| `ALL` | UPTO 上限内**所有**路径 | 极多（可能指数级，慎用） |
| `NOLOOP` | 找**没有环**的路径（点不重复） | 有限多条 |

注意：

- `WITH PROP`：默认 path 只带结构（端点/边引用，读不到属性）；加上后 path 里的点边**携带属性**，才能 `properties(...)`。

- `OVER *`：可跨任意边类型；`UPTO <N> STEPS`：给搜索设**深度上限**，不设最坏可能全图跑。

- `WHERE`：只能过滤路径上的**边属性**，不能过滤点。

语法示例如下：

```ngql
# player102 到 team204 的最短路（允许横跨多种边）
FIND SHORTEST PATH FROM "player102" TO "team204" OVER * YIELD path AS p;

# player100 到 team204，10 跳内的所有路径，并带属性
FIND ALL PATH WITH PROP FROM "player100" TO "team204" OVER * UPTO 10 STEPS YIELD path AS p;
```

---

`GET SUBGRAPH`：以某点为中心，切一片“k 跳关系图”。语法模板如下：

```ngql
GET SUBGRAPH [WITH PROP] [<N> STEPS] FROM <起点们>
[{IN | OUT | BOTH} <边类型们>]
[WHERE ...] YIELD VERTICES AS nodes, EDGES AS rels;
```

其中：

- `IN` 表示只走入边（指向起始点的边），`OUT` 表示只走出边（从起始点指出去的边），`BOTH` 表示出边入边都走。

- `GET SUBGRAPH` 默认 BOTH、1 跳；

- `GET SUBGRAPH` 的 `WHERE` 更窄：仅 `AND`，且只能过滤 `$$`（这一跳的目的点）和边。

语法示例如下

```ngql
# player101 周围 2 跳的人和关系，全带属性
GET SUBGRAPH WITH PROP 2 STEPS FROM "player101" YIELD VERTICES AS nodes, EDGES AS rels;
```

- FIND PATH 类型：SHORTEST / SINGLE SHORTEST / ALL / NOLOOP；`WITH PROP` 显示点边属性；WHERE 只能过滤边属性。
- GET SUBGRAPH 默认 BOTH、1 跳；WHERE 仅支持 AND、只能过滤目的点 `$$.tag.prop` 与边。

---

#### SHOW

| 类 | 语句 | 问谁 | 回答的问题 |
|---|---|---|---|
| 集群 | `SHOW HOSTS` | metad（心跳维护的视图） | 集群里有几台机器、在线吗、分片/leader 分布均不均 |
| 空间 | `SHOW SPACES` / `SHOW CREATE SPACE xxx` | metad | 建了哪些“库”；某“库”当初怎么建的 |
| 库内 schema | `SHOW TAGS` / `SHOW EDGES` / `SHOW TAG INDEXES` / `SHOW TAG INDEX STATUS` / `SHOW CREATE TAG xxx` | metad | 这个 space 里定义了哪些点/边类型、建了哪些索引、索引建好没 |
| 统计 | `SUBMIT JOB STATS` / `SHOW STATS` | metad 派 job 到 storage 扫数，结果存回 metad | 图里各类点边各有多少——喂给优化器/代价模型 |

### 5.7 子句与复合查询

一条查询最后都会产出一张**表**。如何将查询结果写出来？Nebula 给了**两套**说法：

- **原生族**：一条条命令式“图查询语句” + 用管道把它们接起来；出口词叫 **YIELD**。

- **MATCH 族**（openCypher 兼容）：一条完整的声明式查询；出口词叫 **RETURN**。

`YIELD`、`RETURN` 没有本质区别，都是出口词，都是“投影子句”：决定输出表里有哪几列、每列怎么算。而管道 `|` 是“拼接工具”，**它们根本不是同一个东西**。

`GO` / `LOOKUP` / `FETCH` / `FIND PATH` / `GET SUBGRAPH` 都以 `YIELD` 收尾，给出本段产出的列（可用 `AS` 起别名）。比如这一条 GO 写到 YIELD 结束，就是一条完整查询：

```ngql
# 从 player100 出发走 follow，输出“它关注了”，列名叫 who
GO FROM "player100" OVER follow YIELD dst(edge) AS who;
```

而 `MATCH` 不描述“先走一步、再走一步”（命令式），而是描述“我要找的**样子**”（声明式），由引擎自己决定怎么找。比如相同的例子：

```ngql
# 同一个问题：“player100 关注了谁”
MATCH (v:player)-[:follow]->(w:player)
WHERE id(v) == "player100"
RETURN w.player.name AS name;
```

两者差异点在于：

- 出口词是 `RETURN`，不是 `YIELD`；

- **没有管道、没有 `$` 类符号**，中间传递靠变量（`v`/`w`）和 `WITH`；

- 排序/限行直接跟在 `RETURN` 后。比如 `RETURN … ORDER BY name LIMIT 10`，它们是 RETURN 的后置子句，不需要再“接一段”；

- **MATCH 与原生语句不能混用**。比如 `MATCH … | GO …` 是未定义行为。

---

当你想**接着加工这张表**——排序、聚合、限行，或把每一行当作起点**再查一次图**，于是用管道把它接上——当前段的输出，作为输入，传给下一段语句。还是上述相同的例子：

```ngql
GO FROM "player100" OVER follow YIELD dst(edge) AS who
| GO FROM $-.who OVER follow YIELD dst(edge) AS who2   # 把 who 的每一行当新起点，走第二跳
| ORDER BY $-.who2                                     # 给上一步的表排序
| LIMIT 10;                                            # 截断
```

有些别扭的是，排序不是 `GO` 的“一部分”。这是因为，`ORDER BY` 在原生里是一个**独立算子**、子句，作用对象是“上一步已经产出的整张表”，所以不能写成 `GO ... ORDER BY ...`，只能让 `GO` 先把表 `YIELD` 出来，再通过管道把表交给排序；`LIMIT` 同理。它们不是 `GO` 的子句，而是“表的加工算子”，位置永远在管道 `|` 右侧。（对照 MATCH 那边：排序和限行是 `RETURN` 的**后置子句**，直接跟在后面写 `RETURN … ORDER BY name [SKIP n] LIMIT m` 即可，不用再“接一段”。）

所以，**原生 = 把很多小算子用 `|` 串成一条流水线**，一段一段，每段都“一张表进、一张表出”。表要被下一段引用，列就得有名字——所以每段 YIELD 都要用 `AS` 起别名；下一段想读“上游当前这一行”的某个列，就用 `$-.列名`：

```ngql
GO FROM "player100" OVER follow YIELD dst(edge) AS who         # 段1 产表：列 who
| GO FROM $-.who OVER follow YIELD dst(edge) AS who2           # 段2 每行当起点，走第二跳
| GROUP BY $-.who2 YIELD $-.who2, count(*) AS cnt              # 段3 按 who2 分组计数
| ORDER BY $-.cnt                                              # 段4 给整张表排序
| LIMIT 10;                                                    # 段5 截断整张表
```

注意一点：`GROUP BY` 分完组后**必须再补一个 YIELD** 来定义输出列（不能沿用上游的列），聚合是显式写的；MATCH 则相反，`RETURN` 里同时出现 `count(*)` 这类聚合和非聚合列时，它会按非聚合列**自动分组**。

万一这一串太长想**拆开写**，或某段结果想**反复用**，就别用管道一根根串，改用分号 `;` 把“整张表”存进自定义变量。存好后，**同一次复合查询里任意一条后面的语句**都能点名引用 `$id`、反复用多少次都行；但它有两条硬边界：

1. 定义句与引用句必须在**同一次请求**里一起提交给服务端（console 里用行尾 `\` 续行把多句连成一条；且定义句必须以 `;` 收尾）；

2. 引用只能写在定义句**之后**（语句自上而下执行）。这次请求一结束，变量即释放，留不到下一次查询或别的会话

语法示例如下：

```ngql
$id = GO FROM "player100" OVER follow YIELD dst(edge) AS id;   # 存：整张表放进 $id
GO FROM $id.id OVER follow YIELD dst(edge) AS who2;            # 用：以 $id.id 每行为起点再走一跳
```

---

到此 `$` 打头的东西你已经见了好几个——它们引用的是**完全不同的对象**，引用的对象**分属两个世界**。这套符号是**原生族专用，MATCH 里一个都没有**：

- **`$var`、`$-` 活在“结果表的世界”**。复合查询就是一段段小查询用 `|` / `;` 接起来，每段产出一张**表**。这两个符号回答的是“我引用中间产出的哪张表 / 哪一行”。

- **`$^`/`$$`、`src()...` 活在“图遍历那一跳的世界”**。`GO ... OVER <边>` 是沿边一跳一跳地走，每迈一条边产出一行。这两个符号回答的是“此刻我踩的这条边：两个端点是谁、它本身长什么样”。

逐个说清：

- **`$var` = 存下来的“整张表”**。用 `;` 把某条语句的结果整张起名存下（上面的 `$id = GO …;`），同一次复合查询里想引用几次引用几次。它永远是**表**、不是单值，所以取列要写 `$id.id`。

- **`$-` = 管道里“正喂到的这一行”**。管道只认**自己紧挨着的那根管道**，比如在 `A | B | C` 里，写在 C 中的 `$-.x` 指 **B 的输出**，而 B 自己用的 `$-.x` 指 **A 的输出**。每过一根 `|`，`$-` 就**重新绑定**一次，够不到更前面。所以 A 的列想留到 C，要么让 B 把它再 YIELD 一遍带走，要么用 `$var` 存。

- **`$^` / `$$` = 这一跳的起点 / 终点顶点**。每迈一条边产出的一行里，这条边的起点顶点叫 `$^`，终点顶点叫 `$$`。它们是整个**点**，取属性要写进具体 Tag，如 `$^.player.name`、`$$.team.name`。它们只活在“正在迈边”那一句 GO 的 YIELD 里，而且**每做一次 GO 就按新的一跳重新定义**——两跳查询里，第二段的 `$$` 才是真正的目标，第一跳的早已散场。

- **`src()/dst()/type()/rank(edge)` = 把“这条边”解剖成四元组**。`$^`/`$$` 只给两端**点**，给不了这条边本身；这组函数拆的是边：`src(edge)`/`dst(edge)` 给起点/终点的 **VID 标量**（不是点对象——所以 `dst(edge)` 才能直接拿去当下一跳起点），`type(edge)`/`rank(edge)` 给边类型和 rank。FETCH / LOOKUP 处理“边值”时没有“跳”可言，同样靠这组函数拆。

总结如下：

| 符号 | 引用的是 | 什么时候能用 |
|---|---|---|
| `$var` | 前面某条语句**整张**存下的结果表，取列写 `$var.列名` | 整个复合查询内，定义之后随意反复用 |
| `$-` | 管道**紧邻上游**正喂到的**这一行**，取列写 `$-.col` | 只在自己紧挨的那一段里；每根管道重绑一次 |
| `$^` / `$$` | 这一跳这条边的**起点 / 终点顶点**，取属性写 `$^.tag.prop` | 只在逐边遍历（GO 这类）那一句的 YIELD 里；每跳重定义 |
| `src()/dst()/type()/rank()` | 把一个**“边值”**拆回 起点VID / 终点VID / 边类型 / rank | 手里有边值就能用（GO 的 YIELD、FETCH/LOOKUP） |

而 MATCH 族**没有这套符号、也不认管道**。若它同样想“把上一步的结果接着用”，则靠两个自己的子句：

- **`WITH`**：MATCH 的“管道位”。把当前已得到的列接住，可以改名、过滤、聚合，再交给后面的 `MATCH` / `RETURN`。作用上和 `|` 一样是“段与段之间递表”，写法上却更像 SQL 逐层传参：

- **`UNWIND`**：把**一个列表值拆成多行**，行数由列表长度决定。常用来把 `collect()` 聚合出的列表再还原成一行行处理，如 `UNWIND [1,2,3] AS x RETURN x;` 拆成 3 行。

`WITH` 的语法示例如下：

```ngql
# player100 的朋友 w ，再去摸他们效力过的球队
MATCH (v:player)-[:follow]->(w:player)
WHERE id(v) == "player100"
WITH w                              # 把朋友 w 带到下一段
MATCH (w)-[:serve]->(t:team)
RETURN w.player.name AS name, t.team.name AS team;

```

至于“要走两跳”：MATCH 也不用像原生那样接第二段 GO，把模式写长就行（`*1..2` 表示 1~2 跳，由引擎自己决定怎么展开），这就是“声明式”和“命令式”的分水岭：

```ngql
MATCH (v:player)-[:follow*1..2]->(w:player)
WHERE id(v) == "player100"
RETURN DISTINCT w.player.name AS name;
```

最后还有“多张表”之间的拼接——**集合操作**，同样原生族才有：两张结构一致的 YIELD 结果，可以 `UNION`（合并去重）、`UNION ALL`（合并不去重）、`INTERSECT`（取共同）、`MINUS`（前者减后者），前提是两边**列数、列序、类型一致**；要按列值“对上”地横向拼，用 `INNER JOIN`（等值连接，条件写 `==`）：

```ngql
GO FROM "player100" OVER follow YIELD dst(edge) AS id
UNION
GO FROM "player102" OVER follow YIELD dst(edge) AS id;
```

把上面这些（管道、分号、变量、集合操作）串起来的多条语句，合称**复合查询**。它**没有事务/隔离性**：中途某一条失败，**不会回滚**前面已经执行成功的语句。涉及写操作时别指望“要么全做要么不做”，得在脚本里自己安排幂等或补偿。

---

## 6 ★ nGQL 如何被执行

> **追问一个问题：一条 nGQL 是怎么执行并返回结果的？**

### 6.1 路径与时间线

一条 nGQL 从用户在 console 回车、到结果回到屏幕，中间不是“数据库直接查一下”这么简单：graphd（查询引擎所在进程）内部要把它像订单一样沿一条**流水线**依次交给好几拨人处理——先**读懂**你写了什么，再**检查**有没有写错、用的变量名在不在，接着**画一张施工图**定好先干什么后干什么，最后才**照图干活**、把结果一路汇回来。

但动手捋之前，**先分清“谁只做一次、谁每次都要做”**——否则很容易把整条流水线当成“每来一条查询就从头到尾跑一遍”。引擎其实是**两层时间线**：

- **层1 冷启动**：graphd 进程刚起来时做一次，把“家里常备的东西”备好——`registerPlanners()` 注册好各类 Planner、按 `FLAGS_enable_optimizer` 挑好优化规则集、`new` 出一个全局唯一的 `opt::Optimizer`。这一步发生在**任何一条查询到达之前很久**，和你“现在敲的这条”没有关系，更不会每来一条查询就重新注册一遍。
- **层2 每一条请求**：上面说的“读懂->检查->画图->干活”这一整串，**每一条 nGQL 都各自完整地走一遍**。

在源码里，这两层正好落在 `src/graph/service/QueryEngine.cpp` 的两个函数上：`init()`（L27 起）负责层 1,`execute()`（L49 起）负责层 2。

观察层2，一条具体请求从开始到返回的旅程是：

1. **进门打包**：`execute()` 收到请求，先 new 一个 `QueryContext`（装本次请求的 session、当前 space、schema/索引/storage/meta 客户端等上下文），再 new 一个 `QueryInstance` 专职跑这条查询，由 `instance->execute()` 启动。

2. **解析**：词法/语法分析（`src/parser/`,Flex/Bison）把 nGQL 文本咬成一棵 **AST**（`sentence_`）——这里只保证“语法没毛病”，还不管“写没写对”。

3. **校验**：`Validator.cpp` 做语义检查，并建**符号表**——5.7 里那些 `$var`、YIELD 出来的列名，就是在这儿登记、让后续步骤认得的；输出“校验过的上下文”。

4. **生成计划**：按语句是 GO / LOOKUP / MATCH / DDL…挑对应 Planner（`GoPlanner`、`LookupPlanner`、`MatchPlanner`、DDL `MaintainPlanner`…），把它变成一棵由 **PlanNode** 组成的执行计划（施工图；6.2 专讲它长什么样）。

5. **优化（看情况）**：`Optimizer` 在层①就绪，但某句要不要真被改写，取决于 `FLAGS_enable_optimizer` 开着、**且**这句能命中某条改写规则——命中不了就原样往下走。所以它是**（条件性）必经关卡，不是必经之路**。

6. **调度**：`AsyncMsgNotifyBasedScheduler` 把施工图摊成一张 Executor 图，**等齐一个算子的全部依赖才触发它**，互不依赖的分支可以**并发**跑（正因为是无环 DAG 才敢这么调度；6.2 讲）。

7. **执行**：`Executor.cpp` 按 PlanNode 的 `kind` 一一造出真正的执行器，把中间结果按“输出变量名”写进 `ExecutionContext` 供下游读，最后沿 DAG 汇成一张表回到你面前。

整串流程可以压成一句话记：

> **解析（AST） -> 校验（符号表） -> 生成计划（PlanNode DAG） -> （条件性）优化 -> 调度 -> 执行**

把上面散在叙述里的“环节、关键代码、职责”收成一张表，方便以后速查：

| 环节 | 关键代码 | 做什么 |
|---|---|---|
| 服务入口 | `src/graph/service/QueryEngine.cpp` | 层①：`init` 注册 Planner、按 `FLAGS_enable_optimizer` 组规则集、构造 `opt::Optimizer`；层②：`execute` 每请求建 `QueryContext` + `QueryInstance` 并执行 |
| 解析 | `src/parser/`（Flex/Bison） | nGQL → AST（`sentence_`），只查语法 |
| 校验 | `src/graph/validator/Validator.cpp` | 语义校验；建符号表（列/变量）；输出“校验后的上下文” |
| 生成计划 | `src/graph/planner/`（`PlannersRegister.cpp`） | 按语句 Kind 映射 Planner（`GoPlanner`/`LookupPlanner`/`Fetch*Planner`/`PathPlanner`/`SubgraphPlanner`/DDL `MaintainPlanner`/`MatchPlanner`）→ 产出 PlanNode 构成的执行计划（`planner/plan/`） |
| 优化 | `src/graph/optimizer/Optimizer.cpp` | 把计划转成 Memo（OptGroup/OptGroupNode）-> 反复套规则改写 -> 属性裁剪；命中不了规则则原样跳过 |
| 执行调度 | `src/graph/scheduler/AsyncMsgNotifyBasedScheduler.cpp` | 计划 → Executor 图；**DAG 调度**：等齐依赖才触发，无依赖分支**并发**执行 |
| 执行算子 | `src/graph/executor/Executor.cpp` | 按 PlanNode::Kind 映射 Executor 子类；结果按“输出变量名”写入 `ExecutionContext` 供下游读取 |
| 上下文 | `src/graph/context/QueryContext.h` | 聚合 `RequestContext`（session/响应/线程池）、`ValidateContext`（space）、`ExecutionContext`（变量值域）、`SymbolTable`、`ObjectPool` 与各客户端 |

> 注意：表里第 1 行“服务入口”其实是**半个层 1 + 半个层 2**。`init` 的部分属于冷启动，“每请求建 QueryContext + QueryInstance”才属于层 2。它排在开头只是因为是代码入口、方便顺着读，**不代表每次查询前还要再初始化一遍**。

---

### 6.2 执行计划

计划阶段产出的那份“施工图”，执行计划，**本身是一张有向无环图（DAG）**——一条查询要被切成一小串运算，再由调度器按先后去跑（6.1 第 6、7 步）。看懂它只需回答两个问题：**图上的一个点是什么？点和点之间的箭头又是什么意思？** 想清楚这两点，下面的算子清单就不用背——每个名字基本都能从你写过的 nGQL 猜出来。

- **图上的点：PlanNode，执行计划里的一步运算。** Planner 把一条查询“切”成的最小工作单元，就是这样一个节点。它带一个 `kind`，标明“我要做哪类动作”：`Project`=把 YIELD 的列投影出来、`Filter`=套 WHERE 过滤、`GetNeighbors`=GO 那一跳去取邻居、`Aggregate`=分组聚合……计划阶段它们是 `planner/plan/` 下的对象；真跑起来时 `Executor.cpp` 按 `kind()` 造出对应的执行器——所以 6.2 的标题叫“PlanNode ↔ Executor 一一映射”。

- **图上的箭头：数据依赖，“上游算完，把结果喂给下游”。** 这就是为什么它叫“图”而不叫“链”：一个点可以同时等好几个上游（几个分支汇进来），整张图也可以有好几个起点。每个节点都自带一张“我等谁”的清单（上游列表），调度器正是靠它决定谁先跑、谁要等。

**为什么必须是“有向、无环”？** 数据只能从算完的一头流向还没算的一头，所以箭头是单向的；要是 A 依赖 B、B 又依赖 A，那两边都等不到对方的数据，永远算不完。**保证无环 = 保证从任何一个“起点”出发，沿着箭头一定能走到“终点”**——所以调度器才敢“等齐全部依赖再触发”（6.1 第 6 步）。一张计划的样子大致是：

```
Start ──► A 分支的若干节点 ──┐
                           ├──► 收口节点(常见叫 DataCollect / Union) ──► ……结果
Start ──► B 分支的若干节点 ──┘
```

起点在计划里就是 `Start`；一条复合查询（`UNION`、多段拼接）会有**多个 `Start`**，各自算完再汇进一个“收口”节点。上图只是概念示意。

**那“要重复跳好几遍”的查询怎么办？会不会画成环？** 不会。多跳（`GO N STEPS …`）、变长模式（`[:follow*1..2]`）这类“同一段动作反复做”的查询，会被编译成一个 `Loop` 节点，由它把一段子计划**重复执行 N 遍**——静态计划里仍然没有环。顺带一个好消息：你在 5.7 写的管道 `A | B`，编译后**就是**在图上添一条“B 的节点依赖 A 的输出节点”的边——5.7 学的所有子句（YIELD / WHERE / GROUP BY / LIMIT / UNION / 变量）最终都摊成这张图上的节点和依赖边，**没有第二套东西**。

形状讲清楚了，下面是 Planner 实际会造出的**常见节点清单**，按类别记，名字≈nGQL 动作（`executor/query/` 等目录 ls 实证）：

| 类别 | 算子/节点（$\approx$ nGQL 动作） |
|---|---|
| 存储访问/遍历 | `GetVertices`（按 VID 取点）、`GetEdges`（取边）、`GetNeighbors`（GO 一跳取邻居）、`Traverse`/`Expand`/`ExpandAll`/`AppendVertices`（多跳扩展并补取点）、`IndexScan`/`PrefixScan`/`RangeScan`（LOOKUP/MATCH 走索引）、`ScanVertices`/`ScanEdges`（全扫描）、`FulltextIndexScan` |
| 关系代数/管道 | `Project`（投影/YIELD）、`Filter`（WHERE）、`Aggregate`（GROUP BY/聚合）、`Sort`/`TopN`（ORDER BY/LIMIT）、`Limit`、`Dedup`（DISTINCT）、`Unwind`、`Assign`（变量）、`DataCollect`（多输入汇成结果） |
| 连接/集合 | `InnerJoin`/`LeftJoin`、`Union`、`Intersect`、`Minus` |
| 逻辑控制 | `Start`（图的起点/叶子）、`Loop`（把一段子计划重复跑 N 遍，多跳/变长用）、`Select`（条件分支） |
| 其它 | DML → `executor/mutate/`；DDL → `executor/maintain/`；`SHOW TAGS` 等 → `executor/admin/` |

### 6.3 EXPLAIN / PROFILE：把计划打出来看

6.2 那张“施工图”平时藏在引擎里——用户只看得见查询的**最终结果**，看不见它是怎么被规划的。`EXPLAIN` / `PROFILE` 就是把它**打出来给你看**的两条命令。

- **`EXPLAIN` = 只出计划，不执行。** 引擎把这条 nGQL 走完 6.1 的「解析 -> 校验 -> 生成计划 ->（条件性）优化」，在真正派算子去跑**之前**停住，把将要执行的计划打印出来——像“开工前审施工图”：结构对不对、走没走索引，一望便知。但也因为**没真跑**，每个算子“跑了多久、多少行”这类实测数据是**空的**。

- **`PROFILE` = 真执行一遍，顺带逐算子记账。** 除了打印同一张计划，它真的把查询跑完，并把**每个算子实际输出多少行、花了多久**（含发往 storaged 的 RPC 耗时）记在对应行上——像“完工后回看每个工位各用了多久”，调优时看的是它。

所以一言以蔽之：**只想看“这条查询被规划成什么样”→ `EXPLAIN`；想定位“哪一步慢、哪一步行数爆了”→ `PROFILE`。**

两者的语法只差关键字；`format` 只决定**以什么样式打印**：

```ngql
EXPLAIN [format={"row"|"dot"|"tck"}] <nGQL>;   # 只出计划，不执行
PROFILE [format={"row"|"dot"|"tck"}] <nGQL>;   # 真执行 + 出计划与概要
```

其中：

- `row`：表格，每个算子占一行，**就是 6.2 说的“一个 PlanNode 一行”**；

- `dot`：Graphviz 图，把依赖画成真正的箭头（更直观，日常读 row 即可）；

- `tck`：供测试/脚本用的类表格。

下面以 `row` 为例。一张表一个算子一行，5 列含义如下：

| 列 | 含义 |
|---|---|
| `id` | 算子 ID（图里节点的编号） |
| `name` | 算子名——6.2 清单里的 `GetNeighbors`/`Project`/`Loop`…（节点种类） |
| `dependencies` | 依赖的算子 ID 列表——6.2 说的“我等谁”（指向它的依赖边） |
| `profiling data` | `ver`（实现版本）、`rows`（该算子**实际输出行数**）、`execTime`（纯执行耗时）、`totalTime`（含排队/调度总耗时）——**只有 PROFILE 会填** |
| `operator info` | `outputVar`/`inputVar`（算子间传的中间结果变量名，5.7 里那些中间结果在这里有了正式名字），如 `__GetNeighbors_1` |

**注意：表里没有 “est. rows / 预估行数” 这类预估列。** 因为 `EXPLAIN` 不执行，`profiling data` 整列是空的——引擎只能在你**跑完之后**告诉你“实际”多少行，没法在跑之前告诉你“大概”多少行。这本身就是“开源版无基数估计”的表现之一（见 6.4）。

---

### 6.4 优化器

> 教科书会说：优化器要在**一堆等价的执行方式里挑代价最小的那个**（CBO）。那 Nebula 挑了吗？答案是：**它一直在改写，却从没"挑"过。** 6.4 到 6.6 就在讨论这件事，以及它给“运行时反馈”留了多大一个空位。

先把"优化"拆成两个问题，后面才好对照着看：

| 问题 | 意义 |
|-----|-----|
| **能换成哪些写法？**| 同一句查询，执行起来往往不止一种走法——先滤后连还是先连后滤、一路扫过去还是走索引……“换一种等价写法”靠**改写规则**就能答：只要保证换完结果一样，规则一条条套下去，新计划就合法。规则解决的是“有哪些选择”。 |
|**哪种写法真的更快？** | 这要另一套东西：先估算每个算子会**碰多少行**（基数/选择率，原料是统计信息），再按算子的开销模型把整棵树的代价加起来，最后**比价取最小**。这套才是“代价模型”（CBO）。它回答的是“在这些选择里挑哪个”。 |

**Nebula 开源版把第一个问题做得很足，第二个问题整段空缺。** 下面分两个问题讲。

**第一个问题：改写规则，素材很足。** Planner 画出的施工图只是“一种写法”；优化器拿着一沓规则把它换成“另一种写法”。一条规则就是一条模板：“看到这个形状，我帮你换成那个形状。” `optimizer/rule/` 目录下有 **58 个规则实现文件、50 多条具体规则**，扫一眼名字就知道各自在干嘛：

| 类别 | 在干嘛 | 例子 |
|---|---|---|
| 下推类 | 把 Filter/Limit 尽量往数据源头挪，“早滤一行省一路” | `PushFilterDownGetNbrsRule`、`PushFilterDownProjectRule`、`PushLimitDown*`、`PushTopNDownIndexScanRule` |
| 合并/消除类 | 把相邻两步捏成一步、或删掉确实没用的节点 | `MergeGetNbrsAndProjectRule`、`EliminateFilterRule`、`CollapseProjectRule` |
| 索引改写类 | 把“先全扫再过滤”换成“直接走索引取” | `IndexScanRule`、`*IndexFullScanRule`、`UnionAll*IndexScanRule` |

这些规则分三组（`QueryEngine.cpp` 组装）：`DefaultRules()`（索引改写等）始终生效，`QueryRules0()`、`QueryRules()`（下推/合并等）顺序套用。

**第一半的运作方式：`findBestPlan` 就干三件事**（`optimizer/Optimizer.cpp`）：

```
Planner 的施工图
   │ ① prepare：每个节点包成 OptGroup/OptGroupNode（Memo）
   ▼
被规则反复改写，直到"改不动"（② doExploration）
   │ ③ getPlan 取一棵 → postprocess 收尾（补绑输入 + 属性裁剪）
   ▼
新施工图，交给调度器执行
```

| 步骤 | 在干嘛 | 要点 |
|---|---|---|
| **① prepare** | 把计划树“包一层壳”：每个节点套一个 `OptGroup`（组）/`OptGroupNode`（组内成员） | 这层壳是给“同一个子树并存几种等价写法”留的容器（Memo 的标准用途）——**只是包装，不评好坏** |
| **② doExploration** | 反复套规则：自底向上把每条规则在每个组试一遍，有命中就改写、改写完从头再来一轮，直到某轮“什么规则都不再命中”才停 | 上限：外层 **5 轮**、单个组对单条规则 **128 次**（两道保险防死循环）。终止条件是 **“改不动了”，不是“改到最优了”** |
| **③ getPlan + postprocess** | 从组里取一棵计划；收尾只做清理——给个别算子补绑输入变量、**属性裁剪**（把算完也用不上的列掐掉） | 依然没有“选优” |

**优化器把"反复套规则直到不能再改"当成了优化的全部；至于"改完是不是更快"，没有任何环节在问。**

---

**第二个问题的空缺，在源码上体现在**：

1. **作者自己承认了。** `planner/PlannersRegister.cpp` 排 MATCH 起点顺序时有句注释：*"Now we hard code the order of match rules before CBO, put scan rule at the last for we assume it's most inefficient"*——先试索引、把全扫垫底，理由是“**假设**它最没用”。没有统计就没依据、只能靠假设，作者也写明这是在等 CBO。

2. **代价模型要吃的“米”不存在。** 全仓库 grep `cardinality` / `estimate` / `selectivity` / `statistics`，在供执行链路消费的 `src/graph|common|meta|storage` **零命中**——没有任何数据分布可供估算“碰多少行”。

3. **账本留了“金额”栏，但从没人填。** `PlanNode` 挂着 `cost_` 字段和 `calcCost()` 虚函数，但全仓无人给 `cost_` 赋值；`calcCost()` 只有空壳实现，打印一行 "unimplemented cost calculation." 就返回。

4. **名义上“选优”的函数，实际“拿第一个”。** `OptGroup::findMinCostGroupNode()` 确实遍历候选、严格比小——可所有候选 `cost_` 都是 0，比不出差别，于是永远停在第一个。选择退化成“组内先到先得”。

第 4 点顺带解释了另一现象：**一个组里通常只剩一个候选。** 规则改写普遍是“换掉旧的”（transform 完把旧节点 erase）；多候选并存的机制框架其实支持（`TransformResult` 可以只加不删），但**留了也没用**——没有代价就分不出高下，自然没人愿意留。真要做 CBO，这两件事得**同时**发生：规则愿意多留候选，**并且**代价能真算出差别。这比“补一个代价函数”更深一层，是结构性改造点。

那为什么代价都算不出来？因为算代价得先估计“每个算子会碰多少行”，而估计的原料是统计信息——6.5 告诉你：Nebula 有统计，但从不流进这里。

---

### 6.5 统计信息

Nebula 不是没有统计，而是有一套**离线、粗粒度、面向人**的统计：

一条 `SUBMIT JOB STATS` 把统计当**异步 job** 跑：Meta 收下任务、写一个 RUNNING 状态，按分片派给各 storaged；每个 storaged 逐个分片扫，数出**每个 Tag 多少点、每条边类型多少边、整个 space 多少点边**，汇总回 Meta 存下来。之后 `SHOW STATS` 由 graphd 从 Meta 读出打印；还没算过就报 `E_STATS_NOT_FOUND`。

像店里定期盘一次库存、记在总账上——**给店长看**，仅此而已：

- 没有直方图、没有谓词选择率、没有列级分布，只有"整表行数"级别的粗计数；
- 更要紧的是：optimizer / planner / validator **没有任何代码读它**。统计在 Meta 里躺着，管道另一端根本没接。

---

6.3 里 PROFILE 每个算子那行 `rows` / `execTime` 是从哪来的？—— `Executor` 基类（`executor/Executor.h`）自带计测，每个算子跑的时候顺手量：

- `numRows_`：该算子**实际输出行数**（`finish(Result&&)` 时记下）；
- `execTime_`：纯执行耗时（算子内 `SCOPED_TIMER(&execTime_)` 累计）；
- `totalDuration_`：含排队/调度的挂钟总耗时；
- `otherStats_` + `addState()`：算子自定义指标（如发往 storaged 的 RPC 耗时明细）。

上报一条线：`close()` 把它们装成 `ProfilingStats` -> `ExecutionPlan::addProfileStats(node, stats)` -> `PlanDescription` -> 随 EXPLAIN/PROFILE 回给客户端。**6.3 那张表的 profiling data 就是这么来的。**

**课题含义就在这**：真实行数、真实耗时，**每个算子每次执行都在量**，量完顺着 PROFILE 打印出去，**然后就没有然后了**——没有任何机制把 `numRows_`/`execTime_` 沉淀下来、去影响下一次的计划选择。**"运行时反馈"想接"执行后的实测值"，埋点桩已打好，差的只是从这儿回到优化器的一条管道。**

---

### 6.6 课题落点

把 6.4 和 6.5 拼起来，现状就是一张“三截断管”图：

```
                ┌───────────────────────────┐
  计划选择       │ Optimizer：纯规则（RBO）    │ <- 无 cost_、无基数估计、无选择率
                └───────────────────────────┘
                        ▲ (没有任何反馈回路 ✗)
                ┌────────────────────────────┐
  执行过程       │ Executor：有 numRows_/time ✗│ -> 数据只进 PROFILE，执行完即丢
                └────────────────────────────┘
                ┌────────────────────────────┐
  统计信息       │ SUBMIT JOB STATS 离线粗粒度  │ -> 存 Meta，只给 SHOW STATS 看
                └────────────────────────────┘
```

一句话：**执行器能测 -> 优化器不消费 -> 统计离线且不喂优化器。** 三条管子互不相通，正是"运行时反馈 + 自适应代价模型"要补的环：把执行器量到的真实行数/耗时回灌成下一轮代价估算的依据，让优化器从"改到不动"进化成"改到更优"。
