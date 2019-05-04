.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

API: Flows
==========

.. note:: Before reading this page, you should be familiar with the key concepts of :doc:`key-concepts-flows`.

.. note:: 在阅读这里之前，你应该已经熟悉了核心概念 :doc:`key-concepts-flows`。

.. contents::

一个 Flow 的例子
---------------
Before we discuss the API offered by the flow, let's consider what a standard flow may look like.

在我们讨论 flow 提供的 API 之前，让我们来想一下一个标准的 flow 应该像什么样子。

Imagine a flow for agreeing a basic ledger update between Alice and Bob. This flow will have two sides:

* An ``Initiator`` side, that will initiate the request to update the ledger
* A ``Responder`` side, that will respond to the request to update the ledger

我们可以想象一个 Alice 和 Bob 之间同意一个基本的账本更新的 flow。这个 flow 会包含两边：

* ``初始者`` 的一边，会发起更新账本的请求
* ``反馈者`` 的一边，会对更新账本的请求进行反馈

初始者
^^^^^^^^^
In our flow, the Initiator flow class will be doing the majority of the work:

在我们的 flow 中， Initiator flow 类将会处理主要的工作：

*Part 1 - Build the transaction*

1. Choose a notary for the transaction
2. Create a transaction builder
3. Extract any input states from the vault and add them to the builder
4. Create any output states and add them to the builder
5. Add any commands, attachments and time-window to the builder

*Part1 - 创建 transaction

1. 为 transaction 选择一个 notary
2. 创建一个 transaction builder
3. 提取出所有需要的来自 vault 的 input states 并把他们加入到 builder
4. 创建所有需要的 output states 并把他们加入到 builder
5. 向 builder 里添加所有需要的 commands，attachment 和 time-window

*Part 2 - Sign the transaction*

6. Sign the transaction builder
7. Convert the builder to a signed transaction

Part2 - 为 transaction 提供签名

6. 为 transaction builder 提供签名
7. 将这个 builder 转换成一个 signed transaction

*Part 3 - Verify the transaction*

8. Verify the transaction by running its contracts

Part3 - 确认 transaction

8. 通过执行 transaction 的 contracts 来验证这个 transaction

*Part 4 - Gather the counterparty's signature*

9. Send the transaction to the counterparty
10. Wait to receive back the counterparty's signature
11. Add the counterparty's signature to the transaction
12. Verify the transaction's signatures

Part4 - 搜集合作方的签名

9. 将 transaction 发送给 counterparty
10. 等待接收 counterparty 的签名
11. 将 counterparty 的签名添加到 transaction
12. 验证 transaction 的签名

*Part 5 - Finalize the transaction*

13. Send the transaction to the notary
14. Wait to receive back the notarised transaction
15. Record the transaction locally
16. Store any relevant states in the vault
17. Send the transaction to the counterparty for recording

Part5 - 结束 transaction

13. 将 transaction 发送给 notary
14. 等待接收 notarised transaction 的反馈
15. 将 transaction 存储到本地
16. 将所有相关的 states 存储到 vault
17. 将 transaction 发送到 counterparty 去记录

We can visualize the work performed by initiator as follows:

我们可以用下边的 flow 图来表示这个工作流程：

.. image:: resources/flow-overview.png

反馈方
^^^^^^^^^
To respond to these actions, the responder takes the following steps:

为了对这些动作进行反馈， responder 进行一下步骤的操作：

*Part 1 - Sign the transaction*

1. Receive the transaction from the counterparty
2. Verify the transaction's existing signatures
3. Verify the transaction by running its contracts
4. Generate a signature over the transaction
5. Send the signature back to the counterparty

Part1 - 为 transaction 提供签名

1. 从 counterparty 接收 transaction
2. 验证 transaction 中已经存在的签名
3. 通过执行 transaction 的 contracts 来验证 transaction
4. 对该 transaction 生成自己的签名
5. 将签名发送回给 counterparty

*Part 2 - Record the transaction*

6. Receive the notarised transaction from the counterparty
7. Record the transaction locally
8. Store any relevant states in the vault

Part2 - 记录 transaction

6. 从 counterparty 那边接收 notarised transaction
7. 将 transaction 记录到本地
8. 将所有相关的 states 记录到 vault

FlowLogic
---------
In practice, a flow is implemented as one or more communicating ``FlowLogic`` subclasses. The ``FlowLogic``
subclass's constructor can take any number of arguments of any type. The generic of ``FlowLogic`` (e.g.
``FlowLogic<SignedTransaction>``) indicates the flow's return type.

常规来讲，一个 flow 会作为一个或者多个 ``FlowLogic`` 子类被实现的。``FlowLogic`` 子类的构造体能够包含任意数量任意类型的参数。通常的 ``FlowLogic``（比如 ``FlowLogic<SignedTransaction>```）表明了 flow 的返回类型。

.. container:: codeset

   .. sourcecode:: kotlin

        class Initiator(val arg1: Boolean,
                        val arg2: Int,
                        val counterparty: Party): FlowLogic<SignedTransaction>() { }

        class Responder(val otherParty: Party) : FlowLogic<Unit>() { }

   .. sourcecode:: java

        public static class Initiator extends FlowLogic<SignedTransaction> {
            private final boolean arg1;
            private final int arg2;
            private final Party counterparty;

            public Initiator(boolean arg1, int arg2, Party counterparty) {
                this.arg1 = arg1;
                this.arg2 = arg2;
                this.counterparty = counterparty;
            }

        }

        public static class Responder extends FlowLogic<Void> { }

FlowLogic 注解
---------------------
Any flow from which you want to initiate other flows must be annotated with the ``@InitiatingFlow`` annotation.
Additionally, if you wish to start the flow via RPC, you must annotate it with the ``@StartableByRPC`` annotation:

任何你想要用来出发另一个 flow 的 flow，必须要用 ```@InitiatingFlow`` 这个 注解来进行标注。并且，如果你希望通过 RPC 来开始一个 flow，你必须使用 ``@StartableByRPC`` 这个注解：

