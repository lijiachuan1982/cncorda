.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

升级 CorDapp
============================

.. note:: This document only concerns the upgrading of CorDapps and not the Corda platform itself (wire format, node
   database schemas, etc.).

.. note:: 这个文档仅仅关注 CorDapps 的升级而非 Corda 平台自身的升级（wire 格式、节点数据库 schema 等等）。

.. contents::

CorDapp 版本
------------------
.. UPDATE - This is no longer accurate! Needs to talk about the different types of artifacts ( kernel, workflows) each versioned independently

.. UPDATE - 这个对于现在不在准确了！需要针对每个版本单独地说明不同类型的 artifacts（kernel、workflows）

The Corda platform does not mandate a version number on a per-CorDapp basis. Different elements of a CorDapp are
allowed to evolve separately. Sometimes, however, a change to one element will require changes to other elements. For
example, changing a shared data structure may require flow changes that are not backwards-compatible.

Corda 平台没有要求每个 CorDapp 要保持同样一个版本。CorDapp 的不同元素可以分别去升级。有些时候，当对一个元素进行改动之后，可能会需要其他元素也改动。比如，修改了一个共享的数据结构可能会需要 flow 的改动因为这个是不向后兼容的。

Flow 版本
---------------
Any flow that initiates other flows must be annotated with the ``@InitiatingFlow`` annotation, which is defined as:

任何初始化其他 flows 的 flow 必须要使用 @InitiatingFlow 注解，像下边这样定义：

.. sourcecode:: kotlin

   annotation class InitiatingFlow(val version: Int = 1)

The ``version`` property, which defaults to 1, specifies the flow's version. This integer value should be incremented
whenever there is a release of a flow which has changes that are not backwards-compatible. A non-backwards compatible
change is one that changes the interface of the flow.

``version`` 属性默认值为 1，定义了 flow 的版本。当flow 有任何一个新的 release 的时候并且这个 release 包含的变动是非向下兼容的，这个数值应该增加。一个非向下兼容的改动是一个改变了 flow 的接口的变动。

定义一个 flow 的接口
~~~~~~~~~~~~~~~~~~~~~~~~~~~
The flow interface is defined by the sequence of ``send`` and ``receive`` calls between an ``InitiatingFlow`` and an
``InitiatedBy`` flow, including the types of the data sent and received. We can picture a flow's interface as follows:

Flow 的接口是通过在 ``InitiatingFlow`` 和 ``InitiatedBy`` flow 之间有序的 ``send`` 和 ``receive`` 调用来定义的，包括发送和接受的数据的类型。我们可以将 flow 的接口如下图这样表示：

.. image:: resources/flow-interface.png
   :scale: 50%
   :align: center

In the diagram above, the ``InitiatingFlow``:

* Sends an ``Int``
* Receives a ``String``
* Sends a ``String``
* Receives a ``CustomType``

在上边的图中，``InitiatingFlow``：

* 发送了一个 ``Int``
* 接收了一个 ``String``
* 发送了一个 ``String``
* 接收了一个 ``CustomType``

The ``InitiatedBy`` flow does the opposite:

* Receives an ``Int``
* Sends a ``String``
* Receives a ``String``
* Sends a ``CustomType``

``InitiatedBy`` flow 恰恰相反：
* 接收了一个 ``Int``
* 发送了一个 ``String``
* 接收了一个 ``String``
* 发送了一个 ``CustomType``

As long as both the ``InitiatingFlow`` and the ``InitiatedBy`` flows conform to the sequence of actions, the flows can
be implemented in any way you see fit (including adding proprietary business logic that is not shared with other
parties).

只要 ``IntiatingFlow`` 和 ``InitiatedBy`` flows 遵循这个有序的一系列的动作，那么 flows 就可以按照任何你觉得合适的方式来实现（包括添加不共享给其他节点的业务逻辑）

非向下兼容的 flow 改动
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
A flow can become backwards-incompatible in two main ways:

* The sequence of ``send`` and ``receive`` calls changes:

  * A ``send`` or ``receive`` is added or removed from either the ``InitiatingFlow`` or ``InitiatedBy`` flow
  * The sequence of ``send`` and ``receive`` calls changes

* The types of the ``send`` and ``receive`` calls changes

Flow 可以有两种主要的方式会变为非向下兼容的：

* ``send`` 和 ``receive`` 调用的顺序变化：

  * 一个 ``send`` 或者 ``receive`` 从 ``InitiatingFlow`` 或者 ``InitiatedBy`` flow 中被添加或者删除了
  * ``send`` 和 ``receive`` 调用的顺序变了

* ``send`` 和 ``receive`` 调用的类型变了

运行不兼容版本的 flows 的结果
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Pairs of ``InitiatingFlow`` flows and ``InitiatedBy`` flows that have incompatible interfaces are likely to exhibit the
following behaviour:

* The flows hang indefinitely and never terminate, usually because a flow expects a response which is never sent from
  the other side
* One of the flow ends with an exception: "Expected Type X but Received Type Y", because the ``send`` or ``receive``
  types are incorrect
* One of the flows ends with an exception: "Counterparty flow terminated early on the other side", because one flow
  sends some data to another flow, but the latter flow has already ended

带有非兼容接口的 ``InitiatingFlow`` 和 ``InitiatedBy`` flows 可能会出现下边的行为：

