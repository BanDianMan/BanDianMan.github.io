---
title: "ApacheOFBiz  初级指南"
date: 2020-06-11T15:40:51+08:00
draft: true
---

## Apache OFBiz
---
### 介绍
Apache OFBiz是一套业务应用程序，足够灵活，可跨任何行业使用。通用体系结构允许开发人员轻松扩展或增强它以创建自定义功能。OFBiz作为Apache的一个开源项目，从2002年自今已有17年的历史，它每年都会发布一个大版本，中途更新多个小版本它是一个跨平台、跨数据库、跨应用服务器的分布式电子商务类web应用框架，包括实体引擎, 服务引擎, 消息引擎, 工作流引擎, 规则引擎等

### 特性
- 强大的JavaWeb框架
> OFBiz 是基于 Java 的 Web 框架，包括实体引擎、服务引擎和基于小部件的 UI，允许您快速原型设计和开发 Web 应用程序。
- 成熟的 CRM 和 ERP 解决方案
> 作为 Apache 顶级项目 10 年，OFBiz 已将其作为企业范围 ERP 解决方案的稳定性和成熟度，能够灵活应对您的业务进行更改。
- 开发人员友好
> OFBiz 架构非常灵活，开发人员能够使用自定义功能快速轻松地扩展和增强框架。
### 下载安装
可以采用zip解压的方式进行开发[下载地址](https://ofbiz.apache.org/download.html),也可以从源码仓库clone源码,[参考地址](https://ofbiz.apache.org/developers.html)

### 自述文件
 欢迎光临ApacheOFBiz一个强大的顶级Apache软件项目。OFBiz是一个用Java编写的企业资源计划(ERP)系统，包含大量的库、实体、服务和特性，以运行业务的各个方面。
 - [OFBiz文档](https://ofbiz.apache.org/index.html)
 - [OFBiz许可证](http://www.apache.org/licenses/LICENSE-2.0)

#### 系统需求
运行OFBiz的唯一要求是在您的系统上安装了Java Development Kit(JDK) version 8(不仅仅是JRE,而是完整的JDK),您可以从下面的链接下载
- [JDK下载](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
> 注意 : 如果您正在使用Eclipse,请确保在使用Eclipse创建项目之前运行适当的Eclipse命令gradlew Eclipse,此命令将通过创建类路径和.project文件
#### 安全
您可以信任OFBiz项目管理委员会的成员和提交者，他们尽最大努力保护OFBiz不受外部攻击，并在发现漏洞后尽快修复。尽管做出了这些努力，但如果您发现并想要报告某个安全问题，请在公开论坛上公布之前在:security @ ofbiz.apache.org 上报告。
> 注意:如果您计划使用RMI,JNDI,JMX或Spring,或者其他Java类OFBiz没有使用开箱即用(Out Of The Box,OOTB):臭名昭著的Java序列号漏洞,[请务必阅读这个Wiki页面](https://cwiki.apache.org/confluence/display/OFBIZ/The+infamous+Java+serialization+vulnerability)

你可以在这里找到更多关于[金融安全的信息](https://cwiki.apache.org/confluence/display/OFBIZ/Keeping+OFBiz+secure)

#### 快速启动
要快速安装和启动OFBiz,请遵循OFBiz顶级目录(文件夹)的命令行中的下列指示

#### 准备OFBiz
注意:根据您的Internet连接速度,如果您第一此使用OFBiz,可能需要很长时间才能完成此步骤,因为它需要下载所有的依赖项,所以请耐心等待!

- MS Windows 
```
gradlew cleanAll loadDefault
```
- unix操作系统
```
./gradlew cleanAll loadDefault
```

#### 开始OFBiz
- MS Windows 
```
gradlew  ofbiz
```
- unix操作系统
```
./gradlew  ofbiz
```
#### 通过浏览器浏览OFBiz
- [订单后台](https://localhost:8443/ordermgr/control/main)
- [会计后台](https://localhost:8443/accounting/control/main)
- [管理员后台](https://localhost:8443/webtools/control/main)

可以使用用户名: **admin** 密码:**ofbiz**登录

> 注意:默认配置使用嵌入式Java数据库(Apache Derby)和嵌入式应用服务器组件,如Apache Tomcat,Apache Geronimo(事务管理器)等

#### 构建系统的语法
所有的构建任务都是使用嵌入式OFBiz中的Gradle构建系统来执行的,要执行构建任务,请转到OFBiz顶级目录(文件夹)并从那里执行任务

#### 操作系统的语法
在windows和类unix系统之间，任务的语法略有不同
- Windows
```
gradlew <tasks-in-here>
```
- Unix-like
```
./gradlew <tasks-in-here>
```

> 对于本文的其余部分，我们将使用windows语法，如果您使用的是类unix系统，则需要将./添加到gradlew

#### Gradle中的任务类型
Gradle中为OFBiz设计的任务有两种：
- Standard tasks(标准任务) : 执行常规的标准Gradle任务
- OFBiz server tasks(OFBiz服务器任务) : 执行OFBiz启动命令。这些任务以下列单词之一开头
    - **ofbiz**:标准服务器命令
    - **ofbizDebug**:在远程调试模式下运行的服务器命令
    - **ofbizBackground**:在后台分支进程中运行的服务器命令

注意 :
- OFBiz服务器命令需要""命令 例如 : `gradlew "ofbiz -help"`
- 可以通过在任务名称中写入每个单词的第一个字母来使用任务名称的快捷方式。但是，您不能将快捷方式表单用于OFBiz服务器任务。
    - 例如：`gradlew loadAdminUserLogin -PuserLoginId = myadmin` =`gradlew AUL -PuserLoginId = myadmin`

#### 标准任务示例
```
gradlew build
```
```
gradlew cleanAll loadDefault testIntegration
```
#### OFBiz服务器任务示例
```
gradlew "ofbiz --help"
```

```
gradlew "ofbizDebug --test
```

```
gradlew "ofbizBackground --start --portoffset 10000"
```

```
gradlew "ofbiz --shutdown --portoffset 10000"
```

```
gradlew ofbiz (默认为开启)
```
#### 混合任务示例（标准服务器和OFBiz服务器）
```
gradlew cleanAll loadDefault "ofbiz --start"
```
---
#### 快速参考
您可以使用下面的常见任务列表作为控制系统的快速参考。本文档使用Windows任务语法，如果您使用的是类Unix系统，则需要在gradlew中添加`./`，即`./gradlew`。

#### 帮助任务

- 列出OFBiz服务器命令
```
gradlew "ofbiz --help"
```

#### 列出构建任务
- 列出构建系统中的所有可用任务
```
gradlew tasks
```
#### 列出构建项目
- 列出构建系统中的所有可用项目
```
gradlew projects
```

#### Gradle构建系统帮助
- 显示Gradle构建系统的用法和选项
```
gradlew --help
```
#### 服务器命令任务

- 开始OFBiz
```
gradlew "ofbiz --start"
```

> start是默认的服务器任务，因此它也可以工作

```
gradlew ofbiz
```

- 停止OFBiz
```
gradlew "ofbiz --shutdown"
```

- 获取OFBiz状态
```
gradlew "ofbiz --status"
```

#### 强制关闭OFBiz
通过调用适当的操作系统kill命令来终止所有正在运行的OFBiz服务器实例。如果--shutdown命令不起作用，请使用此命令强制终止OFBiz。通常在OFBiz中进行数据加载或测试时需要这样做。

> 警告：请谨慎使用此命令，因为强制终止可能会导致状态/数据不一致

```
gradlew terminateOfbiz
```

#### 在远程调试模式下启动OFBiz
在远程调试模式下启动OFBiz,等待调试器或IDE在端口5005上进行连接
```
gradlew "ofbizDebug --start"
```
或
```
gradlew ofbizDebug
```
#### 在其他端口上启动OFBiz
由--portoffset参数提供的范围偏移的网络端口的起始OFBiz
```
gradlew "ofbiz --start --portoffset 10000"
```

#### 在后台启动OFBiz
在后台启动OFBiz，方法是将其分支到新进程，然后将输出重定向到 **runtime/logs/console.log**
```
gradlew "ofbizBackground --start"
```
或
```
gradlew ofbizBackground
```
您还可以偏移端口，例如 :
```
gradlew "ofbizBackground --start --portoffset 10000"
```
#### 将JVM运行时选项传递给OFBiz
您可以通过指定项目属性`-PjvmArgs`来传递JVM运行时选项。
```
gradlew ofbiz -PjvmArgs="-Xms1024M -Xmx2048M" -Dsome.parameter=hello
```
如果您未指定`jvmArgs`，则默认设置-Xms128M -Xmx1024M

#### 数据加载任务
OFBiz包含以下数据读取器类型:
- seed : OFBiz和外部种子数据-与源一起维护，并在更新系统部署时进行更新
- seed-initial : OFBiz和外部种子数据-与其他种子数据一样与源一起维护，但仅在初始加载，并且在系统更新时不更新，除非手动检查每行
- demo : OFBiz Only演示数据
- ext : 外部常规数据（自定义）
- ext-test : 外部测试数据（自定义）
- ext-demo : 外部演示数据（自定义）

您可以通过以下语法选择要传递的数据读取器:
```
gradlew "ofbiz --load-data readers=<readers-here-comma-separated>"
```
例如 :
```
gradlew "ofbiz --load-data readers=seed,seed-initial,ext,ext-demo"
```
#### 加载默认的OFBiz数据
加载默认数据集；用于初始加载通用OFBiz数据。可以用于开发，测试，演示等目的。请注意，执行此任务可能会导致您选择的数据库中的数据被覆盖。在生产环境中请谨慎使用。默认数据集由数据源使用read-data属性定义，后跟数据集的名称，定义为'entityengine.xml'文件的datasource元素
```
gradlew loadDefault
```
或
```
gradlew "ofbiz --load-data"
```
#### 加载种子数据
仅加载种子数据（不加载种子初始，演示，ext *或其他任何内容）；意味着在更新代码后使用以重新加载种子数据，因为种子数据通常与代码一起维护，并且需要同步操作
```
gradlew "ofbiz --load-data readers=seed"
```
#### 加载外部数据
加载种子，种子初始和外部数据；用于手动/通用测试，开发或使用基于库存OFBiz的派生系统进行生产，其中ext数据基本上替代了演示数据
```
gradlew "ofbiz --load-data readers=seed,seed-initial,ext"
```
#### 加载外部测试数据
从保存实体数据的XML文件中加载数据
```
gradlew "ofbiz --load-data file=foo/bar/FileNameHere.xml"
```
#### 创建一个新的租户
在您的环境中创建一个新的租户，创建委托者，使用admin-user和password加载初始数据（在general.properties中需要multitenant = Y）。传递了以下项目参数：
- tenantId: 强制性的
- tenantName: 可选，默认值为tenantId的值
- domainName: 可选，默认为org.apache.ofbiz
- tenantReaders: 可选，默认值为seed，seed-initial，demo
- dbPlatform:可选，D（Derby），M（MySQL），O（Oracle），P（PostgreSQL）（默认D）
- dbIp: 可选的数据库的IP地址
- dbUser: 可选，数据库的用户名
- dbPassword: 可选，数据库密码

```
gradlew createTenant -PtenantId=mytenant
```
```
gradlew createTenant -PtenantId=mytenant -PtenantName="My Name" -PdomainName=com.example -PtenantReaders=seed,seed-initial,ext -PdbPlatform=M -PdbIp=127.0.0.1 -PdbUser=mydbuser -PdbPassword=mydbpass
```
如果成功运行，系统将创建一个新的租户，其具有：
- delegator: default#${tenandId} (e.g. default#mytenant)
- admin user: ${tenantId}-admin (e.g. mytenant-admin)
- admin user password: ofbiz
#### 加载特定租户的数据
在多租户环境中加载一个特定租户的数据。请注意，必须在general.properties中设置multitenant = Y，并传递以下项目参数：
- tenantId (强制性)
- tenantReaders (可选)
- tenantComponent (可选)
```
gradlew loadTenant -PtenantId=mytenant
```
```
gradlew loadTenant -PtenantId=mytenant -PtenantReaders=seed,seed-initial,demo -PtenantComponent=base
```
#### 测试任务
- 执行所有单元测试
```
gradlew test
```

- 执行所有集成测试
```
gradlew testIntegration
```
或
```
gradlew 'ofbiz --test'
```
- 执行集成测试用例
运行一个测试用例，在此示例中，组件为“entity”，而案例名称为“entity-tests”
```
gradlew "ofbiz --test component=entity --test case=entity-tests"
```
- 在调试模式下执行集成测试用例

在端口5005上监听
```
gradlew "ofbizDebug --test component=entity --test case=entity-tests"
```
- 执行集成测试套件
```
gradlew "ofbiz --test component=widget --test suitename=org.apache.ofbiz.widget.test.WidgetMacroLibraryTests"
```
- 在调试模式下执行集成测试套件

在端口5005上监听
```
gradlew "ofbizDebug --test component=widget --test suitename=org.apache.ofbiz.widget.test.WidgetMacroLibraryTests"
```
#### 杂项任务
- 启动Gradle的图形用户界面

这是Gradle的一个非常方便的功能，它允许用户通过摆动GUI与Gradle进行交互。您可以将常用命令保存在收藏夹列表中，以便经常重复使用
```
gradlew --gui
```
- 在干净的系统上运行所有测试
```
gradlew cleanAll loadDefault testIntegration
```
- 清理所有生成的文件
```
gradlew cleanAll
```
- 刷新生成的文件
```
gradlew clean build
```
- 创建一个管理员用户帐户

创建一个登录名为myusername的管理员用户，其默认密码为“ ofbiz”。首次登录后，biz将要求更改默认密码
```
gradlew loadAdminUserLogin -PuserLoginId=MyUserName
```
- 使用Xlint输出编译Java
```
gradlew -PXlint build
```
- 运行OWASP工具以识别依赖项漏洞（CVE）

下面的命令激活gradle插件（OWASP）并标识和报告OFBiz库依赖项中的已知漏洞（CVE）。该命令执行时间很长，因为它需要下载所有插件依赖项，并且CVE识别过程也很耗时
```
gradlew -PenableOwasp dependencyCheck
```
#### 为OFBiz设置Eclipse项目
由于有了gradle魔术，在eclipse上设置OFBiz非常容易。您所需要做的就是执行一个命令，然后可以将项目导入到Eclipse中。此命令将为Eclipse生成必要的 **.classpath** 和 **.project**文件。
```
gradlew eclipse
```
#### OFBiz插件系统
OFBiz通过插件提供了扩展机制。插件是驻留在特殊目的目录中的标准OFBiz组件。可以手动添加插件，也可以从Maven存储库获取插件。下面列出了用于管理插件的标准任务。

> 注意 : OFBiz插件版本遵循语义版本[2.0.0](https://semver.org/)
#### 自动拉（下载并安装）插件

下载具有所有依赖项（插件）的插件，并从依赖项开始到插件本身一一安装。

```
gradlew pullPlugin -PdependencyId="org.apache.ofbiz.plugin:myplugin:0.1.0"
```
如果插件位于自定义Maven存储库中（而不是jcenter或localhost），则可以使用以下命令指定存储库：

```
gradlew pullPlugin -PrepoUrl="http://www.example.com/custom-maven" -PdependencyId="org.apache.ofbiz.plugin:myplugin:0.1.0"
```
如果您需要用户名和密码来访问定制存储库：
- 如果压缩后解压缩插件
- 将提取的目录放入 **/specialpurpose**
- 运行以下命令
```
gradlew installPlugin -PpluginId=myplugin
```
- 上面的命令实现以下目的
    - 将插件添加到/specialpurpose/component-load.xml
    - 如果存在插件，则在插件的build.gradle文件中执行任务“install”
#### 卸载插件
如果您有一个名为mycustomplugin的现有插件，并且希望卸载，请运行以下命令
```
gradlew uninstallPlugin -PpluginId=myplugin
```
- 上面的命令实现以下目的
    - 如果存在插件，则在插件的build.gradle文件中执行任务“uninstall”
    - 从/specialpurpose/component-load.xml中删除插件
#### 删除插件
在现有插件上调用**uninstallPlugin**，然后将其从文件系统中删除
```
gradlew removePlugin -PpluginId=myplugin
```
#### 创建一个新的插件
创建一个新的插件。传递了以下项目参数：
- pluginId: 强制性的
- pluginResourceName: 可选,默认为pluginId的大写值
- webappName: 可选, 默认是pluginId的值
- basePermission: 可选,默认值为pluginId的UPPERCASE值
```
gradlew createPlugin -PpluginId=myplugin
```
```
gradlew createPlugin -PpluginId=myplugin -PpluginResourceName=MyPlugin -PwebappName=mypluginweb -PbasePermission=MYSECURITY
```
- 上面的命令实现了以下目的
    - 在/specialpurpose/myplugin中创建一个新插件
    - 将插件添加到/specialpurpose/component-load.xml
    
#### 将插件推送到存储库
此任务将OFBiz插件发布到maven包中，然后将其上传到maven存储库。当前，推送仅限于本地主机maven存储库（正在进行中）。要推送插件，需要传递以下参数：
- pluginId: 强制性
- groupId: 可选, 默认为org.apache.ofbiz.plugin
- pluginVersion: 可选,默认为0.1.0-SNAPSHOT
- pluginDescription: 可选, 默认为 to "Publication of OFBiz plugin ${pluginId}"
```
gradlew pushPlugin -PpluginId=myplugin
```
```
gradlew pushPlugin -PpluginId=mycompany -PpluginGroup=com.mycompany.ofbiz.plugin -PpluginVersion=1.2.3 -PpluginDescription="Introduce special functionality X"
```