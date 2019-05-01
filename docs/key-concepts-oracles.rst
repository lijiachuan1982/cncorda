Oracles
=======

.. topic:: 概要

   * *A fact can be included in a transaction as part of a command*
   * *An oracle is a service that will only sign the transaction if the included fact is true*

   * *一个事实（fact）可以作为 command 的一部分被添加到一个 transaction 中*
   * *一个 oracle 是一个服务，它只会为那些包含正确事实的 transaction 提供签名*

.. only:: htmlmode

    Video
    -----
    .. raw:: html
    
        <iframe src="https://player.vimeo.com/video/214157956" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
        <p></p>


概览
--------
In many cases, a transaction's contractual validity depends on some external piece of data, such as the current
exchange rate. However, if we were to let each participant evaluate the transaction's validity based on their own
view of the current exchange rate, the contract's execution would be non-deterministic: some signers would consider the
transaction valid, while others would consider it invalid. As a result, disagreements would arise over the true state
of the ledger.

很多时候 transaction 的合约有效性需要依赖一些外部的数据，比如当前的汇率是多少。如果让每个参与方给予他们对于汇率的观点来验证 transaction 的有效性的话，合约的执行就会变得没有确定性了：一些参与者可能会认为 transaction 是有效的，而其他的参与者可能认为无效。因此，在真正账本中的 state 之上就会提出一些不同的意见。

Corda addresses this issue using *oracles*. Oracles are network services that, upon request, provide commands
that encapsulate a specific fact (e.g. the exchange rate at time x) and list the oracle as a required signer.

Corda 通过使用 *Oracle* 来解决这个问题。Oracle 是一个网络服务，可以根据要求提供包含某一事实的命令（比如在某个时间的汇率）并且将 Oracle 列为要求签名的一方。

If a node wishes to use a given fact in a transaction, they request a command asserting this fact from the oracle. If
the oracle considers the fact to be true, they send back the required command. The node then includes the command in
their transaction, and the oracle will sign the transaction to assert that the fact is true.

如果一个节点希望在一个 transaction 中使用某一个事实，那么它可以提出从 Oracle 来获取该事实的一个命令。如果 Orale 认为这个事实是正确的，它会返回这个要求的命令。然后这个节点就可以把这个命令添加到 transaction 中了，然后 oracle 会为这个事实是真的提供签名。

For privacy purposes, the oracle does not require to have access on every part of the transaction and the only
information it needs to see is their embedded, related to this oracle, command(s). We should also provide
guarantees that all of the commands requiring a signature from this oracle should be visible to
the oracle entity, but not the rest. To achieve that we use filtered transactions, in which the transaction proposer(s)
uses a nested Merkle tree approach to "tear off" the unrelated parts of the transaction. See :doc:`key-concepts-tearoffs`
for more information on how transaction tear-offs work.

为了隐私性的目的，Oracle 不需要能够访问交易的每个部分，他们唯一需要的信息就是看到他们内置的、跟这个 Oracle 相关的 command(s)。我们也应该提供让这些需要提供签名的 Oracle 实体能够看到这些 commands 的保证，但是不包括其他的部分。为了实现这个，我们使用过滤过的交易，是指交易的提案方使用一个内嵌的默克尔树的方式来将一些非相关的交易的部分隐藏掉。查看 :doc:`key-concepts-tearoffs` 了解关于交易如何拿掉工作的详细信息。

If they wish to monetize their services, oracles can choose to only sign a transaction and attest to the validity of
the fact it contains for a fee.

如果他们想为他们的服务定价，Oracles 可以选择只为那些包含服务费的交易提供签名并证明它包含的事实的有效性。