.. container:: codeset

   .. sourcecode:: kotlin

        @InitiatingFlow
        @StartableByRPC
        class Initiator(): FlowLogic<Unit>() { }

   .. sourcecode:: java

        @InitiatingFlow
        @StartableByRPC
        public static class Initiator extends FlowLogic<Unit> { }

Meanwhile, any flow that responds to a message from another flow must be annotated with the ``@InitiatedBy`` annotation.
``@InitiatedBy`` takes the class of the flow it is responding to as its single parameter:

同时，任何一个作为对一个其他 flow 提供反馈的 flow，也必须使用 ``@InitiatedBy`` 这个 注解进行标注。``@InitiatedBy`` 会使用它要反馈的 flow 的 class 作为唯一的一个参数：

.. container:: codeset

   .. sourcecode:: kotlin

        @InitiatedBy(Initiator::class)
        class Responder(val otherSideSession: FlowSession) : FlowLogic<Unit>() { }

   .. sourcecode:: java

        @InitiatedBy(Initiator.class)
        public static class Responder extends FlowLogic<Void> { }

Additionally, any flow that is started by a ``SchedulableState`` must be annotated with the ``@SchedulableFlow``
annotation.

另外，任何由 ``SchedulableState`` 开始的 flow 需要使用 ```@SchedulableFlow`` 这个 注解进行标注。

Call
----
Each ``FlowLogic`` subclass must override ``FlowLogic.call()``, which describes the actions it will take as part of
the flow. For example, the actions of the initiator's side of the flow would be defined in ``Initiator.call``, and the
actions of the responder's side of the flow would be defined in ``Responder.call``.

每一个 ``FlowLogic`` 子类必须要重写 ``FlowLogic.call()```，该方法描述了作为 flow 的一部分要执行怎样的动作。比如，flow 发起方的动作应该在 ``Initiator.call`` 中定义，反馈方的动作应该在 ``Responder.call`` 中定义。

In order for nodes to be able to run multiple flows concurrently, and to allow flows to survive node upgrades and
restarts, flows need to be checkpointable and serializable to disk. This is achieved by marking ``FlowLogic.call()``,
as well as any function invoked from within ``FlowLogic.call()``, with an ``@Suspendable`` annotation.

