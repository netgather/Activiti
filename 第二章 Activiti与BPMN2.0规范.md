# 启动事件
  启动事件都是“捕获型”的，等待第三方触发后才可以启动，在Activiti中可以通过调用API触发启动事件。（结束事件属于“抛出型”事件）
## 空启动事件
    <startEvent id="startEvent" name="Start Event"> </startEvent>
  由于startEvent标签中没有任何其他的元素定义，所以称之为空启动事件，空启动事件是其他启动事件的XML的基本表示。
  Activiti在空启动事件上扩展的属性。

|属性名称|属性说明|示例|
|----|----|----|
|activiti:formKey|用来指定空启动事件关联的表单文件|`<startEvent  id="startEvent" activiti:formKey="apply.form">`|
|activiti:initiator|用来记录启动流程的人的ID，也可以是用户的名称，启动流程之后此属性指定的变量就会自动设置当前人的名称|`<startEvent id="startEvent" activiti:initiator="startUserId">`|
>外置表单
>>表单的内容存放在一个单独的".form"文件中，可以是任何文本内容，一般用HTML书写，启动事件中的activiti:formKey属性就是用来指定使用哪个表单文件。

>内置表单
>>表单中的元素定义在流程定义文件中，包括各种输入框、下拉框等。对应的XML标识为：
`<activiti:formProperty id="name" name="Name" type="string"></activiti:formProperty>`

activiti:formProperty可以有多个，对应一个表单中的一个字段。在实际应用中可以通过引擎提供的API读取、提交这些表单元素。

##定时启动事件
定时启动事件可以用于一次性定时启动、特定时间间隔后启动。定时启动事件常用于定期循环流程或一次性流程。定时启动事件是在基本的空启动事件中嵌套一个定时事件定义timeEventDefinition，对应的XML描述为：
```XML
<startEvent id="timerStartEvent" name="Timer Start Event"><timerEventDefinition><timerCycle></timerCycle></timerEventDefinition></startEvent>
```
通过`timerEventDefinition`可以将一个空启动事件描述为定时启动事件，通过定时启动事件的定时属性可以指定定时启动事件的行为：

|属性名称|属性说明|示例|
|----|----|----|
|timeDate|一次性的定时启动|`<timeDate>2012-12-12T00:00:00</timeDate>`|
|timeDuration|设置多长时间之后启动|`<timeDuration>PT10M</timeDuration>`|
|timeCycle|周期性启动任务|`<timeCycle>R3/PT10H</timeCycle>`|


##异常启动事件
异常启动事件可以触发一个异常子流程。异常启动事件不能通过API方式启动，它总是在另外一个流程抛出异常结束事件的时候被触发。异常启动事件是“捕获型”的，异常结束事件是“抛出型”的，类似于Java中的异常处理机制。

**异常启动事件必须嵌套在一个事件子流程中**

