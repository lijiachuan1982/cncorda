Transactions
============

.. topic:: 概要

   * *Transactions are proposals to update the ledger*
   * *A transaction proposal will only be committed if:*

     * *It doesn't contain double-spends*
     * *It is contractually valid*
     * *It is signed by the required parties*

   * *Transaction 是关于更新账本的提议*
   * *一个 transaction 提议只能在满足以下条件的时候才会被提交：*

     * *它不包含“双花”*
     * *它是合约有效的*
     * *它需要被所有相关方提供签名*

.. only:: htmlmode

   Video
   -----
   .. raw:: html
   
       <iframe src="https://player.vimeo.com/video/213879807" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
       <p></p>


概览
--------
Corda uses a *UTXO* (unspent transaction output) model where every state on the ledger is immutable. The ledger
evolves over time by applying *transactions*, which update the ledger by marking zero or more existing ledger states
as historic (the *inputs*) and producing zero or more new ledger states (the *outputs*). Transactions represent a
single link in the state sequences seen in :doc:`key-concepts-states`.

Corda 使用 *UTXO* (未消费的 transaction output) 模型来使账本上的每条 state 都不可更改。对于账本上数据的变更都是通过使用 *transaction* 的方式来做的，就是将 0 条或多条已经存在的账本 states 变为历史记录（*inputs*），然后再新增0条或多条新的账本 states （*outputs*）。交易代表了 state 顺序中的一个单独的链接，查看 :doc:`key-concepts-states`。

Here is an example of an update transaction, with two inputs and two outputs:

下边是一个更新的 transaction 的例子，包括了两个 inputs 和两个 outputs：

.. image:: resources/basic-tx.png
   :scale: 25%
   :align: center

A transaction can contain any number of inputs, outputs and references of any type:

一个 transaction 中可以包含任何数量任何类型的 inputs 和 outputs：

* They can include many different state types (e.g. both cash and bonds)
* They can be issuances (have zero inputs) or exits (have zero outputs)
* They can merge or split fungible assets (e.g. combining a $2 state and a $5 state into a $7 cash state)

* 可以包含多种类型的 state（cash, bonds）
* 可以是 issuances 类型（有0个input）或者 exists 类型（有0个 output）
* 可以合并或拆分可替换的资产（比如把一个 $2 的 state 和 $5 的 state 合并为 $7 的 cash state）

Transactions are *atomic*: either all the transaction's proposed changes are accepted, or none are.

Transaction 是 *原子性操作*，一个 transaction 里边的所有 changes 必须要全部执行，或者一个也不会执行。

There are two basic types of transactions:

有两种基本类型的 transactions：