为了让节点能够同时运行多个 flows，并且能够让 flows 在节点升级或者重启之后依旧可继续接着执行，flows 需要是 checkpointable 并且可以被序列化到磁盘的。这个可以通过将 ``FlowLogic.call()`` 和由 ``FlowLogic.call()`` 来调用的任何的方法上都带有 ```@Suspendable`` 注解。

.. container:: codeset

   .. sourcecode:: kotlin

        class Initiator(val counterparty: Party): FlowLogic<Unit>() {
            @Suspendable
            override fun call() { }
        }

   .. sourcecode:: java

        public static class InitiatorFlow extends FlowLogic<Void> {
            private final Party counterparty;

            public Initiator(Party counterparty) {
                this.counterparty = counterparty;
            }

            @Suspendable
            @Override
            public Void call() throws FlowException { }

        }

ServiceHub
----------
Within ``FlowLogic.call``, the flow developer has access to the node's ``ServiceHub``, which provides access to the
various services the node provides. We will use the ``ServiceHub`` extensively in the examples that follow. You can
also see :doc:`api-service-hub` for information about the services the ``ServiceHub`` offers.

在 ``FlowLogic.call`` 中，flow 开发者可以访问节点的 ``ServiceHub``，其提供了访问节点所提供的非常多的服务。我们会在例子中非常多的使用 ``ServiceHub``。你也可以查看 :doc:`api-service-hub` 来了解 ``ServiceHub`` 都提供了哪些服务。

常规 flow 任务
-----------------
There are a number of common tasks that you will need to perform within ``FlowLogic.call`` in order to agree ledger
updates. This section details the API for common tasks.

在 ``FlowLogic.call`` 中你可以使用很多常规的任务来同意一个账本的更新。下边的部分会介绍大部分常用的任务。

构建 transaction
^^^^^^^^^^^^^^^^^^^^
The majority of the work performed during a flow will be to build, verify and sign a transaction. This is covered 
in :doc:`api-transactions`.

在一个 flow 中主要要执行的工作就是构建、确认一个 transaction 并提供签名。这个可以查看 :doc:`api-transactions`。

从 vault 中获得 states
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When building a transaction, you'll often need to extract the states you wish to consume from the vault. This is 
covered in :doc:`api-vault-query`.

当构建一个 transaction 的时候，你经常需要从账本上获得你希望去消费掉的 state。这个可以查看 :doc:`api-vault-query`。

获得其他节点的信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
We can retrieve information about other nodes on the network and the services they offer using
``ServiceHub.networkMapCache``.

我们可以使用 ``ServiceHub.networkMapCache`` 来获得网络中其他节点的信息，包括提供哪些服务。

Notaries
~~~~~~~~
Remember that a transaction generally needs a notary to:

* Prevent double-spends if the transaction has inputs
* Serve as a timestamping authority if the transaction has a time-window

一个 transaction 通常大多需要一个 notary 来：

* 如果 transaction 有 input 的话，需要避免双花
* 如果 transaction 有 time-window 的话，要确保 transaction 只能在指定的 time-window 里被执行

There are several ways to retrieve a notary from the network map:

有很多方法来从 network map 那里获得一个 notary：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 01
        :end-before: DOCEND 01
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 01
        :end-before: DOCEND 01
        :dedent: 12

指定 counterparties
~~~~~~~~~~~~~~~~~~~~~~~
We can also use the network map to retrieve a specific counterparty:

我们也可以使用 network map 来获取一个指定的 counterparty 的信息：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 02
        :end-before: DOCEND 02
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 02
        :end-before: DOCEND 02
        :dedent: 12

在 parties 之间进行沟通
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to create a communication session between your initiator flow and the receiver flow you must call
``initiateFlow(party: Party): FlowSession``

为了在你的 initiator flow 和 receiver flow 之间创建一个沟通 session，你必须要调用 ``initiateFlow(party: Party): FlowSession``

``FlowSession`` instances in turn provide three functions:

* ``send(payload: Any)``
    * Sends the ``payload`` object
* ``receive(receiveType: Class<R>): R``
    * Receives an object of type ``receiveType``
* ``sendAndReceive(receiveType: Class<R>, payload: Any): R``
    * Sends the ``payload`` object and receives an object of type ``receiveType`` back

``FlowSession`` 实例提供三个方法：

* ``send(payload: Any)``: 发送 ``payload`` 对象
* ``receive(receiveType: Class<R>): R``: 接收 ``receiveType`` 类型的对象
* ``sendAndReceive(receiveType: Class<R>, payload: Any): R``: 发送 ``payload`` 对象并且接收 ``receiveType`` 类型的对象

In addition ``FlowLogic`` provides functions that batch receives:

* ``receiveAllMap(sessions: Map<FlowSession, Class<out Any>>): Map<FlowSession, UntrustworthyData<Any>>``
  Receives from all ``FlowSession`` objects specified in the passed in map. The received types may differ.
* ``receiveAll(receiveType: Class<R>, sessions: List<FlowSession>): List<UntrustworthyData<R>>``
  Receives from all ``FlowSession`` objects specified in the passed in list. The received types must be the same.

另外，``FlowLogic`` 也提供了批量接收的方法：

* ``receiveAllMap(sessions: Map<FlowSession, Class<out Any>>): Map<FlowSession, UntrustworthyData<Any>>``
 接收来自于传入的 map 中所有 ``FlowSession``。所接收到的类型可能不同。
* ``receiveAll(receiveType: Class<R>, sessions: List<FlowSession>): List<UntrustworthyData<R>>``
 接收来自于传入的 list 中所有 ``FlowSession``对象。所接收到的类型必须相同。

The batched functions are implemented more efficiently by the flow framework.

Flow framework 将批量方法实现的很有效率。

InitiateFlow
~~~~~~~~~~~~

``initiateFlow`` creates a communication session with the passed in ``Party``.

initiateFlow 创建了一个同传进来的 ``Party`` 的一个沟通 session。

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART initiateFlow
        :end-before: DOCEND initiateFlow
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART initiateFlow
        :end-before: DOCEND initiateFlow
        :dedent: 12

Note that at the time of call to this function no actual communication is done, this is deferred to the first
send/receive, at which point the counterparty will either:

1. Ignore the message if they are not registered to respond to messages from this flow.
2. Start the flow they have registered to respond to this flow.

注意当调用这个方法的时候，还没有真实的沟通，这个会被推迟到第一次发送/接收的时候，在那个时间点 counterparty 会：

1. 如果他们没有被注册为这个 flow 提供反馈的话，会忽略这个消息
1. 如果他们被注册为针对这个 flow 要提供反馈的话，会开始这个 flow

Send
~~~~

Once we have a ``FlowSession`` object we can send arbitrary data to a counterparty:

一旦我们有了一个 ``FlowSession`` 对象的话，我们就可以向 counterparty 发送任何的数据了：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 04
        :end-before: DOCEND 04
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 04
        :end-before: DOCEND 04
        :dedent: 12

The flow on the other side must eventually reach a corresponding ``receive`` call to get this message.

在另一方的 flow 最终必须要调用一个对应的 ``receive`` 来获得这个消息。

Receive
~~~~~~~
We can also wait to receive arbitrary data of a specific type from a counterparty. Again, this implies a corresponding
``send`` call in the counterparty's flow. A few scenarios:

* We never receive a message back. In the current design, the flow is paused until the node's owner kills the flow.
* Instead of sending a message back, the counterparty throws a ``FlowException``. This exception is propagated back
  to us, and we can use the error message to establish what happened.
* We receive a message back, but it's of the wrong type. In this case, a ``FlowException`` is thrown.
* We receive back a message of the correct type. All is good.

我们也可以等待从一个 counterparty 那里接收任何的数据。这就意味着在 counterparty 的 flow 中需要调用对应的 ``send`` 方法。以下是几种情况：

* 我们从来没有收到一个返回的消息。在当前的设计中，flow 会被暂停直到节点的 owner 结束了 flow
* counterparty 抛出了一个 ``FlowException`` 而不是返回一个消息。这个异常会传回给我们，我们可以通过这个异常来判断发生了什么错误
* 我们收到了返回的消息，但是是一个错误的类型。这个时候，一个 ``FlowException`` 异常会被抛出
* 我们收到了一个类型正确的消息，一切正常。

Upon calling ``receive`` (or ``sendAndReceive``), the ``FlowLogic`` is suspended until it receives a response.

当调用了 ``receive``（或者 ``sendAndReceive``）方法的时候，``FlowLogic`` 会被挂起直到它收到了一个反馈。

We receive the data wrapped in an ``UntrustworthyData`` instance. This is a reminder that the data we receive may not
be what it appears to be! We must unwrap the ``UntrustworthyData`` using a lambda:

我们收到的数据会被打包在一个 ``UntrustworthyData`` 实例中。这提醒了我们我们收到的数据可能并不像它看起来的那样！我们必须要使用 lambda 来将 ``UntrustworthyData`` 拆包：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 05
        :end-before: DOCEND 05
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 05
        :end-before: DOCEND 05
        :dedent: 12

We're not limited to sending to and receiving from a single counterparty. A flow can send messages to as many parties
as it likes, and each party can invoke a different response flow:

我们也不会限制只能给一个 counterparty 发消息或者只能从一个 counterparty 那里收到消息。一个 flow 可以给任意多的 parties 发送消息，并且每个 party 可以调用不同的 response flow：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 06
        :end-before: DOCEND 06
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 06
        :end-before: DOCEND 06
        :dedent: 12

.. warning:: If you initiate several flows from the same ``@InitiatingFlow`` flow then on the receiving side you must be
   prepared to be initiated by any of the corresponding ``initiateFlow()`` calls! A good way of handling this ambiguity
   is to send as a first message a "role" message to the initiated flow, indicating which part of the initiating flow
   the rest of the counter-flow should conform to. For example send an enum, and on the other side start with a switch
   statement.

.. 如果你从 ``@InitiatingFlow`` flow 中初始了多个 flows 的话，在接收方那边，你应该准备好被任何对应的 ``initiateFlow()`` 来调用。一种处理这个问题的方法是发送第一条 “role” 的消息给被初始化的 flow，说明一下对方的 flow 应该确认的应该是这个 initiating flow 的哪一部分。比如发送了一个枚举值，那么对方就应该使用 switch 语句来处理。

SendAndReceive
~~~~~~~~~~~~~~
We can also use a single call to send data to a counterparty and wait to receive data of a specific type back. The
type of data sent doesn't need to match the type of the data received back:

我们也可以使用一个调用来向 counterparty 发送数据并且等待一个指定类型的返回数据。发送的数据类型不需要必须和收到的返回数据类型一致：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 07
        :end-before: DOCEND 07
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 07
        :end-before: DOCEND 07
        :dedent: 12

Counterparty response
~~~~~~~~~~~~~~~~~~~~~
Suppose we're now on the ``Responder`` side of the flow. We just received the following series of messages from the
``Initiator``:

1. They sent us an ``Any`` instance
2. They waited to receive an ``Integer`` instance back
3. They sent a ``String`` instance and waited to receive a ``Boolean`` instance back

假设我们现在是在 flow 对应的 ``Responder`` 的节点。我们刚刚收到了来自于 ``Initiator`` 的下边的一系列消息：

1. 他们发送给我们 ``Any`` 实例
2. 他们正在等待收到一个 ``Integer`` 类型的返回实例
3. 他们发送了一个 ``String`` 的实例并且在等待收到一个 ``Boolean`` 类型的返回实例

Our side of the flow must mirror these calls. We could do this as follows:

我们这边的 flow 也必须要反映出这样的调用。我们可以：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 08
        :end-before: DOCEND 08
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 08
        :end-before: DOCEND 08
        :dedent: 12

为什么要 Session？
^^^^^^^^^^^^^^^^^^^^

Before ``FlowSession`` s were introduced the send/receive API looked a bit different. They were functions on
``FlowLogic`` and took the address ``Party`` as argument. The platform internally maintained a mapping from ``Party`` to
session, hiding sessions from the user completely.

在 ``FlowSesion`` 被引入之前，send/receive API 看起来是有点不同的。他们是在 ``FlowLogic`` 上的功能并且是将 ``Party`` 作为参数。这个平台在内部会维护一个从 ``Party`` 到 session 的 mapping，对用户完全将 session 隐藏起来。

Although this is a convenient API it introduces subtle issues where a message that was originally meant for a specific
session may end up in another.

尽管这是一个很方便的 API，但它引入了一些小的问题，就是原来针对于一个指定 session 的消息可能最后跑到了另外一个 session 里。

Consider the following contrived example using the old ``Party`` based API:

下边是使用以前的基于 Party 的 API 的例子：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/LaunchSpaceshipFlow.kt
        :language: kotlin
        :start-after: DOCSTART LaunchSpaceshipFlow
        :end-before: DOCEND LaunchSpaceshipFlow

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/LaunchSpaceshipFlow.java
        :language: java
        :start-after: DOCSTART LaunchSpaceshipFlow
        :end-before: DOCEND LaunchSpaceshipFlow

The intention of the flows is very clear: LaunchSpaceshipFlow asks the president whether a spaceship should be launched.
It is expecting a boolean reply. The president in return first tells the secretary that they need coffee, which is also
communicated with a boolean. Afterwards the president replies to the launcher that they don't want to launch.

这个 Flows 的目的很明确：LaunchSpaceshipFlow 在询问长官是否可以让一个宇宙飞船登陆。它期望着一个 boolean 类型的回复（是或否）。长官的回复首先是告诉秘书他们需要 coffee，这个沟通的内容也是是个 boolean 型的回答。然后长官又回复说他们并不希望飞船降落。

However the above can go horribly wrong when the ``launcher`` happens to be the same party ``getSecretary`` returns. In
this case the boolean meant for the secretary will be received by the launcher!

然而上边的情况在 ``launcher`` 和 ``getsecretary`` 返回的是同一个 party 的话会变得很糟糕。如果真的发生了的话，那么这个 boolean 就意味着 secretary 会被 launcher 接收到。

This indicates that ``Party`` is not a good identifier for the communication sequence, and indeed the ``Party`` based
API may introduce ways for an attacker to fish for information and even trigger unintended control flow like in the
above case.

这就说明了 ``Party`` 对于沟通的顺序来说并不是一个很好的身份标识，并且事实上基于 ``Party`` 的 API 也可能会为黑客引入了一个新的方式来钓鱼用户信息甚至像上边说的那样触发一个并不应该的 flow。

Hence we introduced ``FlowSession``, which identifies the communication sequence. With ``FlowSession`` s the above set
of flows would look like this:

因此我们引入了 ``FlowSession``，用来标识沟通的顺序。通过 ``FlowSession``，上边的一系列 flows 会变成下边这样：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/LaunchSpaceshipFlow.kt
        :language: kotlin
        :start-after: DOCSTART LaunchSpaceshipFlowCorrect
        :end-before: DOCEND LaunchSpaceshipFlowCorrect

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/LaunchSpaceshipFlow.java
        :language: java
        :start-after: DOCSTART LaunchSpaceshipFlowCorrect
        :end-before: DOCEND LaunchSpaceshipFlowCorrect

Note how the president is now explicit about which session it wants to send to.

注意现在长官是如何显式地说明他想发送个哪一个 session。

从旧的基于 Party 的 API到新的 API 的转换
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the old API the first ``send`` or ``receive`` to a ``Party`` was the one kicking off the counter-flow. This is now
explicit in the ``initiateFlow`` function call. To port existing code:

在旧的 API 中，对一个 ``Party`` 的第一个 ``send`` 或者 ``receive`` 会是那个开始 counter-flow 的。这个现在是在调用 ``initiateFlow`` 方法中显式地定义的：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART FlowSession porting
        :end-before: DOCEND FlowSession porting
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART FlowSession porting
        :end-before: DOCEND FlowSession porting
        :dedent: 12

Subflows
--------
Subflows are pieces of reusable flows that may be run by calling ``FlowLogic.subFlow``. There are two broad categories
of subflows, inlined and initiating ones. The main difference lies in the counter-flow's starting method, initiating
ones initiate counter-flows automatically, while inlined ones expect some parent counter-flow to run the inlined
counterpart.

Subflows 是一些可能被重用的 flows 并可以通过调用 ``FlowLogic.subFlow`` 来运行。这里有两大类的 subflows，inlined 和 initiating 的。主要的不同在于 counter-flow 的开始方法，initiating subflows 会自动地开始一个 counter-flows，然而 inlined subflows 期望由一个父的 counter-flow 来运行 inlined counter-part。

Inlined subflows
^^^^^^^^^^^^^^^^
Inlined subflows inherit their calling flow's type when initiating a new session with a counterparty. For example, say
we have flow A calling an inlined subflow B, which in turn initiates a session with a party. The FlowLogic type used to
determine which counter-flow should be kicked off will be A, not B. Note that this means that the other side of this
inlined flow must therefore be implemented explicitly in the kicked off flow as well. This may be done by calling a
matching inlined counter-flow, or by implementing the other side explicitly in the kicked off parent flow.

Inlined subflows 在和 counterparty 初始一个新的 session 的时候继承了调用他们的 flow 的类型。比如假设我们有一个 flow A 调用了一个 inlined subflow B，这就会同一个 party 初始了一个 session。FlowLogic 类型会被用来判断哪一个 counter-flow 应该被开始，应该是 A 不是 B。这就意味着这个 inlined flow 的另一侧必须也要在 kicked off flow 中被显式地实现。这个可能通过调用一个匹配的 inlined counter-flow 或者在 kicked off 父 flow 中通过显式地实现另一侧来实现。

An example of such a flow is ``CollectSignaturesFlow``. It has a counter-flow ``SignTransactionFlow`` that isn't
annotated with ``InitiatedBy``. This is because both of these flows are inlined; the kick-off relationship will be
defined by the parent flows calling ``CollectSignaturesFlow`` and ``SignTransactionFlow``.

这样的 flow 的一个例子是 ``CollectSignaturesFlow``。它有一个 counter-flow ``SignTransactionFlow``，这个并没有 ``InitatedBy`` 的注解。这是因为这两个 flow 都是 inlined；这个 kick-off 关系会被父 flows 通过调用 ``CollectSignaturesFlow`` 和 ``SignTransactionFlow`` 来定义的。

In the code inlined subflows appear as regular ``FlowLogic`` instances, `without` either of the ``@InitiatingFlow`` or
``@InitiatedBy`` annotation.

在代码中，inlined subflows 会作为常规的一个 ``FlowLogic`` 的实例，并且没有 ```@InitiatingFlow`` 和 ``@InitiatedBy`` 的注解。

