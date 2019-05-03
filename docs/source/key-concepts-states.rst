States
======

.. topic:: 概要

   * *States represent on-ledger facts*
   * *States are evolved by marking the current state as historic and creating an updated state*
   * *Each node has a vault where it stores any relevant states to itself*

   * *State 代表的是存在账本上的事实*
   * *State 通过将原来的 State 变为历史记录然后添加一条新版本的 state 的方式来对 state 进行更新*
   * *每个节点都有一个 vault 来存储该节点所有相关的 States*

.. only:: htmlmode

   Video
   -----
   .. raw:: html
   
       <iframe src="https://player.vimeo.com/video/213812054" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
       <p></p>


概览
--------
A *state* is an immutable object representing a fact known by one or more Corda nodes at a specific point in time.
States can contain arbitrary data, allowing them to represent facts of any kind (e.g. stocks, bonds, loans, KYC data,
identity information...).

一个 *state* 代表的是一个不可修改的用来代表一个事实的对象，这个事实在某个时间点会被网络中的一个或者多个 Corda 节点所知道。States 可以包含任何的数据，这就允许了它可以代表任何的类型的事实（比如股票，借款，KYC 数据，身份数据等等）。

For example, the following state represents an IOU - an agreement that Alice owes Bob an amount X:

例如，下边的 state 代表了一个 IOU - 一个表示 Alice 欠 Bob 一定数量的钱的协议：

.. image:: resources/state.png
   :scale: 25%
   :align: center

Specifically, this state represents an IOU of £10 from Alice to Bob.

上边的 State 代表了一个从 Alice 到 Bob 的 £10 的 IOU。

As well as any information about the fact itself, the state also contains a reference to the *contract* that governs
the evolution of the state over time. We discuss contracts in :doc:`key-concepts-contracts`.

除了关于这个事实的信息本身，State 还包含了一个 *合约* 的引用，这个合约定义了 state 不断变化的原则。我们在 :doc:`key-concepts-contracts` 会讨论合约。

State 顺序
---------------
As states are immutable, they cannot be modified directly to reflect a change in the state of the world.

因为 states 是不可变更的，他们不能够被直接修改来反应 state 的变化。

Instead, the lifecycle of a shared fact over time is represented by a **state sequence**. When a state needs to be
updated, we create a new version of the state representing the new state of the world, and mark the existing state as
historic.

但是一个共享的事实的生命周期是可以通过 **state 顺序** 来体现。当一个 state 需要更新的时候，我们会创建一个代表新的 state 的新版本的 state，然后将原来的那个 state 标注为历史版本。

This sequence of state replacements gives us a full view of the evolution of the shared fact over time. We can
picture this situation as follows:

这种 state 有序的替换模式能够让我们看到关于一个共享事实的整个生命周期。我们可以用下图来表示这个

.. image:: resources/state-sequence.png
   :scale: 25%
   :align: center

Vault
---------
Each node on the network maintains a *vault* - a database where it tracks all the current and historic states that it
is aware of, and which it considers to be relevant to itself:

Corda 网络中的每一个节点都维护着一个 *vault* - 它是一个数据库，其中跟踪了所有 states 的当前以及历史的 states 数据，以及跟它有关的数据：

.. image:: resources/vault-simple.png
   :scale: 25%
   :align: center

We can think of the ledger from each node's point of view as the set of all the current (i.e. non-historic) states that
it is aware of.

我们可以从每个节点角度将账本理解为节点所关注的一系列当前状态的 states。

参考 states
----------------

Not all states need to be updated by the parties which use them. In the case of reference data, there is a common pattern
where one party creates reference data, which is then used (but not updated) by other parties. For this use-case, the
states containing reference data are referred to as "reference states". Syntactically, reference states are no different
to regular states. However, they are treated different by Corda transactions. See :doc:`key-concepts-transactions` for
more details.

并不是所有的 states 都需要被使用他们的节点来更新的。对于参数数据的情况，有一种常见的模式，一方创建了参考数据，这些数据会被其他方使用（但是不会被更新）。对于这种情况，states 中包含的参考数据被称为 “参考 states”。从语法上讲，参考 states 跟常规的 states 并没有什么不同。然而，Corda 交易会对他们进行不同的处理。浏览 :doc:`key-concepts-transactions` 了解更详细的的信息。