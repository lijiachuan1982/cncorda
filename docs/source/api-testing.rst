.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

API: 测试
============

.. contents::

Flow 测试
------------

MockNetwork
^^^^^^^^^^^

Flow testing can be fully automated using a ``MockNetwork`` composed of ``StartedMockNode`` nodes. Each
``StartedMockNode`` behaves like a regular Corda node, but its services are either in-memory or mocked out.

Flow 的测试可以使用一个 ``MockNetwork`` 和 ``StartedMockNode`` 节点来完全自动的执行。每个 ``StartedMockNode`` 就像是一个常规的 Corda 节点，但是它的服务会在内存中或者是虚构的。

A ``MockNetwork`` is created as follows:

一个 ``MockNetwork`` 向下边这样来创建：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/MockNetworkTestsTutorial.kt
        :language: kotlin
        :start-after: DOCSTART 1
        :end-before: DOCEND 1

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/MockNetworkTestsTutorial.java
        :language: java
        :start-after: DOCSTART 1
        :end-before: DOCEND 1

The ``MockNetwork`` requires at a minimum a list of CorDapps to be installed on each ``StartedMockNode``. The CorDapps are looked up on the
classpath by package name, using ``TestCordapp.findCordapp``. ``TestCordapp.findCordapp`` scans the current classpath to find the CorDapp that contains the given package.
This includes all the associated CorDapp metadata present in its MANIFEST.

``MockNetwork`` 至少需要一个将会被安装在每个 ``StartedMockNode`` 上的 CorDapps 列表。使用 ``TestCordapp.findCordapp`` CorDapps 能够通过包名在 classpath 上被查询。``TestCordapp.findCordapp`` 会扫描当前的 classpth 来找到包含指定包的 CorDapp。这包括了所有在它的 MANIFEST 中展示的相关的 CorDapp metadata。

``MockNetworkParameters`` provides other properties for the network which can be tweaked. They default to sensible values if not specified.

``MockNetworkParameters`` 提供给了对于网络的其他属性。如果没有指定值的话，默认会使用有意义的值。

将节点添加到网络
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Nodes are created on the ``MockNetwork`` using:

节点可以在 ``MockNetwork`` 上被创建：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/MockNetworkTestsTutorial.kt
        :language: kotlin
        :start-after: DOCSTART 2
        :end-before: DOCEND 2

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/MockNetworkTestsTutorial.java
        :language: java
        :start-after: DOCSTART 2
        :end-before: DOCEND 2

Nodes added using ``createNode`` are provided a default set of node parameters. However, it is also possible to
provide different parameters to each node using ``MockNodeParameters``. Of particular interest are ``configOverrides`` which allow you to
override some of the default node configuration options. Please refer to the ``MockNodeConfigOverrides`` class for details what can currently
be overridden. Also, the ``additionalCordapps`` parameter allows you to add extra CorDapps to a specific node. This is useful when you wish
for all nodes to load a common CorDapp but for a subset of nodes to load CorDapps specific to their role in the network.

使用 ``createNode`` 创建的节点会被提供一系列的默认的节点参数。然而，也可以使用 ``MockNodeParameters`` 来为每个节点提供不同的参数。其中一个特别的是 ``configOverrides``，它允许你能够重载一些默认的节点配置。请参考 ``MockNodeConfigOverrides`` 类查看当前都有哪些可以被重载。并且 ``additionalCordapps`` 参数允许你想一个指定的节点添加额外的 CorDapp。这对于如果你想要所有的节点都运行一个通用的 CorDapp，但是对于其中的部分节点会加载针对于他们在这个网络中的角色而特定的 CorDapps 的情况更加有用。

运行网络
^^^^^^^^^^^^^^^^^^^
When using a ``MockNetwork``, you must be careful to ensure that all the nodes have processed all the relevant messages 
before making assertions about the result of performing some action. For example, if you start a flow to update the ledger 
but don't wait until all the nodes involved have processed all the resulting messages, your nodes' vaults may not be in 
the state you expect.

当使用一个 ``MockNetwork`` 的时候，在你想要确认在执行一些操作之后的结果的时候，必须要小心地确保所有的节点都已经处理完所有的消息了。比如，如果你开始一个 flow 来更新账本，但是如果没有等到所有相关的节点已经处理完所有结果的信息的话，你的节点的 vaults 可能并没有到达你想要的状态。

