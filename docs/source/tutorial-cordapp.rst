.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

运行 CorDapp 例子
===========================

.. contents::

The example CorDapp allows nodes to agree IOUs with each other, as long as they obey the following contract rules:

* The IOU's value is strictly positive
* A node is not trying to issue an IOU to itself

这个 CorDapp 的例子允许节点同意彼此间的 IOU，只要他们遵循下边的合约规则：

* IOU 的值是正的
* 一个节点不会尝试给自己生成一个 IOU

We will deploy and run the CorDapp on four test nodes:

* **Notary**, which runs a notary service
* **PartyA**
* **PartyB**
* **PartyC**

我们将会在下边四个测试节点上部署并运行 CorDapp：

* **Notary**, 会运行 notary 服务
* **PartyA**
* **PartyB**
* **PartyC**

Because data is only propagated on a need-to-know basis, any IOUs agreed between PartyA and PartyB become "shared
facts" between PartyA and PartyB only. PartyC won't be aware of these IOUs.

由于数据是基于需要知道的原则来传播的，任何在 PartyA 和 PartyB 同意的 IOUs 仅仅会变为 PartyA 和 PartyB 之间的一个 “共享的事实”。PartyC 是不会知道这些 IOUs 的。

下载 CorDapp 样例
-------------------------------
Start by downloading the example CorDapp from GitHub:

* Set up your machine by following the :doc:`quickstart guide <getting-set-up>`

* Clone the samples repository from using the following command: ``git clone https://github.com/corda/samples``

* Change directories to the ``cordapp-example`` folder: ``cd samples/cordapp-example``

从 GitHub 上下载 CorDapp 样例:

* 跟随 :doc:`快速开始指南 <getting-set-up>` 设置你的电脑
* 用下边的命令 ``git clone https://github.com/corda/samples`` 克隆样例代码
* 进入目录 ``cordapp-example``：``cd samples/cordapp-example``

在 InteliJ 中打开 CorDapp 样例
---------------------------------------
Let's open the example CorDapp in IntelliJ IDEA:

* Open IntelliJ

* A splash screen will appear. Click ``open``, navigate to and select the ``cordapp-example`` folder, and click ``OK``

* Once the project is open, click ``File``, then ``Project Structure``. Under ``Project SDK:``, set the project SDK by
  clicking ``New...``, clicking ``JDK``, and navigating to ``C:\Program Files\Java\jdk1.8.0_XXX`` on Windows or ``Library/Java/JavaVirtualMachines/jdk1.8.XXX`` on MacOSX (where ``XXX`` is the
  latest minor version number). Click ``Apply`` followed by ``OK``

* Again under ``File`` then ``Project Structure``, select ``Modules``. Click ``+``, then ``Import Module``, then select
  the ``cordapp-example`` folder and click ``Open``. Choose to ``Import module from external model``, select
  ``Gradle``, click ``Next`` then ``Finish`` (leaving the defaults) and ``OK``

* Gradle will now download all the project dependencies and perform some indexing. This usually takes a minute or so

让我们在 InteliJ IEAD 中打开这个 CorDapp 样例：

* 打开 InteliJ
* 一个 splash 界面会显示。点击 ``open``，浏览并选择 ``cordapp-example`` 文件夹，然后点击 ``OK``
* 当项目打开后，点击 ``File``，然后 ``Project Structure``。在 ``Project SDK:``，通过点击 ``New...`` 来设置项目的 SDK，在 Windows 中选择 ``C:\Program Files\Java\jdk1.8.0_XXX``，或者在 MacOSX 中选择 ``Library/Java/JavaVirtualMachines/jdk1.8.XXX``（``XXX`` 是最新的小版本号）。点击 ``Apply``
* 再一次，点击 ``File`` 然后 ``Project Structure``，选择 ``Modules``。点击 ``+``，然后 ``Import Module``，然后选择 ``cordapp-example`` 文件夹并点击 ``Open``。选择 ``Import module from external model``，选择 ``Gradle``，点击 ``Next`` 然后 ``Finish``（使用默认）和 ``OK``

