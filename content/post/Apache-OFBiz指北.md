---
title: "ApacheOFBiz指北"
date: 2020-08-11T16:34:36+08:00
draft: true
---

## OFBiz进阶学习

### 1.准备

#### 1.1环境及工具

- Intellij IDEA 2020.1
- JDK1.8.xx
- MySQL5

- OFBIZ版本 (当前发布最新版17.12.04-发布于2020-07-13) 这里使用16.11.05 [下载链接](https://archive.apache.org/dist/ofbiz/apache-ofbiz-16.11.05.zip)

### 2.下载 Apache OfBiz 框架

[下载链接](https://archive.apache.org/dist/ofbiz/apache-ofbiz-16.11.05.zip)

- 下载之后解压

![image-20200727113101448](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727113101448.png)

- Apache OFBiz 自带的数据库为 [Derby](http://db.apache.org/derby/)  后续可以配置成MySQL数据库

### 3.运行 Apache OFBiz

```shell
gradlew cleanAll loadDefault ofbiz
```

以上命令将加载演示数据，（示例数据运行应用程序）

![image-20200727113617571](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727113617571.png)

- [浏览器]( https://localhost:8443/webtools)上打开
  - 用户名 : admin
  - 密码 : ofbiz

### 4.创建一个应用程序

OFBiz 组件是一个文件夹，其中包含一个名为"ofbiz-组件.xml"的特殊 xml 文件，用于描述组件要加载和所需的资源。

OFBiz 本身就是一组组件。

- **框架组件：**这些是为应用程序组件提供技术层和工具的较低级别的组件;这些组件提供的功能通常是任何其他开发框架（数据层、业务逻辑层、事务处理、数据源池等）提供的功能。
- **应用程序组件：**这些是 ERP 应用程序所需的通用业务组件，可以扩展/定制（产品、订单、一方、制造、会计等）;应用程序组件可以访问框架组件提供的服务和工具以及其他应用程序组件发布的服务。
- **特殊用途组件：这些**组件类似于应用程序组件，但用于特殊用途的应用程序，如电子商务，谷歌基础集成，eBay

#### 4.1创建插件/组件

使用命令行设置新的自定义组件

```shell
./gradlew createPlugin -PpluginId=ofbizDemo

```

![image-20200727114838915](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727114838915.png)

#### 4.2运行应用程序

- 在运行第一个组件之前，显示“Hello World”

1. 打开  $OFBIZ_HOME/specialpurpose/ofbizDemo/widget/OfbizDemoScreens.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<screens xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://ofbiz.apache.org/Widget-Screen" xsi:schemaLocation="http://ofbiz.apache.org/Widget-Screen http://ofbiz.apache.org/dtds/widget-screen.xsd">

    <screen name="main">
        <section>
            <actions>
                <set field="headerItem" value="main"/><!-- this highlights the selected menu-item with name "main" -->
            </actions>
            <widgets>
                <decorator-screen name="OfbizDemoCommonDecorator" location="${parameters.mainDecoratorLocation}">
                    <decorator-section name="body">
                        <label text="Hello World!! :)"></label>
                    </decorator-section>
                </decorator-screen>
            </widgets>
        </section>
    </screen>
</screens>
```

- 只添加了 `<label text =“ Hello World !! :)” /></label>`

2. 现在将需要通过重新加载数据 `./gradlew loadDefault ofbiz`来重新启动OFBiz。 这是必需的，因为您已经为组件创建了一些带有一些安全性数据的组件（默认情况下，在组件数据目录中将其设置为OfbizDemoSecurityGroupDemoData.xml），并且在重新启动它时，ofbizdemo组件也会被加载

3. 当OFBiz重新启动时，将浏览器定向到此处的应用程序中 https://localhost:8443/ofbizDemo

4. 您将被要求登录。 使用用户登录：admin 密码：ofbiz。

5. 登录时，您将看到ofbizdemo应用程序，并在屏幕上显示了hello world消息，如下图所示。就是这样，祝贺您的第一个组件已安装并正在运行。

- 我这里进去报错 应该是国际化的问题
  - ![image-20200727122158832](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727122158832.png)

- 解决办法 将apache-ofbiz-16.11.05\specialpurpose\ofbizDemo\widget\CommonScreens.xml 中的**OfbizDemoUiLabels**这行删除
  - ![image-20200727122827605](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727122827605.png)

- 进去再次报错 原因  "元素类型“label”必须以匹配的结束标签“</label>”结束。"
  - ![image-20200727123045492](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727123045492.png)

- 最后加上 label标签 成功进入
  - ![image-20200727123212006](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727123212006.png)
  - ![image-20200727123203985](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727124448015.png)

#### 4.3创建第一个数据库实体（表）

> 定义实体

​	要在数据库中创建自定义实体/表，只需要在ofbizdemo应用程序的$ OFBIZ_HOME / specialpurpose / ofbizDemo / entitydef / entitymodel.xml文件中提供实体定义。 使用Gradle任务设置组件时，已经设置了此文件结构。 只需要进入并提供实体定义，如下所示。 在这里，我们将为ofbizdemo应用程序添加两个新实体。

![image-20200727124128051](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727124742342.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<entitymodel xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://ofbiz.apache.org/dtds/entitymodel.xsd">
    <!-- ========================================================= -->
    <!-- ======================== Defaults ======================= -->
    <!-- ========================================================= -->
    <title>Entity of OfbizDemo Component</title>
    <description>None</description>
    <copyright></copyright>
    <version></version>

    <entity entity-name="OfbizDemoType" package-name="org.apache.ofbiz.ofbizdemo" title="OfbizDemo Type Entity">
        <field name="ofbizDemoTypeId" type="id"><description>primary sequenced ID</description></field>
        <field name="description" type="description"></field>
        <prim-key field="ofbizDemoTypeId"/>
    </entity>

    <entity entity-name="OfbizDemo" package-name="org.apache.ofbiz.ofbizdemo" title="OfbizDemo Entity">
        <field name="ofbizDemoId" type="id"><description>primary sequenced ID</description></field>
        <field name="ofbizDemoTypeId" type="id"></field>
        <field name="firstName" type="name"></field>
        <field name="lastName" type="name"></field>
        <field name="comments" type="comment"></field>
        <prim-key field="ofbizDemoId"/>
        <relation type="one" fk-name="ODEM_OD_TYPE_ID" rel-entity-name="OfbizDemoType">
            <key-map field-name="ofbizDemoTypeId"/>
        </relation>
    </entity>
    
</entitymodel>
```

现在看一下$ OFBIZ_HOME / specialpurpose / ofbizDemo / ofbiz-component.xml文件。 已经在其中进行了资源输入，以便在加载组件时将这些实体从其定义加载到数据库。 如下所示：

```xml
<entity-resource type="model" reader-name="main" loader="main" location="entitydef/entitymodel.xml"/>
```

![image-20200727124448015](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727133535455.png)

要检查，只需重新启动OFBiz（Ctrl + C后跟“ ./gradlew ofbiz”），然后将浏览器定向到https://localhost:8443/webtools/control/entitymaint 并搜索OfbizDemoType和OfbizDemo实体。 您将看到它，如下图所示

![image-20200727124742342](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727124128051.png)

#### 4.4为实体准备数据

设置自定义实体后，现在该为它准备一些样本数据了。 您可以在组件的数据目录下已设置的数据XML文件中执行此操作，即$ OFBIZ_HOME / specialpurpose / ofbizDemo / data / **OfbizDemoTypeData.xml**和$ OFBIZ_HOME / specialpurpose / ofbizDemo / data / **OfbizDemoDemoData.xml**。 如下所示进行设置

> **OfbizDemoTypeData.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<entity-engine-xml>
    <OfbizDemoType ofbizDemoTypeId="INTERNAL" description="Internal Demo - Office"/>
    <OfbizDemoType ofbizDemoTypeId="EXTERNAL" description="External Demo - On Site"/>
</entity-engine-xml>
```

> **OfbizDemoDemoData.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<entity-engine-xml>
    <OfbizDemo ofbizDemoId="SAMPLE_DEMO_1" ofbizDemoTypeId="INTERNAL" firstName="Sample First 1" lastName="Sample Last 1" comments="This is test comment for first record."/>
    <OfbizDemo ofbizDemoId="SAMPLE_DEMO_2" ofbizDemoTypeId="INTERNAL" firstName="Sample First 2" lastName="Sample last 2" comments="This is test comment for second record."/>
    <OfbizDemo ofbizDemoId="SAMPLE_DEMO_3" ofbizDemoTypeId="EXTERNAL" firstName="Sample First 3" lastName="Sample last 3" comments="This is test comment for third record."/>
    <OfbizDemo ofbizDemoId="SAMPLE_DEMO_4" ofbizDemoTypeId="EXTERNAL" firstName="Sample First 4" lastName="Sample last 4" comments="This is test comment for fourth record."/>
</entity-engine-xml>
```

现在再来看看$ OFBIZ_HOME / specialpurpose / ofbizDemo / ofbiz-component.xml文件。 您已经在其中进行了资源输入，以加载以下文件中准备的数据：

> 在**ofbiz-component.xml**中可以看到

```xml
<entity-resource type="data" reader-name="seed" loader="main" location="data/OfbizDemoTypeData.xml"/>
<entity-resource type="data" reader-name="demo" loader="main" location="data/OfbizDemoDemoData.xml"/>
```

![image-20200727125448954](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727123203985.png)

#### 在实体中加载数据

方法一 : 此时要将样本数据加载到定义的实体/表中，您可以在控制台上运行`gradlew loadDefault`

方法二 : 可以直接在webtools中转到此处以加载实体xml https://localhost:8443/webtools/control/EntityImport 只需将xml数据放入“完整XML文档（根标签：entity-engine-xml）：”文本区域，然后单击“导入文本”，如下图所示

- ![image-20200727133535455](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727133549438.png)

- ![image-20200727133549438](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727125448954.png)

已经成功将数据导入数据库表中，非常简单

![image-20200727133658918](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727133925409.png)

### 5.表单和服务

在上面，已经了解了如何创建实体（表），现在该创建一个表单，该表单将允许您在该实体中进行输入。

#### 5.1创建服务

- 在准备表单之前，先编写一个服务以在服务定义xml文件（$ OFBIZ_HOME / specialpurpose / ofbizDemo / servicedef / services.xml）中为OfbizDemo实体在数据库中创建记录

![image-20200727133925409](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727134038953.png)

- 此时这里的服务默认有一个

![image-20200727134038953](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727133658918.png)

这个是虚拟服务，防止空文件和语法错误 - 在此处添加第一个实服务时删除

>**services.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<services xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="http://ofbiz.apache.org/dtds/services.xsd">
    <description>OfbizDemo Services</description>
    <vendor></vendor>
    <version>1.0</version>

    <service name="createOfbizDemo" default-entity-name="OfbizDemo" engine="entity-auto" invoke="create" auth="true">
        <description>Create an Ofbiz Demo record</description>
        <auto-attributes include="pk" mode="OUT" optional="false"/>
        <auto-attributes include="nonpk" mode="IN" optional="false"/>
        <override name="comments" optional="true"/>
    </service>
    
</services>
```

现在再次查看$ OFBIZ_HOME / specialpurpose / ofbizDemo / ofbiz-component.xml文件。 已经在其中输入了用于加载此文件中定义的服务的资源条目：

```xml
<!-- service resources: model(s), eca(s) and group definitions -->
<service-resource type="model" loader="main" location="servicedef/services.xml"/>
```

![image-20200727134242631](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727134242631.png)

要加载此服务定义，您将需要重新启动OFBiz。 

`gradlew ofbiz`

要测试此服务，您可以直接在此处转到webtools-> Run Service选项：https://localhost:8443/webtools/control/runService

![image-20200727134514467](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727135308083.png)

> 通过Web工具运行服务：框架提供的一种智能实用程序，用于运行您的服务。
>
> 提交上述表格后，您将展示一个表格，用于输入服务的IN参数。

- 输入服务名称 
  - ![image-20200727135308083](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727135425260.png)

- 填写参数
  - ![image-20200727135349142](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727134514467.png)
- 运行成功
  - ![image-20200727135256413](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727135256413.png)

- 数据已经添加上去了
  - ![image-20200727135425260](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727135349142.png)

#### 5.2UI国际化的使用（简介）

Apache OFBiz的国际化非常容易，用各种语言定义UI标签，并根据用户的语言环境显示相应的标签。

这是UI标签的示例（创建组件<component-name> UiLabels.xml是默认创建的，在我们的例子中是OfbizDemoUiLabels.xml）

> **OfbizdemouiLabels. xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<resource xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://ofbiz.apache.org/dtds/ofbiz-properties.xsd">
    <property key="OfbizDemoApplication">
        <value xml:lang="en">OfbizDemo Application</value>
        <value xml:lang="zh">OfbizDemo应用程�?</value>
        <value xml:lang="zh-TW">OfbizDemo應用程�?</value>
    </property>
    <property key="OfbizDemoCompanyName">
        <value xml:lang="en">OFBiz: OfbizDemo</value>
        <value xml:lang="zh-TW">OFBiz: OfbizDemo</value>
    </property>
    <property key="OfbizDemoCompanySubtitle">
        <value xml:lang="en">Part of the Apache OFBiz Family of Open Source Software</value>
        <value xml:lang="it">Un modulo della famiglia di software open source Apache OFBiz</value>
        <value xml:lang="zh">开�?软件OFBiz的组�?部分</value>
        <value xml:lang="zh-TW">開�?軟體OFBiz的組�?部分</value>
    </property>
    <property key="OfbizDemoViewPermissionError">
        <value xml:lang="en">You are not allowed to view this page.</value>
        <value xml:lang="zh">�?�?许你�?览这个页�?�。</value>
        <value xml:lang="zh-TW">�?�?許您檢視這個�?�?�.</value>
    </property>
</resource>
```

#### 5.3创建添加(插入数据)表单

让我们为此服务创建我们的第一个表单，为此,让我们编辑现有文件的位置 **$OFBIZ_HOME/specialpurpose/ofbizDemo/widget/OfbizDemoForms.xml**，并添加为 OfbizDemo 创建窗体，如下所示：

> **OfbizDemoForms.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<forms xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xmlns="http://ofbiz.apache.org/Widget-Form" xsi:schemaLocation="http://ofbiz.apache.org/Widget-Form http://ofbiz.apache.org/dtds/widget-form.xsd">

    <form name="AddOfbizDemo" type="single" target="createOfbizDemo">
        <!-- We have this utility in OFBiz to render form based on service definition.
             Service attributes will automatically lookedup and will be shown on form
        -->
        <auto-fields-service service-name="createOfbizDemo"/>
        <field name="ofbizDemoTypeId" title="${uiLabelMap.CommonType}">
            <drop-down allow-empty="false" current-description="">
                <!---We have made this drop down options dynamic(Values from db) using this -->
                <entity-options description="${description}" key-field-name="ofbizDemoTypeId" entity-name="OfbizDemoType">
                    <entity-order-by field-name="description"/>
                </entity-options>
            </drop-down>
        </field>
        <field name="submitButton" title="${uiLabelMap.CommonAdd}"><submit button-type="button"/></field>
    </form>
    
</forms>
```

- 在这里您可以注意到我们已经使用auto-fields-service根据服务定义IN / OUT属性自动生成表单

- 转到Screens xml文件（OfbizDemoScreens.xml），将装饰器主体中的此表单位置添加到用于显示Hello World ...文本的屏幕。 如下所示

> 在主屏幕上添加表格位置
>
> **OfbizDemoScreens.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<screens xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://ofbiz.apache.org/Widget-Screen" xsi:schemaLocation="http://ofbiz.apache.org/Widget-Screen http://ofbiz.apache.org/dtds/widget-screen.xsd">

    <screen name="main">
        <section>
            <actions>
                <set field="headerItem" value="main"/><!-- this highlights the selected menu-item with name "main" -->
            </actions>
            <widgets>
                <decorator-screen name="OfbizDemoCommonDecorator" location="${parameters.mainDecoratorLocation}">
                    <decorator-section name="body">
                        <screenlet title="Add Ofbiz Demo">
                            <include-form name="AddOfbizDemo" location="component://ofbizDemo/widget/OfbizDemoForms.xml"/>
                        </screenlet>
                    </decorator-section>
                </decorator-screen>
            </widgets>
        </section>
    </screen>

</screens>
```

#### 5.4表单的控制器输入

​	在转到表单并从添加表单开始创建OfbizDemo记录之前，您需要在$ OFBIZ_HOME / specialpurpose / ofbizDemo / webapp / ofbizDemo / WEB-INF / controller.xml文件中输入一个条目，以供目标服务调用 表格已提交。 您可以按照如下所示在Ofbizdemo应用程序控制器文件中的“请求映射”下进行操作：

```xml
    <request-map uri="createOfbizDemo">
        <security https="true" auth="true"/>
        <event type="service" invoke="createOfbizDemo"/>
        <response name="success" type="view" value="main"/>
    </request-map>
```

一切都设置好了，让我们看一下我们最近创建的表单https://localhost:8443/ofbizDemo

![image-20200727140458595](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727140458595.png)

主键（ofbizDemoId）不需要与表单一起发送，它将由OFBiz在数据库记录中自动排序。

- 输入参数后添加
  - ![image-20200727140818755](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727140850893.png)

- 可以看到数据已经添加进去了
  - ![image-20200727140850893](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727140818755.png)

#### 5.5创建查找(查询数据)表单

让我们为OfbizDemo实体创建查找表单，以便搜索正在创建的OfbizDemos。

1. 在OfbizDemoForms.xml中添加表单（FindOfbizDemo和ListOfbizDemo）

> **OfbizDemoForms.xml**

```xml
<form name="FindOfbizDemo" type="single" target="FindOfbizDemo" default-entity-name="OfbizDemo">
    <field name="noConditionFind"><hidden value="Y"/> <!-- if this isn't there then with all fields empty no query will be done --></field>
    <field name="ofbizDemoId" title="${uiLabelMap.OfbizDemoId}"><text-find/></field>
    <field name="firstName" title="${uiLabelMap.OfbizDemoFirstName}"><text-find/></field>
    <field name="lastName" title="${uiLabelMap.OfbizDemoLastName}"><text-find/></field>
    <field name="ofbizDemoTypeId" title="${uiLabelMap.OfbizDemoType}">
        <drop-down allow-empty="true" current-description="">
            <entity-options description="${description}" key-field-name="ofbizDemoTypeId" entity-name="OfbizDemoType">
                <entity-order-by field-name="description"/>
            </entity-options>
        </drop-down>
    </field>
    <field name="searchButton" title="${uiLabelMap.CommonFind}" widget-style="smallSubmit"><submit button-type="button" image-location="/images/icons/magnifier.png"/></field>
</form>
  
<form name="ListOfbizDemo" type="list" list-name="listIt" paginate-target="FindOfbizDemo" default-entity-name="OfbizDemo" separate-columns="true"
    odd-row-style="alternate-row" header-row-style="header-row-2" default-table-style="basic-table hover-bar">
    <actions>
       <!-- Preparing search results for user query by using OFBiz stock service to perform find operations on a single entity or view entity -->
       <service service-name="performFind" result-map="result" result-map-list="listIt">
           <field-map field-name="inputFields" from-field="ofbizDemoCtx"/>
           <field-map field-name="entityName" value="OfbizDemo"/>
           <field-map field-name="orderBy" from-field="parameters.sortField"/>
           <field-map field-name="viewIndex" from-field="viewIndex"/>
           <field-map field-name="viewSize" from-field="viewSize"/>
        </service>
    </actions>
    <field name="ofbizDemoId" title="${uiLabelMap.OfbizDemoId}"><display/></field>
    <field name="ofbizDemoTypeId" title="${uiLabelMap.OfbizDemoType}"><display-entity entity-name="OfbizDemoType"/></field>
    <field name="firstName" title="${uiLabelMap.OfbizDemoFirstName}" sort-field="true"><display/></field>
    <field name="lastName" title="${uiLabelMap.OfbizDemoLastName}" sort-field="true"><display/></field>
    <field name="comments" title="${uiLabelMap.OfbizDemoComment}"><display/></field>
</form>
```

窗体或屏幕的动作标签用于视图的数据准备逻辑

> 我们已经使用OOTB OFBiz通用服务performFind进行搜索操作，当您必须在一个实体或一个视图实体上执行搜索时，该操作简单而有效。

2. 在下一步中，我们将这些表格包含在屏幕中，让我们将这些表格添加到OfbizDemoScreens.xml文件中。 为此，包括下面在OfbizDemoScreens.xml中定义的"FindOfbizDemo''屏幕

> **OfbizDemoScreens.xml**

```xml
<!--以表格形式查找并列出所有-->
    <screen name="FindOfbizDemo">
        <section>
            <actions>
                <set field="headerItem" value="findOfbizDemo"/>
                <set field="titleProperty" value="PageTitleFindOfbizDemo"/>
                <set field="ofbizDemoCtx" from-field="parameters"/>
            </actions>
            <widgets>
                <decorator-screen name="main-decorator" location="${parameters.mainDecoratorLocation}">
                    <decorator-section name="body">
                        <section>
                            <condition>
                                <if-has-permission permission="OFBIZDEMO" action="_VIEW"/>
                            </condition>
                            <widgets>
                                <decorator-screen name="FindScreenDecorator" location="component://common/widget/CommonScreens.xml">
                                    <decorator-section name="search-options">
                                        <include-form name="FindOfbizDemo" location="component://ofbizDemo/widget/OfbizDemoForms.xml"/>
                                    </decorator-section>
                                    <decorator-section name="search-results">
                                        <include-form name="ListOfbizDemo" location="component://ofbizDemo/widget/OfbizDemoForms.xml"/>
                                    </decorator-section>
                                </decorator-screen>
                            </widgets>
                            <fail-widgets>
                                <label style="h3">${uiLabelMap.OfbizDemoViewPermissionError}</label>
                            </fail-widgets>
                        </section>
                    </decorator-section>
                </decorator-screen>
            </widgets>
        </section>
    </screen>
```

3. 在controller.xml中添加用于访问此新的Find Ofbiz演示页面的请求映射

> **controller.xml**

```xml
<!-- Request Mapping -->
<request-map uri="FindOfbizDemo"><security https="true" auth="true"/><response name="success" type="view" value="FindOfbizDemo"/></request-map>
   
<!-- View Mapping -->
<view-map name="FindOfbizDemo" type="screen" page="component://ofbizDemo/widget/OfbizDemoScreens.xml#FindOfbizDemo"/>
```

4. 现在，让我们添加一个新菜单来显示查找选项。

   在OFBiz中创建菜单确实非常简单，所有菜单定义为* menus.xml。

   当我们从Gra创建组件时，我们得到一个名为OfbizDemoMenus.xml的文件

   在OfbizDemoMenus.xml文件中输入以下内容。

> **OfbizDemoMenus.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<menus xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://ofbiz.apache.org/Widget-Menu" xsi:schemaLocation="http://ofbiz.apache.org/Widget-Menu http://ofbiz.apache.org/dtds/widget-menu.xsd">
    <menu name="MainAppBar" title="${uiLabelMap.OfbizDemoApplication}" extends="CommonAppBarMenu" extends-resource="component://common/widget/CommonMenus.xml">
        <menu-item name="main" title="${uiLabelMap.CommonMain}"><link target="main"/></menu-item>
        <menu-item name="findOfbizDemo" title="${uiLabelMap.OfbizDemoFind}"><link target="FindOfbizDemo"/></menu-item>
    </menu>
</menus>
```

#### 5.6UI国际化的使用（完成）

如前所述，Apache OFBiz的国际化非常容易，我们以各种语言定义UI标签，并根据用户的语言环境显示相应的标签。

在这里我们完成了UI标签的示例（创建组件<component-name> UiLabels.xml是默认创建的，在我们的例子中是OfbizDemoUiLabels.xml）

> **OfbizDemoUiLabels.xml**

```xml
<property key="OfbizDemoFind">
    <value xml:lang="en">Find</value>
</property>
<property key="OfbizDemoFirstName">
    <value xml:lang="en">First Name</value>
</property>
<property key="OfbizDemoId">
    <value xml:lang="en">OFBiz Demo Id</value>
</property>
<property key="OfbizDemoLastName">
   <value xml:lang="en">Last Name</value>
</property>
```

现在只需重新启动服务器，在ofbizdemo应用程序（https://localhost:8443/ofbizDemo/control/main）下，您将看到“find”菜单选项。

![image-20200727143123284](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727151224885.png)

#### 5.7使用其他引擎的服务

每当需要构建业务逻辑时，您都应该更喜欢编写服务以利用其内置Service Engine中的功能。

您之前创建的服务“ createOfbizDemo”使用的是engine =“ entity-auto”，因此您无需提供其实现，OFBiz会负责创建操作。 当您需要在服务中进行复杂的操作时，涉及要构建的数据库和定制逻辑中的多个实体，您需要为服务提供定制实现。 在本节中，我们将重点讨论这一点。

##### 5.7.1Java服务

您可以按照以下给定步骤按照Java指令实现服务：

1. 定义您的服务，在这里我们将再次与自定义Ofbiz Demo应用程序的同一实体（OfbizDemo）一起运行。 打开服务定义文件$ OFBIZ_HOME / specialpurpose / ofbizDemo / servicedef / services.xml并添加新定义为：

>**services.xml**

```xml
<service name="createOfbizDemoByJavaService" default-entity-name="OfbizDemo" engine="java"
             location="com.companyname.ofbizdemo.services.OfbizDemoServices" invoke="createOfbizDemo" auth="true">
        <description>Create an Ofbiz Demo record using a service in Java</description>
        <auto-attributes include="pk" mode="OUT" optional="false"/>
        <auto-attributes include="nonpk" mode="IN" optional="false"/>
        <override name="comments" optional="true"/>
    </service>
```

> 注意我们这次使用了engine =“ java”

2. 在ofbizDemo组件src / main / java目录中创建“ com.companyname.ofbizdemo.services”包（如果src目录中不存在，则创建它们）。

例如：src / main / java / com / companyname / ofbizdemo / services。 可以在Java目录中放置必须用Java实现的应用程序服务。

3. 在服务目录中的OfbizDemoServices.java文件中定义新的Java类，并实现方法，服务定义将调用该方法，如下所示：

> **OfbizDemoServices.java**

```java
package com.companyname.ofbizdemo.services;

import org.apache.ofbiz.base.util.Debug;
import org.apache.ofbiz.entity.Delegator;
import org.apache.ofbiz.entity.GenericEntityException;
import org.apache.ofbiz.entity.GenericValue;
import org.apache.ofbiz.service.DispatchContext;
import org.apache.ofbiz.service.ServiceUtil;

import java.util.Map;

public class OfbizDemoServices {

    public static final String module = OfbizDemoServices.class.getName();

    public static Map<String,Object> createOfbizDemo(DispatchContext dctx,Map<String , ? extends  Object> context){
        Map<String, Object> result = ServiceUtil.returnSuccess();
        Delegator delegator = dctx.getDelegator();
        try {
            GenericValue ofbizDemo = delegator.makeValue("OfbizDemo");
            ofbizDemo.setNextSeqId();
            ofbizDemo.setNonPKFields(context);
            ofbizDemo = delegator.create(ofbizDemo);
            result.put("ofbizDemoId",ofbizDemo.getString("ofbizDemoId"));
            Debug.log("============这是我在Apache OFBiz中的第一个Java服务实现。 使用ofbizDemoId成功创建了OfbizDemo记录："+ofbizDemo.getString("ofbizDemoId"));
        } catch (GenericEntityException e) {
            Debug.logError(e,module);
            return  ServiceUtil.returnError("在OfbizDemo实体中创建记录时出错。"+module);
        }
        return  result;
    }
}
```

4. 停止服务器并使用“ ./gradlew ofbiz”重新启动，它将编译您的类，并在ofbiz重新启动更新的jar文件时使其可用。

5. 测试使用webtools实现的服务->运行服务选项（https：// localhost：8443 / webtools / control / runService）或仅更新控制器请求调用的服务名称以使用此服务，并在应用程序中使用添加表单 您之前准备的。 这样，您的Add OfbizDemo表单将调用此Java服务。

>```.xml
>controller.xml
>```

```xml
<request-map uri="createOfbizDemo">
        <security https="true" auth="true"/>
        <event type="service" invoke="createOfbizDemoByJavaService"/>
        <response name="success" type="view" value="main"/>
    </request-map>
```

- 点击提交
  - ![image-20200727145856306](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727145856306.png)

- 查看控制台
  - ![image-20200727150026627](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727143123284.png)

> 说明我们成功调用的是Java服务

##### 5.7.2Groovy中的服务 

> 不使用Groovy 则跳过

要利用即时编译和更少的代码行的功能，您可以使用Groovy DSL在OFBiz中实现用于构建业务逻辑的服务。

要使用Groovy实施服务，您可以按照以下给定步骤进行操作：

1. 将新服务定义添加到services / services.xml文件，如下所示：

> **services.xml**

```xml
<service name="createOfbizDemoByGroovyService" default-entity-name="OfbizDemo" engine="groovy"
        location="component://ofbizDemo/script/com/companyname/ofbizdemo/OfbizDemoServices.groovy" invoke="createOfbizDemo" auth="true">
    <description>Create an Ofbiz Demo record using a service in Java</description>
    <auto-attributes include="pk" mode="OUT" optional="false"/>
    <auto-attributes include="nonpk" mode="IN" optional="false"/>
    <override name="comments" optional="true"/>
</service>
```

2. 在此处添加新的Groovy服务文件[component://ofbizDemo/script/com/companyname/ofbizdemo/OfbizDemoServices.groovy](component://ofbizdemo/script/com/companyname/ofbizdemo/OfbizDemoServices.groovy)

3. 将服务实现添加到文件OfbizDemoServices.groovy

> **OfbizDemoServices.groovy**

```groovy
import org.apache.ofbiz.entity.GenericEntityException;

def createOfbizDemo() {
    result = [:];
    try {
        ofbizDemo = delegator.makeValue("OfbizDemo");
        // Auto generating next sequence of ofbizDemoId primary key
        ofbizDemo.setNextSeqId();
        // Setting up all non primary key field values from context map
        ofbizDemo.setNonPKFields(context);
        // Creating record in database for OfbizDemo entity for prepared value
        ofbizDemo = delegator.create(ofbizDemo);
        result.ofbizDemoId = ofbizDemo.ofbizDemoId;
        logInfo("==========This is my first Groovy Service implementation in Apache OFBiz. OfbizDemo record "
                  +"created successfully with ofbizDemoId: "+ofbizDemo.getString("ofbizDemoId"));
      } catch (GenericEntityException e) {
          logError(e.getMessage());
          return error("Error in creating record in OfbizDemo entity ........");
      }
      return result;
} 
```

4. 停止服务器并使用“ ./gradlew ofbiz”重新启动，这一次，我们只需要加载新的服务定义，就不需要显式编译，因为它是Groovy中的服务实现。

5. 测试使用webtools实现的服务->运行服务选项（https：// localhost：8443 / webtools / control / runService）或仅更新控制器请求调用的服务名称以使用此服务，并在应用程序中使用添加表单 您之前为测试准备的。 这样，您的Add OfbizDemo表单将调用此groovy服务。

> **controller.xml**

```xml
<request-map uri="createOfbizDemo">
    <security https="true" auth="true"/>
    <event type="service" invoke="createOfbizDemoByGroovyService"/>
    <response name="success" type="view" value="main"/>
</request-map>
```

要确保正在执行此新服务实现，可以使用Debug.log（...）在控制台日志中检查您已在代码中放入的这一行。 要登录OFBiz，必须始终在Java类中使用Debug类方法。

要获得有关在Apache OFBiz中使用Groovy DSL进行服务和事件实现的更多详细信息，您可以在此处参考由Jacopo Cappellato创建的文档在[OFBiz Wiki](http://cwiki.apache.org/confluence/x/_M_oAQ)中。

### 6.Events(事件)

#### 6.1示范

Apache OFBiz中的事件只是用于HttpServletRequest和HttpServletResponse对象的简单方法。 您无需像使用服务那样提供这些定义。 这些直接从控制器调用。 当您要将自定义服务器端验证添加到输入参数时，事件也很有用。 对于执行数据库操作，您仍然可以从事件中调用预建服务。

要在OFBiz中编写事件，请按照以下步骤操作：

1. 添加新的事件目录到package和新的Events类文件，如下所示：

![image-20200727151224885](https://he-img-test.oss-cn-shanghai.aliyuncs.com/img/image-20200727150026627.png)

> **OfbizDemoEvents.java**

```java
package com.companyname.ofbizdemo.events;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.ofbiz.base.util.Debug;
import org.apache.ofbiz.base.util.UtilMisc;
import org.apache.ofbiz.base.util.UtilValidate;
import org.apache.ofbiz.entity.Delegator;
import org.apache.ofbiz.entity.GenericValue;
import org.apache.ofbiz.service.GenericServiceException;
import org.apache.ofbiz.service.LocalDispatcher;

public class OfbizDemoEvents {

    public static final String module = OfbizDemoEvents.class.getName();

    public static String createOfbizDemoEvent(HttpServletRequest request, HttpServletResponse response) {
        Delegator delegator = (Delegator) request.getAttribute("delegator");
        LocalDispatcher dispatcher = (LocalDispatcher) request.getAttribute("dispatcher");
        GenericValue userLogin = (GenericValue) request.getSession().getAttribute("userLogin");

        String ofbizDemoTypeId = request.getParameter("ofbizDemoTypeId");
        String firstName = request.getParameter("firstName");
        String lastName = request.getParameter("lastName");

        if (UtilValidate.isEmpty(firstName) || UtilValidate.isEmpty(lastName)) {
            String errMsg = "名和姓是表单上的必填字段，不能为空";
            request.setAttribute("_ERROR_MESSAGE_", errMsg);
            return "error";
        }
        String comments = request.getParameter("comments");

        try {
            Debug.logInfo("=======使用服务createOfbizDemoByJavaService在事件中创建OfbizDemo记录=========", module);
            dispatcher.runSync("createOfbizDemoByJavaService", UtilMisc.toMap("ofbizDemoTypeId", ofbizDemoTypeId,
                    "firstName", firstName, "lastName", lastName, "comments", comments, "userLogin", userLogin));
        } catch (GenericServiceException e) {
            String errMsg = "无法在OfbizDemo实体中创建新记录: " + e.toString();
            request.setAttribute("_ERROR_MESSAGE_", errMsg);
            return "error";
        }
        request.setAttribute("_EVENT_MESSAGE_", "OFBiz演示成功创建.");
        return "success";
    }
}
```

2. 将调用此事件的控制器请求添加为：

> **controller.xml**

```xml
<request-map uri="createOfbizDemoEvent">
    <security https="true" auth="true"/>
    <event type="java" path="com.companyname.ofbizdemo.events.OfbizDemoEvents" invoke="createOfbizDemoEvent"/>
    <response name="success" type="view" value="main"/>
    <response name="error" type="view" value="main"/>
</request-map>
```

3. 通过重建服务器来停止和启动服务器，因为我们需要编译在＃1中添加的Java事件类。

4. 现在要测试该事件，您只需将AddOfbizDemo表单目标更改为读取“ createOfbizDemoEvent”，并且由于其提交，它将调用您的事件。

#### 6.2服务与事件之间的区别

以下是服务和事件之间的一些区别，

- 事件用于使用map processor进行验证和转换，而服务用于诸如CRUD操作之类的业务逻辑。
- 服务返回Map。
- 事件返回字符串。
- 服务随服务器一起加载，定义的任何更改（如果在MiniLang中，则不是实现）都需要重新加载。
- 我们可以在事件内部调用服务。 但是我们不能在服务内部调用事件
- 事件是特定的本地片段功能，通常在一个地方用于一种目的，并从其位置调用。
- 服务是一项功能，可以位于网络上的任何位置，大部分时间在几个不同的地方使用，并以其“name”来调用。
- 如果发生事件，您可以访问HttpServletRequest和HttpServletResponse对象，并且可以读取/写入任何所需内容。 如果是服务，则只能访问服务参数。

> [参考资料](https://cwiki.apache.org/confluence/display/OFBIZ/FAQ+-+Tips+-+Tricks+-+Cookbook+-+HowTo#FAQ-Tips-Tricks-Cookbook-HowTo-DifferenceBetweenEventAndService)

| 标准             | 服务                                | 事件                        |
| ---------------- | ----------------------------------- | --------------------------- |
| 用于编写业务逻辑 | Yes                                 | No                          |
| 返回类型         | Map                                 | String                      |
| 需要定义         | Yes                                 | No                          |
| 可以进行作业调度 | Yes                                 | No                          |
| 实施可能性       | 实体自动，Java，简单（XML）和Groovy | Simple (XML), Java & Groovy |

### 7.自定义用户界面

使用FreeMarker模板和Groovy脚本

好的，我们在OFBiz教程的最后一部分中。 在这一部分中，我们将重点介绍为业务管理应用程序（即后端应用程序和esp）定制Apache OFBiz的UI层。 大多数时候，您会发现OFBiz窗口小部件已足够。 但是有时候重要的是根据用户的实际需求开发应用程序。

因此，首先要自定义应用程序的UI部分以使其变得容易，我们将使用Freemarker模板而不是内置的Form Widgets。 首先，我们将了解如何将Freemarker和Groovy脚本与Apache OFBiz一起使用，然后我们将了解如何通过定义自己的装饰器来对其进行自定义样式。 最初，我们将使用OFBiz默认装饰器。

从此处开始，请遵循给出的步骤：

1. 在位置$ OFBIZ_HOME / specialpurpose / ofbizDemo / webapp / ofbizDemo / crud / AddOfbizDemo.ftl和ListOfbizDemo.ftl处添加两个Freemarker文件。

> **AddOfbizDemo.ftl**

```html
<div class="screenlet-body">
  <form id="createOfbizDemoEvent" method="post" action="<@ofbizUrl>createOfbizDemoEvent</@ofbizUrl>">
    <input type="hidden" name="addOfbizDemoFromFtl" value="Y"/>
    <fieldset>
      <div>
        <span class="label">${uiLabelMap.OfbizDemoType}</span>
        <select name="ofbizDemoTypeId" class='required'>
          <#list ofbizDemoTypes as demoType>
            <option value='${demoType.ofbizDemoTypeId}'>${demoType.description}</option>
          </#list>
        </select>*
      </div>
      <div>
        <span class="label">${uiLabelMap.OfbizDemoFirstName}</span>
        <input type="text" name="firstName" id="firstName" class='required' maxlength="20" />*
      </div>
      <div>
        <span class="label">${uiLabelMap.OfbizDemoLastName}</span>
        <input type="text" name="lastName" id="lastName" class='required' maxlength="20" />*
      </div>
      <div>
        <span class="label">${uiLabelMap.OfbizDemoComment}</span>
        <input type="text" name="comments" id="comments" class='inputBox' size="60" maxlength="255" />
      </div>
    </fieldset>
    <input type="submit" value="${uiLabelMap.CommonAdd}" />
  </form>
</div>
```

> **ListOfbizDemo.ftl**

```html
<div class="screenlet-body">
  <#if ofbizDemoList?has_content>
    <table cellspacing=0 cellpadding=2 border=0 class="basic-table">
      <thead><tr>
        <th>${uiLabelMap.OfbizDemoId}</th>
        <th>${uiLabelMap.OfbizDemoType}</th>
        <th>${uiLabelMap.OfbizDemoFirstName}</th>
        <th>${uiLabelMap.OfbizDemoLastName}</th>
        <th>${uiLabelMap.OfbizDemoComment}</th>
      </tr></thead>
      <tbody>
        <#list ofbizDemoList as ofbizDemo>
          <tr>
            <td>${ofbizDemo.ofbizDemoId}</td>
            <td>${ofbizDemo.getRelatedOne("OfbizDemoType").get("description", locale)}</td>
            <td>${ofbizDemo.firstName?default("NA")}</td>
            <td>${ofbizDemo.lastName?default("NA")}</td>
            <td>${ofbizDemo.comments!}</td>
          </tr>
        </#list>
       </tbody>
    </table>
  </#if>
</div>
```

2. 在位置$ OFBIZ_HOME / specialpurpose / ofbizDemo / webapp / ofbizDemo / WEB-INF / actions / crud / ListOfbizDemo.groovy中添加新的Groovy文件以用于数据提取逻辑，并添加如下所示的代码以列出OfbizDemo记录：

```java
ofbizDemoTypes = delegator.findList("OfbizDemoType", null, null, null, null, false);
context.ofbizDemoTypes = ofbizDemoTypes;
ofbizDemoList = delegator.findList("OfbizDemo", null, null, null, null, false);
context.ofbizDemoList = ofbizDemoList;
```

3. 使用Ofbiz默认装饰器将新的屏幕文件添加到OfbizDemoScreens.xml，其中新添加的freemarker和groovy文件为

> **OfbizDemoScreens.xml**

```xml
<screen name="AddOfbizDemoFtl">
    <section>
        <actions>
            <set field="titleProperty" value="PageTitleAddOfbizDemos"/>
            <set field="headerItem" value="addOfbizDemoFtl"/>
            <script location="component://ofbizDemo/webapp/ofbizDemo/WEB-INF/actions/crud/ListOfbizDemo.groovy"/>
        </actions>
        <widgets>
            <decorator-screen name="main-decorator" location="${parameters.mainDecoratorLocation}">
                <decorator-section name="body">
                    <screenlet title="${uiLabelMap.OfbizDemoListOfbizDemos}">
                        <platform-specific>
                            <html><html-template location="component://ofbizDemo/webapp/ofbizDemo/crud/ListOfbizDemo.ftl"/></html>
                         </platform-specific>
                    </screenlet>
                    <screenlet title="${uiLabelMap.OfbizDemoAddOfbizDemoServiceByFtl}">
                        <platform-specific>
                            <html><html-template location="component://ofbizDemo/webapp/ofbizDemo/crud/AddOfbizDemo.ftl"/></html>
                        </platform-specific>
                    </screenlet>
                </decorator-section>
            </decorator-screen>
        </widgets>
    </section>
</screen>
 
```

4. 为OfbizDemo菜单添加新的控制器请求和新项，如下所示：

> **controller.xml**

```xml
<!--Request Mapping-->
<request-map uri="AddOfbizDemoFtl">
    <security https="true" auth="true"/>
    <response name="success" type="view" value="AddOfbizDemoFtl"/>
</request-map>
 
<!--View Mapping-->
<view-map name="AddOfbizDemoFtl" type="screen" page="component://ofbizDemo/widget/OfbizDemoScreens.xml#AddOfbizDemoFtl"/>
 
```

> **OfbizDemoMenus.xml**

```xml
<menu-item name="addOfbizDemoFtl" title="${uiLabelMap.OfbizDemoAddFtl}"><link target="AddOfbizDemoFtl"/></menu-item> 
```

5. 添加您的应用所使用的新UI标签。

6. 运行您的Ofbiz演示应用程序，然后转到您刚刚添加的新标签。

#### 7.1创建自定义装饰器

将用户界面放置在Freemarker中使您可以自由地进行实验，进行CSS调整并使应用程序达到用户想要的方式。 在本节中，我们将看到如何做到这一点。

我们将通过为您的应用程序视图定义自定义装饰器来做到这一点。 OFBiz中的装饰器不过是一个屏幕，您可以通过在其他应用程序屏幕中包含该屏幕来定义和重用该屏幕。 您已经使用OFBiz随附的默认装饰器（主装饰器–> ApplicationDecorator）进行了此操作。 只需观察您到目前为止准备的屏幕，您会发现您正在使用此主装饰器，请参考OfbizDemoScreens.xml中的以下行。

> **OfbizDemoScreens.xml**

```xml
<decorator-screen name="main-decorator" location="${parameters.mainDecoratorLocation}">
```

> **web.xml**

```xml
<context-param>
    <description>The location of the main-decorator screen to use for this webapp; referred to as a context variable in screen def XML files.</description>
    <param-name>mainDecoratorLocation</param-name>
    <param-value>component://ofbizDemo/widget/CommonScreens.xml</param-value>
</context-param>
```

现在是时候使用自定义样式定义自己的装饰器了。

在下面给出的示例中，我们将使用Bootstrap设置在本教程最后一部分中开发的示例Freemarker屏幕的样式。 请按照以下给定的步骤构建您自己的装饰器。

1. 下载Bootstrap v3.3.7目录，您可以从此处下载它并解压缩。
2. 在位置$ OFBIZ_HOME / specialpurpose / ofbizDemo / webapp / ofbizDemo /下创建两个新目录，分别为“ css”和“ js”
3. 将bootstrap-3.3.7 / dist / css / bootstrap.min.css复制到$ OFBIZ_HOME / specialpurpose / ofbizDemo / webapp / ofbizDemo / css
4. 将bootstrap-3.3.7 / dist / js / bootstrap.min.js复制到$ OFBIZ_HOME / specialpurpose / ofbizDemo / webapp / ofbizDemo / js。
5. 打开$ OFBIZ_HOME / specialpurpose / ofbizDemo / webapp / ofbizDemo / WEB-INF / web.xml，最后在allowedPaths中为css和js目录创建条目，如下所示：

>  **web.xml**

```xml
<init-param>
    <param-name>allowedPaths</param-name>
    <param-value>/error:/control:/select:/index.html:/index.jsp:/default.html:/default.jsp:/images:/includes/maincss.css:/css:/js</param-value>
</init-param>
```

6. 在$ OFBIZ_HOME / specialpurpose / ofbizDemo / webapp / ofbizDemo /位置添加名为“ includes”的新目录，并在您刚刚添加的名为PreBody.ftl和PostBody.ftl的新目录中创建两个新文件。 我们将在装饰器屏幕中使用（包括）这两个文件来构建完整的HTML页面。

> **PreBody.ftl**

```html
<html>
  <head>
    <title>${layoutSettings.companyName}</title>
    <meta name="viewport" content="width=device-width, user-scalable=no"/>
    <#if webSiteFaviconContent?has_content>
      <link rel="shortcut icon" href="">
    </#if>
    <#list layoutSettings.styleSheets as styleSheet>
      <link rel="stylesheet" href="${StringUtil.wrapString(styleSheet)}" type="text/css"/>
    </#list>
    <#list layoutSettings.javaScripts as javaScript>
      <script type="text/javascript" src="${StringUtil.wrapString(javaScript)}"></script>
    </#list>
  </head>
  <body data-offset="125">
    <h4 align="center"> ==================Page PreBody Starts From Decorator Screen========================= </h4>
    <div class="container menus" id="container">
      <div class="row">
        <div class="col-sm-12">
          <ul id="page-title" class="breadcrumb">
            <li>
                <a href="<@ofbizUrl>main</@ofbizUrl>">Main</a>
            </li>
            <li class="active"><span class="flipper-title">${StringUtil.wrapString(uiLabelMap[titleProperty])}</span></li>
            <li class="pull-right">
              <a href="<@ofbizUrl>logout</@ofbizUrl>" title="${uiLabelMap.CommonLogout}">logout</i></a>
            </li>
          </ul>
        </div>
      </div>
      <div class="row">
        <div class="col-lg-12 header-col">
          <div id="main-content">
              <h4 align="center"> ==================Page PreBody Ends From Decorator Screen=========================</h4>
              <h4 align="center"> ==================Page Body starts From Screen=========================</h4>
 
```

> **PostBody.ftl**

```html
<#-- Close the tags opened in the PreBody section -->
          </div>
        </div>
      </div>
    </div>
    <h4 align="center"> ==================Page PostBody and Page body in general ends here from Decorator Screen=========================</h4>
  </body>
</html>
```

7. 打开组件的通用屏幕文件$ OFBIZ_HOME / specialpurpose / ofbizDemo / widget / CommonScreens.xml，这是我们将定义自定义装饰器的文件。
8. 更新屏幕名为“ OfbizDemoCommonDecorator”（将用作您的应用程序的自定义装饰器），如下所示：

> **CommonScreens.xml**

```xml
<screen name="OfbizDemoCommonDecorator">
    <section>
        <actions>
            <property-map resource="OfbizDemoUiLabels" map-name="uiLabelMap" global="true"/>
            <property-map resource="CommonUiLabels" map-name="uiLabelMap" global="true"/>
   
            <set field="layoutSettings.companyName" from-field="uiLabelMap.OfbizDemoCompanyName" global="true"/>
            
            <!-- Including custom CSS Styles that you want to use in your application view. [] in field can be used to 
                 set the order of loading CSS files to load if there are multiple -->
            <set field="layoutSettings.styleSheets[]" value="/ofbizDemo/css/bootstrap.min.css"/>
   
            <!-- Including custom JS that you want to use in your application view. [] in field can be used to
                 set the order of loading of JS files to load if there are multiple -->
           <set field="layoutSettings.javaScripts[+0]" value="/ofbizDemo/js/bootstrap.min.js" global="true"/>
        </actions>
        <widgets>
            <section>
                <condition>
                    <if-has-permission permission="OFBIZDEMO" action="_VIEW"/>
                </condition>
                <widgets>
                    <platform-specific><html><html-template location="component://ofbizDemo/webapp/ofbizDemo/includes/PreBody.ftl"/></html></platform-specific>
                    <decorator-section-include name="pre-body"/>
                    <decorator-section-include name="body"/>
                    <platform-specific><html><html-template location="component://ofbizDemo/webapp/ofbizDemo/includes/PostBody.ftl"/></html></platform-specific>
                </widgets>
                <fail-widgets>
                    <label style="h3">${uiLabelMap.OfbizDemoViewPermissionError}</label>
                </fail-widgets>
            </section>
        </widgets>
    </section>
</screen>
```

在上面的代码中，您可能已经注意到了layoutSettings.styleSheets []和layoutSettings.javaScripts [+0]表示法。 您可以使用layoutSettings。 文件的符号。

如果要带空方括号的styleSheets或javaScripts，只需将文件添加到layoutSettings.styleSheets或layoutSettings.javaScripts列表的末尾，并用[+0]将其添加到它的前面。

9. 在上一部分创建的Freemarker屏幕中，将此装饰器用作：

> **OfbizDemoScreens.xml**

```xml
<screen name="AddOfbizDemoFtl">
    <section>
        <actions>
            <set field="titleProperty" value="OfbizDemoAddOfbizDemoFtl"/>
            <set field="headerItem" value="addOfbizDemoFtl"/>
            <script location="component://ofbizDemo/webapp/ofbizDemo/WEB-INF/actions/crud/ListOfbizDemo.groovy"/>
        </actions>
        <widgets>
            <decorator-screen name="OfbizDemoCommonDecorator" location="${parameters.mainDecoratorLocation}">
                <decorator-section name="body">
                     <label style="h4" text="${uiLabelMap.OfbizDemoListOfbizDemos}"/>
                     <platform-specific>
                         <html><html-template location="component://ofbizDemo/webapp/ofbizDemo/crud/ListOfbizDemo.ftl"/></html>
                     </platform-specific>
                     <label style="h4" text="${uiLabelMap.OfbizDemoAddOfbizDemoFtl}"/>
                     <platform-specific>
                         <html><html-template location="component://ofbizDemo/webapp/ofbizDemo/crud/AddOfbizDemo.ftl"/></html>
                     </platform-specific>
                </decorator-section>
            </decorator-screen>
        </widgets>
    </section>
</screen>
```

10. 更新您的FTL文件以遵循HTML Web标准并将CSS应用于以下文件：

> **AddOfbizDemo.ftl**

```html
<form method="post" action="<@ofbizUrl>createOfbizDemoEventFtl</@ofbizUrl>" name="createOfbizDemoEvent" class="form-horizontal">
  <div class="control-group">
    <label class="control-label" for="ofbizDemoTypeId">${uiLabelMap.OfbizDemoType}</label>
    <div class="controls">
      <select id="ofbizDemoTypeId" name="ofbizDemoTypeId">
        <#list ofbizDemoTypes as demoType>
          <option value='${demoType.ofbizDemoTypeId}'>${demoType.description}</option>
        </#list>
      </select>
    </div>
  </div>
  <div class="control-group">
    <label class="control-label" for="firstName">${uiLabelMap.OfbizDemoFirstName}</label>
    <div class="controls">
      <input type="text" id="firstName" name="firstName" required>
    </div>
  </div>
  <div class="control-group">
    <label class="control-label" for="lastName">${uiLabelMap.OfbizDemoLastName}</label>
    <div class="controls">
      <input type="text" id="lastName" name="lastName" required>
    </div>
  </div>
  <div class="control-group">
    <label class="control-label" for="comments">${uiLabelMap.OfbizDemoComment}</label>
    <div class="controls">
      <input type="text" id="comments" name="comments">
    </div>
  </div>
  <div class="control-group">
    <div class="controls">
      <button type="submit" class="btn">${uiLabelMap.CommonAdd}</button>
    </div>
  </div>
</form>
```

> **ListOfbizDemo.ftl**

```html
<table class="table table-bordered table-striped table-hover">
    <thead>
        <tr>
          <th>${uiLabelMap.OfbizDemoId}</th>
          <th>${uiLabelMap.OfbizDemoType}</th>
          <th>${uiLabelMap.OfbizDemoFirstName}</th>
          <th>${uiLabelMap.OfbizDemoLastName}</th>
          <th>${uiLabelMap.OfbizDemoComment}</th>
        </tr>
    </thead>
    <tbody>
        <#list ofbizDemoList as ofbizDemo>
            <tr>
              <td>${ofbizDemo.ofbizDemoId}</td>
              <td>${ofbizDemo.getRelatedOne("OfbizDemoType").get("description", locale)}</td>
              <td>${ofbizDemo.firstName?default("NA")}</td>
              <td>${ofbizDemo.lastName?default("NA")}</td>
              <td>${ofbizDemo.comments!}</td>
            </tr>
        </#list>
    </tbody>
</table> 
```

11. 现在，在web.xml中输入allowedPaths的条目后，重新启动OFBiz。 重新加载时，请访问https：// localhost：8443 / ofbizDemo / control / AddOfbizDemoFtl。您应该看到具有自定义样式的页面，而不是使用默认的OFBiz主题。 

现在，您可以在这里随心所欲地玩它。 尝试更改页眉或使用新的页眉，添加页脚，进行验证等。因此，您可以使用Freemarker模板，CSS和JS自定义OFBiz的UI层。

您可能想要添加自己的CSS或JS文件，可以像对Bootstrap文件那样添加它们。

如果您已按照本教程进行了所有步骤并开发了实践应用程序，那么这将有助于您了解OFBiz中的其他实现。 这些都是在OFBiz中工作的基本基础。 现在您知道了如何在OFBiz中开始开发。 不要留下本教程中提供的额外链接，因为它们将帮助您大量了解此处详细介绍的内容。



这是常见问题解答技巧食谱手册中的另一本好书。



现在，接下来的事情是业务流程，为了真正理解OFBiz和OOTB数据模型中的OOTB流程，确实需要很好地理解它们，因此，可以在：OFBiz相关书中找到这些书。 充分了解OFBiz OOTB可用数据模型和业务流程将有助于在其顶部构建更好的业务解决方案。



现在您可以开始研究了。欢迎来到OFBiz世界。