When ``networkSendManuallyPumped`` is set to ``false``, you must manually initiate the processing of received messages. 
You manually process received messages as follows:

* ``StartedMockNode.pumpReceive()`` processes a single message from the node's queue
* ``MockNetwork.runNetwork()`` processes all the messages in every node's queue until there are no further messages to
  process

当 ``networkSendManuallyPumped`` 被设置为 ``false`` 的时候，你必须要手动地初始一个接收消息的过程。你可以详下边这样手动地处理接收到的消息：

* ``StartedMockNode.pumpReceive()`` 从节点的 queue 中处理一条信息
* ``MockNetwork.runNetwork()`` 处理每个节点的 queue 里的所有消息，直到没有消息需要被处理
      
When ``networkSendManuallyPumped`` is set to ``true``, nodes will automatically process the messages they receive. You 
can block until all messages have been processed using ``MockNetwork.waitQuiescent()``.

当 ``networkSendManuallyPumped`` 被设置为 ``true`` 的时候，节点将会自动地处理接收到的消息。你可以使用 ``MockNetwork.waitQuiescent()`` 来阻塞知道所有的消息都被处理。

.. warning:: If ``threadPerNode`` is set to ``true``, ``networkSendManuallyPumped`` must also be set to ``true``.

.. warning:: 如果 ``threadPerNode`` 被设置为 ``true``，``networkSendManuallyPumped`` 必须也被设置为 ``true``。

运行 flows
^^^^^^^^^^^^^

A ``StartedMockNode`` starts a flow using the ``StartedNodeServices.startFlow`` method. This method returns a future
representing the output of running the flow.

``StartedMockNode`` 使用 ``StartedNodeServices.startFlow`` 启动一个 flow。这个方法返回一个运行这个 flow 会在将来产生的 output。

.. container:: codeset

   .. sourcecode:: kotlin

        val signedTransactionFuture = nodeA.services.startFlow(IOUFlow(iouValue = 99, otherParty = nodeBParty))

   .. sourcecode:: java

        CordaFuture<SignedTransaction> future = startFlow(a.getServices(), new ExampleFlow.Initiator(1, nodeBParty));

The network must then be manually run before retrieving the future's value:

网络在接收将来的值之前必须要被手动地运行：

.. container:: codeset

   .. sourcecode:: kotlin

        val signedTransactionFuture = nodeA.services.startFlow(IOUFlow(iouValue = 99, otherParty = nodeBParty))
        // Assuming network.networkSendManuallyPumped == false.
        network.runNetwork()
        val signedTransaction = future.get();

   .. sourcecode:: java

        CordaFuture<SignedTransaction> future = startFlow(a.getServices(), new ExampleFlow.Initiator(1, nodeBParty));
        // Assuming network.networkSendManuallyPumped == false.
        network.runNetwork();
        SignedTransaction signedTransaction = future.get();

在内部访问 ``StartedMockNode``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

查询节点的 vault
~~~~~~~~~~~~~~~~~~~~~~~

Recorded states can be retrieved from the vault of a ``StartedMockNode`` using:

可以使用下边的代码从一个 ``StartedMockNode`` 的 vault 中获取记录的 states：

.. container:: codeset

   .. sourcecode:: kotlin

        val myStates = nodeA.services.vaultService.queryBy<MyStateType>().states

   .. sourcecode:: java

        List<MyStateType> myStates = node.getServices().getVaultService().queryBy(MyStateType.class).getStates();

This allows you to check whether a given state has (or has not) been stored, and whether it has the correct attributes.

这就允许你能够检查对于一个给定的 state 是否已经被存储了，以及它是否含有正确的属性。

检查一个节点的交易存储
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Recorded transactions can be retrieved from the transaction storage of a ``StartedMockNode`` using:

可以使用下边的代码从一个 ``StartedMockNode`` 的交易存储中获取回来已经记录的交易信息：

.. container:: codeset

   .. sourcecode:: kotlin

        val transaction = nodeA.services.validatedTransactions.getTransaction(transaction.id)

   .. sourcecode:: java

        SignedTransaction transaction = nodeA.getServices().getValidatedTransactions().getTransaction(transaction.getId())

This allows you to check whether a given transaction has (or has not) been stored, and whether it has the correct
attributes.

这就允许你能够检查对于一个给定的交易是否已经被存储了，以及它是否含有正确的属性。

This allows you to check whether a given state has (or has not) been stored, and whether it has the correct attributes.