项目结构
~~~~~~~~~~~~~~~~~
The example CorDapp has the following structure:

CorDapp 样例含有以下的结构：

.. sourcecode:: none

    .
    ├── LICENCE
    ├── README.md
    ├── TRADEMARK
    ├── build.gradle
    ├── clients
    │   ├── build.gradle
    │   └── src
    │       └── main
    │           ├── kotlin
    │           │   └── com
    │           │       └── example
    │           │           └── server
    │           │               ├── MainController.kt
    │           │               ├── NodeRPCConnection.kt
    │           │               └── Server.kt
    │           └── resources
    │               ├── application.properties
    │               └── public
    │                   ├── index.html
    │                   └── js
    │                       └── angular-module.js
    ├── config
    │   ├── dev
    │   │   └── log4j2.xml
    │   └── test
    │       └── log4j2.xml
    ├── contracts-java
    │   ├── build.gradle
    │   └── src
    │       └── main
    │           └── java
    │               └── com
    │                   └── example
    │                       ├── contract
    │                       │   └── IOUContract.java
    │                       ├── schema
    │                       │   ├── IOUSchema.java
    │                       │   └── IOUSchemaV1.java
    │                       └── state
    │                           └── IOUState.java
    ├── contracts-kotlin
    │   ├── build.gradle
    │   └── src
    │       └── main
    │           └── kotlin
    │               └── com
    │                   └── example
    │                       ├── contract
    │                       │   └── IOUContract.kt
    │                       ├── schema
    │                       │   └── IOUSchema.kt
    │                       └── state
    │                           └── IOUState.kt
    ├── cordapp-example.iml
    ├── gradle
    │   └── wrapper
    │       ├── gradle-wrapper.jar
    │       └── gradle-wrapper.properties
    ├── gradle.properties
    ├── gradlew
    ├── gradlew.bat
    ├── lib
    │   ├── README.txt
    │   └── quasar.jar
    ├── settings.gradle
    ├── workflows-java
    │   ├── build.gradle
    │   └── src
    │       ├── integrationTest
    │       │   └── java
    │       │       └── com
    │       │           └── example
    │       │               └── DriverBasedTests.java
    │       ├── main
    │       │   └── java
    │       │       └── com
    │       │           └── example
    │       │               └── flow
    │       │                   └── ExampleFlow.java
    │       └── test
    │           └── java
    │               └── com
    │                   └── example
    │                       ├── NodeDriver.java
    │                       ├── contract
    │                       │   └── IOUContractTests.java
    │                       └── flow
    │                           └── IOUFlowTests.java
    └── workflows-kotlin
        ├── build.gradle
        └── src
            ├── integrationTest
            │   └── kotlin
            │       └── com
            │           └── example
            │               └── DriverBasedTests.kt
            ├── main
            │   └── kotlin
            │       └── com
            │           └── example
            │               └── flow
            │                   └── ExampleFlow.kt
            └── test
                └── kotlin
                    └── com
                        └── example
                            ├── NodeDriver.kt
                            ├── contract
                            │   └── IOUContractTests.kt
                            └── flow
                                └── IOUFlowTests.kt

The key files and directories are as follows:

* The **root directory** contains some gradle files, a README and a LICENSE
* **config** contains log4j2 configs
* **gradle** contains the gradle wrapper, which allows the use of Gradle without installing it yourself and worrying
  about which version is required
* **lib** contains the Quasar jar which rewrites our CorDapp's flows to be checkpointable
* **clients** contains the source code for spring boot integration
* **contracts-java** and **workflows-java** contain the source code for the example CorDapp written in Java
* **contracts-kotlin** and **workflows-kotlin** contain the same source code, but written in Kotlin. CorDapps can be developed in either Java and Kotlin

下边是关键的文件和路径：