* flows 会没有明确原因地停住了并且永远也不会终止，通常是因为一个 flow 在等待这着一个回复，但是这个回复永远不会从另一方返回来
* 其中的一个 flow 会带有异常地结束：“Expected Type X but Received Type Y”，因为 ``send`` 或者 ``receive`` 类型不正确
* 其中的一个 flow 会带有异常地结束：“Counterparty flow terminated early on the other side”，因为一个 flow 向另外一个 flow 发送了一些数据，但是后边这个 flow 已经结束了

确保 flow 的向后兼容性
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The ``InitiatingFlow`` version number is included in the flow session handshake and exposed to both parties via the
``FlowLogic.getFlowContext`` method. This method takes a ``Party`` and returns a ``FlowContext`` object which describes
the flow running on the other side. In particular, it has a ``flowVersion`` property which can be used to
programmatically evolve flows across versions. For example:

``InitiatingFlow`` 的版本号会被包含在 flow session handshake 中并且会通过 ``FlowLogic.getFlowContext`` 方法暴露给双方。这个方法有一个 ``Party`` 并且会返回一个 ``FlowContext`` 对象，这个对象描述了在另一侧运行的 flow。它有一个 ``flowVersion`` 版本，可以用来在不同版本中动态地升级 flows。比如：

.. container:: codeset

    .. sourcecode:: kotlin

        @Suspendable
        override fun call() {
            val otherFlowVersion = otherSession.getCounterpartyFlowInfo().flowVersion
            val receivedString = if (otherFlowVersion == 1) {
                otherSession.receive<Int>().unwrap { it.toString() }
            } else {
                otherSession.receive<String>().unwrap { it }
            }
        }

    .. sourcecode:: java

        @Suspendable
        @Override public Void call() throws FlowException {
            int otherFlowVersion = otherSession.getCounterpartyFlowInfo().getFlowVersion();
            String receivedString;

            if (otherFlowVersion == 1) {
                receivedString = otherSession.receive(Integer.class).unwrap(integer -> {
                    return integer.toString();
                });
            } else {
                receivedString = otherSession.receive(String.class).unwrap(string -> {
                    return string;
                });
            }

            return null;
        }

This code shows a flow that in its first version expected to receive an Int, but in subsequent versions was modified to
expect a String. This flow is still able to communicate with parties that are running the older CorDapp containing
the older flow.

上边的代码演示了当 flow 的第一个版本期望收到一个 Int，但是后续的版本变成了期望收到一个 String。这个 flow 在跟其他仍然运行着包含旧的 flow 的旧的 CorDapp 之间还是能够进行沟通的。

处理对于 inlined subflows 的接口变更
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Here is an example of an in-lined subflow:

下边是一个 in-lined subflow 的例子：

.. container:: codeset

    .. sourcecode:: kotlin

        @StartableByRPC
        @InitiatingFlow
        class FlowA(val recipient: Party) : FlowLogic<Unit>() {
            @Suspendable
            override fun call() {
                subFlow(FlowB(recipient))
            }
        }

        @InitiatedBy(FlowA::class)
        class FlowC(val otherSession: FlowSession) : FlowLogic() {
            // Omitted.
        }

        // Note: No annotations. This is used as an inlined subflow.
        class FlowB(val recipient: Party) : FlowLogic<Unit>() {
            @Suspendable
            override fun call() {
                val message = "I'm an inlined subflow, so I inherit the @InitiatingFlow's session ID and type."
                initiateFlow(recipient).send(message)
            }
        }

    .. sourcecode:: java

        @StartableByRPC
        @InitiatingFlow
        class FlowA extends FlowLogic<Void> {
            private final Party recipient;

            public FlowA(Party recipient) {
                this.recipient = recipient;
            }

            @Suspendable
            @Override public Void call() throws FlowException {
                subFlow(new FlowB(recipient));

                return null;
            }
        }

        @InitiatedBy(FlowA.class)
        class FlowC extends FlowLogic<Void> {
            // Omitted.
        }

        // Note: No annotations. This is used as an inlined subflow.
        class FlowB extends FlowLogic<Void> {
            private final Party recipient;

            public FlowB(Party recipient) {
                this.recipient = recipient;
            }

            @Suspendable
            @Override public Void call() {
                String message = "I'm an inlined subflow, so I inherit the @InitiatingFlow's session ID and type.";
                initiateFlow(recipient).send(message);

                return null;
            }
        }

Inlined subflows are treated as being the flow that invoked them when initiating a new flow session with a counterparty.
Suppose flow ``A`` calls inlined subflow B, which, in turn, initiates a session with a counterparty. The ``FlowLogic``
type used by the counterparty to determine which counter-flow to invoke is determined by ``A``, and not by ``B``. This
means that the response logic for the inlined flow must be implemented explicitly in the ``InitiatedBy`` flow. This can
be done either by calling a matching inlined counter-flow, or by implementing the other side explicitly in the
initiated parent flow. Inlined subflows also inherit the session IDs of their parent flow.

In-lined subflows 是当跟对方初始一个新的 flow session 的时候被调用的 flows。假设 flow A 调用 in-lined subFlow B，B 初始了一个跟对方的会话（session）。对方使用的 ``FlowLogic`` 类型决定应该调用哪个对应的 flow 应该是由 ``A`` 决定的，而不是 ``B``。这意味着 in-lined flow 的 response logic 必须要在 ``InitiateBy`` flow 里被显式地实现。这个可以通过调用一个匹配的 in-lined counter-flow，或者在对方的被初始的父的 flow 中显式地实现。In-lined subflows 也会从他们的父 flow 中继承 session IDs。