这就允许你能够检查对于一个给定的 state 是否已经被存储了，以及它是否含有正确的属性。

更多的例子
^^^^^^^^^^^^^^^^

* See the flow testing tutorial :doc:`here <flow-testing>`
* See the oracle tutorial :doc:`here <oracles>` for information on testing ``@CordaService`` classes
* Further examples are available in the Example CorDapp in
  `Java <https://github.com/corda/samples/blob/release-V|platform_version|/cordapp-example/workflows-java/src/test/java/com/example/flow/IOUFlowTests.java>`_ and
  `Kotlin <https://github.com/corda/samples/blob/release-V|platform_version|/cordapp-example/workflows-kotlin/src/test/kotlin/com/example/flow/IOUFlowTests.kt>`_

* 查看 :doc:`这里 <flow-testing>` 了解 flow 测试教程
* 查看 :doc:`here <oracles>` 了解 Oracle 教程及对于测试 ``@CordaService`` 类的信息
* 在样例 CorDapp 中更多的例子
  `Java <https://github.com/corda/samples/blob/release-V|platform_version|/cordapp-example/workflows-java/src/test/java/com/example/flow/IOUFlowTests.java>`_ and
  `Kotlin <https://github.com/corda/samples/blob/release-V|platform_version|/cordapp-example/workflows-kotlin/src/test/kotlin/com/example/flow/IOUFlowTests.kt>`_

Contract 测试
----------------

The Corda test framework includes the ability to create a test ledger by calling the ``ledger`` function
on an implementation of the ``ServiceHub`` interface.

Corda 测试框架包含了通过在一个 ``ServiceHub`` 接口的实现之上调用 ``ledger`` 方法创建一个测试账本的能力。

测试 identities
^^^^^^^^^^^^^^^

You can create dummy identities to use in test transactions using the ``TestIdentity`` class:

你可以使用 ``TestIdentity`` 类来创建可以用于测试交易的虚构的 identities：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/test/kotlin/net/corda/docs/kotlin/tutorial/testdsl/TutorialTestDSL.kt
        :language: kotlin
        :start-after: DOCSTART 14
        :end-before: DOCEND 14
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/test/java/net/corda/docs/java/tutorial/testdsl/TutorialTestDSL.java
        :language: java
        :start-after: DOCSTART 14
        :end-before: DOCEND 14
        :dedent: 4

``TestIdentity`` exposes the following fields and methods:

``TestIdentity`` 暴露了下边的字段和方法：

.. container:: codeset

   .. sourcecode:: kotlin

        val identityParty: Party = bigCorp.party
        val identityName: CordaX500Name = bigCorp.name
        val identityPubKey: PublicKey = bigCorp.publicKey
        val identityKeyPair: KeyPair = bigCorp.keyPair
        val identityPartyAndCertificate: PartyAndCertificate = bigCorp.identity

   .. sourcecode:: java

        Party identityParty = bigCorp.getParty();
        CordaX500Name identityName = bigCorp.getName();
        PublicKey identityPubKey = bigCorp.getPublicKey();
        KeyPair identityKeyPair = bigCorp.getKeyPair();
        PartyAndCertificate identityPartyAndCertificate = bigCorp.getIdentity();

You can also create a unique ``TestIdentity`` using the ``fresh`` method:

你也可以使用 ``fresh`` 方法创建一个唯一的 ``TestIdentity``：

.. container:: codeset

   .. sourcecode:: kotlin

        val uniqueTestIdentity: TestIdentity = TestIdentity.fresh("orgName")

   .. sourcecode:: java

        TestIdentity uniqueTestIdentity = TestIdentity.Companion.fresh("orgName");

MockServices
^^^^^^^^^^^^

A mock implementation of ``ServiceHub`` is provided in ``MockServices``. This is a minimal ``ServiceHub`` that
suffices to test contract logic. It has the ability to insert states into the vault, query the vault, and
construct and check transactions.

在 ``MockServices`` 中提供了对于 ``ServiceHub`` 的一个虚拟的实现。这是一个最小化的 ``ServiceHub`` 足够用来测试 contract 逻辑。它能够将 states 插入到 vault，查询 vault，以及构建和检查 transactions。

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/test/kotlin/net/corda/docs/kotlin/tutorial/testdsl/TutorialTestDSL.kt
        :language: kotlin
        :start-after: DOCSTART 11
        :end-before: DOCEND 11
        :dedent: 4

    .. literalinclude:: ../../docs/source/example-code/src/test/java/net/corda/docs/java/tutorial/testdsl/TutorialTestDSL.java
        :language: java
        :start-after: DOCSTART 11
        :end-before: DOCEND 11
        :dedent: 4


