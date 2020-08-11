---
title: "ApacheOFBiz  实体引擎与服务引擎"
date: 2020-06-11T15:47:38+08:00
draft: true
---

### Apache OFBiz--实体引擎(Entity Engine)

---

实体引擎是OFBiz最有价值,最核心的,也是最稳定的一个数据层控制器,通过它基本不用编码或很少编码就可以访问数据层,OFBiz提供了用XML写配置文件的方法来操纵数据

OFBIZ 实体引擎提供了一组工具和设计模式来对现实世界中特定的实体（数据对象）进行建模和管理。在系统的上下文环境中,一个实体就是一个由多个数据域 （fields）和该实体与其它实体之间的关系所组成的一个数据对象。这个定义来自于关系型数据库对实体关系模型（Entity-Relation modeling）概念的标准定义。实体引擎的目标是简化企业级应用中对实体数据（对应关系型数据库表）的大量操作，包括定义、维护、通用操作（增、删、 改、查实体和实体之间的关系）的开发工作

实体引擎的一个主要目标是尽可能的提供一种通用的代码结构，来消除在针对每一个实体的事物处理过程中，所有写死（hard code）的代码。 这种系统抽象所关注的问题，与那些把数据从数据库中提取出来，并以报表的形式进行输出和显示处理的报表管理或类似系统是不同的，而是类似于每日都可能发生 很多事物处理的商业应用系统，实体引擎能大量节省构建类似应用系统的开发费用和戏剧性的减少因为系统存在大量写死的事务处理代码所产生的bug。这种类型 的应用系统目前在OFBIZ中实现了一些，如电子商务，入库、出库的帐目管理，任务分配资源管理等等。这些工具能够用来报告和分析系统，但是并不意味着， 它能包容千差万别的客户的应用需求，在实际应用中，我们可以基于它来做一些二次开发。 

实体引擎采用了不少核心J2EE设计模式,如值对象,代表,助手等模式,用户的API接口比较友好

#### 理念

安全可靠的数据存储是数据管理战略的关键业务，OFbiz认真对待数据管理。不把全部繁琐和easy出错的数据管理任务留给应用开发人员。OFbiz在设计和实现阶段非常好的贯彻了这个理念

实体引擎是数据库无关的应用程序开发和部署光看，无缝集成到OFbiz代码中，它能够处理全部的日常数据，安全可靠的管理任务，包含还不限于

- 同一时候连接到随意数量的数据库
- 管理不限数量的数据库连接池
- 负责数据库事务
- 处理数据库错误


#### 实体引擎的好处

- 以前的问题背景 :

1. 你需要借助工具或手工去维护已经存在的或新增加的数据库结构（库结构，表结果等的定义和更新），如果要修改表结构和定义的话，怎么做？
2. 假设你的应用涉及200张表（实体），假设每张表（实体）都存在增、删、改、查，则需要在你的应用中静态构造（硬编码）800个sql语句。
3. 假设这200张表之间存在100种关系，维护每一种关系的增、删、改、查，又需要400个静态构造的sql语句。
4. 假设这些sql语句由10个不同水平的程序员来构造，构造出来的sql语句在执行性能上可能存在巨大差异，而且形态各异。
5. 这些硬编码的sql语句分布在大量Java程序的各个角落，一旦某张表的结构发生变化或者修改某一字段名或表名，意味着什么？意味着混乱

- OFBIZ是如何解决这些问题的：

OFBIZ拒绝这种混乱，一套EntityEngine(实体引擎)机制轻松解决上述所有问题。