* Notary-change transactions (used to change a state's notary - see :doc:`key-concepts-notaries`)
* General transactions (used for everything else)

* Notary-change transactions（用来变更 state 的 notary - 查看 :doc:`key-concepts-notaries`）
* General transactions（其他任何类型的 transaction）

交易链
------------------
When creating a new transaction, the output states that the transaction will propose do not exist yet, and must
therefore be created by the proposer(s) of the transaction. However, the input states already exist as the outputs of
previous transactions. We therefore include them in the proposed transaction by reference.

一个新的 transaction 的 output state 在账本中应该是还不存在的，所以需要提出 transaction 的一方来创建。但是 transaction 中包含的 input 应该是在账本中已经存在的，应该是前一个 transaction 添加进去的 output。所以我们需要在新的 transaction 中引用这些已经存在的记录。

These input states references are a combination of:

这些 Input state 的引用包含两部分：

* The hash of the transaction that created the input
* The input's index in the outputs of the previous transaction

* 创建这个 input 的 transaction 的 hash
* 这个 input 所指的前一个 transaction 带来的 output state 在 output list 中的位置或者索引值

This situation can be illustrated as follows:

下图描述了一个 transaction 链：

.. image:: resources/tx-chain.png
   :scale: 25%
   :align: center

These input state references link together transactions over time, forming what is known as a *transaction chain*.

这些 input state 引用将 transaction 连接在了一起，形成了所谓的 *交易链*。

提交交易
-----------------------
Initially, a transaction is just a **proposal** to update the ledger. It represents the future state of the ledger
that is desired by the transaction builder(s):

初始的时候，一个 transaction 仅仅是一个更新账本的 **提议**。它表示了经过这次更新后账本的新的状态：

.. image:: resources/uncommitted_tx.png
   :scale: 25%
   :align: center

To become reality, the transaction must receive signatures from all of the *required signers* (see **Commands**, below). Each
required signer appends their signature to the transaction to indicate that they approve the proposal:

为了成为真正的一笔交易，transaction 必须要获得所有 *要求的签名*（查看下边的 **command**）。每一个要求的签名者会将签名附加在 transaction 上来表示他们已经同意了这次更新：

.. image:: resources/tx_with_sigs.png
   :scale: 25%
   :align: center

If all of the required signatures are gathered, the transaction becomes committed:

如果得到了所有需要的签名，这个 transaction 就会被提交了：

.. image:: resources/committed_tx.png
   :scale: 25%
   :align: center

This means that:

* The transaction's inputs are marked as historic, and cannot be used in any future transactions
* The transaction's outputs become part of the current state of the ledger

这意味着：

* Transaction 的 input 被标注为历史记录，并且不能再被之后的 transactions 使用了
* Transaction 的 output 变为账本上的当前状态的一部分

交易的有效性
--------------------
Each required signers should only sign the transaction if the following two conditions hold:

   * **Transaction validity**: For both the proposed transaction, and every transaction in the chain of transactions
     that created the current proposed transaction's inputs:

       * The transaction is digitally signed by all the required parties
       * The transaction is *contractually valid* (see :doc:`key-concepts-contracts`)

   * **Transaction uniqueness**: There exists no other committed transaction that has consumed any of the inputs to
     our proposed transaction (see :doc:`key-concepts-consensus`)

每一个被要求的签名方应该只有在满足以下两个条件的时候才应该提供签名：

   * **Transaction 是有效的**：对于当前的 transaction 提案以及产生当前提案的 Input 相关的所有以前的所有 transactions 的链条中：

       * Transaction 应该获得所有相关方的数字签名
       * Transaction 是 *合约有效* 的（查看 :doc:`key-concepts-contracts`）

   * **Transaction 唯一性：本次 transaction 提案要消费的 inputs 没有被任何已经存在的其他的已提交的 transaction 消费过（查看 :doc:`key-concepts-consensus`）。

If the transaction gathers all the required signatures but these conditions do not hold, the transaction's outputs
will not be valid, and will not be accepted as inputs to subsequent transactions.

如果一个 transaction 获得了所有所需的签名，但是以上的条件并没有满足的话，这次 transaction 的 outputs 将会是无效的，也不会被之后的新的 transaction 用来作为 input。

参考 states
----------------

As mentioned in :doc:`key-concepts-states`, some states need to be referred to by the contracts of other input or output
states but not updated/consumed. This is where reference states come in. When a state is added to the references list of
a transaction, instead of the inputs or outputs list, then it is treated as a *reference state*. There are two important
differences between regular states and reference states:

* The specified notary for the transaction **does** check whether the reference states are current. However, reference
  states are not consumed when the transaction containing them is committed to the ledger.
* The contracts for reference states are not executed for the transaction containing them.

正如 :doc:`key-concepts-states` 所描述的，一些 states 需要被其他的 input 或者 output states 的合约代码所引用，但是不需要被修改/消费。这就需要参考 states。当一个 state 被添加到一笔交易的参考 states 列表中，而不是 inputs 或者 outputs 列表的时候，那么它就被作为 *参考 state*。在常规的 states 和参考 states 间有两点区别：

* 交易的 **节点** 指定的 notary 会检查参考 state 是不是当前的。然而，当包含他们的交易被提交的账本的时候，参考 states 是不会被消费的。
* 对于参考 states 的合约代码也不会被包含他们的交易所执行。

其他的交易组件
----------------------------
As well as input states and output states, transactions contain:

* Commands
* Attachments
* Time-Window
* Notary

就像 input states 和 output states 一样，transactions 还可能会包含下边的组件：

* Commands
* Attachments
* Timestamps
* Notary

For example, suppose we have a transaction where Alice uses a £5 cash payment to pay off £5 of an IOU with Bob.
This transaction has two supporting attachments and will only be notarised by NotaryClusterA if the notary pool
receives it within the specified time-window. This transaction would look as follows:

比如一个交易中，Alice 使用 £5 的现金向 Bob 支付了一个 IOU 的 £5。该笔交易包含了两个附件，并且只能够在 notary pool 在指定的时间窗内收到该笔交易的时候被 NotaryClusterA 进行公证，看起来像下边这样：

.. image:: resources/full-tx.png
   :scale: 25%
   :align: center

We explore the role played by the remaining transaction components below.

下边我们看一下剩下的交易组件扮演的角色。

Commands
^^^^^^^^
.. raw:: html

    <iframe src="https://player.vimeo.com/video/213881538" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
    <p></p>

Suppose we have a transaction with a cash state and a bond state as inputs, and a cash state and a bond state as
outputs. This transaction could represent two different scenarios:

假设我们有一个 transaction，其中包含了一个现金 state 和一个债券 state 作为 inputs，一个现金 state 和一个债券 state 作为 outputs。这个 transaction 可以代表两种情况：

* A bond purchase
* A coupon payment on a bond

* 购买债券
* 使用优惠券来支付债券

We can imagine that we'd want to impose different rules on what constitutes a valid transaction depending on whether
this is a purchase or a coupon payment. For example, in the case of a purchase, we would require a change in the bond's
current owner, whereas in the case of a coupon payment, we would require that the ownership of the bond does not
change.

我们可以假设我们要根据这是一个购买债券的交易还是一个使用优惠券支付的交易来制定不同的验证交易的规则。例如，针对购买债券的情况，我们会要求更改债券当前的所有者，但是对于一个使用优惠券付款的情况，我们不会要求改变债券的所有人。

For this, we have *commands*. Including a command in a transaction allows us to indicate the transaction's intent,
affecting how we check the validity of the transaction.

为了实现这个，我们使用 *commands*。在 transaction 中包含一个 command 允许我们能够表示 transaction 的意图，影响我们如何来验证 transaction 有效性。

Each command is also associated with a list of one or more *signers*. By taking the union of all the public keys
listed in the commands, we get the list of the transaction's required signers. In our example, we might imagine that:

* In a coupon payment on a bond, only the owner of the bond is required to sign
* In a cash payment, only the owner of the cash is required to sign

每一个命令也会关联一个或多个 *签名人*。通过在 commands 中列出的所有的公钥信息，我们就知道了这个 transaction 里所有要求的签名人的列表。在我们这个例子中，我们可以认为：

* 对于使用优惠券购买债券的情况，只有债券的所有者需要提供签名
* 对于一个现金支付的情况，只有现金的所有者需要提供给签名

We can visualize this situation as follows:

我们可以通过下图表示这个情况：

.. image:: resources/commands.png
   :scale: 25%
   :align: center

Attachments
^^^^^^^^^^^
.. raw:: html

    <iframe src="https://player.vimeo.com/video/213879328" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
    <p></p>

Sometimes, we have a large piece of data that can be reused across many different transactions. Some examples:

* A calendar of public holidays
* Supporting legal documentation
* A table of currency codes

有些时候，我们会有一些数据可以在不同的 transactions 中被重用。比如：

* 一个公共假期的 calendar
* 支持的法律文档
* 一个货币代码的表格

For this use case, we have *attachments*. Each transaction can refer to zero or more attachments by hash. These
attachments are ZIP/JAR files containing arbitrary content. The information in these files can then be
used when checking the transaction's validity.

针对这些情况，我们使用 *附件*。一个 transaction 可以通过 hash 引用 0 个或者多个附件。这些附件是 ZIP/JAR 文件，可以包含任何内容。这些附件中信息可以用来验证 transaction 的有效性。

Time-window
^^^^^^^^^^^
In some cases, we want a transaction proposed to only be approved during a certain time-window. For example:

* An option can only be exercised after a certain date
* A bond may only be redeemed before its expiry date

一些时候，我们希望一个交易仅仅在一个指定的时间点被批准执行。例如：

* 在一个指定的日期之后执行一个选项
* 一个债券只能在它的过期日期前被赎回

In such cases, we can add a *time-window* to the transaction. Time-windows specify the time window during which the
transaction can be committed. We discuss time-windows in the section on :doc:`key-concepts-time-windows`.

在这些情况下，我们给 transaction 添加一个 *time-window*。time-windows 制定了交易会在哪个时间点被提交。我们在 :doc:`key-concepts-time-windows` 讨论了 time-windows。

Notary
^^^^^^
A notary pool is a network service that provides uniqueness consensus by attesting that, for a given transaction,
it has not already signed other transactions that consume any of the proposed transaction’s input states.
The notary pool provides the point of finality in the system.

一个 Notary pool 是一个提供唯一性共识的网络服务，通过证明对于一个指定的新的交易提案的 inputs，不会有任何该服务之前提供过签名的交易已经消费掉该 inputs。Notary pool 在这个系统中提供了终结点。

Note that if the notary entity is absent then the transaction is not notarised at all. This is intended for
issuance/genesis transactions that don't consume any other states and thus can't double spend anything.
For more information on the notary services, see :doc:`key-concepts-notaries`.

注意如果 notary 实体缺失的话，交易是完全不能被公证的。这个是为 issuance/genesis 交易，这类的交易不会消费任何其他的 states，因此不能够重复消费任何 states，因此不会产生双花。更多关于 notary 服务的信息，请查看 :doc:`key-concepts-notaries`。