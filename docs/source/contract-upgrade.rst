.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

升级合约
===================

While every care is taken in development of contract code, inevitably upgrades will be required to fix bugs (in either
design or implementation). Upgrades can involve a substitution of one version of the contract code for another or
changing to a different contract that understands how to migrate the existing state objects. When state objects are
added as outputs to transactions, they are linked to the contract code they are intended for via the
``StateAndContract`` type. Changing a state's contract only requires substituting one ``ContractClassName`` for another.

虽然在开发合约代码的过程中我们会非常小心，但是由于解决 bug（设计或者实施）升级还是不可避免的。升级可能会涉及到将合约代码从一个版本升级到另一个版本，或者变成另一个不同的合约，这个合约知道应该如何升级已有的 state 对象。当 state 对象作为 outputs 被添加到 transactions 的时候，通过 ``StateAndContract`` 类型，他们被连关联到他们要用的合约代码。修改 state 的合约代码只需要将 ``ContractClassName`` 替换成另外的名字。

Workflow
--------
Here's the workflow for contract upgrades:

1. Banks A and B negotiate a trade, off-platform

2. Banks A and B execute a flow to construct a state object representing the trade, using contract X, and include it in
   a transaction (which is then signed and sent to the consensus service)

3. Time passes

4. The developer of contract X discovers a bug in the contract code, and releases a new version, contract Y. The
   developer will then notify all existing users (e.g. via a mailing list or CorDapp store) to stop their nodes from
   issuing further states with contract X

5. Banks A and B review the new contract via standard change control processes and identify the contract states they
   agree to upgrade (they may decide not to upgrade some contract states as these might be needed for some other
   obligation contract)

6. Banks A and B instruct their Corda nodes (via RPC) to be willing to upgrade state objects with contract X to state
   objects with contract Y using the agreed upgrade path

7. One of the parties (the ``Initiator``) initiates a flow to replace state objects referring to contract X with new
   state objects referring to contract Y

8. A proposed transaction (the ``Proposal``), with the old states as input and the reissued states as outputs, is
   created and signed with the node's private key

9. The ``Initiator`` node sends the proposed transaction, along with details of the new contract upgrade path that it
   is proposing, to all participants of the state object

10. Each counterparty (the ``Acceptor`` s) verifies the proposal, signs or rejects the state reissuance accordingly, and
    sends a signature or rejection notification back to the initiating node

11. If signatures are received from all parties, the ``Initiator`` assembles the complete signed transaction and sends
    it to the notary

下边是升级合约的流程：

1. Bank A 和 B 在线下协商了一笔交易
2. Bank A 和 B 执行了一个 flow 来构建了一个 state 对象来表示这笔交易，使用的是合约 X，并将它放到了一个 transaction 中（接下来会被签名并发送到共识服务，即 Notary Service）
3. 一段时间过去了
4. 合约 X 的开发者发现了合约代码中的一个 bug，然后就发布了一个新的版本，合约 Y。开发者接下来会通知所有已有的用户（比如使用一个邮件列表或者 CorDapp store）来停止他们的节点继续用合约 X 来初始新的 states 对象
5. Bank A 和 B 通过标准的改动控制流程（change control process）来 review 新的合约代码并且识别他们同意升级的合约 states（他们可能会选择不升级某些合约 states，因为这些 states 可能被其他的一些债务合约在使用）
6. Bank A 和 B 会告诉他们的节点（通过 RPC）将会使用双方同意的更新途径来将使用合约 X 的 state 对象更新到使用合约 Y 的 state 对象
7. 相关方的某一方（``Initiator``）会初始一个 flow 来将使用合约 X 的 state 对象替换为新的使用合约 Y 的 state 对象
8. 一个将老的 states 作为 input，将新的 states 作为 output 的 transaction（``Proposal``）被提出并且被使用节点的私钥进行了签名。
9. 初始（``Initiator``）节点将这个 transaction 和关于如何更新到新的合约的途径详细信息发送给 state 对象的所有参与者。
10. 每个合作方（``Acceptor``）验证这个提议，签名或者拒绝这个重新生成 state 的提议，然后将签名或者拒绝的结果发送给初始节点
11. 如果从所有相关节点都收到了签名的话，``Initiator`` 节点会整理这个完全被签署过的 transaction 并且发送给 notary

批准一个更新
----------------------
Each of the participants in the state for which the contract is being upgraded will have to instruct their node that
they agree to the upgrade before the upgrade can take place. The ``ContractUpgradeFlow`` is used to manage the
authorisation process. Each node administrator can use RPC to trigger either an ``Authorise`` or a ``Deauthorise`` flow
for the state in question.

