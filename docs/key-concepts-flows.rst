Flows
=====

.. topic:: 概要

   * *Flows automate the process of agreeing ledger updates*
   * *Communication between nodes only occurs in the context of these flows, and is point-to-point*
   * *Built-in flows are provided to automate common tasks*

   * *Flows 使同意更新账本的流程变得自动化*
   * *节点之间的沟通只能够在这些 Flows 的上下文中发生，并且是点对点的*
   * *内置的 flows 提供了常用的一些任务*

.. only:: htmlmode

    Video
    -----
    .. raw:: html
    
        <iframe src="https://player.vimeo.com/video/214046145" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
        <p></p>


动机
----------
Corda networks use point-to-point messaging instead of a global broadcast. This means that coordinating a ledger update
requires network participants to specify exactly what information needs to be sent, to which counterparties, and in
what order.

Corda 网络使用点对点的消息传输而不是全局广播。也就是说协调一个关于账本的更新需要网络上的参与者明确的指定需要发送什么信息，发送给谁，按照什么顺序发送。

Here is a visualisation of the process of agreeing a simple ledger update between Alice and Bob:

下边的图片动态地展示了对于一个简单在 Alice 和 Bob 之间的同意账本更新的流程。

.. only:: htmlmode

   .. image:: resources/flow.gif
      :scale: 25%
      :align: center


.. only:: pdfmode

   .. image:: resources/flow.png
      :scale: 25%
      :align: center


Flow 框架
------------------
Rather than having to specify these steps manually, Corda automates the process using *flows*. A flow is a sequence
of steps that tells a node how to achieve a specific ledger update, such as issuing an asset or settling a trade.

Corda 使用了 *flows* 来使上边的步骤变得自动化而不是手动地来处理这些步骤。一个 flow 是一系列有顺序的步骤来告诉一个节点应该如何实现一个指定的账本更新，比如发行一个资产或者结算一笔交易。

Here is the sequence of flow steps involved in the simple ledger update above:
下边是一个上边图片所描述的简单账本更新所涉及到的顺序的流程：

.. image:: resources/flow-sequence.png
   :scale: 25%
   :align: center

运行 flows
-------------
Once a given business process has been encapsulated in a flow and installed on the node as part of a CorDapp, the node's
owner can instruct the node to kick off this business process at any time using an RPC call. The flow abstracts all
the networking, I/O and concurrency issues away from the node owner.

一旦一个业务流程被封装在了一个 flow 中并且在节点中作为 CorDapp 的一部分被安装好之后，节点的所有者可以在任何时间通过使用一个 RPC call 来告诉节点开始这个业务流程。Flow 将所有的网络，I/O 和并发问题都抽象了出来，这个节点 owner 就不需要关注这些了。

All activity on the node occurs in the context of these flows. Unlike contracts, flows do not execute in a sandbox,
meaning that nodes can perform actions such as networking, I/O and use sources of randomness within the execution of a
flow.

节点上所有的动作都是发生在这些 flows 的上下文上的。与 contract 不同，flows 不是在 sandbox 里执行的，也就是说节点可以在执行一个 flow 的过程中来进行一些动作比如 networking，I/O 或者随机地使用一些资源。

节点内部通信
^^^^^^^^^^^^^^^^^^^^^^^^
Nodes communicate by passing messages between flows. Each node has zero or more flow classes that are registered to
respond to messages from a single other flow.

节点间是通过在不同的 flows间传递消息来进行沟通的。每个节点有0个或者多个注册的 flow classes 来回复另外个一个单独的 flow 的消息。

Suppose Alice is a node on the network and wishes to agree a ledger update with Bob, another network node. To
communicate with Bob, Alice must:

* Start a flow that Bob is registered to respond to
* Send Bob a message within the context of that flow
* Bob will start its registered counterparty flow

假设 Alice 是网络中的一个节点，并且她希望同 Bob（网络中的另一个节点） 一起同意一次账本的更新。为了跟 Bob 进行沟通， Alice 必须：

* 开始一个 Bob 已经注册过的 flow
* Alice 在这个 flow 的上下文中给 Bob 发送一个消息
* Bob 会启动它注册的这个 conterparty flow

Now that a connection is established, Alice and Bob can communicate to agree a ledger update by passing a series of
messages back and forth, as prescribed by the flow steps.

连接已经建立起来了，Alice 和 Bob 就可以像 flow 步骤中描述的那样来回地沟通关于一个更新账本的改动并且最终达成一致。

Subflows
^^^^^^^^
Flows can be composed by starting a flow as a subprocess in the context of another flow. The flow that is started as
a subprocess is known as a *subflow*. The parent flow will wait until the subflow returns.

Flows 可以通过在另外一个 flow 的上下文中开始一个新的 flow 座位一个子流程的方式被组成。作为子流程被启动的 Flow 被称为 *subflow*。父 flow 需要等待所有的 subflow 完成后才会继续运行。

Flow 类库
~~~~~~~~~~~~~~~~
Corda provides a library of flows to handle common tasks, meaning that developers do not have to redefine the
logic behind common processes such as:

* Notarising and recording a transaction
* Gathering signatures from counterparty nodes
* Verifying a chain of transactions

Corda 对于一些常规的任务都提供了一套代码库，所以开发者就不需要自己去定义这些常见流程背后的逻辑了，比如：

* 公正和记录一个 transaction
* 从相关节点搜集签名
* 确认一个交易链

Further information on the available built-in flows can be found in :doc:`api-flows`.

更多关于内置的 flows 的信息能够在 :doc:`api-flows` 中找到。

并发
-----------
The flow framework allows nodes to have many flows active at once. These flows may last days, across node restarts and even upgrades.

Flow 框架允许节点可以同时运行多个 flows。这些 flows 可能由于节点的重启甚至升级会持续几天。

This is achieved by serializing flows to disk whenever they enter a blocking state (e.g. when they're waiting on I/O
or a networking call). Instead of waiting for the flow to become unblocked, the node immediately starts work on any
other scheduled flows, only returning to the original flow at a later date.

这个可以通过在 flow 变成阻塞的状态的时候，将 flows 序列化到硬盘中的方式来实现（比如他们在等待 I/O 或者是网络的调用）。出现这种情况的时候，节点不会等待这个阻塞状态的 flow变成非阻塞的状态，而会立即运行其他的 flow，只会在稍后返回到原来这个阻塞的flow。