.. note:: Inlined flows aren't versioned; they inherit their parent flow's version.

.. note:: Inlined flows 并没有自己的版本，他们会继承他们父 flows 的版本。

Initiating subflows
^^^^^^^^^^^^^^^^^^^
Initiating subflows are ones annotated with the ``@InitiatingFlow`` annotation. When such a flow initiates a session its
type will be used to determine which ``@InitiatedBy`` flow to kick off on the counterparty.

Initiating subflows 是这些带有 ``@InitiatingFlow`` 注解的 subflows。当这样的 flow 初始了一个 session 的时候，它的类型会被用来确定哪一个 ``@InitiatedBy`` 的flow 会在对方那里被开始。

An example is the ``@InitiatingFlow InitiatorFlow``/``@InitiatedBy ResponderFlow`` flow pair in the ``FlowCookbook``.

一个例子就是 ``FlowCookbook`` 中的 ``@InitiatingFlow InitiatorFlow``/``@InitiatedBy ResponderFlow`` flow 对。

.. note:: Initiating flows are versioned separately from their parents.

.. note:: Initiating flows 有自己的版本，跟它的父 flows 是分开的。

.. note:: The only exception to this rule is ``FinalityFlow`` which is annotated with ``@InitiatingFlow`` but is an inlined flow. This flow
   was previously initiating and the annotation exists to maintain backwards compatibility with old code.

