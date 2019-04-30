账本
==========

.. topic:: 概要

   * *The ledger is subjective from each peer's perspective*
   * *Two peers are always guaranteed to see the exact same version of any on-ledger facts they share*

   * *每个账本是针对于每一个节点的*
   * *对于账本上的共享事实，共享的两方（或多方）总是能够保证存在他们自己的账本中的事实是完全一致的*

.. only:: htmlmode

   Video
   -----
   .. raw:: html
   
       <iframe src="https://player.vimeo.com/video/213812040" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
       <p></p>


概览
--------
In Corda, there is **no single central store of data**. Instead, each node maintains a separate database of known
facts. As a result, each peer only sees a subset of facts on the ledger, and no peer is aware of the ledger in its
entirety.

在 Corda 中是 **没有唯一的中心化存储的数据** 的。相反，每个节点维护这一个独立的数据库，其中包含了所知道的事实。所以每个 peer 只能够看到账本中的事实中的一部分，没有节点能够知道所有的内容。

For example, imagine a network with five nodes, where each coloured circle represents a shared fact:

例如，设想一个网络中有五个节点，每一个彩色的圆圈代表了一个共享的事实

.. image:: resources/ledger-venn.png
   :scale: 25%
   :align: center

We can see that although Carl, Demi and Ed are aware of shared fact 3, **Alice and Bob are not**.

我们可以看到，尽管 Carl，Demi 和 Ed 了解共享的事实 3，但是 **Alice 和 Bob 是不知道的**。

Equally importantly, Corda guarantees that whenever one of these facts is shared by multiple nodes on the network, it evolves
in lockstep in the database of every node that is aware of it:

同样重要的是，Corda 确保了一旦这些事实中的一个被网络中的多个节点间共享了的话，网络中的所有知道这个事实的节点的数据库会同时被更新：

.. image:: resources/ledger-table.png
   :scale: 25%
   :align: center

For example, Alice and Bob will both see the **exact same version** of shared facts 1 and 7.

例如， Alice 和 Bob 将会都能够看到 **完全一致版本** 的共享事实 1 和 7。