1. 涉及1张表（实体）的增、删、改、查，它提供一套处理机制（不到12个类，大约5千行代码），应用的规模是10000张表，它还是这套处理机制（不到12个类，大约5千行代码），而且这些处理机制由JAVA程序高手生成和维护，可以保证其合理性、可靠性和安全性。
2. EntityEngine提供了一个构造复杂sql操纵语句的机制，你可以根据需要随时构造任意复杂的sql语句，完成你想要做的事情，这样你可以在开发过程中，随时修改你的数据库定义，OFBIZ在系统启动时会自动加载并检测数据库中的不一致性和参考完整性。
3. 实体引擎大大简化了涉及关系型数据库的管理和维护，但这还只是一小块好处，大的好处是你在实现一个复杂需求的应用时，实体引擎用为数不多的几个类解决了你所有的问题，实现任意复杂的数据库存取业务和商业逻辑，而且与需求的复杂度和数量无关。

#### 实体建模

在OFBiz的实体引擎中通过两个XML文件来完成,一个是关于实体建模的,另一个是关于字段类型建模的

OFBiz的主要实体模型XML文件能够在\specialpurpose\webpos\entitydef下找到,最初的所有实体都在文件entitymodel.xml中,但是现在它们被拆分到了不同的文件中(包括entitymodel.xml中),以下列模式命名:entitymodel_*.xml

#### 实体定义

```
<entity entity-name="ProdCatalog"
            package-name="org.ofbiz.product.catalog"
            title="Catalog Entity" default-resource-name="ProductEntityLabels">
      <field name="prodCatalogId" type="id-ne"></field>
      <field name="catalogName" type="name"></field>
      <field name="useQuickAdd" type="indicator"></field>
      <field name="styleSheet" type="url"></field>
      <field name="headerLogo" type="url"></field>
      <field name="contentPathPrefix" type="long-varchar"></field>
      <field name="templatePathPrefix" type="long-varchar"></field>
      <field name="viewAllowPermReqd" type="indicator"></field>
      <field name="purchaseAllowPermReqd" type="indicator"></field>
      <prim-key field="prodCatalogId"/>
    </entity>
```

#### 标准实体

- 属性
  - entity-name： 实体名
  - table-name：表名
  - package-name：包名
  - default-resource-name：缺省资源文件名
  - dependent-on：指定父级实体和依赖的实体，仅用来指定层次化实体结构
  - sequence-bank-size：序列号步长
  - enable-lock：是否在这个实体上使用优化锁
- 子元素：
  - description：说明
  - field：字段
  - prim-key：主键
  - relation：关系
  - copyright：版权
  - index：索引

### Apache OFBiz--服务引擎(Service Engine)

#### 介绍

服务引擎是OFBiz的另一个核心组件,OFBiz只有这两个核心组件,实体引擎代表业务数据,而服务引擎代表了业务逻辑

引入服务引擎的另一个价值是,它是的OFBiz业务框架不限于Web应用,非Web的客户端包括Java应用,EJB,都可以直接调用,这样,框架的可扩展性非常好

服务引擎的服务可以分为同步,异步(关心还是忽略结果),支持JMS,具体实现方式可以是一个Java静态方法,工作流,Bean Shell脚本等

服务框架是 OFBiz 2.0 新增加的功能。服务定义为一段独立的逻辑程序，当多个服务组合在一起时可完成不同类型的业务需求。服务有很多类型：Workflow, Rules, Java, SOAP, BeanShell等。

Java 类型的服务更像一个静态方法实现的事件，然而使用服务框架就不会局限在 Web 应用程序中。服务需要使用 Map 传入参数，结果同样从 Map 中返回。这样很妙，因为 Map 可以被序列化并保存或者通过HTTP(SOAP)传输。

服务通过服务定义来定义并指派给具体的服务引擎。每个服务引擎 通过适当的方式负责调用服务定义。因为服务没有和 web应用程序帮定在一起，就允许在没有响应对象可用时仍可以执行。这就允许在指定时间由 工作调度程序 在后台调用服务。服务能调用其他服务。因此，将多个小的服务串联起来实现一个大的任务使重用更容易。