.. note:: 这个规则的唯一一个例外是 ``FinalityFlow``，它是带有 ``@InitiatingFlow`` 注解的，但是它是一个 inlined flow。这个 flow 是之前被初始化的，并且这个注解的存在是为了维护跟旧代码的兼容性。

核心 initiating subflows
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Corda-provided initiating subflows are a little different to standard ones as they are versioned together with the
platform, and their initiated counter-flows are registered explicitly, so there is no need for the ``InitiatedBy``
annotation.

Corda 提供的 initiating subflows 针对于标准的 subflows 有一点点不同，就是他们是跟着平台的版本在一起的，并且他们初始的 counter-flows 是被显式地注册的，所以就不需要有 ``InitiatedBy`` 这个注解了。

Flows 类库
^^^^^^^^^^^^^
Corda installs four initiating subflow pairs on each node by default:

* ``NotaryChangeFlow``/``NotaryChangeHandler``, which should be used to change a state's notary
* ``ContractUpgradeFlow.Initiate``/``ContractUpgradeHandler``, which should be used to change a state's contract
* ``SwapIdentitiesFlow``/``SwapIdentitiesHandler``, which is used to exchange confidential identities with a
  counterparty

Corda 在每个节点中会默认安装 4 个 initiating subflow：

* ``NotaryChangeFlow``/``NotaryChangeHandler``，用来变更一个 state 的 notary
* ``ContractUpgradeFlow.Initiate``/``ContractUpgradeHandler``, 用来变更 state 的 contract
* ``SwapIdentitiesFlow``/``SwapIdentitiesHandler``, 用来交换一个 counterparty 的 confidential identities

.. warning:: ``SwapIdentitiesFlow``/``SwapIdentitiesHandler`` are only installed if the ``confidential-identities`` module 
   is included. The ``confidential-identities`` module  is still not stabilised, so the
   ``SwapIdentitiesFlow``/``SwapIdentitiesHandler`` API may change in future releases. See :doc:`corda-api`.

