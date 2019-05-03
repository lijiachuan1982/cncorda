Contracts
=========

.. topic:: 概要

   * *A valid transaction must be accepted by the contract of each of its input and output states*
   * *Contracts are written in a JVM programming language (e.g. Java or Kotlin)*
   * *Contract execution is deterministic and its acceptance of a transaction is based on the transaction's contents alone*

   * *一个有效的 transaction 必须要被它的所有 input 和 output states中的 contract 接受*
   * *Contracts 需要使用 JVM 编程语言编写（java 或者 kotlin）*
   * *Contract 的执行是一定要有一个确定性结果的，并且它对于一个 transaction 的接受是仅仅基于 transaction 的内容*

.. only:: htmlmode

   Video
   -----
   .. raw:: html

       <iframe src="https://player.vimeo.com/video/214168839" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
       <p></p>


Transaction 验证
------------------------
Recall that a transaction is only valid if it is digitally signed by all required signers. However, even if a
transaction gathers all the required signatures, it is only valid if it is also **contractually valid**.

一个 transaction 仅仅当被所有要求的签名方提供了签名之后才会被认为是有效的。但是，除了获得到所有人的签名之后，还必须要满足 **合约有效** 才会被最终认为有效。

**Contract validity** is defined as follows:

* Each transaction state specifies a *contract* type
* A *contract* takes a transaction as input, and states whether the transaction is considered valid based on the
  contract's rules
* A transaction is only valid if the contract of **every input state** and **every output state** considers it to be
  valid

**合约有效** 的定义包含以下几点：

* 每个 state 都指定了一个 *合约* 类别
* 一个 *合约* 将一个 transaction 作为输入，并且介于合约的规则来声明一个 transaction 是否被认为是有效的
* 一个 transaction 会在 **每个 input state** 和 **每个 output state** 中定义的合约都认为它是有效的情况下，才会被认为是有效的

We can picture this situation as follows:

我们可以用下图来描述这个关系：

.. image:: resources/tx-validation.png
   :scale: 25%
   :align: center

The contract code can be written in any JVM language, and has access to the full capabilities of the language,
including:

* Checking the number of inputs, outputs, commands, time-window, and/or attachments
* Checking the contents of any of these components
* Looping constructs, variable assignment, function calls, helper methods, etc.
* Grouping similar states to validate them as a group (e.g. imposing a rule on the combined value of all the cash
  states)

Contract code 可以用任何的 JVM 语言编写，并且具有该语言的所有能力，包括：

* 检查 inputs，outputs，commands 的数量，时间，和/或者 附件
* 检查这些组件中的任何一个的内容
* 循环一个结构，给变量赋值，调用方法，帮助函数等等
* 将一些类似的 states 分组来验证（比如对于所有的现金 state 的组合定义一个规则）

A transaction that is not contractually valid is not a valid proposal to update the ledger, and thus can never be
committed to the ledger. In this way, contracts impose rules on the evolution of states over time that are
independent of the willingness of the required signers to sign a given transaction.

一个 transaction 如果不是合约有效的话，是不会被视为一个对账本的有效更新，也就不可能被提交至账本。通过这种方式，合约会通过定义规则来不断地更新 states 的状态，并且每次更新必须要获得所有相关的签名。

The contract sandbox
--------------------
Transaction verification must be *deterministic* - a contract should either **always accept** or **always reject** a
given transaction. For example, transaction validity cannot depend on the time at which validation is conducted, or
the amount of information the peer running the contract holds. This is a necessary condition to ensure that all peers
on the network reach consensus regarding the validity of a given ledger update.

Transaction 验证必须是 *一个确定性的结果* - 一个 contract 必须或者 **总是接受**，或者 **总是拒绝** 一个给定的 transaction。比如 transaction 是否有效不能够取决于你在什么时间做的 verify 或者是基于某一方具有的信息量的多少来决定是有效的还是无效的。这是一个很重要的条件来确保网络上的相关节点能够在这个对账本的更新的操作达成共识。

Future versions of Corda will evaluate transactions in a strictly deterministic sandbox. The sandbox has a whitelist that
prevents the contract from importing libraries that could be a source of non-determinism. This includes libraries
that provide the current time, random number generators, libraries that provide filesystem access or networking
libraries, for example. Ultimately, the only information available to the contract when verifying the transaction is
the information included in the transaction itself.

未来版本的 Corda 将会以一个严格的确定性的 sandbox 来考量交易。这个 sandbox 有一个白名单（whitelist）能够用来防止引入一些会造成不确定性的外部库。这些库包括提供当前日期的，产生随机数的，提供访问系统文件的或者访问网络的。本质上来讲，当验证一个 transaction 的时候，对于 contract 的信息只应该来自于 transaction 自身的信息。

Developers can pre-verify their CorDapps are determinsitic by linking their CorDapps against the deterministic modules
(see the :doc:`Deterministic Corda Modules <deterministic-modules>`).

开发者可以通过将他们的 CorDapps 和确定模块相连接的方式来预先验证他们的 CorDaps 是结果确定的（查看:doc:`确定性的 Corda 模块 <deterministic-modules>`）。

Contract 的局限性
--------------------
Since a contract has no access to information from the outside world, it can only check the transaction for internal
validity. It cannot check, for example, that the transaction is in accordance with what was originally agreed with the
counterparties.

因为 contract 没有办法访问到外部的信息，它只能检查 transaction 内部的有效性，比如它不能够检查确认当前这个 transaction 是不是已经同其他相关方达成了共识取得了其他方的确认。

Peers should therefore check the contents of a transaction before signing it, *even if the transaction is
contractually valid*, to see whether they agree with the proposed ledger update. A peer is under no obligation to
sign a transaction just because it is contractually valid. For example, they may be unwilling to take on a loan that
is too large, or may disagree on the amount of cash offered for an asset.

所以在各方提供最终的签名确认之前，各方应该对transaction 的内容进行检查来确定他们是否同意这个对账本的更新，*即使这个 transaction 是合约有效的*。任何一方是不会因为 transaction 是 contractually valid 就能够去提供签名。比如他们可能不愿意去提供一个巨额的借款，或者可能不会同意购买一个资产花费的钱的金额。

Oracles
-------
Sometimes, transaction validity will depend on some external piece of information, such as an exchange rate. In
these cases, an oracle is required. See :doc:`key-concepts-oracles` for further details.

有的时候 transaction validity 需要取决于一些外部的信息，比如兑换汇率。这种情况下就需要使用 oracle 了。查看 :doc:`key-concepts-oracles` 了解更多信息。

Legal prose
-----------

.. raw:: html

    <iframe src="https://player.vimeo.com/video/213879293" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
    <p></p>

Each contract also refers to a legal prose document that states the rules governing the evolution of the state over
time in a way that is compatible with traditional legal systems. This document can be relied upon in the case of
legal disputes.

每一个合约也会引用一个 legal prose 文档，这个文档中定义了合约中规定的内容，legal prose 也会被传统的法律系统所接受。这个文档会在发生法律纠纷的时候被用来进行判定依据。