* **根目录** 包含了一些 gradle 文件，一个 README 和一个 LICENSE
* **config** 包含了 log4j2 配置文件
* **gradle** 包含了 gradle wrapper，这允许你可以直接使用 gradle 而不用自己安装并考虑应该使用哪个版本
* **lib** 包含了 Quasar jar，它重写了我们的 CorDapp 的 flows 成为 checkpointable
* **clients** 包含了 sprint boot 集成的源代码
* **contract-java** 和 **workflow-java** 包含了使用 Java 编写的 CorDapp 样例的源代码
* **contracts-kotlin** 和 **workflows-kotlin** 包含了使用 Kotlin 编写的 CorDapp 样例的源代码。CorDapps 可以用 Java 或者 Kotlin 编写

运行 CorDapp 样例
---------------------------
There are two ways to run the example CorDapp:

* Via the terminal
* Via IntelliJ

有两种方式运行 CorDapp 样例：

* 通过 terminal
* 通过 IntelliJ

Both approaches will create a set of test nodes, install the CorDapp on these nodes, and then run the nodes. You can
read more about how we generate nodes :doc:`here <generating-a-node>`.

两种方式都会创建一系列的测试节点，在这些节点上安装 CorDapp，并且运行这些节点。你可以在 :doc:`这里 <generating-a-node>` 阅读更多关于如何生成节点的信息。

从 terminal 运行 CorDapp 样例
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

构建 CorDapp 样例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Open a terminal window in the ``cordapp-example`` directory

* Run the ``deployNodes`` Gradle task to build four nodes with our CorDapp already installed on them:

  * Unix/Mac OSX: ``./gradlew deployNodes``

  * Windows: ``gradlew.bat deployNodes``

* 在 ``cordapp-example`` 路径下打开一个 terminal 窗口
* 运行 ``deployNodes`` gradle 任务来构建四个节点：

  * Unix/Mac OSX: ``./gradlew deployNodes``
  * Windows: ``gradlew.bat deployNodes``

.. note:: CorDapps can be written in any language targeting the JVM. In our case, we've provided the example source in
   both Kotlin and Java. Since both sets of source files are functionally identical, we will refer to the Kotlin version
   throughout the documentation.

.. note:: CorDapps 可以由任何目标是 JVM 的语言编写。在我们的例子中，我们提供了 Java 和 Kotlin 的源代码。因为两套代码具有完全一样的功能，我们会在这个文档中只使用 Kotlin 版本。

* After the build finishes, you will see the following output in the ``workflows-kotlin/build/nodes`` folder:

  * A folder for each generated node
  * A ``runnodes`` shell script for running all the nodes simultaneously on osX
  * A ``runnodes.bat`` batch file for running all the nodes simultaneously on Windows

* 在 build 结束后，在 ``workflows-kotlin/build/nodes`` 文件夹里，你应该能够看到下边的输出：

  * 为每个节点生成的文件夹
  * 一个 ``runnodes`` shell 脚本，用来在 OSX 上同时运行所有的节点
  * 一个 ``runnodes.bat`` batch 文件，用来在 Windows 上同时运行所有的节点

* Each node in the ``nodes`` folder will have the following structure:

* ``nodes`` 文件夹下的每个节点将会有以下的结构：

  .. sourcecode:: none
      
      . nodeName
      ├── additional-node-infos  // 
      ├── certificates
      ├── corda.jar              // The Corda node runtime
      ├── cordapps               // The node's CorDapps
      │   ├── corda-finance-contracts-|corda_version|.jar
      │   ├── corda-finance-workflows-|corda_version|.jar
      │   └── cordapp-example-0.1.jar
      ├── drivers
      ├── logs
      ├── network-parameters
      ├── node.conf              // The node's configuration file
      ├── nodeInfo-<HASH>        // The hash will be different each time you generate a node
      └── persistence.mv.db      // The node's database