.. warning:: ``SwapIdentitiesFlow``/``SwapIdentitiesHandler`` 只会在包含了 ``confidential-identities`` 模块的时候会被安装。``confidential-identities`` 模块现在还不是稳定版本，所以 ``SwapIdentitiesFlow``/``SwapIdentitiesHandler`` API 模块在之后的 release 中可能会有变更。查看  :doc:`corda-api`。

Corda also provides a number of built-in inlined subflows that should be used for handling common tasks. The most
important are:

* ``FinalityFlow`` which is used to notarise, record locally and then broadcast a signed transaction to its participants
  and any extra parties.
* ``ReceiveFinalityFlow`` to receive these notarised transactions from the ``FinalityFlow`` sender and record locally.
* ``CollectSignaturesFlow`` , which should be used to collect a transaction's required signatures
* ``SendTransactionFlow`` , which should be used to send a signed transaction if it needed to be resolved on
  the other side.
* ``ReceiveTransactionFlow``, which should be used receive a signed transaction

Corda 提供了很多内置的 flows 用来处理常见的任务。比较重要的有：

* ``FinalityFlow``，用来公正（notarise）和记录 transaction 并且将一个签过名的 transaction 广播给它的所有参与者以及任何额外的 parties
* ``ReceiveFinalityFlow``，用来接收来自于 ``FinalityFlow`` 的发送方的已经被公证过的 transaction 并且存储到本地
* ``CollectSignaturesFlow``，用来搜集一个 transaction 所要求的签名
* ``SendTransactionFlow``，用来发送一个签了名的 transaction，如果这个 transaction 需要自另一方去处理的话
* ``ReceiveTransactionFlow``，用来接收一个已经被签名了的 transaction

Let's look at some of these flows in more detail.

我们来看这些常见的 subflow 例子。

FinalityFlow
~~~~~~~~~~~~
``FinalityFlow`` allows us to notarise the transaction and get it recorded in the vault of the participants of all
the transaction's states:

``FinalityFlow`` 允许我们来公证一个 transaction 并且让所有参与者都可以将 transaction 的 states 记录到账本中：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 09
        :end-before: DOCEND 09
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 09
        :end-before: DOCEND 09
        :dedent: 12

We can also choose to send the transaction to additional parties who aren't one of the state's participants:

我们也可以将 transaction 发送给额外的 parties 即使他们不是 state 的参与者：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 10
        :end-before: DOCEND 10
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 10
        :end-before: DOCEND 10
        :dedent: 12

Only one party has to call ``FinalityFlow`` for a given transaction to be recorded by all participants. It **must not**
be called by every participant. Instead, every other particpant **must** call ``ReceiveFinalityFlow`` in their responder
flow to receive the transaction:

对于一个 transaction 仅仅需要一方来调用 ``FinalityFlow`` 来让所有的参与者记录它。这 **不需要** 每一方分别自己去调用。每个其他的参与方 **必须** 在他们的 responder flow 中调用 ``ReceiveFinalityFlow`` 来接收交易：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART ReceiveFinalityFlow
        :end-before: DOCEND ReceiveFinalityFlow
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART ReceiveFinalityFlow
        :end-before: DOCEND ReceiveFinalityFlow
        :dedent: 12

``idOfTxWeSigned`` is an optional parameter used to confirm that we got the right transaction. It comes from using ``SignTransactionFlow``
which is described below.

``idOfTxWeSigned`` 是一个可选的参数可以用来确认我们得到了一个正确的交易。它是使用从下边描述的 ``SignTransactionFlow`` 得到的。

**错误处理行为**

