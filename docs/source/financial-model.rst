.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

金融模型
===============

Corda provides a large standard library of data types used in financial applications and contract state objects.
These provide a common language for states and contracts.

Corda 提供了大量的在金融应用中使用的数据类型的标准库和合约 state 对象。这个对于 states 和 合约提供了一个通用的语言。

数量
------

The `Amount <api/kotlin/corda/net.corda.core.contracts/-amount/index.html>`_ class is used to represent an amount of
some fungible asset. It is a generic class which wraps around a type used to define the underlying product, called
the *token*. For instance it can be the standard JDK type ``Currency``, or an ``Issued`` instance, or this can be
a more complex type such as an obligation contract issuance definition (which in turn contains a token definition
for whatever the obligation is to be settled in). Custom token types should implement ``TokenizableAssetInfo`` to allow the
``Amount`` conversion helpers ``fromDecimal`` and ``toDecimal`` to calculate the correct ``displayTokenSize``.

`Amount <api/kotlin/corda/net.corda.core.contracts/-amount/index.html>`_ 类被用来代表某种 fungible asset 的数量。这是一个 generic 类，被提升成为一种类型来定义底层的叫做 *token* 的产品。例如他可以是标准的 JDK 的 ``Currency`` 类型，或者是一个被 ``发行``（Issued）的实例，或者是一个更复杂的类型，比如作为一个债务合约发行的定义（这反过来包括了一个 token 类型的定义，这个 token 会用来结算这笔债务）。自定义的 token 类型应该实现 ``TokenizableAssetInfo`` 来允许 ``Amount`` 转换 helpers ``fromDecimal`` 和 ``toDecimal`` 来计算正确的 ``displayTokenSize``。

.. note:: Fungible is used here to mean that instances of an asset is interchangeable for any other identical instance,
          and that they can be split/merged. For example a £5 note can reasonably be exchanged for any other £5 note, and
          a £10 note can be exchanged for two £5 notes, or vice-versa.

.. note:: 这里的 fungible 代表着能够跟其他的唯一的实例进行互换，并且他们还可以被拆分/合并。例如一个 £5 的纸币（note）可以被合理地转换成任何其他的 £5 纸币，并且一个 £10 的纸币还可以被换成两个 £5 的纸币，反过来也可以。

Here are some examples:

下边是一些例子：

.. container:: codeset

   .. sourcecode:: kotlin

      // A quantity of some specific currency like pounds, euros, dollars etc.
      Amount<Currency>
      // A quantity of currency that is issued by a specific issuer, for instance central bank vs other bank dollars
      Amount<Issued<Currency>>
      // A quantity of a product governed by specific obligation terms
      Amount<Obligation.Terms<P>>

``Amount`` represents quantities as integers. You cannot use ``Amount`` to represent negative quantities,
or fractional quantities: if you wish to do this then you must use a different type, typically ``BigDecimal``.
For currencies the quantity represents pennies, cents, or whatever else is the smallest integer amount for that currency,
but for other assets it might mean anything e.g. 1000 tonnes of coal, or kilowatt-hours. The precise conversion ratio
to displayable amounts is via the ``displayTokenSize`` property, which is the ``BigDecimal`` numeric representation of
a single token as it would be written. ``Amount`` also defines methods to do overflow/underflow checked addition and subtraction
(these are operator overloads in Kotlin and can be used as regular methods from Java). More complex calculations should typically
be done in ``BigDecimal`` and converted back to ensure due consideration of rounding and to ensure token conservation.

``Amount`` 会以整数的形式来代表数量。你不可以用 ``Amount`` 来表示一个负的数量：如果你想要这么做的话那你比需要使用另外一种类型，通常应该是 ``BigDecimal``。对于货币来说，数量代表着便士（pennies），美分（cents）或者任何对于该货币是最小的整数数量，但是对于其他类型的资产，这可能会代表任何东西，比如 1000 公吨的煤，或者是千瓦小时。对于可显示的数量的精确转换率是要通过 ``displayTokenSize`` 属性来实现的，它是 ``BigDecimal`` 对于一个单独的 token 的数字展现。``Amount`` 也定义了方法来对 overflow/underflow checked 加法和减法（这个在 Kotlin 中是操作符重载 operator overloads，在 Java 追踪可以向常规方法那样被使用）。更复杂的计算通常应该使用 ``BigDecial`` 来做然后再转换回来，以确保进位（rounding）的考虑和确保 token 的转换。

``Issued`` refers to a product (which can be cash, a cash-like thing, assets, or generally anything else that's
quantifiable with integer quantities) and an associated ``PartyAndReference`` that describes the issuer of that contract.
An issued product typically follows a lifecycle which includes issuance, movement and exiting from the ledger (for example,
see the ``Cash`` contract and its associated *state* and *commands*)