As such, an interface change to an inlined subflow must be considered a change to the parent flow interfaces.

因此，一个 in-lined subflow 的一个借口的改动必须要考虑对父 flow 接口也要有一个改动。

An example of an inlined subflow is ``CollectSignaturesFlow``. It has a response flow called ``SignTransactionFlow``
that isn’t annotated with ``InitiatedBy``. This is because both of these flows are inlined. How these flows speak to
one another is defined by the parent flows that call ``CollectSignaturesFlow`` and ``SignTransactionFlow``.

一个 in-lined subflow 的例子是 ``CollectSignaturesFlow``。他有一个没有 ``InitiateBy`` 注解的 response 的叫 ``SignTransactionFlow`` 的 flow。这是因为这两个 flows 都是 in-lined。这两个 flows 是如何彼此交流的是通过调用成为 ``CollectSignaturesFlow`` 和 ``SignTransactionFlow`` 他们的父 flows 来定义的。

In code, inlined subflows appear as regular ``FlowLogic`` instances without either an ``InitiatingFlow`` or an
``InitiatedBy`` annotation.

在代码中，in-lined subflows 看起来就是一个常规的 ``FlowLogic`` 的实例，但是没有 ``InitiatingFlow`` 或者 ``InitiatedBy`` 注解。

Inlined flows are not versioned, as they inherit the version of their parent ``InitiatingFlow`` or ``InitiatedBy``
flow.

In-lined subflows 是没有版本的，因为他们的版本是继承于他们的父 ``InitiatingFlow`` 和 ``InitiatedBy`` flow。

Flows which are not an ``InitiatingFlow`` or ``InitiatedBy`` flow, or inlined subflows that are not called from an
``InitiatingFlow`` or ``InitiatedBy`` flow, can be updated without consideration of backwards-compatibility. Flows of
this type include utility flows for querying the vault and flows for reaching out to external systems.

不是 ``InitiatingFlow`` 或者 ``InitiatedBy`` flow，也不是由一个 ``InitiatingFlow`` 或者 ``InitiatedBy`` flow 调用的 in-lined subflows ，更新的时候可以不考虑向下兼容的问题。这种类型的 flows 包括用来查询 vault 的 utility flows，或者对外部系统进行查询的 flows。

进行 flow 升级
~~~~~~~~~~~~~~~~~~~~~~~~

1. Update the flow and test the changes. Increment the flow version number in the ``InitiatingFlow`` annotation
2. Ensure that all versions of the existing flow have finished running and there are no pending ``SchedulableFlows`` on
   any of the nodes on the business network. This can be done by :ref:`draining_the_node`
3. Shut down the node
4. Replace the existing CorDapp JAR with the CorDapp JAR containing the new flow
5. Start the node

1. 更新 flow 并测试这些变化。在 ``InitiatingFlow`` 注解中递增 flow 版本号
2. 确保所有版本的已经存在的 flow 都已经运行完毕，在这个业务网络中的任何节点上没有未结束的 ``SchedulableFlows``。这个可以通过 :ref:`draining_the_node` 来实现。
3. 关闭节点
4. 将已经存在的 CorDapp JAR 替换为包含新的 flow 的 CorDapp JAR
5. 启动节点

If you shut down all nodes and upgrade them all at the same time, any incompatible change can be made.

如果你关闭了所有的节点并在同一时间升级他们的话，任何的非兼容的改动都可以。

In situations where some nodes may still be using previous versions of a flow and thus new versions of your flow may
talk to old versions, the updated flows need to be backwards-compatible. This will be the case for almost any real
deployment in which you cannot easily coordinate the roll-out of new code across the network.

当一些节点还需要使用一个 flow 之前的版本的时候，那么你的新版本的 flow 就需要跟旧的版本的 flow 进行对话，这个升级后的 flow 就需要有向后兼容性。这可能是最有可能的真是的部署场景，你可能很难在整个网络中发布一个新的代码。

.. _draining_the_node:

Flow 排空
~~~~~~~~~~~~~~~~~

A flow *checkpoint* is a serialised snapshot of the flow's stack frames and any objects reachable from the stack.
Checkpoints are saved to the database automatically when a flow suspends or resumes, which typically happens when
sending or receiving messages. A flow may be replayed from the last checkpoint if the node restarts. Automatic
checkpointing is an unusual feature of Corda and significantly helps developers write reliable code that can survive
node restarts and crashes. It also assists with scaling up, as flows that are waiting for a response can be flushed
from memory.

一个 flow *检查点*是一个序列化的 flow 的堆栈结构（stack frames） 和 任何可以从堆栈中拿到的对象的 snapshot。检查点会在一个 flow 挂起后者恢复的时候被自动存到数据中，这个通常会在发送或者接收消息的时候发生。当节点重启的时候，一个 flow 可能会从最后一个检查点开始重新运行。自动的创建检查点是 Corda 提供的 一个非常规的功能，这会很大地帮助开发者编写可靠的代码来确保当节点重启或者 crash 之后节点还能够继续正常运行。这个也帮助了向上扩展（scaling up），因为当 flows 在等待一个 response 的时候，他们会被从内存中清理掉。