.. note:: ``deployNodes`` is a utility task to create an entirely new set of nodes for testing your CorDapp. In production, 
   you would instead create a single node as described in :doc:`generating-a-node` and build your CorDapp JARs as described 
   in :doc:`cordapp-build-systems`.

.. note:: ``deployNodes`` 是一个 utility 任务来创建一系列全新用来测试你的 CorDapp 的节点。在生产环境中，你可能会像 :doc:`generating-a-node` 描述的那样只生成一个节点，并且像 :doc:`cordapp-build-systems` 描述的那样创建你的 CorDapp JARs。

运行 CorDapp 样例
^^^^^^^^^^^^^^^^^^^^^^^^^^^
Start the nodes by running the following command from the root of the ``cordapp-example`` folder:

* Unix/Mac OSX: ``workflows-kotlin/build/nodes/runnodes``
* Windows: ``call workflows-kotlin\build\nodes\runnodes.bat``

从 ``cordapp-example`` 文件夹的根目录下运行下边的命令来启动节点：

* Unix/Mac OSX: ``workflows-kotlin/build/nodes/runnodes``
* Windows: ``call workflows-kotlin\build\nodes\runnodes.bat``

Each Spring Boot server needs to be started in its own terminal/command prompt, replace X with A, B and C:

* Unix/Mac OSX: ``./gradlew runPartyXServer``
* Windows: ``gradlew.bat runPartyXServer``

每个 Spring Boot server 需要在它自己的 terminal/command 窗口中被启动，将 X 替换为 A, B 和 C：

* Unix/Mac OSX: ``./gradlew runPartyXServer``
* Windows: ``gradlew.bat runPartyXServer``

Look for the Started ServerKt in X seconds message, don't rely on the % indicator.

查看 Started ServerKt in X seconds 消息，不要依赖 % 的指示符

.. warning:: On Unix/Mac OSX, do not click/change focus until all seven additional terminal windows have opened, or some
   nodes may fail to start.

.. warning:: 在 Unix/Mac OSX，不要点击/改变焦点知道所有 7 个额外的 terminal 窗口都被打开，或者一些节点可能会启动失败。

For each node, the ``runnodes`` script creates a node tab/window:

对于每个节点，``runnodes`` 语句创建了一个节点 tab/window：

