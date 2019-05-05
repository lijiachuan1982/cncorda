合约目录
==================

There are a number of contracts supplied with Corda, which cover both core functionality (such as cash on ledger) and
provide examples of how to model complex contracts (such as interest rate swaps). There is also a ``Dummy`` contract.
However it does not provide any meaningful functionality, and is intended purely for testing purposes.

在 Corda 中提供了很多类型的合约，既包括核心功能（比如现金和账本），也提供了应该如何构建复杂合约的例子（日股汇率交换）。还包括 ``Dummy`` 合约。然而这里并没有提供任何有意义的功能，所以这纯是为了测试的目的。

现金
----

The ``Cash`` contract's state objects represent an amount of some issued currency, owned by some party. Any currency
can be issued by any party, and it is up to the recipient to determine whether they trust the issuer. Generally nodes
are expected to have criteria (such as a whitelist) that issuers must fulfil for cash they issue to be accepted.

``Cash`` 合约的 states 对象代表了一些发行的货币和谁拥有这些货币。然和节点都可以发行任何的货币，所以这个取决于接收方来决定他们是否新人货币的发行方。总体来说，节点是应该具有一些审核条件（就像是白名单），货币发行方必须要满足这些条件他们发行的货币才会被接受。

Cash state objects implement the ``FungibleAsset`` interface, and can be used by the commercial paper and obligation
contracts as part of settlement of an outstanding debt. The contracts' verification functions require that cash state
objects of the correct value are received by the beneficiary as part of the settlement transaction.

现金的 state 对象实现了 ``FungibleAsset`` 接口，并且可以被商业票据（commercial paper）和债券（obligation）合约作为一个借款清算的一部分被使用。合约的校验方法要求，作为清算 transaction 的一部分，具有正确价值的现金 state 对象被收款人接收到了。

The cash contract supports issuing, moving and exiting (destroying) states. Note, however, that issuance cannot be part
of the same transaction as other cash commands, in order to minimise complexity in balance verification.

现金合约支持发行（issuing），转移（moving）和销毁（exiting）states。注意，发行的操作不能够跟其他现金命令放在同一个 transaction 中，以最小化余额验证的难度。

Cash shares a common superclass, ``OnLedgerAsset``, with the Commodity contract. This implements common behaviour of
assets which can be issued, moved and exited on chain, with the subclasses handling asset-specific data types and
behaviour.

现金合约同商品合约共享了一个通用的 superclass OnLedgerAsset。它实现了在区块链上可以被发行、转移和销毁的资产（assets）的常见行为，它的子类会处理特定资产数据类型和行为。

.. note:: Corda supports a pluggable cash selection algorithm by implementing the ``CashSelection`` interface.
          The default implementation uses an H2 specific query that can be overridden for different database providers.
          Please see ``CashSelectionH2Impl`` and its associated declaration in
          ``META-INF\services\net.corda.finance.contracts.asset.CashSelection``

.. note:: Corda 支持通过实现 ``CashSelection`` 接口的方式来支持可插拔（pluggable）的现金选择算法。默认的实现是使用一个特定的 H2 查询，这个查询对于不同的数据库提供商（database provider）都可以被重写。请查看 ``META-INF\services\net.corda.finance.contracts.asset.CashSelection`` 路径下的 ``CashSelectionH2Impl`` 和相关的声明。

商品
---------

The ``Commodity`` contract is an early stage example of a non-currency contract whose states implement the ``FungibleAsset``
interface. This is used as a proof of concept for non-cash obligations.

``Cmmodity`` 合约是一个非货币合约的早期阶段的例子，它的 states 实现了 ``FungibleAsset`` 接口。这个被用来作为对于非现金的债务的一个概念验证（proof of concept）。

商业票据
----------------

``CommercialPaper`` is a very simple obligation to pay an amount of cash at some future point in time (the maturity
date), and exists primarily as a simplified contract for use in tutorials. Commercial paper supports issuing, moving
and redeeming (settling) states. Unlike the full obligation contract it does not support locking the state so it cannot
be settled if the obligor defaults on payment, or netting of state objects. All commands are exclusive of the other
commercial paper commands. Use the ``Obligation`` contract for more advanced functionality.

``CommercialPaper`` 是在将来支付一定现金的一个很简单的债务，也是在教程中被使用的一个简化的合约。商业票据支持发行、转移和履约（结算） states。跟完全的债务合约不同，他不支持将 state 锁住，所以如果债务人拒绝支付或者 netting of state objects，它是不会被清算的。每个商业票据的命令都是独有的。使用 ``Obligation`` 合约来做一些更高级的功能。

利率交换
------------------

The Interest Rate Swap (IRS) contract is a bilateral contract to implement a vanilla fixed / floating same currency
interest rate swap. In general, an IRS allows two counterparties to modify their exposure from changes in the underlying
interest rate. They are often used as a hedging instrument, convert a fixed rate loan to a floating rate loan, vice
versa etc.

利率交换合约是一个双边的合约，实现一个 vanilla 固定 / 浮动相同的货币利率交换。大体上说，一个 IRS 允许了交易双方从对底层利率的改的东来改变他们的 exposure。 他们经常被用来作为套期工具（hedging instrument），将一个固定利率的贷款转换为一个浮动利率的贷款，或者做一个相反的操作。

See ":doc:`contract-irs`" for full details on the IRS contract.

查看 ":doc:`contract-irs`" 了解关于 IRS 合约的详细内容。

债务
----------

The obligation contract's state objects represent an obligation to provide some asset, which would generally be a
cash state object, but can be any contract state object fulfilling the ``FungibleAsset`` interface, including other
obligations. The obligation contract uses objects referred to as ``Terms`` to group commands and state objects together.
Terms are a subset of an obligation state object, including details of what should be paid, when, and to whom.

债务合约的 state 对象代表了一个需要提供某些资产的债务，通常会是一个现金 state 对象，但是也可以是任何满足 ``FungibleAsset`` 接口的合约 state 对象，包括其他类型的债务。债务合约使用的对象是作为条款（Terms）来将命令（commands）和 state 对象组合在一起。条款是一个债务 state 对象的子集，包括了什么需要被支付，什么时间以及支付给谁的详细信息。

Obligation state objects can be issued, moved and exited as with any fungible asset. The contract also supports state
object netting and lifecycle changes (marking the obligation that a state object represents as having defaulted, or
reverting it to the normal state after marking as having defaulted). The ``Net`` command cannot be included with any
other obligation commands in the same transaction, as it applies to state objects with different beneficiaries, and
as such applies across multiple terms.

债务 state 对象像其他任何的 fungible asset 一样可以被发行、转移和清除。合约还支持 state 对象 netting 和生命周期变动（marking the obligation that a state object represents as having defaulted, or reverting it to the normal state after marking as having defaulted）。``Net`` 命令不能和其他任何债务命令一同包含在同一个 transaction 中，因为它会被应用到不同受益人的 state 对象中，还因为这个会应用到不同的条款中。

All other obligation contract commands specify obligation terms (what is to be delivered, by whom and by when)
which are used as a grouping key for input/output states and commands. Issuance and lifecycle commands are mutually
exclusive of other commands (move/exit) which apply to the same obligation terms, but multiple commands can be present
in a single transaction if they apply to different terms. For example, a contract can have two different ``Issue``
commands as long as they apply to different terms, but could not have an ``Issue`` and a ``Net``, or an ``Issue`` and
``Move`` that apply to the same terms.

Netting of obligations supports close-out netting (which can be triggered by either obligor or beneficiary, but is
limited to bilateral netting), and payment netting (which requires signatures from all involved parties, but supports
multilateral netting).