在不同应用程序中使用的服务可以通过创建全局服务定义文件(只能创建一个)或者一个应用程序的特定服务(这样的服务受限制且只能用于这个应用程序)。当在 web 应用程序中使用时，服务可以用于 web 事件，这允许时间在服务框架中(stay small?) 并重用现成的逻辑。同样，服务可以定义成 'exportable'，允许外部程序访问。目前，SOAP EventHandler 允许服务通过 SOAP 来产生。其他形式的远程访问将来会加入到框架中

#### 服务定义

		服务定义在服务定义文件中。有全局(global )定义文件，所有服务派遣者都可以调用，同时也有只和单一服务派遣者相关联单独服务定义文件。当 LocalDispatcher 被创建，他会传递指向服务定义文件的 Arils 的一个集合。这些文件由 XML 写成，并定义了调用一个服务的必须信息。
	
		服务定义有一个唯一名字，明确的服务引擎名，明确定义的输入输出参数。

```xml
<service name="userLogin" engine="java"
 location="org.ofbiz.commonapp.security.login.LoginServices" invoke="userLogin">
 <description>Authenticate a username/password; create a UserLogin object</description>
 <attribute name="login.username" type="String" mode="IN"/>
 <attribute name="login.password" type="String" mode="IN"/>
 <attribute name="userLogin" type="org.ofbiz.core.entity.GenericValue" mode="OUT" 
optional="true"/>
</service>
```

#### Service 元素 

- name  : 服务唯一的名字
- engine : 服务引擎的名字 
- location : 服务类的包或其他位置
- invoke : 服务的方法名
- auth : 服务是否需要通过验证(true/false  默认false)
- export : 是否通过SOAP/HTTP/JMS(true/false  默认false) 访问
- validate :  是否对下面属性的名字和类型进行验证(true/false  默认true) 

##### **Implements 元素**

- sevice  : 这个服务实现的服务的名字,所有属性都被继承

##### Attribute 元素

- name : 这个属性的名字
- type : 这个对象的类型(String,java.utle.Data等)
- mode : 这个参数是输入,输出或输入输出类型 (IN/OUT/ INOUT)
- optional : 是否可选(true/false  默认false)

> 由上面面可以看出服务名是 userLogin，使用 *java* 引擎。这个服务需要两个必须的输入参数：
>
> *login.username* 和 *login.password*。必须的参数在服务调用之前会做检验。如果参数和名字及对
>
> 象类型不符服务就不会被调用。参数是否应该传给服务定义为 **optional**。服务调用后，输出参
>
> 数也被检验。只有需要的参数被检验，但是，如果传递了一个没有定义为可选的参数或者必须
>
> 的参数没有通过校验，将会导致服务失败。这个服务没有要求输出参数，因此只是简单返回



### Apache OFBiz--工作流引擎

#### 介绍

			OFBiz 工作流引擎基于 WfMC 和 OMG 规范。它是服务框架的成员之一，与 EntityEngine 紧密集成。工作流引擎把 entitymodel_workflow.XML 文件找到的实体用作定义信息，而把 entitymode_workeffort 文件找到的实体用作运行时刻存储。
	
		一个流程或任务（activity）都是实时的。因此，工作流引擎不是运行在一个线程上，而只是简单的一组 API 和通用对象在处理流程。当工作流发生改变时，引擎马上就处理这个变化，处理结束后，引擎返回结果。因此，当一个应用垮了（或系统重启），重启时工作流接着从停下的位置继续执行。
	
		工作流引擎不是为一个 web 站点的处理流程而设计的。这是一个普遍的错误概念。web 站点的流转由控制 Servlet 处理。工作流是为了达到一个目标而进行的手动和自动任务（activitie）处理。
	
		OFBiz工作流引擎把 XPDL 作为自己的流程定义语言。这是一个开放的标准，十分灵活。在 XPDL规范没有明确或留给厂商实现的地方，我们在 XPDL 扩展节说明。























### 消息引擎

### 

### 规则引擎