``Issued`` 指的是一个产品（这个产品可以是现金、和现金类似的事物，资产或者任何可以用整数来代表的可数量化的事物）和一个相关的 ``PartyAndReference``，这个 reference 代表了该合约的发行者。一个被发行的产品通常会遵循一个生命周期，其中会包括发行，转移和从账本中消除（比如，查看 ``Cash`` 合约和它相关的 *state* 和 *commands*）

To represent movements of ``Amount`` tokens use the ``AmountTransfer`` type, which records the quantity and perspective
of a transfer. Positive values will indicate a movement of tokens from a ``source`` e.g. a ``Party``, or ``CompositeKey``
to a ``destination``. Negative values can be used to indicate a retrograde motion of tokens from ``destination``
to ``source``. ``AmountTransfer`` supports addition (as a Kotlin operator, or Java method) to provide netting
and aggregation of flows. The ``apply`` method can be used to process a list of attributed ``Amount`` objects in a
``List<SourceAndAmount>`` to carry out the actual transfer.

为了展现 ``Amount`` 的转移，tokens 使用 ``AmountTransfer`` 类型，它记录了数量和一个转移的目的。正数的数量值表示 tokens 从一个 ``source``（比如一个 ``Party`` 或者 ``CompositeKey``）转移到了 ``destination``。负数值可以用来表示从 ``destination`` 向 ``source`` 的一个颠倒的转移。``AmountTransfer`` 支持更多的（作为一个 Kotlin 的操作符，或者 Java 的方法）来提供 netting 和聚合 flows。``apply`` 方法可以用来处理在一个 ``List<SourceAndAmount>`` 中定义好的 ``Amount`` 对象列表来开始真正的转移。

金融 states
----------------
In additional to the common state types, a number of interfaces extend ``ContractState`` to model financial state such as:

  ``LinearState``
    A state which has a unique identifier beyond its StateRef and carries it through state transitions.
    Such a state cannot be duplicated, merged or split in a transaction: only continued or deleted. A linear state is
    useful when modelling an indivisible/non-fungible thing like a specific deal, or an asset that can't be
    split (like a rare piece of art).

  ``DealState``
    A LinearState representing an agreement between two or more parties. Intended to simplify implementing generic
    protocols that manipulate many agreement types.

  ``FungibleAsset``
    A FungibleAsset is intended to be used for contract states representing assets which are fungible, countable and issued by a
    specific party. States contain assets which are equivalent (such as cash of the same currency), so records of their existence
    can be merged or split as needed where the issuer is the same. For instance, dollars issued by the Fed are fungible and
    countable (in cents), barrels of West Texas crude are fungible and countable (oil from two small containers can be poured into one large
    container), shares of the same class in a specific company are fungible and countable, and so on.

在常见的 state 类型以外，还有很多扩展了 ``ContractState`` 的接口来作为金融 state，比如：

  ``LinearState``：这种类型的 states 在它的 StateRef 之上具有一个唯一的标识，这个标识在不同的 state 交换中一直存在。这种类型的 states 在一个 transaction 中不能够被复制、合并或者拆分：只能是或者继续使用或者删除。Linear state 对于生成一个不可分割（indivisible）/non-fungible 的事物（比如一笔指定的交易，或者一个不可拆分的资产，就像一个稀少的艺术品）是非常有用的。

  ``DealState``：LinearState 代表一个在两方或多方之间的一个协议。目的是要简化实现通用的协议（generic protocols），该协议可以处理很多种协议类型。

  ``FungibleAsset``：一个 FungibleAsset 表示的是可以替换的，可计算的并且可以被制定的一方发行的资产。States 包含了相同的资产（比如同币种的现金），所以如果他们的额发行者是同一个的话，他们可以被合并或者被拆分。比如政府发行的美元是可以替换（fungible）并计算（countable）的（用美分），桶装的西德克萨斯石油是可以替换并计算的（油可以从两个小的容器倒进一个大的容器中），某个公司的同种类别的股份是可替换并计算的，等等。

The following diagram illustrates the complete Contract State hierarchy:

下边的图表展示了整个 Contract State 的结构：

.. image:: resources/financialContractStateModel.png

Note there are currently two packages, a core library and a finance model specific library.
Developers may re-use or extend the Finance types directly or write their own by extending the base types from the Core library.

注意当前有两个包，一个核心类库和一个金融模型特殊的类库。开发者可以直接重用或者扩展金融类型，或者通过从核心类库中扩展基本的类型来编写自己的类型。
