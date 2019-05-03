共识
=========

.. topic:: 概要

   * *To be committed, transactions must achieve both validity and uniqueness consensus*
   * *Validity consensus requires contractual validity of the transaction and all its dependencies*
   * *Uniqueness consensus prevents double-spends*

   * *为了交易能够被提交，transaction 必须要同时满足有效性和 唯一性的共识*
   * *有效性共识需要 transaction 和 它的所有依赖都是合约有效的*
   * *唯一性共识可以避免“双花”*

.. only:: htmlmode

   Video
   -----
   .. raw:: html
   
       <iframe src="https://player.vimeo.com/video/214138438" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
       <p></p>


两种类型的共识
----------------------
Determining whether a proposed transaction is a valid ledger update involves reaching two types of consensus:

* *Validity consensus* - this is checked by each required signer before they sign the transaction
* *Uniqueness consensus* - this is only checked by a notary service

判断一个交易的提案是否是一次有效的账本更新要达到两种类型的共识：

* *有效性共识*：这给是交易所要求的签名者在提供他们签名之前去校验的
* *唯一性共识*：这个只会被 notary service 去验证

有效性共识
------------------
Validity consensus is the process of checking that the following conditions hold both for the proposed transaction,
and for every transaction in the transaction chain that generated the inputs to the proposed transaction:

* The transaction is accepted by the contracts of every input and output state
* The transaction has all the required signatures

有效性共识是关于验证下边所描述的条件对于提交的 transaction 和生成该该 transaction 的 inputs 的交易链中的每次 transaction 都必须要满足：

* Transaction 中的每个 input 和 output 的 contracts 所接受
* Transaction 得到了所有要求的签名

It is not enough to verify the proposed transaction itself. We must also verify every transaction in the chain of
transactions that led up to the creation of the inputs to the proposed transaction.

仅仅检查交易提案本身信息是不够的。我们还需要检查跟产生当前这个 transaction 的 inputs 有关的所有以前的 transaction 链。

This is known as *walking the chain*. Suppose, for example, that a party on the network proposes a transaction
transferring us a treasury bond. We can only be sure that the bond transfer is valid if:

* The treasury bond was issued by the central bank in a valid issuance transaction
* Every subsequent transaction in which the bond changed hands was also valid

这个被称作 *walking the chain*。假设，例如网络中的一个节点提交了一个交换债券的一笔交易。我们只有了解下边的情况才能确保这个债券的交换是有效的：

* 这个债券应该是由中心银行发行的，而且应该是在一次有效的发行交易中
* 关于这个债券的后续交易记录也应该都是有效的

The only way to be sure of both conditions is to walk the transaction's chain. We can visualize this process as follows:

确保两点都满足的唯一方式就是查看整个交易链。我们可以用下图表示：

.. image:: resources/validation-consensus.png
   :scale: 25%
   :align: center

When verifying a proposed transaction, a given party may not have every transaction in the transaction chain that they
need to verify. In this case, they can request the missing transactions from the transaction proposer(s). The
transaction proposer(s) will always have the full transaction chain, since they would have requested it when
verifying the transaction that created the proposed transaction's input states.

当确认一个交易提案的时候，给定的一方可能没有它需要验证的交易链上的所有交易信息。这种情况下，他可以向交易的提出方索要缺少的那部分交易。交易的提出方应该永远会有整个的交易链信息，因为他们应该在验证之前的交易中已经获取了相关的交易链信息。

唯一性共识
--------------------
Imagine that Bob holds a valid central-bank-issued cash state of $1,000,000. Bob can now create two transaction
proposals:

* A transaction transferring the $1,000,000 to Charlie in exchange for £800,000
* A transaction transferring the $1,000,000 to Dan in exchange for €900,000

设想一下 Bob 持有有效的由中央银行发行的 $1,000,000 现金 state。Bob 可以创建两个交易提案：

* 一笔交易要跟 Charlie 用这 $1,000,000 交换 £800,000
* 一笔交易要跟 Dan 用这 $1,000,000 交换 €900,000

This is a problem because, although both transactions will achieve validity consensus, Bob has managed to
"double-spend" his USD to get double the amount of GBP and EUR. We can visualize this as follows:

这会是一个问题，因为尽管这两笔交易都可以通过有效性共识，但是 Bob 确实现了一次“双花 double spend” 他的美元来获得了两倍价值的 GBP 和 EUR。我们可以用下图表示这个流程：

.. image:: resources/uniqueness-consensus.png
   :scale: 25%
   :align: center

To prevent this, a valid transaction proposal must also achieve uniqueness consensus. Uniqueness consensus is the
requirement that none of the inputs to a proposed transaction have already been consumed in another transaction.

为了避免这样的问题发生，一个有效的交易提案同时也要满足唯一性共识。唯一性共识要求一个 transaction 的 input 不能被任何其他的 transaction 消费掉过。

If one or more of the inputs have already been consumed in another transaction, this is known as a *double spend*,
and the transaction proposal is considered invalid.

当一个交易中的一个或多个 inputs 已经被其他的交易消费掉的情况，通常被称为 *双花*，那么相关的交易应该被视为无效的交易。

Uniqueness consensus is provided by notaries. See :doc:`key-concepts-notaries` for more details.

唯一性共识是有 notaries 提供的。查看 :doc:`key-concepts-notaries` 了解更多详细信息。