.. sourcecode:: none

      ______               __
     / ____/     _________/ /___ _
    / /     __  / ___/ __  / __ `/         Top tip: never say "oops", instead
   / /___  /_/ / /  / /_/ / /_/ /          always say "Ah, Interesting!"
   \____/     /_/   \__,_/\__,_/

   --- Corda Open Source corda-|corda_version| (4157c25) -----------------------------------------------


   Logs can be found in                    : /Users/joeldudley/Desktop/cordapp-example/workflows-kotlin/build/nodes/PartyA/logs
   Database connection url is              : jdbc:h2:tcp://localhost:59472/node
   Incoming connection address             : localhost:10005
   Listening on port                       : 10005
   Loaded CorDapps                         : corda-finance-corda-|corda_version|, cordapp-example-0.1, corda-core-corda-|corda_version|
   Node for "PartyA" started up and registered in 38.59 sec


   Welcome to the Corda interactive shell.
   Useful commands include 'help' to see what is available, and 'bye' to shut down the node.

   Fri Mar 02 17:34:02 GMT 2018>>>

It usually takes around 60 seconds for the nodes to finish starting up. To ensure that all the nodes are running, you
can query the 'status' end-point located at ``http://localhost:[port]/api/status`` (e.g.
``http://localhost:50005/api/status`` for ``PartyA``).

通常需要 60 秒钟左右节点能够完成启动。为了确保所有节点是运行的，你可以在 ``http://localhost:[port]/api/status`` 查询 'status' end-point（比如对于 PartyA 来说，``http://localhost:50005/api/status``）。

在 IntelliJ 中运行 CorDapp 样例
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
* Select the ``Run Example CorDapp - Kotlin`` run configuration from the drop-down menu at the top right-hand side of
  the IDE

* Click the green arrow to start the nodes:

  .. image:: resources/run-config-drop-down.png
    :width: 400

* To stop the nodes, press the red square button at the top right-hand side of the IDE, next to the run configurations

* 在 IDE 右上角的下拉菜单中选择 ``Run Example CorDapp - Kotlin`` 来运行配置
* 点击绿色的箭头来启动节点：

  .. image:: resources/run-config-drop-down.png
    :width: 400

* 想要停止节点，点击 IDE 右上角的红色的方块按钮，在运行配置的旁边

跟 CorDapp 样例进行互动
------------------------------------

通过 HTTP
~~~~~~~~
The Spring Boot servers run locally on the following ports:

Spring Boot servers 在下边的端口上运行：

* PartyA: ``localhost:50005``
* PartyB: ``localhost:50006``
* PartyC: ``localhost:50007``

These ports are defined in ``clients/build.gradle``.

这些端口在 ``clients/build.gradle`` 中被定义。

Each Spring Boot server exposes the following endpoints:

每个 Spring Boot server 暴露了一下的 endpoints：

* ``/api/example/me``
* ``/api/example/peers``
* ``/api/example/ious``
* ``/api/example/create-iou`` with parameters ``iouValue`` and ``partyName`` which is CN name of a node

There is also a web front-end served from the home web page e.g. ``localhost:50005``.

这里也有一个来自于 home web page 的一个 web 前端，比如 ``localhost:50005``。

.. warning:: The content is only available for demonstration purposes and does not implement
   anti-XSS, anti-XSRF or other security techniques. Do not use this code in production.

.. warning:: 这些内容仅仅是为了演示的目的并且没有实现 anti-XSS、anti-XSRF 或者其他的安全技术。不要把它应用于生产环境。

通过 endpoint 创建一个 IOU
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
An IOU can be created by sending a PUT request to the ``/api/example/create-iou`` endpoint directly, or by using the
the web form served from the home directory.

一个 IOU 可以直接通过发送一个 PUT 请求给 ``/api/example/create-iou`` endpoint 来创建，或者使用来自 home 路径的 web form 来创建。

To create an IOU between PartyA and PartyB, run the following command from the command line:

想要创建一个在 PartyA 和 PartyB 之间的 IOU，在命令行中运行下边的命令：

.. sourcecode:: bash

   curl -X PUT 'http://localhost:50005/api/example/create-iou?iouValue=1&partyName=O=PartyB,L=New%20York,C=US'

Note that both PartyA's port number (``50005``) and PartyB are referenced in the PUT request path. This command
instructs PartyA to agree an IOU with PartyB. Once the process is complete, both nodes will have a signed, notarised
copy of the IOU. PartyC will not.

注意 PartyA 的端口号 (``50005``) 和 PartyB 的都在 PUT 请求路径中被引用。这个命令告诉 PartyA 去同意一个跟 PartyB 的 IOU。当这个过程结束之后，两个节点都会有一个签过名的，经过公证的 IOU 的副本。PartyC 则不会有。

通过 web 前端提交一个 IOU
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To create an IOU between PartyA and PartyB, navigate to the home directory for the node, click the "create IOU" button at the top-left
of the page, and enter the IOU details into the web-form. The IOU must have a positive value. For example:

想要在 PartyA 和 PartyB 之间创建一个 IOU，浏览节点的 home 路径，点击页面左上角的 "create IOU" 按钮，在 web-form 中输入 IOU 详细信息。IOU 必须是正数。比如：

.. sourcecode:: none

  Counterparty: Select from list
  Value (Int):   5

And click submit. Upon clicking submit, the modal dialogue will close, and the nodes will agree the IOU.

点击 Submit。点击 submit 之后，这个模态窗口会被关闭，节点将会同意这个 IOU。

检查 output
^^^^^^^^^^^^^^^^^^^
Assuming all went well, you can view the newly-created IOU by accessing the vault of PartyA or PartyB:

假设一切运行良好，你可以通过访问 PartyA 或者 PartyB 的 vault 来浏览新创建的 IOU：

*Via the HTTP API:*

* PartyA's vault: Navigate to http://localhost:50005/api/example/ious
* PartyB's vault: Navigate to http://localhost:50006/api/example/ious

*Via home page:*

* PartyA: Navigate to http://localhost:50005 and hit the "refresh" button
* PartyB: Navigate to http://localhost:50006 and hit the "refresh" button

The vault and web front-end of PartyC (at ``localhost:50007``) will not display any IOUs. This is because PartyC was
not involved in this transaction.

PartyC 的 vault 以及 web 前端（``localhost:50007``）不会显示任何的的 IOU。这是因为 PartyC 并没有参与这个交易。

通过 shell (terminal only)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Nodes started via the terminal will display an interactive shell:

通过 terminal 启动的节点将会显示一个可交互的 shell：

.. sourcecode:: none

    Welcome to the Corda interactive shell.
    Useful commands include 'help' to see what is available, and 'bye' to shut down the node.

    Fri Jul 07 16:36:29 BST 2017>>>

Type ``flow list`` in the shell to see a list of the flows that your node can run. In our case, this will return the
following list:

在 shell 中输入 ``flow list`` 将会看到你的节点运行的一个 flows 列表。在我们的例子中，将会返回下边的列表：

.. sourcecode:: none

    com.example.flow.ExampleFlow$Initiator
    net.corda.core.flows.ContractUpgradeFlow$Authorise
    net.corda.core.flows.ContractUpgradeFlow$Deauthorise
    net.corda.core.flows.ContractUpgradeFlow$Initiate
    net.corda.finance.flows.CashExitFlow
    net.corda.finance.flows.CashIssueAndPaymentFlow
    net.corda.finance.flows.CashIssueFlow
    net.corda.finance.flows.CashPaymentFlow
    net.corda.finance.internal.CashConfigDataFlow

通过 shell 创建一个 IOU
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
We can create a new IOU using the ``ExampleFlow$Initiator`` flow. For example, from the interactive shell of PartyA,
you can agree an IOU of 50 with PartyB by running
``flow start ExampleFlow$Initiator iouValue: 50, otherParty: "O=PartyB,L=New York,C=US"``.

我们可以通过使用 ``ExampleFlow$Initiator`` flow 来创建一个新的 IOU。比如，在 PartyA 的 shell 中，你可以通过运行 ``flow start ExampleFlow$Initiator iouValue: 50, otherParty: "O=PartyB,L=New York,C=US"`` 来同意跟 PartyB 的一个 50 的 IOU。

This will print out the following progress steps:

这会打印出下边的进度步骤：

.. sourcecode:: none

    ✅   Generating transaction based on new IOU.
    ✅   Verifying contract constraints.
    ✅   Signing transaction with our private key.
    ✅   Gathering the counterparty's signature.
        ✅   Collecting signatures from counterparties.
        ✅   Verifying collected signatures.
    ✅   Obtaining notary signature and recording transaction.
        ✅   Requesting signature by notary service
                Requesting signature by Notary service
                Validating response from Notary service
        ✅   Broadcasting transaction to participants
    ✅   Done

检查输出
^^^^^^^^^^^^^^^^^^^
We can also issue RPC operations to the node via the interactive shell. Type ``run`` to see the full list of available
operations.

我们也可以通过 shell 对节点来初始 RPC 操作。输入 ``run`` 来查看可用的操作的全部列表。

You can see the newly-created IOU by running ``run vaultQuery contractStateType: com.example.state.IOUState``.

你可以通过运行 ``run vaultQuery contractStateType: com.example.state.IOUState`` 来查看新创建的 IOU。

As before, the interactive shell of PartyC will not display any IOUs.

像以前一样，PartyC 的 shell 中不会显示任何的 IOUs。

通过 h2 web console
~~~~~~~~~~~~~~~~~~~~~~
You can connect directly to your node's database to see its stored states, transactions and attachments. To do so,
please follow the instructions in :doc:`node-database`.

你也可以直接连接到节点的数据库来查看它存储的 states、transactions 以及附件。可以根据 :doc:`node-database` 中的指导做。

在不同的机器上运行节点
-----------------------------
The nodes can be configured to communicate as a network even when distributed across several machines:

即使在不同的机器上，这些节点也是可以被配置成一个可交互的网络的：

* Deploy the nodes as usual:

  * Unix/Mac OSX: ``./gradlew deployNodes``
  * Windows: ``gradlew.bat deployNodes``

* Navigate to the build folder (``workflows-kotlin/build/nodes``)
* For each node, open its ``node.conf`` file and change ``localhost`` in its ``p2pAddress`` to the IP address of the machine
  where the node will be run (e.g. ``p2pAddress="10.18.0.166:10007"``)
* These changes require new node-info files to be distributed amongst the nodes. Use the network bootstrapper tool
  (see :doc:`network-bootstrapper`) to update the files and have them distributed locally:

  ``java -jar network-bootstrapper.jar workflows-kotlin/build/nodes``

* Move the node folders to their individual machines (e.g. using a USB key). It is important that none of the
  nodes - including the notary - end up on more than one machine. Each computer should also have a copy of ``runnodes``
  and ``runnodes.bat``.

  For example, you may end up with the following layout:

  * Machine 1: ``Notary``, ``PartyA``, ``runnodes``, ``runnodes.bat``
  * Machine 2: ``PartyB``, ``PartyC``, ``runnodes``, ``runnodes.bat``

* After starting each node, the nodes will be able to see one another and agree IOUs among themselves


即使在不同的机器上，这些节点也是可以被配置成一个可交互的网络的：

* 像常规那样部署节点:

  * Unix/Mac OSX: ``./gradlew deployNodes``
  * Windows: ``gradlew.bat deployNodes``

* 浏览至 build 文件夹 (``workflows-kotlin/build/nodes``)
* 对于每个节点，打开它的 ``node.conf`` 文件并且在它的 ``p2pAddress`` 改动 ``localhost`` 为节点将要运行的机器的 IP 地址 (比如 ``p2pAddress="10.18.0.166:10007"``)
* 这个改动需要将新的 node-info 文件分发到所有节点。使用 bootstrapper 工具
  (see :doc:`network-bootstrapper`) 来更新这些文件并在本地分发他们:

  ``java -jar network-bootstrapper.jar workflows-kotlin/build/nodes``

* 将节点文件夹放到他们自己的机器上 (比如使用 USB)。很关键的一点是，所有这些节点，包括 notary 都不应该存在于多于一台机器上. 每台电脑上都应该有 ``runnodes`` 和 ``runnodes.bat`` 的副本.

  比如，你可能有下边这样的结构:

  * Machine 1: ``Notary``, ``PartyA``, ``runnodes``, ``runnodes.bat``
  * Machine 2: ``PartyB``, ``PartyC``, ``runnodes``, ``runnodes.bat``

* 在启动每个节点之后，节点就能够看到彼此并在彼此间同意 IOUs 了

.. warning:: The bootstrapper must be run **after** the ``node.conf`` files have been modified, but **before** the nodes 
   are distributed across machines. Otherwise, the nodes will not be able to communicate.

.. warning:: bootstrapper 必须要在 ``node.conf`` 修改 **之后** 并且在节点被分发到不同机器 **之前** 运行。否则节点是不能够进行通信的。

.. note:: If you are using H2 and wish to use the same ``h2port`` value for two or more nodes, you must only assign them that
   value after the nodes have been moved to their individual machines. The initial bootstrapping process requires access to 
   the nodes' databases and if two nodes share the same H2 port, the process will fail.

.. note:: 如果你在使用 H2 并且你想要给两个或多个节点使用相同的 ``h2port`` 的话，你必须在节点被放到他们自己的机器之后再设置这个值。这个初始的 bootstrapping 流程是需要访问节点的数据库的，所以如果两个节点共享了相同的 H2 端口的话，这个过程会失败。

测试你的 CorDapp
--------------------

Corda provides several frameworks for writing unit and integration tests for CorDapps.

Corda 提供了不同的框架来为 CorDapps 编写单元和集成测试。

Contract 测试
~~~~~~~~~~~~~~
You can run the CorDapp's contract tests by running the ``Run Contract Tests - Kotlin`` run configuration.

你可以通过运行 ``Run Contract Tests - Kotlin`` 运行配置来运行 CorDapp 的 contract 测试。

Flow 测试
~~~~~~~~~~
You can run the CorDapp's flow tests by running the ``Run Flow Tests - Kotlin`` run configuration.

你可以通过运行 ``Run Flow Tests - Kotlin`` 运行配置来运行 CorDapp 的 flow 测试。

集成测试
~~~~~~~~~~~~~~~~~
You can run the CorDapp's integration tests by running the ``Run Integration Tests - Kotlin`` run configuration.

你可以通过运行 ``Run Integration Tests - Kotlin`` 运行配置来运行 CorDapp 的集成测试。

.. _tutorial_cordapp_running_tests_intellij:

在 IntelliJ 中运行测试
~~~~~~~~~~~~~~~~~~~~~~~~~

We recommend editing your IntelliJ preferences so that you use the Gradle runner - this means that the quasar utils
plugin will make sure that some flags (like ``-javaagent`` - see :ref:`below <tutorial_cordapp_alternative_test_runners>`) are
set for you.

我们建议变更你的 IntelliJ 的首选项，所以你会使用 Gradle runner - 这意味着 quasar utils plugin 将会确保一些 flags（比如 ``-javaagent`` - 查看 :ref:`下边的 <tutorial_cordapp_alternative_test_runners>`）会为你设置好。

To switch to using the Gradle runner:

想要换成使用 Gradle runner：

* Navigate to ``Build, Execution, Deployment -> Build Tools -> Gradle -> Runner`` (or search for `runner`)

  * Windows: this is in "Settings"
  * MacOS: this is in "Preferences"

* Set "Delegate IDE build/run actions to gradle" to true
* Set "Run test using:" to "Gradle Test Runner"

.. _tutorial_cordapp_alternative_test_runners:

If you would prefer to use the built in IntelliJ JUnit test runner, you can add some code to your ``build.gradle`` file and
it will copy your quasar JAR file to the lib directory. You will also need to specify ``-javaagent:lib/quasar.jar``
and set the run directory to the project root directory for each test.

如果你想要使用 IntelliJ 内置的 Junit test runner，你可以向你的 ``build.gradle`` 文件中添加一些代码，它会将你的 quasar JAR 文件 copy 到 lib 目录。你还需要指定 ``-javaagent:lib/quasar.jar`` 并且设置运行的路径为每个测试的项目的根路径。

Add the following to your ``build.gradle`` file - ideally to a ``build.gradle`` that already contains the quasar-utils plugin line:

将下边的代码添加到你的 ``build.gradle`` 文件 - 理想的是到一个 ``build.gradle`` 已经包含了 quasar-utils plugin line：

.. sourcecode:: groovy

    apply plugin: 'net.corda.plugins.quasar-utils'

    task installQuasar(type: Copy) {
        destinationDir rootProject.file("lib")
        from(configurations.quasar) {
            rename 'quasar-core(.*).jar', 'quasar.jar'
        }
    }


and then you can run ``gradlew installQuasar``.

然后你可以运行 ``gradlew installQuasar``。

Debugging 你的 CorDapp
----------------------

查看 :doc:`debugging-a-cordapp`.
