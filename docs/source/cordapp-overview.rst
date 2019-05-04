什么是 CorDapp？
==================

CorDapps (Corda Distributed Applications) are distributed applications that run on the Corda platform. The goal of a
CorDapp is to allow nodes to reach agreement on updates to the ledger. They achieve this goal by defining flows that
Corda node owners can invoke over RPC:

CorDapps（Corda 分布式应用 Corda Distributed Applications）是在 Corda 平台上运行的分布式应用程序。CorDapp 的目标是允许节点间达成协议来更新账本。为了实现这个目标，Corda 定义了可以由 Corda 节点所有者通过 RPC 调用的 flows：

.. image:: resources/node-diagram.png
   :scale: 25%
   :align: center

CorDapp 组件
------------------
CorDapps take the form of a set of JAR files containing class definitions written in Java and/or Kotlin.

CorDapps 是由一系列的包含使用 Java 和/或者 Kotlin 编写的类定义的 JAR 文件形式构成的。

These class definitions will commonly include the following elements:

* Flows: Define a routine for the node to run, usually to update the ledger
  (see :doc:`Key Concepts - Flows <key-concepts-flows>`). They subclass ``FlowLogic``
* States: Define the facts over which agreement is reached (see :doc:`Key Concepts - States <key-concepts-states>`).
  They implement the ``ContractState`` interface
* Contracts, defining what constitutes a valid ledger update (see
  :doc:`Key Concepts - Contracts <key-concepts-contracts>`). They implement the ``Contract`` interface
* Services, providing long-lived utilities within the node. They subclass ``SingletonSerializationToken``
* Serialisation whitelists, restricting what types your node will receive off the wire. They implement the
  ``SerializationWhitelist`` interface

这些类定义通常包含下边的元素：

* Flows：定义了节点的日常工作，通常是更新账本（查看 :doc:`核心概念 - Flows <key-concepts-flows>`）。他们是 ``FlowLogic`` 的子类
* States：定义了要达成协议的事实（查看 :doc:`核心概念 - States <key-concepts-states>`）。他们实现了 ``ContractState`` 接口
* Contracts：定义了构成有效账本更新的内容（查看 :doc:`核心概念 - Contracts <key-concepts-contracts>`）。他们实现了 ``Contract`` 接口
* Services：在节点中提供了长期的 utilities。他们是 ``SingletonSerializationToken`` 的子类
* 序列化白名单：限制了你的节点会接收什么样的信息。他们实现了 ``SerializationWhitelist`` 接口

But the CorDapp JAR can also include other class definitions. These may include:

* APIs and static web content: These are served by Corda's built-in webserver. This webserver is not
  production-ready, and should be used for testing purposes only
* Utility classes

但是 CorDapp JAR 也能够包含其他的类定义。可能包括：

* APIs 和静态 web 内容：这个是由 Corda 内置的 webserver 来服务的。这个 webserver 还不适合于生产环境，应该仅仅被用于测试的目的
* Utility 类

一个例子
----------
Suppose a node owner wants their node to be able to trade bonds. They may choose to install a Bond Trading CorDapp with
the following components:

* A ``BondState``, used to represent bonds as shared facts on the ledger
* A ``BondContract``, used to govern which ledger updates involving ``BondState`` states are valid
* Three flows:

    * An ``IssueBondFlow``, allowing new ``BondState`` states to be issued onto the ledger
    * A ``TradeBondFlow``, allowing existing ``BondState`` states to be bought and sold on the ledger
    * An ``ExitBondFlow``, allowing existing ``BondState`` states to be exited from the ledger

假设一个节点的所有者想让他的节点能够交易债券。他可能会选择安装一个包含下边组件的债券交易的 CorDapp：

* 一个 ``BondState``，用来体现以共享的事实的形式存在于账本上的债券
* 一个 ``BondContract``，用来证明哪些涉及到 ``Bondstate`` 的账本更新是有效的
* 三个 flows：

    * 一个 ``IssueBondFlow``，允许新的 ``BondState`` 可以被初始化发行到账本中
    * 一个 ``TradeBondFlow``，允许对于一个已经存在的 ``BondState`` 在账本中进行购买和出售
    * 一个 ``ExitBondFlow``，允许将一个已经存在的 ``BondState`` 从账本中清理掉

After installing this CorDapp, the node owner will be able to use the flows defined by the CorDapp to agree ledger
updates related to issuance, sale, purchase and exit of bonds.

当安装完这个 CorDapp 之后，节点的所有者便能够通过 CorDapp 定义的 flows 来同意关于发行、出售、购买和清理债券的一些列账本更新了。

同时在 Corda（开源版本）和 Corda 企业版上编写和构建应用程序
-----------------------------------------------------------------------------------
Corda and Corda Enterprise are compatible and interoperable, which means you can write a CorDapp that can run on both.
To make this work in practice you should follow these steps:

Corda 和 Corda 企业版是兼容的，并且可以互操作，也就是说你可以编写一个 CorDapp 能够同时在两个 Corda 中运行。你需要遵循下边的步骤来实现这个：

1. Ensure your CorDapp is designed per :doc:`Structuring a CorDapp <writing-a-cordapp>` and annotated according to :ref:`CorDapp separation <cordapp_separation_ref>`.
   In particular, it is critical to separate the consensus-critical parts of your application (contracts, states and their dependencies) from
   the rest of the business logic (flows, APIs, etc).
   The former - the **CorDapp kernel** - is the Jar that will be attached to transactions creating/consuming your states and is the Jar
   that any node on the network verifying the transaction must execute.

.. note:: It is also important to understand how to manage any dependencies a CorDapp may have on 3rd party libraries and other CorDapps.
   Please read :ref:`Setting your dependencies <cordapp_dependencies_ref>` to understand the options and recommendations with regards to correctly Jar'ing CorDapp dependencies.

2. Compile this **CorDapp kernel** Jar once, and then depend on it from your workflows Jar (or Jars - see below). Importantly, if
   you want your app to work on both Corda and Corda Enterprise, you must compile this Jar against Corda, not Corda Enterprise.
   This is because, in future, we may add additional functionality to Corda Enterprise that is not in Corda and you may inadvertently create a
   CorDapp kernel that does not work on Corda open source. Compiling against Corda open source as a matter of course prevents this risk, as well
   as preventing the risk that you inadvertently create two different versions of the Jar, which will have different hashes and hence break compatibility
   and interoperability.

.. note:: As of Corda 4 it is recommended to use :ref:`CorDapp Jar signing <cordapp_build_system_signing_cordapp_jar_ref>` to leverage the new signature constraints functionality.

3. Your workflow Jar(s) should depend on the **CorDapp kernel** (contract, states and dependencies). Importantly, you can create different workflow
   Jars for Corda and Corda Enterprise, because the workflows Jar is not consensus critical. For example, you may wish to add additional features
   to your CorDapp for when it is run on Corda Enterprise (perhaps it uses advanced features of one of the supported enterprise databases or includes
   advanced database migration scripts, or some other Enterprise-only feature).

In summary, structure your app as kernel (contracts, states, dependencies) and workflow (the rest) and be sure to compile the kernel
against Corda open source. You can compile your workflow (Jars) against the distribution of Corda that they target.
