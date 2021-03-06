---
title: "ApacheNiFi介绍"
date: 2020-05-06T15:23:58+08:00
draft: true
---

## Apache NiFi基础及架构
---
![image-20200506152500071](https://i.loli.net/2020/05/06/1WKwck7239SrXHe.png)
### Apache NiFi介绍 
- Apache NiFi是一个易于使用，功能强大且可靠的系统，可通过编排和执行数据流来处理和分发数据
    - Apache NiFiis使系统之间的数据流自动化
    - 基于Web的用户界面
    - 高度可配置
        - 容忍损失与保证交付
        - 低延迟与高吞吐量 
    - 资料来源
        - 从头到尾跟踪数据流
    - 专为扩展而设计
        - 构建定制处理器
    - 安全
        - SSL，SSH，HTTPS，加密内容等
### NiFi子项目-MiNiFi
- MiNiFi是一种补充性的数据收集方法，是对NiFiin数据流管理的核心原则的补充，侧重于在创建源时收集数据
    - NiFilives位于数据中心中-为它提供企业服务器或它们的集群
    - MiNiFiLive距离数据出生地很近，并且是该设备或系统上的访客
### NiFi功能
- 保证交付
- 数据缓冲
- 优先排队
- 流特定的QoS
    - 延迟与吞吐量
    - 损耗容限
- 资料来源
- 支持推拉模型
- 视觉命令与控制
- 流模板
- 可插拔/多角色安全
- 专为扩展而设计
    - 丰富的内置处理器
    - 广泛的支持和3rdcustom处理器
- 聚类
### 核心概念
- FlowFile – 表示在系统中移动的每个对象
- Proessor(处理器) – 执行实际工作，例如系统之间的数据路由，转换或中介
    - 处理器可以访问给定FlowFile及其内容流的属性
    - 处理器在给定单元中进行提交或回滚工作
- Connection(连接) – 通过充当队列来提供处理器之间的实际链接
- Flow Controller(流控制器)–充当代理，促进按计划在处理器之间交换FlowFiles
- Process Group(流程组) – 是一组特定的流程及其联系
    - 允许创建可重复使用的组件
### Apache NiFi架构
![image-20200506152510976](https://i.loli.net/2020/05/06/wVpPA5FsOJ9a8Zu.png)
### Apache NiFi集群
![image-20200506152530273](https://i.loli.net/2020/05/06/tYpyBf7ZzC85Sjq.png)
- 零主集群
- 每个节点对数据执行相同的任务，但对不同的数据集执行相同的任务
- 管理员选择了一个集群协调员，负责协调集群中各节点之间的连接性。
### NiFi Site-to-Site
- 两个NiFi instances之间的直接通信
- 推到接收器上的输入端口，或从信源上的输出端口拉
- 处理负载平衡和可靠的交付
- 使用证书的安全连接（可选）
### Site-to-Site Push
- 源将远程进程组连接到目标上的输入端口
- Site-to-Site 负责群集中各个节点之间的负载平衡
### Site-to-Site Pull
- 目标将远程进程组连接到源上的输出端口
- 如果源是集群，则每个节点将从群集中的每个节点中提取
### Site-to-Site Client
> 任何要推动或退出NiFi的客户
> ![image-20200506152538942](https://i.loli.net/2020/05/06/DyNnumOHAhxUeKs.png)
### Flow File
- FlowFile表示在系统中移动的每个对象
    - Header:  key/value 对属性
    - Content: 零个或多个字节
### FlowFileProcessor
- 处理器实际执行工作:
    - 操作FlowFilecontent
        - 数据路由，数据转换等
    - 提交或回滚工作
    - 使用/添加/更新属性
### Connection
- 连接提供了处理器之间的实际联系
    - 这些充当队列，并允许各种进程以不同的速率进行交互
    - 可以动态地对这些队列进行优先级排序，并可以在负载上设置上限，从而实现Back Pressure
        - Back Pressure是指在不再计划运行作为连接源的组件之前，应允许队列中存在多少数据
        - NiFi提供了两个Back Pressure配置元素
            - FolwFiles数
            - 数据大小
### Flow Controller
- NiFi dataflow调度程序和执行引擎:
    - 维护有关进程如何连接和管理所有进程使用的线程及其分配的知识
    - 充当代理，促进处理器之间的FlowFile交换
### Process Group
- 流程组是一组特定的流程及其连接
    - 通过输入端口接收数据，并通过输出端口发送数据
    - 流程组允许仅通过组合其他组件来创建全新的组件
        - 输入口
        - 输出口
### Processors(处理器)
- 数据提取处理器
    - ListSFTP, FetchSFTP, ListFiles, FetchFiles, GetFile, ListHDFS, FetchHDFS, ... etc
- 路由和中介处理器
    - 路由和中介处理器
- 数据库访问处理器
    - ExecuteSQL，PutSQL，ListDatabaseTables等
- 属性提取处理器
    - UpdateAttribute，EvaluateJSONPath，ExtractText等
- 系统交互处理器
    - ExecuteScript，ExecuteProcess，ExecuteStreamCommand等
- 数据转换处理器
    - ReplaceText，SplitText，MergeContent，...
- 发送数据处理器
    - PutSFTP, PutFile, PutHDFS, PublishKafka, ConsumeKafka, ...
### 处理器关系
- 流文件通过使用处理器之间的关系进行验证的连接从一个处理器移动到另一个处理器
    - 每当建立连接时，开发人员都会在这些处理器之间选择一个或多个关系
    - SUCCESS
        - 流文件成功完成流程后移至下一个流程
    - FAILURE
        - 流文件在失败的流程后移至下一个流程
### DEMO
...