Once a transaction has been notarised and its input states consumed by the flow initiator (eg. sender), should the participant(s) receiving the
transaction fail to verify it, or the receiving flow (the finality handler) fails due to some other error, we then have a scenario where not
all parties have the correct up to date view of the ledger (a condition where eventual consistency between participants takes longer than is
normally the case under Corda's `eventual consistency model <https://en.wikipedia.org/wiki/Eventual_consistency>`_). To recover from this scenario,
the receiver's finality handler will automatically be sent to the :doc:`node-flow-hospital` where it's suspended and retried from its last checkpoint
upon node restart, or according to other conditional retry rules explained in :ref:`flow hospital runtime behaviour <flow-hospital-runtime>`.
This gives the node operator the opportunity to recover from the error. Until the issue is resolved the node will continue to retry the flow
on each startup. Upon successful completion by the receiver's finality flow, the ledger will become fully consistent once again.

当一笔交易被证明并且 flow initiator（比如 sender）也消费了它的 states 之后，如果参与方接收到了交易验证没通过，或者由于一些其他的错误，接收的 flow（finality 处理）失败了的话，那么就会出现不是所有的参与方都有一个正确的最新的账本的视图（在 Corda 的 `最终一致性模型 <https://en.wikipedia.org/wiki/Eventual_consistency>`_ 下，在这种条件下载参与方之间的最终一致性要比常规的花费更长的时间）。为了能够从这个场景中恢复，接收方的 finality handler 会被自动地发送到 :doc:`node-flow-hospital`，在那里它会被挂起并且在节点重启或者根据在 :ref:`flow hospital runtime behaviour <flow-hospital-runtime>` 中解释的其他条件下的重试规则会尝试在它的最后一个 checkpoint 那里重试。这就给了节点的维护者机会来从错误中恢复。节点会在每次重启的时候不断的重试这个 flow 直到问题被解决。一旦接收方的 finality flow 成功结束了，那么账本将会变得再次完全一致。

.. warning:: It's possible to forcibly terminate the erroring finality handler using the ``killFlow`` RPC but at the risk of an inconsistent view of the ledger.

.. warning:: 使用 ``killFlow`` RPC 来强制结束错误的 finality handler 是可以的，但是会造成账本的不一致的视图。

.. note:: A future release will allow retrying hospitalised flows without restarting the node, i.e. via RPC.

.. note:: 之后的 release 会允许不需要重启节点就能够重试有问题的 flows，比如通过 RPC。

CollectSignaturesFlow/SignTransactionFlow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The list of parties who need to sign a transaction is dictated by the transaction's commands. Once we've signed a
transaction ourselves, we can automatically gather the signatures of the other required signers using
``CollectSignaturesFlow``:

都要由哪些 parties 来为 transaction 提供签名是在 transaction 的 commands 中定义的。一旦我们为 transaction 提供了自己的签名，我们可以使用 ``CollectSignaturesFlow`` 来搜集其他必须提供签名的 parties 的签名：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 15
        :end-before: DOCEND 15
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 15
        :end-before: DOCEND 15
        :dedent: 12

Each required signer will need to respond by invoking its own ``SignTransactionFlow`` subclass to check the
transaction (by implementing the ``checkTransaction`` method) and provide their signature if they are satisfied:

每一个要求提供签名的 party 需要调用他们自己的 ``SignTransactionFlow`` 子类来检查 transaction（通过实现 ``checkTransaction`` 方法） 并且在满足要求后提供自己的签名：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 16
        :end-before: DOCEND 16
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 16
        :end-before: DOCEND 16
        :dedent: 12

Types of things to check include:

    * Ensuring that the transaction received is the expected type, i.e. has the expected type of inputs and outputs
    * Checking that the properties of the outputs are expected, this is in the absence of integrating reference
      data sources to facilitate this
    * Checking that the transaction is not incorrectly spending (perhaps maliciously) asset states, as potentially
      the transaction creator has access to some of signer's state references

需要检查的事情包括：

    * 确保接收到的 transaction 是期待的类型，比如是否具有期待类型的 inputs 和 outputs
    * 检查 outputs 的属性是不是正确，这是因为没有继承引用的数据源来协调
    * 检查交易没有错误地消费（可能是恶意的） asset states，因为很有可能交易的创建者能够访问一些签名者的 state references

SendTransactionFlow/ReceiveTransactionFlow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Verifying a transaction received from a counterparty also requires verification of every transaction in its
dependency chain. This means the receiving party needs to be able to ask the sender all the details of the chain.
The sender will use ``SendTransactionFlow`` for sending the transaction and then for processing all subsequent
transaction data vending requests as the receiver walks the dependency chain using ``ReceiveTransactionFlow``:

验证一个从 counterparty 发送来的 transaction 也需要验证 transaction 依赖链（dependency chain）上的每一个 transaction。这就意味着接收方需要能够向发送方要求这个依赖链的所有详细内容。发送方就可以使用 ``SendTransactionFlow`` 来发送 transaction，接收方就可以通过使用 ``ReceiveTransactionFlow`` 来查看所有依赖链的内容：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 12
        :end-before: DOCEND 12
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 12
        :end-before: DOCEND 12
        :dedent: 12

We can receive the transaction using ``ReceiveTransactionFlow``, which will automatically download all the
dependencies and verify the transaction:

我们可以使用 ``ReceiveTransactionFlow`` 来接收 transaction，这会自动地下载所有的依赖并且确认 transaction：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 13
        :end-before: DOCEND 13
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 13
        :end-before: DOCEND 13
        :dedent: 12

We can also send and receive a ``StateAndRef`` dependency chain and automatically resolve its dependencies:

我们也可以发送和接收一个 ``StateAndRef`` 依赖链并且自动解决了它的依赖：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 14
        :end-before: DOCEND 14
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 14
        :end-before: DOCEND 14
        :dedent: 12

为什么要用 inlined subflows?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Inlined subflows provide a way to share commonly used flow code `while forcing users to create a parent flow`. Take for
example ``CollectSignaturesFlow``. Say we made it an initiating flow that automatically kicks off
``SignTransactionFlow`` that signs the transaction. This would mean malicious nodes can just send any old transaction to
us using ``CollectSignaturesFlow`` and we would automatically sign it!

Inlined subflows 提供了一种分享常用的 flow code 的方式，这种方式要求 `用户必须要创建一个父的 flow`。比如 ``CollectSignaturesFlow`` 这个例子。假设我们创建了一个 initiating flow 来自动开始一个 ``SignTransactionFlow`` 来为 transaction 提供签名。这意味着恶意的节点能够通过使用 ``CollectSignaturesFlow`` 只需向我们发送任何一个旧的 transaction，然后我们就会自动地为其提供签名！

By making this pair of flows inlined we provide control to the user over whether to sign the transaction or not by
forcing them to nest it in their own parent flows.

为了使这对 flows 在同一个等级范围，我们通过强制用户将这个 flow 嵌套到他们自己的父 flows 中的方式来允许用户决定他们是否要为这个 transaction 提供签名。

In general if you're writing a subflow the decision of whether you should make it initiating should depend on whether
the counter-flow needs broader context to achieve its goal.

总体上来说，如果你在写一个 flow 的话，你是否应该将其定义为一个 initiating flow 应该基于 counter-flow 是否需要更广泛的上下文来达到它的目标。

FlowException
-------------
Suppose a node throws an exception while running a flow. Any counterparty flows waiting for a message from the node
(i.e. as part of a call to ``receive`` or ``sendAndReceive``) will be notified that the flow has unexpectedly
ended and will themselves end. However, the exception thrown will not be propagated back to the counterparties.

假设一个节点在运行 flow 的时候抛出了一个异常。其他任何在等待该节点返回信息的节点（比如作为调用 ``receive`` 或者 ``sendAndReceive`` 的一部分）会被提示该 flow 异常终止并且自我结束。然而抛出的异常不会被发回到 counterparties。

If you wish to notify any waiting counterparties of the cause of the exception, you can do so by throwing a
``FlowException``:

如果你想告知任何等待的 counterparties 异常的原因的话，你可以通过抛出一个 ``FlowException`` 来实现：

.. container:: codeset

    .. literalinclude:: ../../core/src/main/kotlin/net/corda/core/flows/FlowException.kt
        :language: kotlin
        :start-after: DOCSTART 1
        :end-before: DOCEND 1

The flow framework will automatically propagate the ``FlowException`` back to the waiting counterparties.

Flow framework 会自动地将这个 ``FlowException`` 返回给等待的 counterparties。

There are many scenarios in which throwing a ``FlowException`` would be appropriate:

* A transaction doesn't ``verify()``
* A transaction's signatures are invalid
* The transaction does not match the parameters of the deal as discussed
* You are reneging on a deal

以下的情况是适合返回一个 ``FlowException`` 的：

* 没有 ``verify()`` 方法的 transaction
* 一个 transaction 的签名是无效的
* Transaction 跟讨论的交易参数不匹配
* 交易违规

ProgressTracker
---------------
We can give our flow a progress tracker. This allows us to see the flow's progress visually in our node's CRaSH shell.

我们可以给我们的 flow 一个进度跟踪器。这个使我们能够在我们节点的 CRaSH shell 中看到 flow 的进展。

To provide a progress tracker, we have to override ``FlowLogic.progressTracker`` in our flow:

为了提供一个 progress tracker，我们需要在我们的 flow 中重写 ``FlowLogic.progressTracker`` ：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 17
        :end-before: DOCEND 17
        :dedent: 4

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 17
        :end-before: DOCEND 17
        :dedent: 8

We then update the progress tracker's current step as we progress through the flow as follows:

然后我们就可以按照下边的方式来根据 flow 的进展来更新 progress tracker 的当前步骤：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 18
        :end-before: DOCEND 18
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 18
        :end-before: DOCEND 18
        :dedent: 12

HTTP and 数据库调用
-----------------------
HTTP, database and other calls to external resources are allowed in flows. However, their support is currently limited:

* The call must be executed in a BLOCKING way. Flows don't currently support suspending to await the response to a call to an external resource

  * For this reason, the call should be provided with a timeout to prevent the flow from suspending forever. If the timeout elapses, this should be treated as a soft failure and handled by the flow's business logic
  
* The call must be idempotent. If the flow fails and has to restart from a checkpoint, the call will also be replayed

HTTP、数据库和其他对于外部资源的调用在 flow 中是允许的。然而，对于这些的支持现在是有限的：

* 这个调用必须要以一种 阻塞 的方式来执行。Flows 当前还不支持挂起并等待对于外部资源调用的反馈

  * 因此，这个调用应该提供一个超时来避免 flow 会被永远挂起。如果达到超时的时间，这个应该被触发一个 soft failure 并被 flow 的业务逻辑来处理

* 这个调用必须是密等的。如果这个 flow 失败了并且不得不从某个 checkpoint 重启的话，那么这次调用也会被重新执行

并发，锁和等待
--------------------------------
Corda is designed to:

* run many flows in parallel
* persist flows to storage and resurrect those flows much later
* (in the future) migrate flows between JVMs

Corda 是被设计用来：

* 同时运行多个 flows
* 可能会将 flows 持久化到 storage 并在稍后恢复这些 flows
* （在将来）在 JVMs 之间迁移 flows

Because of this, care must be taken when performing locking or waiting operations.

因此，在执行锁或者等待的操作的时候必须要小心。

锁
^^^^^^^
Flows should avoid using locks or interacting with objects that are shared between flows (except for ``ServiceHub`` and other 
carefully crafted services such as Oracles.  See :doc:`oracles`). Locks will significantly reduce the scalability of the 
node, and can cause the node to deadlock if they remain locked across flow context switch boundaries (such as when sending 
and receiving from peers, as discussed above, or sleeping, as discussed below).

Flows 应该避免使用锁，甚至通常也不应该尝试同 flows 之间共享的对象来进行交互（除了 ``ServiceHub`` 和其他仔细地设计过的服务，查看 :doc:`oracles`）。锁会很大程度上减弱节点的可扩展性，并且如果他们在 flow 上下文的转换间（比如像上边讨论的那样当发送和从 peer 那里接收，或者想下边讨论的休眠）依旧保持被锁的状态的话，还会造成节点的死锁。

等待
^^^^^^^
A flow can wait until a specific transaction has been received and verified by the node using `FlowLogic.waitForLedgerCommit`. 
Outside of this, scheduling an activity to occur at some future time should be achieved using ``SchedulableState``.

一个 flow 能够等待直到一个特定的交易被收到并且通过了由节点使用 `FlowLogic.waitForLedgerCommit` 进行的验证。除此之外，在将来的某个时间预约一个动作会发生也可以通过使用 ``SchedulableState`` 来实现。

However, if there is a need for brief pauses in flows, you have the option of using ``FlowLogic.sleep`` in place of where you
might have used ``Thread.sleep``. Flows should expressly not use ``Thread.sleep``, since this will prevent the node from 
processing other flows in the meantime, significantly impairing the performance of the node.

然而，如果需要在 flows 中停止一段时间，你可以在你已经使用 ``Thread.sleep`` 的地方使用 ``FlowLogic.sleep``。Flows 很明显应该不使用 ``Thread.sleep``，因为这会组织节点在同一时间处理其他的 flows，这会严重地影响节点的效率。

Even ``FlowLogic.sleep`` should not be used to create long running flows or as a substitute to using the ``SchedulableState``
scheduler, since the Corda ethos is for short-lived flows (long-lived flows make upgrading nodes or CorDapps much more 
complicated).

甚至 ``FlowLogic.sleep`` 也不应该被用来创建一个长时间运行的 flows 或者作为使用 ``SchedulableState`` scheduler 的后续操作，因为 Corda 的精神是为了短生命的 flows（长时间运行的 flows 会将升级节点或 CorDapps 变得更复杂）。

For example, the ``finance`` package currently uses ``FlowLogic.sleep`` to make several attempts at coin selection when 
many states are soft locked, to wait for states to become unlocked:

比如，``finance`` 包当前使用 ``FlowLogic.sleep``来进行不同的尝试来进行 coin 的选择，当有多个 states 被 soft locked，来等待其他的新的 states 变成了未被锁的状态。

    .. literalinclude:: ../../finance/workflows/src/main/kotlin/net/corda/finance/workflows/asset/selection/AbstractCashSelection.kt
        :language: kotlin
        :start-after: DOCSTART CASHSELECT 1
        :end-before: DOCEND CASHSELECT 1
        :dedent: 8