Alternatively, there is a helper constructor which just accepts a list of ``TestIdentity``. The first identity provided is
the identity of the node whose ``ServiceHub`` is being mocked, and any subsequent identities are identities that the node
knows about. Only the calling package is scanned for cordapps and a test ``IdentityService`` is created
for you, using all the given identities.

或者，这里还有一个仅仅接收一个 ``TestIdentity`` 列表的 helper 构造函数。提供的第一个 identity 是模拟 ``ServiceHub`` 的节点的 identity，后续的 identities 是这个节点了解的其他的节点的 identities。只有这个调用的包会被扫描 CorDapps 并且一个测试的 ``IdentityService`` 会使用所有给定的属性被创建。

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/test/kotlin/net/corda/docs/kotlin/tutorial/testdsl/TutorialTestDSL.kt
        :language: kotlin
        :start-after: DOCSTART 12
        :end-before: DOCEND 12
        :dedent: 4

    .. literalinclude:: ../../docs/source/example-code/src/test/java/net/corda/docs/java/tutorial/testdsl/TutorialTestDSL.java
        :language: java
        :start-after: DOCSTART 12
        :end-before: DOCEND 12
        :dedent: 4


使用一个测试账本来编写测试
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``ServiceHub.ledger`` extension function allows you to create a test ledger. Within the ledger wrapper you can create
transactions using the ``transaction`` function. Within a transaction you can define the ``input`` and
``output`` states for the transaction, alongside any commands that are being executed, the ``timeWindow`` in which the
transaction has been executed, and any ``attachments``, as shown in this example test:

``ServiceHub.ledger`` 扩展方法允许你能够创建一个测试的账本。在账本的 wrapper 中，你可以使用 ``transaction`` 方法来创建 transactions。在一个 transaction 中，你可以为 transaction 定义 ``input`` 和 ``output`` states，以及任何的会被执行的 commands，transaction 需要遵循的 ``timeWindow``，和任何的 ``attachments``，就像下边的测试例子：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/test/kotlin/net/corda/docs/kotlin/tutorial/testdsl/TutorialTestDSL.kt
        :language: kotlin
        :start-after: DOCSTART 13
        :end-before: DOCEND 13
        :dedent: 4

    .. literalinclude:: ../../docs/source/example-code/src/test/java/net/corda/docs/java/tutorial/testdsl/TutorialTestDSL.java
        :language: java
        :start-after: DOCSTART 13
        :end-before: DOCEND 13
        :dedent: 4

Once all the transaction components have been specified, you can run ``verifies()`` to check that the given transaction is valid.

当所有的 transaction 组件都被指定，你可以运行 ``verifies()`` 来检查给定的 transaction 是否是有效的。

检查失败的 states
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In order to test for failures, you can use the ``failsWith`` method, or in Kotlin the ``fails with`` helper method, which
assert that the transaction fails with a specific error. If you just want to assert that the transaction has failed without
verifying the message, there is also a ``fails`` method.

为了测试失败的情况，你可以使用 ``failsWith`` 方法，或者在 kotlin 中 ``fails with`` helper 方法，它可以造成 transaction 由于一个指定的错误而失败。如果你只是想造成 transaction 失败而不需要指定消息的话，也可以使用 ``fails`` 方法。

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/test/kotlin/net/corda/docs/kotlin/tutorial/testdsl/TutorialTestDSL.kt
        :language: kotlin
        :start-after: DOCSTART 4
        :end-before: DOCEND 4
        :dedent: 4

    .. literalinclude:: ../../docs/source/example-code/src/test/java/net/corda/docs/java/tutorial/testdsl/TutorialTestDSL.java
        :language: java
        :start-after: DOCSTART 4
        :end-before: DOCEND 4
        :dedent: 4

.. note::

    The transaction DSL forces the last line of the test to be either a ``verifies`` or ``fails with`` statement.

.. note:: Transaction DSL 强制这个测试的最后一行或者是一个 ``verifies`` 或者是 ``fails with`` 语句。

一次测试多个场景
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Within a single transaction block, you can assert several times that the transaction constructed so far either passes or
fails verification. For example, you could test that a contract fails to verify because it has no output states, and then
add the relevant output state and check that the contract verifies successfully, as in the following example:

