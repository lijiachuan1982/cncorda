安全编码指南
========================

The platform does what it can to be secure by default and safe by design. Unfortunately the platform cannot
prevent every kind of security mistake. This document describes what to think about when writing applications
to block various kinds of attack. Whilst it may be tempting to just assume no reasonable counterparty would
attempt to subvert your trades using flow level attacks, relying on trust for software security makes it
harder to scale up your operations later when you might want to add counterparties quickly and without
extensive vetting.

Corda 平台的设计默认已经考虑到安全因素。但是不幸的是平台没办法避免每种安全的错误。这篇文章描述了当编写应用的时候，都要考虑哪些方面来阻止各种类型的攻击。我们假设对方会尝试着使用 flow 级别的攻击来破坏你的交易，当你可能要快速添加更多的合作伙伴并且没有过多的验证的时候，对于软件安全的依赖会让后来的维护工作变得很难。

Flows
-----

:doc:`flow-state-machines` are how your app communicates with other parties on the network. Therefore they
are the typical entry point for malicious data into your app and must be treated with care.

:doc:`flow-state-machines` 是你的应用如何同网络中的其他节点进行沟通的方式。因此他们是恶意数据进入到你的应用的一个典型的入口，所以必须要谨慎地对待。

The ``receive`` methods return data wrapped in the ``UntrustworthyData<T>`` marker type. This type doesn't add
any functionality, it's only there to remind you to properly validate everything that you get from the network.
Remember that the other side may *not* be running the code you provide to take part in the flow: they are
allowed to do anything! Things to watch out for:

* A transaction that doesn't match a partial transaction built or proposed earlier in the flow, for instance,
  if you propose to trade a cash state worth $100 for an asset, and the transaction to sign comes back from the
  other side, you must check that it points to the state you actually requested. Otherwise the attacker could
  get you to sign a transaction that spends a much larger state to you, if they know the ID of one!
* A transaction that isn't of the right type. There are two transaction types: general and notary change. If you
  are expecting one type but get the other you may find yourself signing a transaction that transfers your assets
  to the control of a hostile notary.
* Unexpected changes in any part of the states in a transaction. If you have access to all the needed data, you
  could re-run the builder logic and do a comparison of the resulting states to ensure that it's what you expected.
  For instance if the data needed to construct the next state is available to both parties, the function to
  calculate the transaction you want to mutually agree could be shared between both classes implementing both
  sides of the flow.

``receive`` 方法返回的数据会被包装在 ``UntrustworthyData<T>`` marker 类型中。这个类型并没有添加任何的方法，它被放在这仅仅是为了提醒你对于从网络中获得的任何信息都要正确的验证。记住，在另一端的节点可能并没有运行你提供给他的代码：他们被允许做任何事情！你需要注意的事情包括：

* 一个同部分的 transaction build 或者之前在 flow 中提出的 transaction 不匹配，比如你提出了一个对于某个资产（asset）价值 $100 现金 state 的交易，然后这个 transaction 从对方节点签了名被返回来，你必须要检查它确实只想了你真正请求的那个 state。否则的话如果黑客知道某个资产的 ID，那么他们会欺骗你为一个会花费你更多的 state 的 transaction 签名。
* 一个不是正确类型的 transaction。这里主要有两种 transaction 类型：通用类型和 notary 变更类型。如果你期望收到一种类型但是得到的是另外一种类型，那么你可能会发现自己为一个将自己的资产转移到了 恶意的一个 notary 的 transaction 提供了签名。
* 在一个 transaction 中的任何的 states 中存在非期望的变动。如果你对所有需要的数据都有访问权限的话，你可以重新运行 builder 逻辑，然后将结果 states 进行对比来确保它确实是你期望得到的。比如如果用来构建下一个 state 的数据对于双方都是可用的话，用来对你想要互相同意的 transaction 进行计算的方法可以在 flow 的两方被共享。

The theme should be clear: signing is a very sensitive operation, so you need to be sure you know what it is you
are about to sign, and that nothing has changed in the small print! Once you have provided your signature over a
transaction to a counterparty, there is no longer anything you can do to prevent them from committing it to the ledger.

思路应该很清晰了：提供签名是一个非常敏感的操作，所以你必须要确定你将要提供签名的是什么，并且在 small print 中没有什么被改动了！一旦你向对方在一个 transaction 中提供了签名，你就没法做任何事情来阻止这些提交到账本中了。

Contracts
---------

Contracts are arbitrary functions inside a JVM sandbox and therefore they have a lot of leeway to shoot themselves
in the foot. Things to watch out for:

* Changes in states that should not be allowed by the current state transition. You will want to check that no
  fields are changing except the intended fields!
* Accidentally catching and discarding exceptions that might be thrown by validation logic.
* Calling into other contracts via virtual methods if you don't know what those other contracts are or might do.

合约是在一个 JVM 沙盒（sandbox）中的任何的方法，因此 they have a lot of leeway to shoot themselves in the foot。需要关注的点包括：

* 在当前的 state 交换中是不允许 states 的变动的。你需要检查除了那些想要去改动的字段以外，是不应该有其他字段的变动的。
* 意外的捕获和忽略可能会由验证逻辑抛出来的异常。
* 如果你不知道其他的合约是什么或者会做什么的话，那么通过虚拟方法来调用其它的合约
