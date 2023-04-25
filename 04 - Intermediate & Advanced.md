> 我们今天的主题是关于Data Vault 2.0数据建模的一些高级技术。我们将用一些医药行业的例子来解释。

## Same-As Link

> 以一个典型的医药企业为例，通常会有很多不同的业务部门来负责生产和销售不同种类的药物，比如普通药品事业部、特殊药品事业部（典型特征是高净值，如肿瘤，小分子靶向药，皮肤病特应性皮炎特效药达必拓）、疫苗事业部（新生儿疫苗，流感疫苗）等；
> 
> 对于医院来说，按照《医院分级管理标准》，医院经过评审，确定为三级，每级再划分为甲、乙、丙三等，其中三级医院增设特等级别，因此医院共分三级十等。如果是三级特等医院，可能涉及的医学范围就很大，也会同时覆盖普药、特药和疫苗等不同的医学领域。
>
> 从医生的角度来说，也是相同的道理，在其职业生涯中，可能会涉及多个医学领域，比如刚开始是普药，后来进入疫苗，最后进入特药领域。
> 
> 然而由于医药企业的IT规划在最初规划主数据管理的时候，不同的BU系统之间相互割裂，对于HCO（医疗组织机构），HCP(医疗专业人士)的数据存储和管理也是在不同的业务系统中。如下图所示：
>

普药和疫苗事业部的不同系统，在存储相同的医疗组织机构（HCO）时，由于企业的信息孤岛，导致两个系统对于相同的四家医院，生成了不同的业务键，即医院编码；在Data Vault 2.0的数据模型体系中，我们将这些具有业务含义的键，叫做业务键（Business Key，BK）；

![HCO Pharmcy & Vaccine](images/SAL%20-%20HCO%20Pharmcy%20Vaccine.png)

如果我们对这两类数据进行数据建模，建立统一的枢纽（HUB）实体，首先要符合两个条件：

1. 两者从编码体系上，是不会相互重复的，这个例子中普药医院管理系统使用的是纯字母编码，疫苗医院管理系统使用的纯数字编码；
2. 两个系统的表都是在描述医疗组织机构（HCO），从业务上理解是一个相同的概念，且数据颗粒度是一致的。

因此两者经建模之后，形成的枢纽（HUB）模型表如下所示：
一部分数据来自普药系统，一部分来自疫苗系统。

![HCO HUB](images/SAL%20HUB%20HCO.png)

由于我们已知，两个系统中有相同的医院，并且也可以获悉不同系统中相同医院的对应关系，那么就可以通过Same-As Link来把这些数据关联起来，表明了不同系统之间相同实体之间的关系，如下所示：

![SAL Model From HCO](images/SAL%20Model%20from%20Hub.png)

Same-As Link的表结构如下所示：

![SAL of HCO](images/SAL%20HCO.png)

Same-As Link的完整流程：

![SAL Full](images/SAL%20Full.png)

> 1. 需要注意的是，Same-As Link中使用不同数据源，对于相同业务对象的业务键不会相互覆盖，只有在这种情况下，Same-As Link才能完全识别，并体现相同业务对象的关联关系；
> 2. 除了Same-As Link的处理方法，另外两种可选的方案，一是按数据源创建不同的Hubs，并通过普通的Link表将这些Hubs链接在一起；二是将第二个系统中的业务键作为Hub模型中业务键的一部分，形成单独的Hub表，这么做的缺点是，这样的Hub无法体现不同系统之间的关联关系。
> 3. Same-As Link，是属于Business Vault的组件之一，如果两个业务对象的关联关系并不是由某一个源系统生成的，而是由人工或算法计算出来的。
> 

---
## Point-In-Time

Point In Time，PIT表属于辅助查询表（Query assistance table），它是为从原始数据库中获取数据的查询性能而建立的，为了提高数据在Data Mart层消费时的查询效率；PIT为基于视图的维度和基于视图的事实提供了等价连接的能力，它们是基于Teradata编写的Join Indexes原则，只是时间点结构可以在任何平台上实现。

Data Vault的Data model，其核心在与满足数据在加载到数据仓库时的数据一致性，可追溯和可审计性，并强调了数据模型对于业务变化的适应性、健壮性和可扩展性；但其类似第三范式的设计原理，无法满足数据在数据集市层或者通过BI工具进行高效查询的场景。这也是为什么在数据消费阶段，需要设计多维数据模型，或者PIT表来进行数据筛选、抽取及转换的必要性。

简单总结如下：

* PIT是一种特殊类型的卫星表，能够提高连接效率从而改善数据查询的性能；
* PIT表通常是在数据集市层，可以包含其他计算字段，或者其他的为业务设计的开始或结束日期字段等；
* PIT 表是，我们创建PIT表，通常是为了将同一个枢纽（Hub）或链接（Link）的多个卫星表（Satellite）合并在一起，从而能够在一个连续的时间序列中的某一个时间点（Point-in-Time），获取有效的所有描述性信息，即Descriptive Data。
* 通常不会限制在PIT中卫星表（Satellite）的数量，如果有新的卫星表（Satellite）被添加到枢纽（Hub）中，那么这些卫星表（Satellite）也会添加在PIT表中


### PIT 常见结构

PIT_Name

![PIT Structure](images/pit%20structure.png)

1. PIT表的主键是由业务键和快照时间通过哈希方法计算出来的；
2. Hub表中的Hashkey会作为代理键；
3. Satellites的中的Hashkey也会作为代理键；

