编写一个 CorDapp
=====================

.. contents::

.. _cordapp-structure:

模块
-------
The source code for a CorDapp is divided into one or more modules, each of which will be compiled into a separate JAR.
Together, these JARs represent a single CorDapp. Typically, a Cordapp contains all the classes required for it to be
used standalone. However, some Cordapps are only libraries for other Cordapps and cannot be run standalone.

CorDapp 的源代码被分为一个或者多个模块，其中的每个都会被编译为一个单独的 JAR 文件。这些 JAR 文件在一起便代表了一个 CorDapp。通常，一个 CorDapp 包含的所有所需的类应该被独立地使用。然而，一些 CorDapps 仅仅是被其他 CorDapps 所使用的类库，他们是不能够独立运行的。

A common pattern is to have:

* One module containing only the CorDapp's contracts and/or states, as well as any required dependencies
* A second module containing the remaining classes that depend on these contracts and/or states

一个常规的模式是具有：

* 一个只含有 CorDapp 的 contracts 和/或者 states，以及他们所需的依赖的模块
* 第二个模块包含依赖于这些 contracts 和/或者 states 的剩余的类

This is because each time a contract is used in a transaction, the entire JAR containing the contract's definition is
attached to the transaction. This is to ensure that the exact same contract and state definitions are used when
verifying this transaction at a later date. Because of this, you will want to keep this module, and therefore the
resulting JAR file, as small as possible to reduce the size of your transactions and keep your node performant.

这是因为每次一个 contract 在一个 transaction 中被使用的时候，包含这个 contract 定义的整个 JAR 文件就会被附加在 transaction 里。这是为了在之后的某个时间验证 transaction 的时候，确保使用的是完全一致的 contract 和 state 定义。因为这个，你会想要保留着个模块，并且因此产生的这个 JAR 文件，应该尽可能的小，以此来减少你的 transaction 的尺寸，保持节点的效率。

However, this two-module structure is not prescriptive:

* A library CorDapp containing only contracts and states would only need a single module

* In a CorDapp with multiple sets of contracts and states that **do not** depend on each other, each independent set of
  contracts and states would go in a separate module to reduce transaction size

* In a CorDapp with multiple sets of contracts and states that **do** depend on each other, either keep them in the
  same module or create separate modules that depend on each other

* The module containing the flows and other classes can be structured in any way because it is not attached to
  transactions

然而，这种两个模块的结构并不是必须的：

* 一个类库类型的 CorDapp 仅仅包含 contracts 和 states 将会只需要一个模块
* 对于一个具有多套的 contracts 和 states 并且彼此 **没有** 依赖性的 CorDapp，每套独立的 contracts 和 states 都应该放在单独的一个模块，来减小 transaction 的尺寸
* 对于一个具有多套的 contracts 和 states 并且彼此 **有** 依赖性的 CorDapp，可以把他们保存在相同的模块，或者创建彼此依赖的不同的模块
* 包含 flows 和其他类的模块可以按照任何方式来构建，因为它不会被附加到 transaction 中

CorDapps 模板
-----------------
You should base your project on one of the following templates:

你应该基于 Java 或者 Kotlin 的模板结构来创建你的项目

* `Java Template CorDapp <https://github.com/corda/cordapp-template-java>`_ (for CorDapps written in Java)
* `Kotlin Template CorDapp <https://github.com/corda/cordapp-template-kotlin>`_ (for CorDapps written in Kotlin)

Please use the branch of the template that corresponds to the major version of Corda you are using. For example,
someone building a CorDapp on Corda 4.1 should use the ``release-V4`` branch of the template.

请使用你在使用的 Corda 的版本对应的分支的模板。比如，如果是基于 Corda 4.1 构建一个 CorDapp 的话，应该选择 ``release-V4`` 分支的模板。

Build system
^^^^^^^^^^^^

The templates are built using Gradle. A Gradle wrapper is provided in the ``wrapper`` folder, and the dependencies are
defined in the ``build.gradle`` files. See :doc:`cordapp-build-systems` for more information.

模板是使用 Gradle 来构建的。一个 Gradle wrapper 在 ``wrapper`` 文件夹下被提供，并且这些依赖项是在 ``build.gradle`` 文件中定义的。查看 :doc:`cordapp-build-systems` 了解更多信息。

No templates are currently provided for Maven or other build systems.

现在没有针对于 Maven 或者其他的 build systems 的模板。

模块
^^^^^^^
The templates are split into two modules:

* A ``cordapp-contracts-states`` module containing the contracts and states
* A ``cordapp`` module containing the remaining classes that depends on the ``cordapp-contracts-states`` module

模板分为两个模块：

* 一个 ``cordapp-contracts-states`` 模块，包含了 contracts 和 states
* 一个 ``cordapp`` 模块包含了依赖于 ``cordapp-contracts-states`` 模块的剩余的类

These modules will be compiled into two JARs - a ``cordapp-contracts-states`` JAR and a ``cordapp`` JAR - which
together represent the Template CorDapp.

这些模块会被编译到两个 JARs - 一个 ``cordapp-contracts-states`` JAR 和一个 ``cordapp`` JAR - 他们俩共同代表了这个 CorDapp 模板。

第一个模块 - cordapp-contract-states
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Here is the structure of the ``src`` directory for the ``cordapp-contracts-states`` module of the Java template:

下边是 Java 模板的 ``cordapp-contracts-states`` 模块的 ``src`` 路径结构：

.. parsed-literal::

    .
    └── main
        └── java
            └── com
                └── template
                    ├── TemplateContract.java
                    └── TemplateState.java

The directory only contains two class definitions:

这个路径仅包含两个类定义：

* ``TemplateContract``
* ``TemplateState``

These are definitions for classes that we expect to have to send over the wire. They will be compiled into their own
CorDapp.

这些是我们希望在网络中进行传输的内容的类定义。他们会被编译成为自己的 CorDapp。

第二个模块 - cordapp
~~~~~~~~~~~~~~~~~~~~
Here is the structure of the ``src`` directory for the ``cordapp`` module of the Java template:

下边是 Java 模板的 ``cordapp`` 模块的 ``src`` 路径结构：

.. parsed-literal::

    .
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── template
    │   │           ├── TemplateApi.java
    │   │           ├── TemplateClient.java
    │   │           ├── TemplateFlow.java
    │   │           ├── TemplateSerializationWhitelist.java
    │   │           └── TemplateWebPlugin.java
    │   └── resources
    │       ├── META-INF
    │       │   └── services
    │       │       ├── net.corda.core.serialization.SerializationWhitelist
    │       │       └── net.corda.webserver.services.WebServerPluginRegistry
    │       ├── certificates
    │       └── templateWeb
    ├── test
    │   └── java
    │       └── com
    │           └── template
    │               ├── ContractTests.java
    │               ├── FlowTests.java
    │               └── NodeDriver.java
    └── integrationTest
        └── java
            └── com
                └── template
                    └── DriverBasedTest.java

The ``src`` directory is structured as follows:

* ``main`` contains the source of the CorDapp
* ``test`` contains example unit tests, as well as a node driver for running the CorDapp from IntelliJ
* ``integrationTest`` contains an example integration test

``src`` 路径结构包括：

* ``main`` 包含了 CorDapp 源代码
* ``test`` 包含了单元测试代码，还包括一个能够在 IntelliJ 中运行 CorDapp 的节点 driver
* ``integrationTest`` 包含了集成测试的例子

Within ``main``, we have the following directories:

* ``java``, which contains the source-code for our CorDapp:

    * ``TemplateFlow.java``, which contains a template ``FlowLogic`` subclass
    * ``TemplateState.java``, which contains a template ``ContractState`` implementation
    * ``TemplateContract.java``, which contains a template ``Contract`` implementation
    * ``TemplateSerializationWhitelist.java``, which contains a template ``SerializationWhitelist`` implementation
    * ``TemplateApi.java``, which contains a template API for the deprecated Corda webserver
    * ``TemplateWebPlugin.java``, which registers the API and front-end for the deprecated Corda webserver
    * ``TemplateClient.java``, which contains a template RPC client for interacting with our CorDapp

* ``resources/META-INF/services``, which contains various registries:

    * ``net.corda.core.serialization.SerializationWhitelist``, which registers the CorDapp's serialisation whitelists
    * ``net.corda.webserver.services.WebServerPluginRegistry``, which registers the CorDapp's web plugins

* ``resources/templateWeb``, which contains a template front-end

在 ``main`` 中, 我们有以下的目录:

* ``java``, 包含了 CorDapp 的源代码:

    * ``TemplateFlow.java``, 包含了一个 ``FlowLogic`` 子类
    * ``TemplateState.java``, 包含了一个 ``ContractState`` 的实现
    * ``TemplateContract.java``, 包含了一个 ``Contract`` 的实现
    * ``TemplateSerializationWhitelist.java``, 包含了一个 ``SerializationWhitelist`` 的实现
    * ``TemplateApi.java``, 包含了一个为了已经废弃的 Corda webserver 的一个模板 API
    * ``TemplateWebPlugin.java``, 注册了一个为了已经废弃的 Corda web server 的 API 和前端
    * ``TemplateClient.java``, 包含了一个跟 CorDapp 互动的一个 RPC 客户端

* ``resources/META-INF/services``, 包含了很多不同的注册:

    * ``net.corda.core.serialization.SerializationWhitelist``, 注册了 CorDapp 的 serialisation 白名单
    * ``net.corda.webserver.services.WebServerPluginRegistry``, 注册了 CorDapp 的 web plugins

* ``resources/templateWeb``, 包含了一个前端

In a production CorDapp:

* We would remove the files related to the deprecated Corda webserver (``TemplateApi.java``,
  ``TemplateWebPlugin.java``, ``resources/templateWeb``, and ``net.corda.webserver.services.WebServerPluginRegistry``)
  and replace them with a production-ready webserver

* We would also move ``TemplateClient.java`` into a separate module so that it is not included in the CorDapp

在一个生产环境的 CorDapp：

* 我们会移除掉已经被废弃的 Corda webserver 相关的文件（``TemplateApi.java``, ``TemplateWebPlugin.java``, ``resources/templateWeb``, 和 ``net.corda.webserver.services.WebServerPluginRegistry``）并且把他们替换成一个适用于生产环境的 webserver
* 我们也会把 ``TemplateClient.java`` 移动到一个单独的模块，所以它不会被包含在 CorDapp 中