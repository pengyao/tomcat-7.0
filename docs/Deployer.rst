Tomcat Web应用部署
==================================

.. contents::



简介
--------------------

部署指的是在Tomcat server中安装web应用(第三方war包或者自定义的web应用)的过程.

在Tomcat server中可以通过以下方式实现部署web应用.

   * 静态部署: Tomcat完成启动前应用已经安装(setup)
   * 动态部署: 通过连接Tomcat Manager web应用完成部署或者利用已经部署的web应用 (译者注：或后边的内容尚需要推敲)
   
Tomcat Manager提供通过URL进行web应用部署的特性。也有一个叫"Client Deployer"的shell工具，它通过与Tomcat Manager交互完成部署并提供附加的特性如对打包的web应用资源(web application resource)进行编译和校验.


安装
--------------------

如果静态部署方式保护署web应用的话没有前置安装需求，尽管在Tomcat Manager手册中项目描述了一些需要前置配置的配置,在使用Tomcat Manager来部署的话也没有前置安装需求. 如果想使用Tomcat Client Deployer(TCD)方式进行部署的话，就需要先进行安装.

TCD并没有包含在Tomcat core版本中，需要从下载区域进行下载，下载的文件通常标记为 *apache-tomcat-7.0.x-deployer*

安装TCD需要先安装Apache Ant 1.6.2+及JAVA。 操作系统中应定义下ANT_HOME(指向Ant安装目录)、JAVA_HOME(指向JAVA安装目录)环境变量. 同时也需要确保操作系统的命令行能够运行Ant的ant、Java的javac编译命令

1. 下载TCD
2. TDC不需要解压到已经安装的Tomcat目录, 它可以解压到其他地方
3. 阅读 `USing the Tomcat Client Deployer <http://tomcat.apache.org/tomcat-7.0-doc/deployer-howto.html#Deploying using the Client Deployer Package>`_


一句话描述Context
-----------------------------

在谈论部署web应用时，需要先理解 *Context*  这个概念. Context描述Tomcat如何调用web应用.

配置Context是需要使用 *Context描述(descriptor)* . A Context Descriptor is simply an XML file that contains Tomcat related configuration for a Context, 如命名资源(naming resources)或session管理配置. 
早期的Tomcat版本中，Context描述经常包含在主配置文件 *server.xml* 中.新的版本中不建议这做(尽管依然能使用)

Context描述不仅仅应用于Tomcat如何去配置Context,也常常应用于Tomcat Manager和TCD以便它们能够使用Context描述去正确执行它们的规则.

本地的Context描述有:

   1. $CATALINA_BASE/conf/[enginename]/[hostname]/context.xml
   2. $CATALINA_BASE/webapps/[webappname]/META-INF/context.xml
   
文件(1)中文件名为[webappname].xml，文件(2)中文件名为context.xml. 如果没有配置Context描述，Tomcat将使用它的默认值.


在Tomcat启动时部署
------------------------------

如果对Tomcat Manager或TCD没有兴趣，你需要在Tomcat启动时中静态部署你的应用. web应用部署在每个Host上指定的 *appBase* 里,你可以拷贝一个叫做 *exploded web application* 如未压缩的目录，或者压缩后的web应用资源 .war文件

只有当Host的 *deployOnStartup* 属性为"true"时，放置在Host(默认Host为"localhost")的 *appBase* 属性(默认appBase为"${CATALINA_BASE/webapps}")目录下的web应用会在Tomcat启动时会进行部署.

Tomcat启动时部署顺序如下所示:

   1. 含有Context描述的会优先部署
   2. 没有Context描述的已Exploded web应用会被部署，如果在appBase下有关联的.WAR文件，同时.WAR文件比已经exploded web应用要新的话，会删除到exploded目录，并重新通过.WAR部署
   3. .WAR文件部署
   
需要注意的是，对于每一个已经部署的应用，一个Context描述将会创建，除非已经存在.


在运行中的Tomcat server进行部署
--------------------------------------------------

在运行状态的Tomcat server，是有可能进行web应用部署的

如果Host的 *autoDeploy* 属性设置的为"true"，Host将动态部署并更新应用. 例如一个新的 .WAR放在了 *appBase* 中. 为了完成这个工作，Host需要有一个默认配置的后台进程进行处理.