![异常启动事件](http://d.pr/i/80Xw/ErrorStartEvent.png)
```xml
<subProcess id="eventsubprocess1" name="Event sub Process" triggeredByEvent="true">
      <startEvent id="errorstartevent1" name="Error start">
        <errorEventDefinition></errorEventDefinition>
      </startEvent>
      <userTask id="usertask2" name="User Task"></userTask>
      <endEvent id="endevent3" name="End"></endEvent>
      <sequenceFlow id="flow6" sourceRef="errorstartevent1" targetRef="usertask2"></sequenceFlow>
      <sequenceFlow id="flow7" sourceRef="usertask2" targetRef="endevent3"></sequenceFlow>
</subProcess>
```
##消息启动事件
消息启动事件可以通过一个消息名称触发，从而启动一个流程实例；可以结合消息抛出事件一起使用，由流程自动根据消息的名称启动对应的流程；还可以通过Activiti提供的API触发消息启动事件：`runtimeService.startProcessByMessage("消息名称")`启动一个对应消息名称的流程。
```XML
<message id="reSendFile" name="重新发送文件"/>
<process id="myProcess" name="My process" isExecutable="true">
  <startEvent id="messagestartevent1" name="Message start">
    <messageEventDefinition messageRef="reSendFile" />
  </startEvent>
</process>
```

#结束事件
结束事件和启动事件通常成对出现在流程定义文件中，当然在定义流程时允许没有结束事件，但是不能没有开始事件。结束事件可以细分为以下三种类型：

* 空结束事件
* 异常结束事件
* 终止结束事件
* 取消结束事件

结束事件表示流程（包括子流程）的结束，结束事件总是抛出型的，也就是当流程执行到结束事件时会抛出一个执行结果。

##空结束事件
所有的结束事件都是“抛出型”的，空结束事件也不例外，可以简单的将其理解为“抛出了空值”。
空结束事件的XML描述为：
```XML
<endEvent id="end" name="my end event"></endEvent>
```

##异常结束事件
异常结束事件定义了需要抛出的异常代码，如果找到了**异常开始事件定义的异常代码**，则会触发异常开始事件，否则按照空结束事件规则处理。异常结束事件的XML描述为：
```XML
<endEvent id="endevent1" name="ErrorEnd">
  <errorEventDefinition errorRef="AIA-001"></errorEventDefinition>
</endEvent>
```
##终止结束事件
终止结束事件可以终止一个流程实例的运行。空结束事件结束的是一条输出流，终止结束事件结束的是整个流程实例。
>终止结束事件的应用场景：在付费流程中因为某些原因导致流程不能继续运行，最终的结果是取消本次付费流程实例，所以需要提前结束流程实例的运行，此时可以把某个输出流指向到 *终止结束事件*

终止结束事件的XML描述为：
```XML
<endEvent id="terminateendevent1" name="Terminate End Event">
  <terminateEndEventDefinition></timerEventDefinition>
</endEvent>
```

##取消结束事件
取消结束事件可以取消一个 **事务子流程** 的执行，同时取消结束事件只能在子流程中使用。
>当子流程执行过程中出现异常需要取消时，可以设置一个取消结束事件 *当输出流指向到取消结束事件时流程将会中断执行*。
>取消结束事件可以和取消边界事件配合使用，使用取消边界事件来对取消操作做后续处理。

取消结束事件的XML描述为：
```XML
<endEvent id="myCancelEndEvent">
  <cancelEventDefinition/>
</endEvent>
```

#顺序流
顺序流是两个模型的连接者，用于连接不同的活动、事件。如果一个元素在流程执行期间被访问，那么流程会沿着该元素所有输出流继续执行。

顺序流可以细分为两种：
* 标准顺序流
* 条件顺序流

##标准顺序流
标准顺序流可以用来连接两个或多个模型，以使这些模型建立联系。标准顺序流的XML描述如下：

```XML
<sequenceFlow id="flow" sourceRef="stat" targetRef="userTask"></sequenceFlow>
```

其中`sequenceFlow`表示一个顺序流，`sourceRef`属性指定顺序流的源，`targetRef`属性指定顺序流的目的模型。
>sourceRef 和 targetRef 属性的值 就是对应模型的ID

#任务
根据业务需求的不同，任务可以分为以下几种类型：
* 用户任务
* 脚本任务
* WebService任务
* 业务规则任务
* 邮件任务
* Mule任务
* Camel任务
* 手动任务
* Java Service任务
* Shell任务

>所有的任务的图形化表示都是以一个矩形为基础，在左上角添加具体类型的图标。

>大多数任务都有单独的XML定义，用来描述一种特定任务类型，也有部分任务是基于serviceTask扩展的。

-------

##用户任务
用户任务需要人来参与，因为它必须被任务触发
>用户任务可以定义任务名称、优先级、到期日、任务处理人

用户任务的XML描述如下：
```XML
<userTask id="usertask1" name="领导审批">
  <humanPerformer>
    <resourceAssignmentExpression>
      <formalExpression>
        10908
      </formalExpression>
    </resourceAssignmentExpression>
  </humanPerformer>
</userTask>
```
> `userTask`定义了一个用户任务；`humanPerformer`标签表示把这个用户任务分配给一个人；`resourceAssignmentExpression`和`formalExpression`表示将这个用户任务分配给了用户10908办理

对应的表示如下所示：

![用户任务](https://d.pr/OC3y/userTask.png)

用户任务除了可以分配给一个用户之外，还可以分配给组或者用户和组的混合。
对应的XML描述如下所示：

```XML
<userTask id="usertask1" name="领导审批">
    <potentialOwner>
    	<resourceAssignmentExpression>
    		<formalExpression>
    			user(10908),group(zhongtai),manager
    		</formalExpression>
    	</resourceAssignmentExpression>
    </potentialOwner>
</userTask>
```
>`potentialOwner`标签用来描述一个潜在的用户、组集合，并使用`resourceAssignmentExpression`标签和`formalExpression`标签将这个用户任务同时分配给用户10908、组zhongtai和组manager。
***如果不添加user关键字或group关键字说明候选人是什么类型，那么Activiti默认为组***


将用户任务指派给人、组、人和组的区别主要是：
* `humanPerformer`标签和`potentialOwner`标签
* `user`关键字和`group`关键字

Activiti在BPMN2.0的基础上进行了扩展，可以简化设置用户、组的方式、支持动态获取用户或组分配用户任务，同时还可以为用户任务设置*创建*、*分配*、*完成监听*。
Activiti扩展的用户任务属性有：

|属性名称|属性说明|属性示例|
|----|----|----|
|activiti:assignee|用来指定用户任务的处理人，可以代替humanPerformer标签|`<userTask id="leaderAudit" name="领导审批" activiti:assignee="henryyan">`|
|activiti:cadidateUsers|用来指定用户任务的候选人，多个候选人用逗号分隔，可以用来代替potentialOwner标签|`<userTask id="leaderAudit" name="领导审批" activiti:cadidateUsers="henryyan,10908">`|
|activiti:cadidateGroups|用来指定候选组，多个候选组使用逗号分隔|`<userTask id="leaderAudit" name="领导审批" activiti:cadidateGroups="leader,manager">`|
|activiti:dueDate|设置用户任务到期日，通常使用变量代替而不是指定一个具体日期|`<userTask id="leaderAudit" name="领导审批" activiti:dueDate="${overDate}">`|
|activiti:priority|用户任务优先级，取值区间[0,100]。通常使用变量来动态设置用户任务的优先级|`<userTask id="leaderAudit" name="领导审批" activiti:priority="${priority}">`|

接下来使用Activiti扩展的属性来重写之前的两段用户任务代码：

```XML
<userTask id="leaderAudit" name="领导审批" activiti:assignee="10908"></userTask>
```

```XML
<userTask id="leaderAudit" name="领导审批" activiti:cadidateUsers="10908,henryyan" activiti:cadidateGroups="leader,manager"></userTask>
```

除了以上属性，Activiti还在用户任务添加了任务监听的子元素，监听的选项有：
* 创建任务-create
* 分配任务-assignment
* 完成任务-complete

```XML
<userTask id="leaderAudit" name="领导审批" activiti:cadidateUsers="10908,henryyan" activiti:cadidateGroups="leader,manager">
    	<extensionElements>
    		<activiti:taskListener event="create" class="你的类路径"></activiti:taskListener>
    	</extensionElements>
</userTask>
```

##脚本任务
脚本任务使得Activiti可以运行支持的脚本语言，例如Activiti默认支持的Groovy、JavaScript、Juel，但是脚本任务的代码需要符合 [JSR-223](http://jcp.org/en/jsr/detail?id=223) 规范。Activiti除了默认支持的三种脚本语言Groovy、JavaScript、Juel，还允许使用其他的脚本语言，前提是把依赖的jar添加到classpath中。
****************

脚本任务的图形化表示如下所示：

脚本任务的基本XML描述如下：

```XML
<scriptTask id="initvars" name="初始化变量">

</scriptTask>
```

示例：使用Groovy作为脚本引擎的脚本任务：

```XML
<scriptTask id="initvars" name="初始化变量" scriptFormat="groovy">
    <script>
    	<![CDATA[
    		def name = "Henryyan";
    		execution.setVariable('name',name);
    	]]>
    </script>
</scriptTask>
<scriptTask id="printvars" name="输出变量" scriptFormat="groovy">
    <script>
    	<![CDATA[
    		out:println name;
    	]]>
    </script>
</scriptTask>
```

>如果在脚本任务中使用了脚本，那么必须指定脚本的类型：`scriptFormat=""`

除了scriptFormat属性之外，Activiti支持的脚本任务属性见下表：

|属性名称|属性说明|属性示例|
|----|----|----|
|activiti:resultvariable|将脚本处理的结果保存到一个变量中,*activiti:resultVariable指定的变量名称需要在脚本中预先定义才能使用*|`<scriptTask id="st1" name="保存变量" scriptFormat="juel" activiti:resultVariable="name"></scriptTask>`|

##Java Service任务
***Java Service任务不属于BPMN2.0规范，而是Activiti扩展的专门应用于Java语言的服务任务***。需要使用`serviceTask`标签来定义Java Service

Java Service任务允许定义一个实现了指定接口（`JavaDelete`或`ActivityBehavior`）的Java类，或者执行一个表达式，还可以像脚本任务一样把结果保存到一个变量中。
接口JavaDelegate和接口ActivityBehavior中只有一个抽象方法：

```Java
void execute(DelegateExecution execution) throws Exception;
```

Java Service的XML描述如下所示：

```XML
<serviceTask id="javaServiceTask" name="Java Service Task" activiti:class="你的Java Service类"></serviceTask>
```

> `activiti:class`属性用于表示该`serviceTask`是一个Java Service，同时“你的Java Service类”必须要实现JavaDelegate或ActivityBehavior中的一个接口。除了使用`activiti:class`指定一个Java类之外，还可以使用其他方式定义Java Service任务需要执行的Java对象（*见下表*）。

在指定一个Java类的同时还可以配置执行Service时传入的变量，这样在执行Java类的时候可以读取预先设置的变量值。

|属性名称|属性说明|属性示例|
|----|----|----|
|activit:class|用于指定Java Service需要执行的Java类，activit:class的属性值需要实现接口JavaDelete或ActivityBehavior|`<ServiceTask id="JavaServiceTask1" name="Java Service Task" activiti:class="你的Java类"/>`|
|activit:expression|使用UEL定义需要执行的任务内容：计算公式、调用Bean对象的方法（Bean示例可以使用new创建，也可以使用Spring代理）。在执行Java Service任务的时候也可以使用流程变量作为参数|`<serviceTask id="JavaServiceTask1" name="Java Service Task" activit:expression="#{leaveService.back()}"/>` leaveService作为一个流程变量存在，需要实现JavaDelegate接口或ActivityBehavior接口，此外还需要实现java.io.Serializable接口，以便引擎可以序列化流程变量`leaveService`|
|activiti:delegateExpression|功能和`activiti:expression`类似同样需要实现JavaDelegate或ActivityBehavior中的一个接口，指定的Bean可以使用new方式创建也可以使用Spring去代理|`<ServiceTask id="JavaServiceTask1" name="Java Service Task" activiti:delegateExpression="${leaveBackDelegate}"/>`|
|activiti:resultVariable|该属性仅适用于activit:expression类型的Java Service，可以把一个表达式的执行结果保存到activiti:resultVariable指定的变量名称中|`<serviceTask id="javaServiceTask" name="Java Service Task" activiti:resultvariable="backDate" activiti:expression="#{leaveService.back()}"/>`将leaveService.back()的执行结果保存到变量backDate中|

>在示例代码中，`activiti:expression`使用的是UEL表达式 **"#{leaveService.back()}"**指定了具体的bean名称以及bean中的方法。`activiti:delegateExpression`使用的是**"${leaveBackDelegate}"**方式在运行时动态的设置bean，并调用bean中实现`JavaDelegate`接口或`AvtivityBehavior`接口中的execute()方法。

##Web Service任务
通过Web Service任务可以调用外部Web Service资源，并且支持标准的Web Service和REST风格的Service。同Java Service一样，Web Service是通过`serviceTask`标签定义的，需要使用`implementation="##WebService"`属性标识该serviceTask任务为一个Web Service任务，Web Service任务的XML描述如下所示：

```XML
<serviceTask id="webServiceTask" name="WebService Call" implementation="##WebService">
	<ioSpecification>
		<dataInput itemSubjectRef="tns:prettyPrintCountRequestItem" id="dataInputOfServiceTask"></dataInput>
		<dataOutput itemSubjectRef="tns:prettyPrintCountResponseItem" id="dataOutputOfServiceTask"></dataOutput>
		<inputSet>
			<dataInputRefs>dataInputOfServiceTask</dataInputRefs>
		</inputSet>
		<outputSet>
			<dataOutputRefs>dataOutputOfServiceTask</dataOutputRefs>
		</outputSet>
	</ioSpecification>
	<dataInputAssociation>
		<sourceRef>PrefixVariable</sourceRef>
		<targetRef>prefix</targetRef>
	</dataInputAssociation>
	<dataInputAssociation>
		<sourceRef>SuffixVariable</sourceRef>
		<targetRef>suffix</targetRef>
	</dataInputAssociation>
	<dataOutputAssociation>
		<sourceRef>prettyPrint</sourceRef>
		<targetRef>OutputVariable</targetRef>
	</dataOutputAssociation>    
</serviceTask>
```

>* 通过 `implementation="##WebService"` 属性设置serviceTask为Web Service类型。
>* 通过 `ioSpecification` 标签定义输入、输出参数。
>* 通过 `dataInputAssociation` 标签定义数据输入关系。
>* 通过 `dataOutputAssociation` 标签定义数据输出关系。


###业务规则任务
*业务规则任务* 可以根据流程变量的值来处理预设的业务规则。在实际应用时只需要把含有 *业务规则任务* 的流程定义文件和规则引擎文件一同打包部署到系统中，同时把[Drools](http://wwww.jboss.org/drools)的jar包添加到classpath中即可实现Activiti驱动规则引擎。

> 规则引擎：简单来说就是把业务数据交由规则引擎处理，规则引擎根据不同的业务规则（各种条件的判断）计算得出最终结果，最后把结果返回给调用者。（在BPMN2.0中的调用者为流程引擎，在Activiti中的调用者即为Activiti）

业务规则任务的图形化表示如下所示：


业务规则任务的XML描述如下所示：

```XML
<businessRuleTask id="businessruletask1" name="Business rule task"></businessRuleTask>
```

Activiti扩展的业务规则任务的属性有：

|属性名称|属性说明|属性示例|
|----|----|----|----|
|activiti:rules|在规则文件“.drl”中定义的规则名称，多个规则使用逗号分隔。要执行规则文件中的全部规则，只需将该属性的值设置为空即可|`<businessRuleTask id="businessruletask1" activiti:rules="rule1,rule2"/>`|
|activiti:execlude|boolean类型，用来设置是否排除某些规则。如果值为`false`，则不排除，按照`activiti:rules`指定的规则执行；如果值为`true`，则忽略`activiti:rules`指定的规则；如果`activiti:execlude`为`true`，且`activiti:rules`的属性值为空，则不执行任何规则|`<businessRuleTask id="businessruletask1" activiti:rules="rule2" activiti:execlude="true"/>` 表示忽略规则rule2，仅执行rule1|
|activiti:ruleVariablesInput|业务规则执行时需要使用的数据源，使用 `${fooVar}` 方式定义数据源，多个数据源使用逗号分隔|`<businessRuleTask id="businessruletask1" activiti:ruleVariablesInput="${message}"/>`|
|activiti:resultVariableName|业务规则任务执行的结果保存在属性 `activiti:resultVariableName`指定的变量中，该变量的类型为`ArrayList`|`<businessRuleTask id="businessruletask1" activiti:resultVariableName="rulesOutputVariablesList"/>` |


##邮件任务
通过邮件任务可以发送邮件，其中的邮件信息通过变量方式传递，邮件任务不属于BPMN2.0规范，而是和Java Service任务类似，由Activiti扩展而来专门用于处理邮件任务，通过在`serviceTask`上添加扩展属性`activiti:type="mail"`实现邮件任务。
邮件任务的图形化表示如下所示：

邮件任务的XML描述如下所示：
```XML
<serviceTask id="mailtask1" name="Mail Task" activiti:type="mail">
</serviceTask>
```

发送邮件需要配置邮件服务器信息到流程引擎中，可以在activiti.cfg.xml定义的引擎属性中设置，配置邮件服务器所需的属性见下表：

|属性名称|是否必须|属性描述|
|----|----|----|----|
|mailServerHost|否|邮件服务器的主机名，默认为localhost|
|mailServerPort|是|SMTP通信端口，默认为25；如果使用SSL，默认为465|
|mailServerDefaultFrom|否|发件人的email，如果不设置，默认为activiti@activiti.org|
|mailServerUsername|否|邮件服务认证账号，默认为空|
|mailServerPassword|否|邮件服务认证密码，默认为空|

以上是在配置邮件服务器时需要使用的属性，如果需要使用邮件任务，还需要配置邮件任务，邮件任务统一使用activiti:field元素来配置。以下为邮件任务的属性列表：

|属性名称|属性说明|属性示例|
|----|----|----|
|to|邮件的收件人，多个收件人使用逗号分隔|`<activiti:field name="to" expression="${to}"/>`|
|from|邮件的发件人。如果不设置，则默认使用引擎的mailServerDefaultFrom属性指定的发件人|`<activiti:field name="from" expression="${from}"/>`|
|subject|邮件的主题|`<activiti:field name="subject" expression="${subject}"/>`|
|cc|抄送|`<activiti:field name="cc" expression="${cc}"/>`|
|bcc|密送|`<activiti:field name="bcc" expression="${bcc}"/>`|
|charset|邮件内容字符集，建议使用utf-8，以防止中文乱码|`<activiti:field name="charset" expression="${charset}"/>`|
|text|纯文本格式的邮件内容|`<activiti:field name="text" expression="${mailContent}"/>`|
|html|html格式的邮件内容|`<activiti:field name="html" expression="mailContent"/>`|

>在配置邮件任务的属性时，可以将属性值写死，也可以使用变量来动态设置属性值。例如配置html格式的邮件内容：
```XML
<activiti:field name="html">
  <activiti:expression>
    <![CDATA[
      <html>
        <body>
          你好，我是<b>${userName}</b>
        </body>
      </html>
    ]]>
  </activiti:expression>
</activiti:field>
```

##Camel任务
[Camel](http://camel.apache.org/)是由Apache开源组织开发的基于[Apache 2协议](http://www.apache.org/licenses/LICENSE-2.0.html)的企业系统整合框架，用来解决消息路由的框架，类似的产品还有：Mule、Spring Intergretion等。
Camel任务是使用Java语言编写的，并不包含在BPMN 2.0规范中，而是由Activiti在serviceTask的基础上扩展的一个任务模型。可以通过 `activiti:delegateExpression` 属性指定Camel的上下文，在运行时由引擎调用Camel路由处理任务。
Camel任务的图形表示如下所示：

Camel任务的XML描述如下所示：
```XML
<serviceTask id="camelTask" name="Camel Task" activiti:delegateExpression="${camel}"></serviceTask>
```

> `${camel}`表示有Activiti调用名称为camel的Bean对象处理Camel任务，该Bean对象可以定义在Activiti配置文件中或由Spring代理

##Mule任务
Mule任务和Camel任务功能类似，但是他们是基于不同的标准实现的，[Mule任务](http://www.mulesoft.org/)是由MuleSoft开发的一款轻量级开源ESB（Enterprise System Bus，企业系统总线）产品。在使用方式上Camel任务使用链式编程的方式配置路由，而Mule任务使用非入侵方式，通过可视化的IDE工具使用流程图模式配置路由。
Mule任务和Camel任务一样，也不是BPMN 2.0 规范中的一部分，而是Activiti针对Mule任务进行了整合。Activiti对Camel的扩展使用的是`serviceTask`，而对Mule任务使用了`sendTask`。
Mule任务的图形化表示如下所示：

Mule任务的XML描述如下所示：
```XML
<sendTask id="muleTask" name="Mule Task" activiti:type="mule">
    <extensionElements>
    	<activiti:fieldname="endpointUrl">
    		<activiti:string>vm://in</activiti:string>
    	</activiti:field>
    	<activiti:fieldname="language">
    		<activiti:string>juel</activiti:string>
    	</activiti:field>
    	<activiti:fieldname="payloadExpression">
    		<activiti:string>"hi"</activiti:string>
    	</activiti:field>
    	<activiti:fieldname="resultVariable">
  			<activiti:string>resultVar</activiti:string>
  		</activiti:field>
  	</extensionElements>
</sendTask>
```
> * 通过 `activiti:type="mule"` 属性来指定任务类型为MuleSoft
> * 通过 `<extensionElements>` 标签配置一系列调用Mule的参数
> * 通过 `fieldname="endpointUrl"` 属性指定协议类型为VM，类似于java虚拟机，还可以指定其他的协议来调用Mule，例如：JMS或其他远程访问协议
> * 通过 `fieldname="language"` 属性指定调用payloadExpression的语言，在实际使用时可以通过Activiti的任务变量动态设置
> * 通过 `fieldname="payloadExpression"` 指定Mule消息的payload名称
> * 通过 `fieldname="resultVariable"` 属性来保存Mule任务的执行结果，该结果可以通过Activiti API获取。

##手动任务

手动任务不做任何事情，用来定义BPMN不能完成的任务，表示该任务需要人工参与。流程引擎不需要关注如何处理它，Activiti把手动任务当成一个空任务来处理，当到达此任务时由引擎自动完成并转向下一个任务。
手动任务的图形化表示如下所示：

手动任务的XML描述如下所示：
```XML
<manualTask id="manualtask1" name="Manual Task"></manualTask>
```

##接收任务
接收任务是一个功能简单且单一的任务，在任务创建之后开始等待消息的到来，直到消息到来被触发才完成任务。实际上，Activiti把 `接收任务` 作为一个Java类型的接受任务，仅能通过`RuntimeService`接口的`signal()`方法触发`接收任务`。在调用Activiti的API触发流程的`接收任务`后，引擎把当前流程由等待状态恢复为可执行状态。
接收任务的图形化表示如下所示：

接收任务的XML描述如下所示：
```XML
<receiveTask id="receiveTask" name="Receive Task"></receiveTask>
```

##Shell任务
Shell任务允许在流程运行过程中执行本地操作系统中的脚本、命令，是Activiti基于serviceTask扩展的一种任务。
Shell任务的图形化表示如下所示：

Shell任务的XML描述如下所示：
```XML
<serviceTask id="shellTask" name="Shell Task" activiti:type="shell"></serviceTask>
```
配置Shell任务的参数和配置邮件任务类似，都是在 `<extensionElements>`标签下使用`<activiti:field>`标签配置，Shell任务的属性及其含义如下所示：

|属性名称|是否必须|属性类型|默认值|属性说明|
|----|----|----|----|----|
|command|是|String|执行的脚本、命令||
|arg0~arg5|否|String|参数，在执行的时候用空格分隔多个参数||
|wait|否|boolean|是否等待脚本执行完成|true|
|redirectError|否|boolean|是否合并错误输出至标准输出false|
|cleanEnv|否|boolean|是否继承当前的环境变量|false|
|outputVariable|否|String|保存脚本、命令执行结果的变量|空，不记录|
|errorCodeVariable|否|String|保存脚本、命令执行失败反馈的错误编码的变量|空，为绑定错误|
|directory|否|String|在哪个目录执行脚本、命令|当前项目目录|


##多实例
多实例允许业务流程中某一个任务或者子流程可以重复执行多次，多实例可以选择顺序执行，还可以选择并行执行多实例任务或者子流程。
> * 一个申请由多人审批是多实例的典型应用场景

多实例的图形化描述是在原任务的基础上添加了3个平行线（顺序执行）和3个垂直线（并行执行）。顺序执行的多实例的图形化描述如下所示：

并行执行的多实例的图形化描述如下所示：

多实例支持的任务类型有：
* 用户任务
* 脚本任务
* Java Service任务
* Web Service任务
* 业务规则任务
* 邮件任务
* 手动任务
* 接收任务
* 嵌入式子流程
* 调用活动式子流程

在BPMN 2.0规范中规定的几个多实例的属性变量如下：
* nrOfInstances ：实例总数
* nrOfActiveInstances ：当前活动的实例数量。对于顺序执行的多实例，该值总是为1
* nrOfCompleteInstances：已完成的实例数量
* loopCounter：多实例运行过程中，for-each循环中当前的索引值

多实例是通过在相应的任务下添加`<multiInstanceLoopCharacteristics>`标签实现的

多实例的配置元素及属性如下表所示：

|属性名称|属性说明|属性示例|
|----|----|----|
|isSequential|是否按照顺序执行任务，值为true时表示按照顺序执行，只有在前一个任务处理完成之后才会继续创建后一个任务；值为false时表示不按照顺序执行，而是一次创建多个任务，多个任务并行处理|`<userTask id="audi" name="任务分发" activiti:assignee="henryyan"><multiInstanceLoopCharacteristics isSequential="false"><loopCardinality>3</loopCardinality></multiInstanceLoopCharacterisTics></userTask>`|
|loopCardinality|实例的数量。可以使用常量设置，也可以使用表达式动态设置|`<userTask id="audi" name="任务分发" activiti:assignee="henryyan"><multiInstanceLoopCharacteristics isSequential="false"><loopCardinality>3</loopCardinality></multiInstanceLoopCharacterisTics></userTask>`表示一次创建三个实例。也可以使用UEL表达式动态设置：`<loopCardinality>${allTaskCounter - completedTaskCounter}</loopCardinality>`|
|loopDateInputRef|任务参与人列表，以参与人集合的数量决定创建多少个实例。***loopDateInputRef和loopCardinality是互斥的关系***|`<userTask id="audit" name="领导审批"><multiInstanceLoopCharacteristics isSequential="false"><loopDataInputRef>assignList</loopDataInputRef><inputDataItem name="assignee"/></multiInstanceLoopCharacterisTics></userTask>` `assignList`作为一个运行时变量存在，实际上是包含了一系列用户ID的数组|
|inputDataItem|需要配合`loopDateInputRef`使用,用来指定在loopDateInputRef中定义的办理人集合中循环过程中单个变量的名称|`<userTask id="audit" name="领导审批" activiti:assignee="${assignee}"><multiInstanceLoopCharacteristics isSequential="false"><loopDataInputRef>assigneeList</loopDataInputRef><inputDataItem name="assignee"/></multiInstanceLoopCharacterisTics></userTask>` 动态设置任务参与人列表，并且指定循环过程中单个变量的名称为`assignee`,然后将该用户任务动态的指派给该变量所对应的用户assignee|
|completionCondition|多实例循环结束条件，当满足一定条件时退出多实例循环。例如在投票表决场景中当通过率达到70%时结束任务，剩余未办理的任务不再执行|`<userTask id="audit" name="领导审批"><multiInstanceLoopCharacteristics isSequential="false"><completionCondition>${nrOfCompleteInstances/nrOfInstances>=0.7}</completionCondition></multiInstanceLoopCharacterisTics></userTask>`|

此外，Activiti还基于BPMN 2.0 规范以扩展的属性形式简化了配置，Activiti扩展的多实例属性如下表：

|属性名称|属性说明|属性示例|
|----|----|----|
|activiti:collection|用来简化loopDataInputRef，即任务参与人列表|`<multiInstanceLoopCharacteristics isSequential="false" activiti:collection="assigneeList"></multiInstanceLoopCharacterisTics>`|
|activiti:elementVariable|用来简化inputDataItem，即任务办理人列表在循环过程中单个变量的名称|`<multiInstanceLoopCharacteristics isSequential="false" activiti:collection="assigneeList" activiti:elementVariable="assignee"></multiInstanceLoopCharacterisTics>`|


#网关
网关用于控制流程走向（在BPMN 2.0规范中称为“执行令牌”）。根据功能不同可以划分为4中网关：
* 排他网关
* 并行网关
* 包容网关
* 事件网关

##排他网关
排他网关用来对流程中的决定进行建模。流程执行到排他网关时，按照输出流的殊勋逐个进行计算，当条件计算结果为true时，执行当前网关的输出流；如果多个线路的计算结果都为true时，只会执行第一个值为true的网关，忽略其他值为true的网关；如果多个网关计算结果没有为true的值，则引擎会排除异常。排他网关类似于java中的`if(...) else if(...) else...`语句。
排他网关的图形化表示如下所示：

排他网关的XML描述如下所示：
```XML
<exclusiveGateway id="exclusivegateway1" name="Exclusive Gateway"></exclusiveGateway>
```
在排他网关中可以使用`default=""`属性来指定默认的输出流

排他网关需要和条件顺序流配合使用，一个排他网关可以连接多个条件顺序流，每个条件顺序流设置一个条件，在运行时由引擎进行计算并根据计算结果是否为true决定执行与否。
典型的排他网关使用场景如下所示：

对应的XML描述如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<exclusiveGateway id="exclusivegateway1" name="Exclusive Gateway"></exclusiveGateway>
<sequenceFlow id="flow" sourceRef="startevent1" targetRef="exclusivegateway1"></sequenceFlow>
<userTask id="usertask1" name="任务二"></userTask>
<sequenceFlow id="flow1" name="${type==1}" sourceRef="exclusivegateway1" targetRef="usertask1">
   <conditionExpression xsi:type="tFormalExpression">
    		<![CDATA[
    			${type==1}
    		]]>
   </conditionExpression>
</sequenceFlow>
<userTask id="usertask2" name="任务一"></userTask>
<sequenceFlow id="flow2" name="${type==2}" sourceRef="exclusivegateway1" targetRef="usertask2">
  <conditionExpression xsi:type="tFormalExpression">
    		<![CDATA[
    			${type==2}
    		]]>
  </conditionExpression>
</sequenceFlow>
<userTask id="usertask3" name="任务三"></userTask>
<sequenceFlow id="flow3" name="${type==3}" sourceRef="exclusivegateway1" targetRef="usertask3">
  <conditionExpression xsi:type="tFormalExpression">
    		<![CDATA[
    			${type==3}
    		]]>
  </conditionExpression>
</sequenceFlow>
```

在这段示例中，使用了`exclusiveGateway`标签定义了排他网关，并定义了三个顺序流且统一设置条件顺序流的sourceRef的属性值为`exclusiveGateway`的ID，以次使得条件顺序流和排他网关进行关联。

> 当所有条件都不满足时（即网关计算结果都不为true时），引擎会抛出异常，因此在使用排他网关时最好添加`default=""`属性，当所有条件都不满足时，排他网关会默认执行`exclusiveGateway`的`default`属性指定的条件条件顺序流。

##并行网关
并行网关用来对并发的任务进行流程建模，它能把单条线路拆分成多个路径并行执行或将多条线路合并。具体来讲：
* 拆分 ：并行执行所有的输出顺序流，并且为每一条顺序流创建一个并行执行线路
* 合并 ：所有从并行网关拆分并执行完成的线路均在此等候，直到所有的线路都执行完成才继续向下执行。

> 并行网关不会计算线路上设置的条件，如果设置了，则会被直接忽略。

并行网关的图形化表示如下所示：

并行网关的XML描述如下所示：
```XML
<parallelGateway id="parallelgateway1" name="Parallel Gateway"></parallelGateway>
```

典型的并行网关用例如图所示：

对应的XML描述如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="请假申请"></userTask>
<sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
 <parallelGateway id="parallelgateway1" name="Parallel Gateway"></parallelGateway>
<sequenceFlow id="flow2" sourceRef="usertask1" targetRef="parallelgateway1"></sequenceFlow>
<userTask id="usertask2" name="部门领导审批"></userTask>
<userTask id="usertask3" name="人事审批"></userTask>
<sequenceFlow id="flow3" sourceRef="parallelgateway1" targetRef="usertask2"></sequenceFlow>
<sequenceFlow id="flow4" sourceRef="parallelgateway1" targetRef="usertask3"></sequenceFlow>
<parallelGateway id="parallelgateway2" name="Parallel Gateway"></parallelGateway>
<sequenceFlow id="flow5" sourceRef="usertask2" targetRef="parallelgateway2"></sequenceFlow>
<sequenceFlow id="flow6" sourceRef="usertask3" targetRef="parallelgateway2"></sequenceFlow>
<serviceTask id="servicetask1" name="考勤归档"></serviceTask>
<sequenceFlow id="flow7" sourceRef="parallelgateway2" targetRef="servicetask1"></sequenceFlow>
<endEvent id="endevent1" name="End"></endEvent>
<sequenceFlow id="flow8" sourceRef="servicetask1" targetRef="endevent1"></sequenceFlow>
```

并行网关还允许在线路上嵌套并行网关，即在并行网关拆分出的线路上再添加若干个并行网关，只要保证最后有一个并行网关合并拆分的线路即可。所有并行网关不需要“成对出现”，只要保证“有始有终”即可。示例如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="请假申请"></userTask>
<sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
<parallelGateway id="parallelgateway1" name="Parallel Gateway"></parallelGateway>
<sequenceFlow id="flow2" sourceRef="usertask1" targetRef="parallelgateway1"></sequenceFlow>
<userTask id="usertask2" name="部门领导审批"></userTask>
<userTask id="usertask3" name="人事审批"></userTask>
<userTask id="usertask4" name="系统记录"></userTask>
<sequenceFlow id="flow3" sourceRef="parallelgateway1" targetRef="usertask2"></sequenceFlow>
<sequenceFlow id="flow4" sourceRef="parallelgateway1" targetRef="usertask3"></sequenceFlow>
<sequenceFlow id="flow5" sourceRef="parallelgateway1" targetRef="usertask4"></sequenceFlow>
<parallelGateway id="parallelgateway2" name="Parallel Gateway"></parallelGateway>
<sequenceFlow id="flow6" sourceRef="usertask2" targetRef="parallelgateway2"></sequenceFlow>
<sequenceFlow id="flow7" sourceRef="usertask3" targetRef="parallelgateway2"></sequenceFlow>
<parallelGateway id="parallelgateway3" name="Parallel Gateway"></parallelGateway>
<sequenceFlow id="flow8" sourceRef="usertask4" targetRef="parallelgateway3"></sequenceFlow>
<serviceTask id="servicetask1" name="Service Task"></serviceTask>
<sequenceFlow id="flow9" sourceRef="parallelgateway2" targetRef="servicetask1"></sequenceFlow>
<sequenceFlow id="flow10" sourceRef="parallelgateway3" targetRef="servicetask1"></sequenceFlow>
<endEvent id="endevent1" name="End"></endEvent>
<sequenceFlow id="flow11" sourceRef="servicetask1" targetRef="endevent1"></sequenceFlow>
```

对应的图形化表示如下所示：




##包容网关
包容网关融合了排他网关和并行网关的特性，排他网关允许在每条线路上设置条件；并行网关可以同时执行多条线路，包容网关既可以同时执行多条线路，又可以在网关上设置条件。
包容网关的图形化表示如下所示：

包容网关的XML描述如下所示：
```XML
<inclusiveGateway id="inclusivegateway1" name="Inclusive Gateway"></inclusiveGateway>
```

包容网关的功能取决于输入顺序流和输出顺序流：
* 计算每条线路上的表达式，当表达式的计算结果为true时，创建一个并行线路并执行
* 所有从并行网关拆分并执行完成的线路均在此等候，直到所有的线路都执行完成才继续向下执行。
基于包容网关实现的请假流程图如下所示：

>该请假流程在启动的时候根据条件顺序流判断是否需要领导或人事审批，如果两个条件都满足，则“部门领导审批”和“人事审批”都会被创建

对应的XML描述如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="请假申请"></userTask>
<sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
<inclusiveGateway id="inclusivegateway1" name="Inclusive Gateway"></inclusiveGateway>
<sequenceFlow id="flow2" sourceRef="usertask1" targetRef="inclusivegateway1"></sequenceFlow>
<userTask id="usertask2" name="部门领导审批"></userTask>
<userTask id="usertask3" name="人事审批"></userTask>
<sequenceFlow id="flow3" sourceRef="inclusivegateway1" targetRef="usertask2"></sequenceFlow>
<sequenceFlow id="flow4" sourceRef="inclusivegateway1" targetRef="usertask3"></sequenceFlow>
<inclusiveGateway id="inclusivegateway2" name="Inclusive Gateway"></inclusiveGateway>
<sequenceFlow id="flow5" sourceRef="usertask2" targetRef="inclusivegateway2"></sequenceFlow>
<sequenceFlow id="flow6" sourceRef="usertask3" targetRef="inclusivegateway2"></sequenceFlow>
<serviceTask id="servicetask1" name="考勤归档"></serviceTask>
<sequenceFlow id="flow7" sourceRef="inclusivegateway2" targetRef="servicetask1"></sequenceFlow>
<endEvent id="endevent1" name="End"></endEvent>
<sequenceFlow id="flow8" sourceRef="servicetask1" targetRef="endevent1"></sequenceFlow>
```

##事件网关
事件网关是专门为 *中间捕获事件* 设置的。事件网关允许设置多个输出流来指向不同中间捕获事件。当流程执行到事件网关后，当前流程处于“等待状态”（因为中间捕获事件需要依赖中间抛出事件触发，才能更改“等待状态”为“活动状态”）。
事件网关的图形化表示如下所示：

事件网关的XML描述如下所示：
```XML
<eventBasedGateway id="eventgateway1" name="Event Gateway"></eventBasedGateway>
```
典型的事件网关的应用场景如图所示：

>在该场景中，分别定义了*定时器中间捕获事件*和*信号中间捕获事件*。在流程启动之后两个输入流均处于“等待”状态。中间捕获事件在满足了条件之后会被自动触发继续执行输出流的活动。

对应的XML描述如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<eventBasedGateway id="eventgateway1" name="Event Gateway"></eventBasedGateway>
<intermediateCatchEvent id="timerintermediatecatchevent1" name="定时事件">
  <timerEventDefinition>PT1S</timerEventDefinition>
</intermediateCatchEvent>
<sequenceFlow id="flow1" sourceRef="eventgateway1" targetRef="timerintermediatecatchevent1"></sequenceFlow>
<intermediateCatchEvent id="signalintermediatecatchevent1" name="信号捕获事件">
  <signalEventDefinition signal="alertSignal"></signalEventDefinition>
</intermediateCatchEvent>
<sequenceFlow id="flow2" sourceRef="eventgateway1" targetRef="signalintermediatecatchevent1"></sequenceFlow>
<scriptTask id="scripttask1" name="定时任务被触发后执行" activiti:autoStoreVariables="false" scriptFormat="groovy">
  <script><![CDATA[out:println "after time event"]]></script>
</scriptTask>
<serviceTask id="servicetask1" name="捕获到信号事件后执行" scriptFormat="groovy">
  <script><![CDATA[out:println "after signal event"]]></script> 
</serviceTask>
<sequenceFlow id="flow3" sourceRef="timerintermediatecatchevent1" targetRef="scripttask1"></sequenceFlow>
<sequenceFlow id="flow4" sourceRef="signalintermediatecatchevent1" targetRef="servicetask1"></sequenceFlow>
<endEvent id="endevent1" name="End"></endEvent>
<sequenceFlow id="flow5" sourceRef="scripttask1" targetRef="endevent1"></sequenceFlow>
<sequenceFlow id="flow6" sourceRef="servicetask1" targetRef="endevent1"></sequenceFlow>
<sequenceFlow id="flow7" sourceRef="startevent1" targetRef="eventgateway1"></sequenceFlow>
```

关于事件网关需要***注意***一下几点：
* 事件网关的输出流数量必须大于等于两个
* 事件网关的输出流类型只能是中间捕获事件
* 中间捕获事件的输出流只能有一个

#子活动与调用过程
在实际的应用中，业务流程通常比较复杂，一般将复杂的业务流程划分成多个不同的模块，可以把这些模块划分为一个子流程作为主流程的一部分；由于子流程是嵌套在主流程中，因此为了保持某些模块的通用性，需要把某些模块作为活动由主流程“调用”，这就是调用活动。`调用活动`即包含了子流程的“封装”的特性，又具有通用性。

##子流程
把一系列需要处理的任务归结到一起，作为一个大流程中的一部分，可以把子流程作为一个独立的流程来理解，只不过有一些限制和表现形式不同而已。因为子流程嵌入在主流程，所以也把子流程称为“嵌入式子流程“。
子流程需要把一些模型包含在一个实线矩形中，其图形化表示如下所示：

子流程的XML描述如下所示：
```XML
<subProcess id="subprocess1" name="Sub Process"></subProcess>
```
注意：
> * 在BPMN 2.0中，只能且仅能包含一个空启动事件
> * 在BPMN 2.0中，至少有一个结束事件
> * 在BPMN 2.0中，子流程中的顺序流不能直接设置输出流到子流程之外的活动上，如果需要，可以使用边界事件代替
> * 在BPMN 2.0中，允许子流程不包含启动事件、结束事件；在Activiti中，子流程必须包含启动事件、结束事件

子流程的一个简单示例如图所示：

对应的XML描述如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="下单"></userTask>
<sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
<subProcess id="subprocess1" name="付款子流程">
    <startEvent id="startevent2" name="Start"></startEvent>
    <userTask id="usertask2" name="向银行付款"></userTask>
    <sequenceFlow id="flow2" sourceRef="startevent2" targetRef="usertask2"></sequenceFlow>
    <userTask id="usertask3" name="发送通知邮件"></userTask>
    <sequenceFlow id="flow3" sourceRef="usertask2" targetRef="usertask3"></sequenceFlow>
    <endEvent id="endevent1" name="End"></endEvent>
    <sequenceFlow id="flow4" sourceRef="usertask3" targetRef="endevent1"></sequenceFlow>
</subProcess>
<sequenceFlow id="flow5" sourceRef="usertask1" targetRef="subprocess1"></sequenceFlow>
```
> * 在流程运行中，流程引擎会自动为主流程和子流程建立关联关系，子流程可以通过API获取主流程的一些信息及变量。
> * 子流程支持多实例特性。例如部门领导为几个业务员工分配任务，而每项任务都是一个子流程，部门领导下发任务之后每个业务员工都会拥有一个独立的子流程实例，等所有的业务员工都处理完成之后再汇总给部门领导。

##调用活动
由于子流程是嵌套在主流程中的，所以子流程不具有通用性，而`调用活动`解决的问题就是子流程通用的局限性。调用活动的图形化表示如下所示：

调用活动的XML描述如下所示：
```XML
<callActivity id="callactivity1" name="Call activity"></callActivity>
```
在使用调用活动时，只需要创建一个调用活动并指定调用活动的ID作为主流程的一个活动即可。简单示例如下所示：

对应的XML描述如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="用户下单"></userTask>
<sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
<callActivity id="callactivity1" name="Call activity" callElement="payment">
  <extensionElements>
    <activiti:in source="amount" target="amount"/>
    <activiti:out source="paid" target="paid"/>
    <activiti:out source="${payTime" target="payTime"/>
  </extensionElements>
</callActivity>
<sequenceFlow id="flow2" sourceRef="usertask1" targetRef="callactivity1"></sequenceFlow>
<endEvent id="endevent1" name="End"></endEvent>
<sequenceFlow id="flow3" sourceRef="callactivity1" targetRef="endevent1"></sequenceFlow>
```
调用活动的属性见下表：

|属性名称|属性说明|属性示例|
|----|----|----|
|callElement|调用活动名称，用于主流程调用，每个调用活动应该唯一|<callActivity id="callactiviti " name="Call Activiti" calledElement="payment"></callActivity>|
|activiti:in|外部流程传入的变量|`<activiti:in source="amount" target="amount"/>` **source**：主流程中的变量名称，**target** ：将变量传递给调用活动时，在调用活动中使用的变量名称|
|activiti:out|调用活动执行完成之后的结果|`<activiti:out source="paid" target="paid">` **source** :主流程中的变量名称，用于存储计算结果，**target**:将处理结果传递给主流程时的变量名|

###事件子流程
事件子流程与子流程类似，不同的是事件子流程不能直接启动，而是由其他事件触发启；子流程的图形化表示使用的是实线矩形，而事件子流程使用的是虚线矩形；子流程嵌套在主流程中，作为主流程输出流中的一个输出活动，而事件子流程是独立在主流程中的。
事件子流程的图形化表示如下所示：

对应的XML描述吐下所示：
```XML
<subProcess id="eventSubPrcess" name="支付失败--完善账号信息" triggeredByEvent="true">
</subProcess>
```
> 事件子流程与子流程在XML上的区别是：添加triggeredByEvent属性表示此子流程只能由事件触发后被动启动

可以触发事件子流程的事件有：
* 异常事件
* 信号事件
* 消息事件
* 定时器事件
* 补偿事件
事件子流程的一个简单应用如下图所示：

对应的XML描述如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="支付费用"></userTask>
<sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
<exclusiveGateway id="exclusivegateway1" name="Exclusive Gateway"></exclusiveGateway>
<sequenceFlow id="flow2" sourceRef="usertask1" targetRef="exclusivegateway1"></sequenceFlow>
<endEvent id="endevent1" name="End"></endEvent>
<sequenceFlow id="flow3" sourceRef="exclusivegateway1" targetRef="endevent1"></sequenceFlow>
<endEvent id="errorendevent1" name="ErrorEnd">
      <errorEventDefinition>A001</errorEventDefinition>
</endEvent>
<sequenceFlow id="flow4" sourceRef="exclusivegateway1" targetRef="errorendevent1"></sequenceFlow>
<subProcess id="eventsubprocess1" name="支付费用失败--完善账号信息" triggeredByEvent="true">
      <startEvent id="errorstartevent1" name="Error start">
        <errorEventDefinition>A001</errorEventDefinition>
      </startEvent>
      <userTask id="usertask2" name="完善账号信息"></userTask>
      <endEvent id="endevent2" name="End"></endEvent>
      <sequenceFlow id="flow5" sourceRef="errorstartevent1" targetRef="usertask2"></sequenceFlow>
      <sequenceFlow id="flow6" sourceRef="usertask2" targetRef="endevent2"></sequenceFlow>
</subProcess>
```
> 在这个简单应用中，事件子流程与排他网关、异常结束事件结合使用：通过排他网关判断正常情况与异常情况，在异常情况下执行异常结束事件，抛出编号为A001的异常，由于定义的异常编号相同，所以会触发事件子流程中的异常启动事件。

###事务子流程
事务子流程也被称为”事务块“，用来处理一组必须在同一个事务中完成的活动，这些活动要么一起成功，要么一起失败。事务子流程中的活动具有[ACID](http://zh.wikiedia.org/zh-hk/ACID)特性，如果其中有一个活动失败或取消，则整个事务子流程的所有活动全部回滚。
事务子流程的图形化表示如下所示：

对应的XML描述如下所示：
```XML
<transactioin id="transaction">

</transaction>
```
典型的事务子流程的应用场景如下图所示：


> 根据业务流程，可以将执行结果划分为以下几种：
* 事务成功：事务子流程中的两个全部执行成功，正常结束
* 事务失败：事务子流程在执行时出现了不可处理的异常，既不能补偿不能回滚，通过边界异常捕获事件捕获到异常，然后执行“记录系统异常信息“的任务，然后结束流程。
* 事务取消：在执行到“向收款方账户汇款“的任务时发生了异常，通过附加在对应边界上的`边界异常捕获事件`捕获异常，然后执行`取消结束事件`,改取消结束事件被附加在事件子流程中的`边界取消结束事件`捕获，然后执行”通知客户汇款失败“的任务，与此同时会触发发夹在”从付款方扣除金额“任务上的`边界补偿事件`然后执行”退回已扣金额“的补偿任务。

对应的XML描述如下所示：
```XML

```

##中间事件
中间事件提供的特殊功能可以用来处理流程执行过程中抛出、捕获的事件。中间事件包括：
* 边界事件
* 中间捕获事件
* 中间抛出事件

###边界事件
边界事件是绑定在活动上的“捕获型”事件，会一直监听所有处于运行中活动的某种事件的触发。
* 在捕获到事件之后中断活动，然后从边界事件类型的输出流继续执行
* 一旦触发边界事件，当前的活动就会被中断，然后按照边界事件之后的输出流执行
* 每个边界事件类型都是通过属性`attachedToRef`指定“附加“到抛出边界事件的活动上
边界事件的基本XML描述如下所示：
```XML
<boundaryEvent id="boundaryEvent" attachedToRef="someActivity">
  <xxxEventDefinition />
</boundaryEvent>
```
> `cancelActivity`属性可以取true、false两个值，用来指定在捕获到边界事件之后是否取消执行输出流指定的活动，但是这个属性仅适用于部分边界事件。

####定时器边界事件
定时器边界事件需要附属在一个非自动任务、调用活动、子流程上，在上游任务执行完成之后开始倒计时预设的时间，到达预设事件之后触发定时器边界事件的输出流（输出流可以是并行的）。
定时器边界事件的图形化表示如下所示：

对应的XML描述如下所示：
```XML
<boundaryEvent id="boundayTime" cancelActivity="false" attachedToRef="hrAudit"><timerEventDefinition><timeDuration>P3D</timeDuration></timerEventDefinition></boundaryEvent>
```
定时器边界事件的简单应用如下图所示：

![定时器边界事件的简单应用](http://d.pr/i/fxVu/timeBoundaryEvent.png)

对应的XML描述如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="部门领导审批"></userTask>
<sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
<userTask id="usertask2" name="人事审批"></userTask>
<sequenceFlow id="flow2" sourceRef="usertask1" targetRef="usertask2"></sequenceFlow>
<userTask id="usertask3" name="销假"></userTask>
<sequenceFlow id="flow3" sourceRef="usertask2" targetRef="usertask3"></sequenceFlow>
<endEvent id="endevent1" name="End"></endEvent>
<sequenceFlow id="flow4" sourceRef="usertask3" targetRef="endevent1"></sequenceFlow>
<boundaryEvent id="boundarytimer1" name="Timer" attachedToRef="usertask2" cancelActivity="true">
      <timerEventDefinition>
        <timeDuration>P3D</timeDuration>
      </timerEventDefinition>
</boundaryEvent>
<serviceTask id="servicetask1" name="发送提醒邮件"></serviceTask>
<endEvent id="endevent2" name="End"></endEvent>
<sequenceFlow id="flow5" sourceRef="boundarytimer1" targetRef="servicetask1"></sequenceFlow>
<sequenceFlow id="flow6" sourceRef="servicetask1" targetRef="endevent2"></sequenceFlow>
```

####异常边界事件
异常边界事件用来捕获 **嵌入子流程 **或** 调用活动 **抛出的异常，异常在抛出之后被异常边界事件捕获，同时嵌入子流程中的活动或调用活动中的活动也被中断执行
异常边界事件的图形化表示如下所示：

对应的XML描述如下所示：
```XML
<boundaryEvent id="errorBoundaryEvent" cancelActivity="false" attachToRef="subprocess"><errorEventDefinition errorRef="A001" /></boundaryEvent>
```
异常边界事件的简单应用如下所示：
![异常边界事件的简单应用](http://d.pr/i/zWio/errorBoundaryEvent.png)
对应的XML描述如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="用户下单"></userTask>
<subProcess id="subprocess1" name="付款子流程">
      <startEvent id="startevent2" name="Start"></startEvent>
      <userTask id="usertask2" name="向银行付款"></userTask>
      <exclusiveGateway id="exclusivegateway1" name="Exclusive Gateway"></exclusiveGateway>
      <serviceTask id="mailtask1" name="邮件通知" activiti:type="mail"></serviceTask>
      <endEvent id="endevent1" name="End"></endEvent>
      <sequenceFlow id="flow1" sourceRef="mailtask1" targetRef="endevent1"></sequenceFlow>
      <sequenceFlow id="flow2" sourceRef="exclusivegateway1" targetRef="mailtask1"></sequenceFlow>
      <sequenceFlow id="flow3" sourceRef="startevent2" targetRef="usertask2"></sequenceFlow>
      <sequenceFlow id="flow4" sourceRef="usertask2" targetRef="exclusivegateway1"></sequenceFlow>
      <endEvent id="errorendevent1" name="ErrorEnd">
        <errorEventDefinition>A001</errorEventDefinition>
      </endEvent>
      <sequenceFlow id="flow5" sourceRef="exclusivegateway1" targetRef="errorendevent1"></sequenceFlow>
</subProcess>
<boundaryEvent id="boundaryerror1" name="Error" attachedToRef="subprocess1">
      <errorEventDefinition>A001</errorEventDefinition>
</boundaryEvent>
<serviceTask id="servicetask1" name="处理异常"></serviceTask>
<sequenceFlow id="flow6" sourceRef="boundaryerror1" targetRef="servicetask1"></sequenceFlow>
<sequenceFlow id="flow7" name="重新付款" sourceRef="servicetask1" targetRef="subprocess1"></sequenceFlow>
```

####信号边界事件
信号边界事件可以用来捕获流程执行过程中抛出的信号，信号边界事件可以“附加”在各种活动和子流程上。信号边界事件不仅可以捕获本流程的信号，还可以捕获到其他流程的信号事件，而且如果在一个活动或子流程上定义了多个信号边界事件并同时监听同一个信号，则会同时出发，因为对应的信号抛出事件是全局的。
信号边界事件的图形化表示如下所示：

对应的XML描述如下所示：
```XML
<boundaryEvent id="boundary" attachedToRef="subprocess1" cancelActivity="true"><signalEventDefinition signalRef="A001" /><boundaryEvent>
```

####取消边界事件
**取消边界事件是专门针对事务子流程所设立的，用来捕获子流程中抛出的取消事件**
取消边界事件的XML描述如下所示：
```XML
<boundaryEvent id="cancelBoundaryEvent" attachedToRef="transactionsubprocess1"><cancelEventDefinition /><boundaryEvent>
```
> 取消边界事件不需要设置cancelActiviti属性，因为它总是”中断“取消边界事件附加任务后续活动的执行。

取消边界事件在结合事务子流程时需要注意的方面：
* 一个事务子流程只允许附加一个取消边界事件
* 对于多实例的事务子流程，如果其中一个实例触发了取消边界事件，那么其他的实例也同样会触发取消边界事件
* 如果事务子流程中嵌套了子流程，仅仅触发已经完成了的子流程的补偿事件。

####补偿边界事件
补偿边界事件用于事务子流程中针对事务失败后的业务逻辑进行补偿。补偿边界事件的输出流不是一个顺序流而是一个“关联”，用虚线表示。
补偿边界事件的XML描述如下所示：
```XML
<boundaryEvent id="Event_1" name="捕获取消结束事件" attachedToRef="task_1"><compensateEventDefinition id="eventDef_1" /></boundaryEvent>
<association id="association_1" sourceRef="event_1" targetRef="task_2" associationDirection="One" />
<serviceTask id="task_2" name="退回已扣金额" isForCompensation="true" />
```
>在代码中，使用`<compensateEventDefinition>`来标识一个补偿边界事件;用`<association>`关联两个对象，和顺序流类似，使用sourceRef、targetRef表示两个关联活动的ID；使用`isFOrCOmpensation`属性，用来声明这时一个补偿类型的活动。

###中间捕获事件
根据事件的不同，中间事件需要使用不同的方式才能继续执行后续的输出流的活动；中间事件必须连接一个输入流和一个输出流。
中间捕获事件的基本XML描述如下所示：
```XML
<intermediateCatchEvent id="event1">
  <xxxxEventDefinition />
</intermediateCatchEvent>
```

####定时器中间捕获事件
定时器中间捕获事件是在一个特定的时间或指定间隔多长时间之后触发，从而执行输出流后面的活动。
定时器中间捕获事件的图形化表示和定时器边界事件的图形化表示相同，只是两种事件所处位置不同，定时器中间捕获事件的图形化表示如下所示：
![定时器中间捕获事件](http://d.pr/i/1Bfk/timerIntermediateCatchEvent.png)
定时器中间捕获事件的XML描述如下所示：
```XML
<intermediateCatchEvent id="timerintermediatecatchevent1" name="TimerCatchEvent">
  <timerEventDefinition>
    <timeDuration>PT5M</timeDuration>
  </timerEventDefinition>
</intermediateCatchEvent>
```

定时器中间捕获事件的简单应用如下图所示：
![定时器中间捕获事件的典型应用](http://d.pr/i/4TQE/useTimerIntermendiateCatchEvent.png)
对应的XML如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="任务一"></userTask>
<userTask id="usertask2" name="任务二"></userTask>
<endEvent id="endevent1" name="End"></endEvent>
<intermediateCatchEvent id="timerintermediatecatchevent1" name="TimerCatchEvent">
      <timerEventDefinition>
      	<timeDuration>PT5M</timeDuration>
      </timerEventDefinition>
</intermediateCatchEvent>
<sequenceFlow id="flow1" sourceRef="usertask1" targetRef="timerintermediatecatchevent1"></sequenceFlow>
<sequenceFlow id="flow2" sourceRef="timerintermediatecatchevent1" targetRef="usertask2"></sequenceFlow>
<sequenceFlow id="flow3" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
<sequenceFlow id="flow4" sourceRef="usertask2" targetRef="endevent1"></sequenceFlow>
```

***当流程执行到定时器中间捕获事件后，流程会被暂时中止一定时间****

####信号中间捕获事件
信号中间捕获事件用来捕获被当前流程或其他流程抛出的信号事件，捕获的条件就是信号的ID一致。**当抛出的信号事件匹配到多个相同的信号捕获事件时，每个匹配到的事件均会被执行，类似于广播*。
信号捕获事件的图形化表示如下所示:

![信号中间捕获事件](http://d.pr/i/9Tgh/signalIntermediateCatchEvent.png)
对应的XML描述如下所示：
```XML
<intermediateCatchEvent id="signalintermediatecatchevent1" name="SignalCatchEvent">
      <signalEventDefinition signalRef="A001"></signalEventDefinition>
</intermediateCatchEvent>
```
信号捕获事件的简单应用如下所示：

![信号中间捕获事件的简单应用](http://d.pr/i/gU5N/useSignalIntemediateCatchEvent.png)
>在“任务一”的用户任务完成之后，流程到达信号中间捕获事件，此时流程处于等待状态，一直接收到等待的信号才能继续执行。

对应的XML描述如下所示：
```XML
<signal id="signal-001" name="触发信号中间捕获事件的信号"
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="任务一"></userTask>
<intermediateCatchEvent id="signalintermediatecatchevent1" name="SignalCatchEvent">
      <signalEventDefinition signalRef="signal-001"></signalEventDefinition>
</intermediateCatchEvent>
<endEvent id="endevent1" name="End"></endEvent>
<sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
<sequenceFlow id="flow2" sourceRef="usertask1" targetRef="signalintermediatecatchevent1"></sequenceFlow>
<sequenceFlow id="flow3" sourceRef="signalintermediatecatchevent1" targetRef="endevent1"></sequenceFlow>
```

###消息中间捕获事件
消息中间捕获事件和信号中间捕获事件类似，不同的是信号中间捕获事件是“广播式”传播，而消息中间捕获事件是定向一对一的传递。
消息中间捕获事件的图形化表示如下所示：

对应的XML描述如下所示：
```XML
<intermediateCatchEvent id="messageintermediatecatchevent1" name="MessageCatchEvent">
      <messageEventDefinition messageRef="message-001"></messageEventDefinition>
</intermediateCatchEvent>
```

消息捕获事件的简单应用如下所示：


对应的XML描述如下所示：
```XML
<message id="message-001" name="触发消息中间捕获事件的消息" />
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="任务一"></userTask>
<intermediateCatchEvent id="messageintermediatecatchevent1" name="MessageCatchEvent">
      <messageEventDefinition messageRef="message-001"></messageEventDefinition>
</intermediateCatchEvent>
<endEvent id="endevent1" name="End"></endEvent>
<sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
<sequenceFlow id="flow2" sourceRef="usertask1" targetRef="messageintermediatecatchevent1"></sequenceFlow>
<sequenceFlow id="flow3" sourceRef="messageintermediatecatchevent1" targetRef="endevent1"></sequenceFlow>
```
>在第一个用户任务“任务一”完成之后，用过输出流流转到消息中间捕获事件，使流程处于等待状态，当此流程实例收到消息之后，流程重新被激活继续执行后续活动

###中间抛出事件
中间抛出事件和中间捕获事件是两个相互依赖的关系，中间捕获事件需要有事件抛出才能被触发，而中间抛出事件需要有对应的捕获事件接收才有意义。
中间抛出事件一般用在一个任务完成后需要发送通知或执行其他系统任务的场景，工作流引擎会对抛出的事件进行广播。
中间抛出事件的基本XML描述如下所示：
```XML
<intermateThrowEvent id="throwEvent">
  <xxxEventDefinition />
</intermateThrowEvent>
```

####空中间抛出事件
空中间抛出事件中没有任何功能，因此执行到空中间抛出事件时直接跳过。
空中间抛出事件的图形化表示如下所示：

空中间抛出事件的XML描述如下所示：
```XML
<intermediateThrowEvent id="noneintermediatethrowevent1" name="NoneThrowEvent"></intermediateThrowEvent>
```

从业务功能上讲，可以把空中间抛出事件作为中间状态，借助Activiti的扩展功能为空中间抛出事件添加监听器来执行我们预设的功能。
如下代码为空中间抛出事件添加了监听器：
```XML
<intermediateThrowEvent id="noneIntermediateThrowEvent" name="添加了监听器的空中间抛出事件">
  <extensionElements>
    <activiti:extensionListener class="yourClass"></activiti:extensionListener>
  </extensionElements>
</intermediateThrowEvent>
```

####信号中间抛出事件
信号中间抛出事件可以抛出一个信号，然后交给引擎传播信号事件。信号中间抛出事件的图形化表示如下所示：

信号中间抛出事件的XML描述如下所示：
```XML
<intermediateThrowEvent id="signalintermediatethrowevent1" name="SignalThrowEvent">
      <signalEventDefinition signalRef="A001"></signalEventDefinition>
</intermediateThrowEvent>
```

信号中间抛出事件的简单应用如下图所示：

对应的XML描述如下所示：
```XML
<signal id="signal-001" name="发送通知邮件"></signal>
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="任务一"></userTask>
<userTask id="usertask2" name="任务二"></userTask>
<endEvent id="endevent1" name="End"></endEvent>
<intermediateThrowEvent id="signalintermediatethrowevent1" name="SignalThrowEvent">
      <signalEventDefinition signalRef="signal-001"></signalEventDefinition>
</intermediateThrowEvent>
<sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
<sequenceFlow id="flow2" sourceRef="usertask1" targetRef="signalintermediatethrowevent1"></sequenceFlow>
<sequenceFlow id="flow3" sourceRef="signalintermediatethrowevent1" targetRef="usertask2"></sequenceFlow>
<sequenceFlow id="flow4" sourceRef="usertask2" targetRef="endevent1"></sequenceFlow>
```
>信号中间抛出事件可以使用异步方式、同步方式（默认是同步方式）。在同步方式下，在“任务一”执行完成之后直接抛出信号事件并等待匹配的信号捕获事件执行完成，如果在执行过程中抛出异常，那么其他已经执行过的信号事件全部回滚，否则继续执行信号中间抛出事件的输出流；在异步方式下，不会等待其他信号事件的执行结果而继续执行信号中间抛出事件的输出流。

信号中间抛出事件的异步方式的XML描述：
```XML
<intermediateThrowEvent id="signalThrowEvent" name="信号中间抛出事件">
  <signalEventDefinition signalRef="signal-001" activiti:async="true"></signalEventDefinition>
</intermateThrowEvent>
```


###监听器

####执行监听器
执行监听器允许在执行流程时执行java代码或表达式，执行监听器可以捕获的事件有：
* 流程实例启动、结束
* 活动启动、结束
* 路由开始、结束
* 中间事件开始、结束
* 触发开始事件、触发结束事件
* 输出流捕获
监听器都是在对应的监听对象内通过在`<extensionElements>`中添加`activiti:executionListener`标签来实现的。通过event属性指定监听事件的类型，并且可以选择监听器的执行类型。监听器的执行类型如下表所示：

|监听器执行类型|属性说明|属性示例|
|----|----|----|
|class|类执行类型，该类需要实现org.activiti.engine.delegate.ExecutionListener|`<activiti:extensionListener enevt="start" class="yourClass" />`|
|expression|表达式执行类型|`<activiti:extensionListener event="start" expression="${pojo.method(execution.eventName)}" />` pojo是一个Bean的名称，可以由Spring进行代理|
|delegateExpression|动态执行类型|class属性必须显示指定一个class的全路径，delegateExpression属性允许指定一个实现了监听接口的类的name`<activiti:executionListener event="start" delegateExpression="${executionListenerBean}" />|

执行监听器的简单示例如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="任务一"></userTask>
<userTask id="usertask2" name="任务二"></userTask>
<endEvent id="endevent1" name="End"></endEvent>
<sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
<sequenceFlow id="flow2" sourceRef="usertask1" targetRef="usertask2">
    	<extensionElements>
    		<activiti:executionListener event="toke" class="you.class">
    		
    		</activiti:executionListener>
    	</extensionElements>
</sequenceFlow>
<sequenceFlow id="flow3" sourceRef="usertask2" targetRef="endevent1"></sequenceFlow>
```

此外，还可以在监听中使用`<activiti:field>`注入字段，在流程运行时监听实现类可以通过接口中的对象获取字段的值。
简单示例如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
    <userTask id="usertask1" name="任务一"></userTask>
    <userTask id="usertask2" name="任务二"></userTask>
    <endEvent id="endevent1" name="End"></endEvent>
    <sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
    <sequenceFlow id="flow2" sourceRef="usertask1" targetRef="usertask2">
    	<extensionElements>
    		<activiti:executionListener event="toke" class="you.class">
    			<activiti:field name="firstVar" stringValue="Henry Yan"></activiti:field>
    			<activiti:field name="secondVar" expression="${'user'+counter}"></activiti:field>
    		</activiti:executionListener>
    	</extensionElements>
    </sequenceFlow>
    <sequenceFlow id="flow3" sourceRef="usertask2" targetRef="endevent1"></sequenceFlow>
```
>通过`activiti:field`字段为监听器注入了名称为firstVar的变量，这样在监听器中就可以通过Activiti引擎的方法获取变量的值，此外也可以通过表达式的方式为变量赋值。

####任务监听器
任务监听器只能应用于用户任务，用来监听以下三种事件：
* create ： 在任务被创建且所有的任务属性被设置完成之后触发
* assignment ： 在任务被分配给某个便利人之后触发（assignment事件总是在create事件之前被触发，因为任务的办理人是一种属性）
* complete ： 在任务完成时触发
任务监听器的三种监听器的执行类型：

|监听器执行类型|属性说明及示例|
|----|----|
|class|需要实现接口：org.activiti.engine.delegate.TaskListener `<extensionElements><activiti:taskListener class="you.class"/></extensionElements>`|
|expression|定义一个表达式 `<extensionElements><activiti:taskListener expression="${pojo.method(execution.eventName)}" event="complete"/></extensionElements>`|
|delegateExpression|动态指定监听器 `<extensionElements><activiti:taskListener delegateExpression="${taskListenerBean}" event="start"/></extensionElements>` taskListenerBean是一个beand的名称，可以由Spring代理|
任务监听器的简单示例如下所示：

对应的XML描述如下所示：
```XML
<startEvent id="startevent1" name="Start"></startEvent>
<userTask id="usertask1" name="任务一">
      <extensionElements>
    		<activiti:taskListener event="complete" class="you.class">
    			
    		</activiti:taskListener>
    	</extensionElements>
</userTask>
<endEvent id="endevent1" name="End"></endEvent>
<sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"></sequenceFlow>
<sequenceFlow id="flow2" sourceRef="usertask1" targetRef="endevent1"></sequenceFlow>
```
>通过属性`event="complete"`设置当任务完成时触发，从而执行监听器类
此外，任务监听器与执行监听器类似，都可以通过`<activiti:field>`为监听器注入字段。


##Markdown实时预览