However, this means that restoring an old checkpoint to a new version of a flow may cause resume failures. For example
if you remove a local variable from a method that previously had one, then the flow engine won't be able to figure out
where to put the stored value of the variable.

然而，这也意味着将 flow 从一个旧版本恢复到一个新的版本的时候，可能会造成重启失败。比如如果你从一个方法中删除了一个本地变量，这个变量在以前的版本中是有的，那么 flow 引擎是无法找出之前存储的变量值应该放在哪里的。

For this reason, in currently released versions of Corda you must *drain the node* before doing an app upgrade that
changes ``@Suspendable`` code. A drain blocks new flows from starting but allows existing flows to finish. Thus once
a drain is complete there should be no outstanding checkpoints or running flows. Upgrading the app will then succeed.

因此，在当前版本的 Corda 中，在做一个改变了 ```@Suspendable`` 代码更新的一个应用升级之前，你必须要 *排空节点*。排空操作会组织开始一个的 flows，但是仍旧允许完成已经存在的 flows。因此当一次排空操作完成的时候，就不应该有任何特别的检查点或者是正在运行的 flows 了。这样升级应用才会成功。

A node can be drained or undrained via RPC using the ``setFlowsDrainingModeEnabled`` method, and via the shell using
the standard ``run`` command to invoke the RPC. See :doc:`shell` to learn more.

一个节点可以使用 ``setFlowsDrainingModeEnabled`` 方法来决定要排空还是不要排空，这个可以通过 shell ，使用标准的 ``run`` 命令来调用 RPC 来实现。

.. _contract_upgrading_ref:

Contract 和 state 版本
-----------------------------

There are two types of contract/state upgrade:

1. *Implicit:* By allowing multiple implementations of the contract ahead of time, using constraints. See
   :doc:`api-contract-constraints` to learn more
2. *Explicit:* By creating a special *contract upgrade transaction* and getting all participants of a state to sign it
   using the contract upgrade flows

这里有两种类型的 contract/state 升级：

1. *隐式的升级*：使用约束（constraints）允许提前对于 contract 开发多种实现。查看 :doc:`api-contract-constraints` 了解更多
1. *显式的升级*：创建一个特殊的 *更新合约的 transaction* 然后使用升级合约 flows 来获得 state 的所有参与者的签名

The general recommendation for Corda 4 is to use **implicit** upgrades for the reasons described :ref:`here <implicit_vs_explicit_upgrades>`.

对于 Corda 4 推荐的方式是使用 **隐式升级**，在 :ref:`here <implicit_vs_explicit_upgrades>` 里描述了原因。

.. _explicit_contract_upgrades_ref:

进行显式的 contract 和 state 升级
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In an explicit upgrade, contracts and states can be changed in arbitrary ways, if and only if all of the state's
participants agree to the proposed upgrade. To ensure the continuity of the chain the upgraded contract needs to declare the contract and
constraint of the states it's allowed to replace.

在显式的升级中，contracts 和 state 可以按照任何的方式来变化，这些变化仅仅在 state 的所有参与者对这个升级都同意的条件下才会生效。为了确保链的连续性，升级的 contract 需要声明它允许替换的 states 的 contract 和约束。

.. warning:: In Corda 4 we've introduced the Signature Constraint (see :doc:`api-contract-constraints`). States created or migrated to
            the Signature Constraint can't be explicitly upgraded using the Contract upgrade transaction. This feature might be added in a future version.
            Given the nature of the Signature constraint there should be little need to create a brand new contract to fix issues in the old contract.

.. warning:: 在 Corda 4，我们引入了签名约束（查看 :doc:`api-contract-constraints`）。新建的或者迁移到签名约束的 states 不能使用 contract 升级 transaction 进行显式的升级。这个功能可能会在将来的版本中添加。基于签名约束的本质特点，这可能有很小的需求来创建一个全新的 contract 来解决在旧的 contract 中的问题。

1. 保留已经存在的 state 和 contract 的定义
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Currently, all nodes must **permanently** keep **all** old state and contract definitions on their node's classpath if the explicit upgrade
process was used on them.

当前，如果使用显式的升级，所有节点必须要在他们节点的 classpath 上 **永久地** 保存 **所有** 就的 state 和 contract 的定义。

.. note:: This requirement will go away in a future version of Corda. In Corda 4, the contract-code-as-attachment feature was implemented
          only for "normal" transactions. ``Contract Upgrade`` and ``Notary Change`` transactions will still be executed within the node classpath.

.. note:: 这个需要会在将来版本的 Corda 中去除。在 Corda 4 中，contract-code-as-attachment 的功能仅仅对于 “常规” 的 transaction 实现了。``Contract Upgrade`` 和 ``Notary Change`` 还是会在节点的 classpath 中被执行。

2. 编写新的 state 和 contract 定义
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Update the contract and state definitions. There are no restrictions on how states are updated. However,
upgraded contracts must implement the ``UpgradedContract`` interface. This interface is defined as:

更新 contract 和/或 state 定义。对于如何更新 states，并没有任何的限制。但是更新 contracts 必须要实现 ``UpgradedContract`` 接口。接口定义如下

.. sourcecode:: kotlin

    interface UpgradedContract<in OldState : ContractState, out NewState : ContractState> : Contract {
        val legacyContract: ContractClassName
        fun upgrade(state: OldState): NewState
    }

The ``upgrade`` method describes how the old state type is upgraded to the new state type.

``upgrade`` 方法描述了旧的 state 类型是如何更新成新的 state 类型的。

By default this new contract will only be able to upgrade legacy states which are constrained by the zone whitelist (see :doc:`api-contract-constraints`).

新的 contract 默认只能够更新在白名单中的已有的 states（查看 :doc:`api-contract-constraints`）。

.. note:: The requirement for a ``legacyContractConstraint`` arises from the fact that when a transaction chain is verified and a ``Contract Upgrade`` is
          encountered on the back chain, the verifier wants to know that a legitimate state was transformed into the new contract. The ``legacyContractConstraint`` is
          the mechanism by which this is enforced. Using it, the new contract is able to narrow down what constraint the states it is upgrading should have.
          If a malicious party would create a fake ``com.megacorp.MegaToken`` state, he would not be able to use the usual ``MegaToken`` code as his
          fake token will not validate because the constraints will not match. The ``com.megacorp.SuperMegaToken`` would know that it is a fake state and thus refuse to upgrade it.
          It is safe to omit the ``legacyContractConstraint`` for the zone whitelist constraint, because the chain of trust is ensured by the Zone operator
          who would have whitelisted both contracts and checked them.

.. note:: 当一个 transaction 链被验证并且在之前的 chain 上遇到了一个 ``Contract Upgrade`` 的时候，验证着想要知道一个正确的 state 被转换成为一个新的 contract，由于这样一个事实，对于一个 ``legacyContractConstraint`` 的需求就被提了出来。``legacyContractConstraint`` 是一种强制执行这个的机制。使用它，新的 contract 能够知道这个 state 的升级使用的约束是什么。如果一个恶意节点创建了一个虚假的 ``com.megacorp.MegaToken`` state，它应该不能够使用常规的 ``MegaToken`` 代码，因为它的虚假的 token 由于不满足约束而不是正确的。``com.megacorp.SuperMegaToken`` 将会知道它是一个虚假的 state 因此就会拒绝更新它。这个对于 zone 白名单来说可以安全的省略 ``legacyContractConstraint``，因为具有 contracts 白名单并且会验证他们的 zone 维护者会确保 trust 链。

If the hash constraint is used, the new contract should implement ``UpgradedContractWithLegacyConstraint``
instead, and specify the constraint explicitly:

如果使用了 hash 约束类型话，新的 contract 必须要实现 ``UpgradedContractWithLegacyConstraint``，并且需要显式地指明是哪种约束：

.. sourcecode:: kotlin

    interface UpgradedContractWithLegacyConstraint<in OldState : ContractState, out NewState : ContractState> : UpgradedContract<OldState, NewState> {
        val legacyContractConstraint: AttachmentConstraint
    }

For example, in case of hash constraints the hash of the legacy JAR file should be provided:

比如，如果是 hash 约束的话，那么原始的 JAR 文件的 hash 需要被提供：

.. sourcecode:: kotlin

    override val legacyContractConstraint: AttachmentConstraint
        get() = HashAttachmentConstraint(SecureHash.parse("E02BD2B9B010BBCE49C0D7C35BECEF2C79BEB2EE80D902B54CC9231418A4FA0C"))

3. 创建新的 CorDapp JAR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Produce a new CorDapp JAR file. This JAR file should only contain the new contract and state definitions.

生成一个新的 CorDapp JAR 文件。这个 JAR 文件应该只包含新的 contract 和 state 定义。

4. 分发新的 CorDapp JAR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Place the new CorDapp JAR file in the ``cordapps`` folder of all the relevant nodes. You can do this while the nodes are still 
running.

将新的 CorDapp JAR 文件放在所有相关节点的 ``cordapps`` 文件夹下。你可以在节点还在运行的情况下做这些。

5. 关闭节点
^^^^^^^^^^^^^^^^^
Have each node operator stop their node. If you are also changing flow definitions, you should perform a 
:ref:`node drain <draining_the_node>` first to avoid the definition of states or contracts changing whilst a flow is 
in progress.

让每个节点维护者停止他们的节点。如果你也改变了 flow 定义的话，你需要首先执行 :ref:`排空节点 <draining_the_node>` ，来避免在一个 flow 仍在运行的过程中来引入新定义的 states 和 contracts。

6. 重新运行网络 bootstrapper (仅仅在你想要把新的 contract 加到白名单)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
If you're using the network bootstrapper instead of a network map server and have defined any new contracts, you need to 
re-run the network bootstrapper to whitelist the new contracts. See :doc:`network-bootstrapper`.

如果正在使用 network bootstrapper 而不是一个 network map server 并且定义了新的 contracts 的话，你需要重新运行 network bootstrapper 来将新的 contract 添加到白名单里。查看 :doc:`network-bootstrapper`。

7. 重启节点
^^^^^^^^^^^^^^^^^^^^
Have each node operator restart their node.

让每个节点维护者重启节点。

8. 升级授权
^^^^^^^^^^^^^^^^^^^^^^^^
Now that new states and contracts are on the classpath for all the relevant nodes, the nodes must all run the
``ContractUpgradeFlow.Authorise`` flow. This flow takes a ``StateAndRef`` of the state to update as well as a reference
to the new contract, which must implement the ``UpgradedContract`` interface.

如果新的 states 和 contracts 已经被放到了所有节点的 classpath 下之后，下一步就是每个节点去运行 ``ContractUpgradeFlow.Authorise`` flow。这个 flow 会带有一个需要更新的 ``StateAndRef`` 的 state，还有一个对新的 contract 的引用，这个 contract 必须要实现 ``UpgradedContract`` 接口。

At any point, a node administrator may de-authorise a contract upgrade by running the
``ContractUpgradeFlow.Deauthorise`` flow.

在任何时间，节点的管理员都可以通过运行 ``ContractUpgradeFlow.Deauthorise`` flow 来不通过一个 contract 的升级。

9. 执行升级
^^^^^^^^^^^^^^^^^^^^^^
Once all nodes have performed the authorisation process, a **single** node must initiate the upgrade via the
``ContractUpgradeFlow.Initiate`` flow for each state object. This flow has the following signature:

当所有的节点都执行完了授权流程后，必须要选择 **一个** 参与节点通过 ``ContractUpgradeFlow.Initiate`` flow 来初始对每个 state 对象的更新。这个 flow 有这样的特点：

.. sourcecode:: kotlin

    class Initiate<OldState : ContractState, out NewState : ContractState>(
        originalState: StateAndRef<OldState>,
        newContractClass: Class<out UpgradedContract<OldState, NewState>>
    ) : AbstractStateReplacementFlow.Instigator<OldState, NewState, Class<out UpgradedContract<OldState, NewState>>>(originalState, newContractClass)

This flow sub-classes ``AbstractStateReplacementFlow``, which can be used to upgrade state objects that do not need a
contract upgrade.

这个 flow 是 ``AbstractStateReplacementFlow`` 的子类（sub-class），它也可以用来对不需要更新 contract 的 state 对象进行更新。

One the flow ends successfully, all the participants of the old state object should have the upgraded state object
which references the new contract code.

当 flow 成功结束后，所有参与节点的旧的 state 对象应该被更新为升级过的 state 对象了，他们也会指向新的 contract code。

10. 将升级过的 state 从 zone 约束迁移到签名约束
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
遵循 :doc:`api-contract-constraints` 里的步骤。

需要注意的点
~~~~~~~~~~~~~~

Contract 更新 flows 的能力
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Despite its name, the ``ContractUpgradeFlow`` handles the update of both state object definitions and contract logic
* The state can completely change as part of an upgrade! For example, it is possible to transmute a ``Cat`` state into
  a ``Dog`` state, provided that all participants in the ``Cat`` state agree to the change
* If a node has not yet run the contract upgrade authorisation flow, they will not be able to upgrade the contract
  and/or state objects
* State schema changes are handled separately

* 不需要管它的名字，``ContractUpgradeFlow`` 同样可以处理 state 对象和 contract 逻辑定义的更新
* 在一次升级中，State 可以彻底的发生改变。比如可以从一个 ``猫`` state 变成 ``狗`` state，只需要确保所有 猫 state 的参与者都同意这个变化
* 如果一个节点没有运行 contract 升级授权 flow 的话，他们将不会更新 contract 和/或 state 对象的更新
* State schema 改动需要单独处理

过程
^^^^^^^^^
* All nodes need to run the contract upgrade authorisation flow to upgrade the contract and/or state objects
* Only node administrators are able to run the contract upgrade authorisation and deauthorisation flows
* Upgrade authorisations can subsequently be deauthorised
* Only one node should run the contract upgrade initiation flow. If multiple nodes run it for the same ``StateRef``, a
  double-spend will occur for all but the first completed upgrade
* Upgrades do not have to happen immediately. For a period, the two parties can use the old states and contracts
  side-by-side
* The supplied upgrade flows upgrade one state object at a time

* 所有节点需要运行 contract 升级授权 flow 来升级 contract 和/或者 state 对象
* 只有节点管理者能够运行 contract 升级授权和结束授权 flows
* 升级授权可以在后续被停止授权
* 只有一个节点应该运行初始 contract 升级 flow。如果多个节点对相同的 ``StateAndRef`` 运行了初始化 flow，一个“双花”问题会在双方产生，最先完成的会生效
* 升级不需要马上执行。在一段时期内，双方还是可以继续使用旧的 state 和 contracts
* 这里提供的升级 flows 每次只会升级一个 state 对象

State schema 版本
-----------------------
By default, all state objects are serialised to the database as a string of bytes and referenced by their ``StateRef``.
However, it is also possible to define custom schemas for serialising particular properties or combinations of
properties, so that they can be queried from a source other than the Corda Vault. This is done by implementing the
``QueryableState`` interface and creating a custom object relational mapper for the state. See :doc:`api-persistence`
for details.

默认的，所有的 state 对象都会以被序列化为字节格式的字符串而存到数据库中，并且会被他们的 ``StateRef`` 引用。然而对某些特定的属性或者一些属性的集合的序列化也是可以定义自定义的 schemas 的，所以他们就可以从一个数据源被检索而不是直接检索 Corda Vault。这个是通过实现 ``QueryableState`` 接口并且对这个 state 创建一个自定义的 ORM(Object Relational Mapper) 来实现的。查看 :doc:`api-persistence` 了解更详细的信息。

For backwards compatible changes such as adding columns, the procedure for upgrading a state schema is to extend the
existing object relational mapper. For example, we can update:

针对于向后兼容性，像添加新的 columns 这样的改动，升级一个 state schema 的过程其实是对已经存在的 ORM 进行扩展。比如，我们可以将下边的 schema：

.. container:: codeset

    .. sourcecode:: kotlin

        object ObligationSchemaV1 : MappedSchema(Obligation::class.java, 1, listOf(ObligationEntity::class.java)) {
            @Entity @Table(name = "obligations")
            class ObligationEntity(obligation: Obligation) : PersistentState() {
                @Column var currency: String = obligation.amount.token.toString()
                @Column var amount: Long = obligation.amount.quantity
                @Column @Lob var lender: ByteArray = obligation.lender.owningKey.encoded
                @Column @Lob var borrower: ByteArray = obligation.borrower.owningKey.encoded
                @Column var linear_id: String = obligation.linearId.id.toString()
            }
        }

    .. sourcecode:: java

        public class ObligationSchemaV1 extends MappedSchema {
            public ObligationSchemaV1() {
                super(Obligation.class, 1, ImmutableList.of(ObligationEntity.class));
            }
        }

        @Entity
        @Table(name = "obligations")
        public class ObligationEntity extends PersistentState {
            @Column(name = "currency") private String currency;
            @Column(name = "amount") private Long amount;
            @Column(name = "lender") @Lob private byte[] lender;
            @Column(name = "borrower") @Lob private byte[] borrower;
            @Column(name = "linear_id") private UUID linearId;

            protected ObligationEntity(){}

            public ObligationEntity(String currency, Long amount, byte[] lender, byte[] borrower, UUID linearId) {
                this.currency = currency;
                this.amount = amount;
                this.lender = lender;
                this.borrower = borrower;
                this.linearId = linearId;
            }

            public String getCurrency() {
                return currency;
            }

            public Long getAmount() {
                return amount;
            }

            public byte[] getLender() {
                return lender;
            }

            public byte[] getBorrower() {
                return borrower;
            }

            public UUID getLinearId() {
                return linearId;
            }
        }

变为：

.. container:: codeset

    .. sourcecode:: kotlin

        object ObligationSchemaV1 : MappedSchema(Obligation::class.java, 1, listOf(ObligationEntity::class.java)) {
            @Entity @Table(name = "obligations")
            class ObligationEntity(obligation: Obligation) : PersistentState() {
                @Column var currency: String = obligation.amount.token.toString()
                @Column var amount: Long = obligation.amount.quantity
                @Column @Lob var lender: ByteArray = obligation.lender.owningKey.encoded
                @Column @Lob var borrower: ByteArray = obligation.borrower.owningKey.encoded
                @Column var linear_id: String = obligation.linearId.id.toString()
                @Column var defaulted: Bool = obligation.amount.inDefault               // NEW COLUMN!
            }
        }

    .. sourcecode:: java

        public class ObligationSchemaV1 extends MappedSchema {
            public ObligationSchemaV1() {
                super(Obligation.class, 1, ImmutableList.of(ObligationEntity.class));
            }
        }

        @Entity
        @Table(name = "obligations")
        public class ObligationEntity extends PersistentState {
            @Column(name = "currency") private String currency;
            @Column(name = "amount") private Long amount;
            @Column(name = "lender") @Lob private byte[] lender;
            @Column(name = "borrower") @Lob private byte[] borrower;
            @Column(name = "linear_id") private UUID linearId;
            @Column(name = "defaulted") private Boolean defaulted;            // NEW COLUMN!

            protected ObligationEntity(){}

            public ObligationEntity(String currency, Long amount, byte[] lender, byte[] borrower, UUID linearId, Boolean defaulted) {
                this.currency = currency;
                this.amount = amount;
                this.lender = lender;
                this.borrower = borrower;
                this.linearId = linearId;
                this.defaulted = defaulted;
            }

            public String getCurrency() {
                return currency;
            }

            public Long getAmount() {
                return amount;
            }

            public byte[] getLender() {
                return lender;
            }

            public byte[] getBorrower() {
                return borrower;
            }

            public UUID getLinearId() {
                return linearId;
            }

            public Boolean isDefaulted() {
                return defaulted;
            }
        }

Thus adding a new column with a default value.

因此当添加一个新的 column 的时候，给它一个默认值。

To make a non-backwards compatible change, the ``ContractUpgradeFlow`` or ``AbstractStateReplacementFlow`` must be
used, as changes to the state are required. To make a backwards-incompatible change such as deleting a column (e.g.
because a property was removed from a state object), the procedure is to define another object relational mapper, then
add it to the ``supportedSchemas`` property of your ``QueryableState``, like so:

对于一个非向后兼容的改动，那么必须要使用 ``ContractUpgradeFlow`` 或者 ``AbstractStateReplacementFlow``，因为必须要对 state 也要做改动。对于一个非向后兼容的改动，比如删除了一个 column（比如因为某个属性需要从 state 对象中被删除），更新的过程应该是定义另外一个 ORM，然后将它添加到你的 ``QueryableState`` 的 ``supportedSchemas`` 属性中，像下边这样：

.. container:: codeset

    .. sourcecode:: kotlin

        override fun supportedSchemas(): Iterable<MappedSchema> = listOf(ExampleSchemaV1, ExampleSchemaV2)

    .. sourcecode:: java

        @Override public Iterable<MappedSchema> supportedSchemas() {
            return ImmutableList.of(new ExampleSchemaV1(), new ExampleSchemaV2());
        }

Then, in ``generateMappedObject``, add support for the new schema:

然后在 ``generateMappedObject`` 中添加对新的 schema 的支持：

.. container:: codeset

    .. sourcecode:: kotlin

        override fun generateMappedObject(schema: MappedSchema): PersistentState {
            return when (schema) {
                is DummyLinearStateSchemaV1 -> // Omitted.
                is DummyLinearStateSchemaV2 -> // Omitted.
                else -> throw IllegalArgumentException("Unrecognised schema $schema")
            }
        }

    .. sourcecode:: java

        @Override public PersistentState generateMappedObject(MappedSchema schema) {
            if (schema instanceof DummyLinearStateSchemaV1) {
                // Omitted.
            } else if (schema instanceof DummyLinearStateSchemaV2) {
                // Omitted.
            } else {
                throw new IllegalArgumentException("Unrecognised schema $schema");
            }
        }

With this approach, whenever the state object is stored in the vault, a representation of it will be stored in two
separate database tables where possible - one for each supported schema.

通过这种方式，当 state 对象被存储到 vault 中的时候，它的代表（representation）会被分别存储到两个数据库表中，每个代表着一个支持的 schema。

序列化
-------------

Corda 序列化格式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Currently, the serialisation format for everything except flow checkpoints (which uses a Kryo-based format) is based
on AMQP 1.0, a self-describing and controllable serialisation format. AMQP is desirable because it allows us to have
a schema describing what has been serialized alongside the data itself. This assists with versioning and deserialising
long-ago archived data, among other things.

当前，所有的序列化格式除了 flow checkpoints（使用 Kryo-based 格式） 以外都是基于 AMQP 1.0，一个自描述（self-describing）和可控的序列化格式。AMQP 是正确的选择因为除了被序列化的数据本身，它允许我们可以定义一个 schema 来描述什么被序列化了。这个协助了版本以及反序列化很久以前 archive 的数据，和其他的事情。

编写满足序列化格式需求的类
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Although not strictly related to versioning, AMQP serialisation dictates that we must write our classes in a particular way:

* Your class must have a constructor that takes all the properties that you wish to record in the serialized form. This
  is required in order for the serialization framework to reconstruct an instance of your class
* If more than one constructor is provided, the serialization framework needs to know which one to use. The
  ``@ConstructorForDeserialization`` annotation can be used to indicate the chosen constructor. For a Kotlin class
  without the ``@ConstructorForDeserialization`` annotation, the primary constructor is selected
* The class must be compiled with parameter names in the .class file. This is the default in Kotlin but must be turned
  on in Java (using the ``-parameters`` command line option to ``javac``)
* Your class must provide a Java Bean getter for each of the properties in the constructor, with a matching name. For
  example, if a class has the constructor parameter ``foo``, there must be a getter called ``getFoo()``. If ``foo`` is
  a boolean, the getter may optionally be called ``isFoo()``. This is why the class must be compiled with parameter
  names turned on
* The class must be annotated with ``@CordaSerializable``
* The declared types of constructor arguments/getters must be supported, and where generics are used the generic
  parameter must be a supported type, an open wildcard (*), or a bounded wildcard which is currently widened to an open
  wildcard
* Any superclass must adhere to the same rules, but can be abstract
* Object graph cycles are not supported, so an object cannot refer to itself, directly or indirectly

虽然并不是跟版本有着很严格的联系，AMQP 序列化要求我们要以一种特别的方式来编写我们的类：

* 你的类必须要有个构造器，这个构造器需要有你所有想要以被序列化的形式记录的所有属性。之所以需要这样是为了序列化框架能够重现你的类的实例
* 如果提供了不止一个构造器的话，序列化框架需要知道应该使用哪一个。``@ConstructorForDeserialization`` 注解可以用来指定选择的构造器。对于一个没有 ```@ConstructorForDeserialization`` 注解的 Kotlin 的类，主的构造器会被选择
* 类必须要含有 .class 文件中的参数名字，从而被编译。这在 Kotlin 中是默认的但是在 Java 中必须要被开启（对于 javac 使用 ```-parameters`` 命令行选项）
* 你的类对于在构造器中的每个属性都需要提供一个 Java Bean getter，而且名字要跟构造器中的一样。比如，如果一个类含有一个构造器参数 ``foo``，那么必须要有一个名字为 ``getFoo()`` 的 getter。如果 ``foo`` 是一个 boolean 类型的，那么 getter 可能需要被命名为 ``isFoo()```。这也是为什么类必须要以将参数名字开启的方式被编译
* 类必须要有 ```@CordaSerializable`` 的注解
* 必须要支持针对于构造器参数/getters 定义的类型，当有 generics 被使用的时候， generic 参数也必须是一个被支持的类型，一个打开的通配符（）或者一个现在对一个打开的通配符进行扩展的有限的通配符
* 任何的超级类（superclass）也必须要遵循这个原则，但是可以是个抽象类
* 对象 graph 周期当前还不支持，所以一个对象是不能够直接或间接的引用它自己的