在一个 transaction 块中，对于构建好的一个 transaction，你可以制造出多次的成功或者失败的验证。比如，你可以测试一个 contract 由于没有 output states 而失败，然后添加相关的 output state 并且检查这个 contract 是否能够成功，像下边的例子那样：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/test/kotlin/net/corda/docs/kotlin/tutorial/testdsl/TutorialTestDSL.kt
        :language: kotlin
        :start-after: DOCSTART 5
        :end-before: DOCEND 5
        :dedent: 4

    .. literalinclude:: ../../docs/source/example-code/src/test/java/net/corda/docs/java/tutorial/testdsl/TutorialTestDSL.java
        :language: java
        :start-after: DOCSTART 5
        :end-before: DOCEND 5
        :dedent: 4

You can also use the ``tweak`` function to create a locally scoped transaction that you can make changes to
and then return to the original, unmodified transaction. As in the following example:

你也可以使用 ``tweak`` 方法来创建一个本地范围的 transaction，你就可以对它进行改动然后返回给原始的没有改变过的 transaction。像下边的例子那样：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/test/kotlin/net/corda/docs/kotlin/tutorial/testdsl/TutorialTestDSL.kt
        :language: kotlin
        :start-after: DOCSTART 7
        :end-before: DOCEND 7
        :dedent: 4

    .. literalinclude:: ../../docs/source/example-code/src/test/java/net/corda/docs/java/tutorial/testdsl/TutorialTestDSL.java
        :language: java
        :start-after: DOCSTART 7
        :end-before: DOCEND 7
        :dedent: 4


将 transactions 链起来
~~~~~~~~~~~~~~~~~~~~~

The following example shows that within a ``ledger``, you can create more than one ``transaction`` in order to test chains
of transactions. In addition to ``transaction``, ``unverifiedTransaction`` can be used, as in the example below, to create
transactions on the ledger without verifying them, for pre-populating the ledger with existing data. When chaining transactions,
it is important to note that even though a ``transaction`` ``verifies`` successfully, the overall ledger may not be valid. This can
be verified separately by placing a ``verifies`` or ``fails`` statement  within the ``ledger`` block.

下边的例子显示了在一个 ``ledger`` 中，你可以创建多于一个的 ``transaction`` 来测试 transactions 链。除了 ``transaction``,``unverifiedTransaction`` 也可以像下边的例子那样被用来在账本上创建 transaction 而不需要验证它们，以此向账本中预先录入一些已经存在的数据。当把 transactions 链起来的时候，很重要的需要注意的一点是尽管一个 ``transaction`` ``verifies`` 成功了，但是整个账本可能不是有效的。这个可以通过使用一个在 ``ledger`` 中的 ``verifies`` 或者 ``fails`` 语句分别来验证。

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/test/kotlin/net/corda/docs/kotlin/tutorial/testdsl/TutorialTestDSL.kt
        :language: kotlin
        :start-after: DOCSTART 9
        :end-before: DOCEND 9
        :dedent: 4

    .. literalinclude:: ../../docs/source/example-code/src/test/java/net/corda/docs/java/tutorial/testdsl/TutorialTestDSL.java
        :language: java
        :start-after: DOCSTART 9
        :end-before: DOCEND 9
        :dedent: 4


更多的例子
^^^^^^^^^^^^^^^^

* See the flow testing tutorial :doc:`here <tutorial-test-dsl>`
* Further examples are available in the Example CorDapp in
  `Java <https://github.com/corda/samples/blob/release-V|platform_version|/cordapp-example/workflows-java/src/test/java/com/example/flow/IOUFlowTests.java>`_ and
  `Kotlin <https://github.com/corda/samples/blob/release-V|platform_version|/cordapp-example/workflows-kotlin/src/test/kotlin/com/example/flow/IOUFlowTests.kt>`_

* 在 :doc:`这里 <tutorial-test-dsl>` 查看 flow 测试教程
* 在 CorDapp 例子中更过的例子在
  `Java <https://github.com/corda/samples/blob/release-V|platform_version|/cordapp-example/workflows-java/src/test/java/com/example/flow/IOUFlowTests.java>`_ and
  `Kotlin <https://github.com/corda/samples/blob/reease-V|platform_version|/cordapp-example/workflows-kotlin/src/test/kotlin/com/example/flow/IOUFlowTests.kt>`_