当 *autoDeploy* 设置为"true"时，运行中的Tomcat可以进行完成如下操作:

   * 部署copy到 *appBase* 下的.WAR文件
   * 部署copy到 *appBase* 目录下的已经exploded web应用
   * 对于通过.WAR已经部署的web应用，当新的.WAR文件提供过来后会重新进行部署. 对于已经exploded的web应用将会被删除掉，然后.WAR将会重新解压. 注意当Host的 *unpackWARs* 设置为 *false* (.WAR包不会自动exploded)时并不会这样做，而是简单的重新部署下压缩包
   * 当 **/WEB-INF/web.xml** (or any other resource defined as a WatchedResource)被更新时，web应用会重新加载.
   * 当之前已经部署的web应用中的Context描述文件更新时，会重新部署web应用.
   * 当之前定义的全局或单Host的Context描述文件更新时，会重新部署web应用
   * 当Context描述文件添加到 $CATALINA_BASE/conf/[enginename]/[hostname]/目录后，会重新部署web应用
   * 当document base(docBase)被删除时，应用会undeployment。 Note that on Windows, this assumes that anti-locking features (see Context configuration) are enabled, otherwise it is not possible to delete the resources of a running web application.
   
Note that web application reloading can also be configured in the loader, in which case loaded classes will be tracked for changes.


通过Tomcat Manager进行部署
------------------------------------------

如果使用Tomcat Manager进行部署，请访问 http://tomcat.apache.org/tomcat-7.0-doc/manager-howto.html

使用Client Deployer Package进行部署
-------------------------------------------

最终，可以使用Tomcat Client Deployer进行web应用部署. 该工具常用于验证、编译、压缩成.WAR、在生产环境或开发Tomcat环境进行web应用部署. 需要注意的是它需要使用Tomcat Manager，并且确保目标Tomcat服务处于运行中

It is assumed the user will be familiar with Apache Ant for using the TCD. Apache Ant is a scripted build tool. The TCD comes pre-packaged with a build script to use. Only a modest understanding of Apache Ant is required (installation as listed earlier in this page, and familiarity with using the operating system command shell and configuring environment variables).

The TCD includes Ant tasks, the Jasper page compiler for JSP compilation before deployment, as well as a task which validates the web application Context Descriptor. The validator task (class org.apache.catalina.ant.ValidatorTask) allows only one parameter: the base path of an exploded web application.

The TCD uses an exploded web application as input (see the list of the properties used below). A web application that is programmatically deployed with the deployer may include a Context Descriptor in /META-INF/context.xml.

The TCD includes a ready-to-use Ant script, with the following targets:

   * compile (default): Compile and validate the web application. This can be used standalone, and does not need a running Tomcat server. The compiled application will only run on the associated Tomcat 7.0.x server release, and is not guaranteed to work on another Tomcat release, as the code generated by Jasper depends on its runtime component. It should also be noted that this target will also compile automatically any Java source file located in the /WEB-INF/classes folder of the web application.
   * deploy: Deploy a web application (compiled or not) to a Tomcat server.
   * undeploy: Undeploy a web application
   * start: Start web application
   * reload: Reload web application
   * stop: Stop web application
   
In order for the deployment to be configured, create a file called deployer.properties in the TCD installation directory root. In this file, add the following name=value pairs per line:

Additionally, you will need to ensure that a user has been setup for the target Tomcat Manager (which TCD uses) otherwise the TCD will not authenticate with the Tomcat Manager and the deployment will fail. To do this, see the Tomcat Manager page.

   * build: The build folder used will be, by default, ${build}/webapp/${path}. After the end of the execution of the compile target, the web application .WAR will be located at ${build}/webapp/${path}.war.
   * webapp: The directory containing the exploded web application which will be compiled and validated. By default, the folder is myapp.
   * path: Deployed context path of the web application, by default /myapp.
   * url: Absolute URL to the Tomcat Manager web application of a running Tomcat server, which will be used to deploy and undeploy the web application. By default, the deployer will attempt to access a Tomcat instance running on localhost, at http://localhost:8080/manager/text.
   * username: Tomcat Manager username (user should have a role of manager-script)
   * password: Tomcat Manager password.
   
Comments
--------------------------------------

      





 
 

   
   