在升级可以开始之前，将要被升级的 state 中的参与者们（participants）需要告诉他们的节点他们已经同意了这个更新。``ContractUpgradeFlow`` 被用来管理这个授权的流程。对于将要执行的更新 state 的 flow，每个节点的管理员可以使用 RPC 来触发一个 ``Authorise`` 或 ``Deauthorise`` 的命令。

.. literalinclude:: ../../core/src/main/kotlin/net/corda/core/flows/ContractUpgradeFlow.kt
    :language: kotlin
    :start-after: DOCSTART 1
    :end-before: DOCEND 1
    :dedent: 4

.. literalinclude:: ../../core/src/main/kotlin/net/corda/core/flows/ContractUpgradeFlow.kt
    :language: kotlin
    :start-after: DOCSTART 2
    :end-before: DOCEND 2
    :dedent: 4

提出一个更新
--------------------
After all parties have authorised the contract upgrade for the state, one of the contract participants can initiate the
upgrade process by triggering the ``ContractUpgradeFlow.Initiate`` flow. ``Initiate`` creates a transaction including
the old state and the updated state, and sends it to each of the participants. Each participant will verify the
transaction, create a signature over it, and send the signature back to the initiator. Once all the signatures are
collected, the transaction will be notarised and persisted to every participant's vault.

当所有的节点都批准了对于 state 的合约更新之后，某一个合约的参与者可以通过触发 ``ContractUpgradeFlow.Initiate`` flow 来初始这个更新的流程。``Initiate`` 创建了一个包含旧的 state 和更新的 state 的 transaction，然后发送给了每一个 state 的参与者。每个参与者都会检查这个 transaction，提供签名并将签名发送回给变更初始者。当收到所有的签名之后，这个 transaction 会被公正并且永久地被存储到每个参与者的账本（vault）中。

示例
-------
Suppose Bank A has entered into an agreement with Bank B which is represented by the state object
``DummyContractState`` and governed by the contract code ``DummyContract``. A few days after the exchange of contracts,
the developer of the contract code discovers a bug in the contract code.

假设 Bank A 和 Bank B 达成了一个协议，这个协议以 ``DummyContractState`` state 对象来表示并且由合约代码 ``DummyContract`` 来管理。当交换完合约之后的某天，合约代码的开发者发现额合约代码中的一个 bug。

Bank A and Bank B decide to upgrade the contract to ``DummyContractV2``:

Bank A 和 Bank B 决定将合约升级为 ``DummyContractV2``：

1. The developer creates a new contract ``DummyContractV2`` extending the ``UpgradedContract`` class, and a new state
   object ``DummyContractV2.State`` referencing the new contract.

   开发者通过扩展 ``UpgradedContract`` 类创建了一个新的合约 ``DummyContractV2``，和一个新的针对新合约的 state 对象 ``DummyContractV2.State``。

.. literalinclude:: /../../testing/test-utils/src/main/kotlin/net/corda/testing/contracts/DummyContractV2.kt
    :language: kotlin
    :start-after: DOCSTART 1
    :end-before: DOCEND 1

2. Bank A instructs its node to accept the contract upgrade to ``DummyContractV2`` for the contract state.
   Bank A 告诉他的节点接受这个针对于 contract state 的合约更新为 ``DummyContractV2``。

.. container:: codeset

   .. sourcecode:: kotlin
   
      val rpcClient : CordaRPCClient = << Bank A's Corda RPC Client >>
      val rpcA = rpcClient.proxy()
      rpcA.startFlow(ContractUpgradeFlow.Authorise(<<StateAndRef of the contract state>>, DummyContractV2::class.java))

3. Bank B initiates the upgrade flow, which will send an upgrade proposal to all contract participants. Each of the
   participants of the contract state will sign and return the contract state upgrade proposal once they have validated
   and agreed with the upgrade. The upgraded transaction will be recorded in every participant's node by the flow.

   Bank B 初始了这个升级的 flow，这会给所有的合约的参与者节点发送一个更新的提议。合约 state 的每个参与者，当他们验证完并且同意这个更新之后，将会提供签名并返回合约 state 的更新提议。Flow 会将被更新的 transaction 记录到每个参与节点中。

.. container:: codeset

   .. sourcecode:: kotlin
      
      val rpcClient : CordaRPCClient = << Bank B's Corda RPC Client >>
      val rpcB = rpcClient.proxy()
      rpcB.startFlow({ stateAndRef, upgrade -> ContractUpgradeFlow(stateAndRef, upgrade) },
          <<StateAndRef of the contract state>>,
          DummyContractV2::class.java)
          
.. note:: See ``ContractUpgradeFlowTest`` for more detailed code examples.

.. note:: 查看 ``ContractUpgradeFlowTest`` 了解更多样例代码。