### PIT 查询性能

> PIT表由于需要连接所有Hub及Link之下的Satelltes，因此其性能会受到Satellte数量的影响，下面给出了一些常见的提升PIT查询性能的建议，大家可以在具体工作中进行实践。

1. PIT表应该在有限的时间内使用，旧的数据应该在不再使用时被删除，通常可以保留3个月的数据以提高查询性能；
2. 在给定的天数或月数之后删除PIT表的快照，如在几年之内，每年保留多少个Snapshot；每年、每季度或者每周保留多少个Snapshot等；
3. 按时间间隔对PIT表进行分区，可以按Snapshot时间日期进行分区；
4. 业务团队应该同意在PIT表中保留多长时间的数据。

### PIT的例子

下面通过一个例子来讲解PIT的实际使用情况。

如下图，有一个人员信息的Hub，及其附加的3个卫星表，分别是出生信息、健康信息及财务信息。

![PIT Sample](images/pit%20sample.png)

**Hub**

人员信息Hub表中，包含了基本的Hashkey, Business Key, 加载时间及数据来源，来自妇产科医院系统；

![PIT Hub](images/pit%20hub.png)

**Satellite**

Satellite，包含了基本的Hash Key, Hash Diff, Sequence ID，加载时间和数据来源；
* Hashdiff，用于计算描述性信息的变化情况；
* Sequence ID，用于记录数据的变化顺序；

下面图中可以看出，（假设）SA001的人员：
1. 其2017年4月12日出生后，出生信息未发生过变化；
2. 6岁后由于生病，2023年4月3日 ～ 2023年4月5日连续3天去社区医院看病，因此健康信息公变化了3次；
3. 2023年4月第一周和第二周发放了两次工资；

![pit satellite](images/pit%20satellite.png)

假设PIT的观察周期是从2023年4月1日 ～ 4月8日，那么我们得到的PIT表会是下面这个情况：

![pit table](images/pit%20table.png)

观察PIT表，我们发现：

1. PIT表按snapshot字段显示，按时间顺序将所有卫星的哈希值+加载时间信息拼接在了一起；
2. PIT表的主键通过业务键和snapshot时间戳按哈希方法计算出来；
3. 满足snapshot时间戳的卫星表，会按时间顺序显示在PIT表中；
4. 如卫星表在PIT的snapshot时间戳中，并没有存在记录，则会显示Null值，表示不存在；

由于在PIT表的实践过程中，需要通过equaljoin来达到最佳的数据库性能，因此我们需要在卫星表中避免Null值；

**ghost value**

关于ghost value，可以理解为在Satellite卫星表中增加一行数据值，该数值不与其他任何枢纽、链接或卫星表相连，不具有任何业务含义，仅为满足PIT在具体实施过程中，弥补因为有Null值导致无法使用equaljoin的目的。

#### 通过Ghost value来增强Satellite

我们在每一个Satellite中补充了Hashkey值为000000000000000000，加载时间为1900-01-01的数据，用来说明这行数据是为PIT表衍生出来的。

![Satellite with ghost value](images/pit%20satellite%20with%20ghost%20code.png)

接下来通过equaljoin将所有Satellite表链接起来之后，得到了增强的PIT表，如下所示：

![pit with ghost value](images/pit%20table%20with%20ghost%20data.png)

#### 通过Sequence ID来继续优化PIT结构

 各位一定注意到了，在每个Satellite表中都有一个字段：Sequence ID，用来标识数据在表中出现的顺序；合理利用Sequence ID，能够将PIT数据的查询性能达到极致，具体做法就是在equaljoin的时候，通过Sequence ID来做连接，并在PIT中仅通过Sequence ID来标识Satellite的数据行项目，如下图所示：

![pit with sequence id](images/pit%20with%20sequence%20id.png)

PIT表的结构通过Sequence ID进一步简化：
1. Sequence ID的数据类型通常为Int或Bigint的数字型数值，在最终的equal-join中性能将更加优秀；
2. Sequence ID替代了Hashkey与Load Datetime的组合，PIT将可以包含更多的Satellite；

### 总结
> 1. PIT时刻表是Data Vault数据建模中的一种特殊的卫星表，其通常会位于Data Mart数据集市层；
> 2. 当卫星表的数量过多时使用，且是可以删除的，其目的是为了加速数据从Raw Data Vault的查询性能，随时可以被删除。
> 3. PIT中的Snapshot时间快照区间通常根据业务需要进行设置，按年、季度、月、周、天都可以，常用的做法还会根据Snapshot时间快照进行分区；
> 4. PIT在实施过程中，通常使用EQUAL JOIN来获取查询的最佳性能，并且会为卫星表设置Ghost Value，以及Sequence ID来加速整体查询性能；
> 
---
## Bridges

brings together hub and link keys that support a particular business case.


---
## Reference

provides static lookup data such as: calendars, currencies, countries and transaction codes.



## Reference Materials

* Building a Scalable Data Warehouse with Data Vault 2.0 - 5.1 HUB Application
* [Data Vault Implementation and Automation A pattern for Data Mart Delivery](https://roelantvos.com/blog/wp-content/uploads/2019/01/Data-Vault-Implementation-and-Automation-A-pattern-for-Data-Mart-delivery.pdf)
* [Data Vault Techniques on Snowflake: Point-in-Time (PIT) Constructs and Join Trees](https://www.snowflake.com/blog/point-in-time-constructs-and-join-trees/)