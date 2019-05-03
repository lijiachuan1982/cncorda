.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

API: Identity
=============

.. contents::

Party
-----
Parties on the network are represented using the ``AbstractParty`` class. There are two types of ``AbstractParty``:

* ``Party``, identified by a ``PublicKey`` and a ``CordaX500Name``
* ``AnonymousParty``, identified by a ``PublicKey`` only

Corda 网络中的 parties 是通过使用 ``AbstractParty`` 类来代表的。主要有两种类型的 ``AbstractParty``：

* ``Party``，通过一个 ``PublicKey`` 和 ``CordaX500Name`` 来识别
* ``AnonymousParty``，只能通过一个 ``PublicKey`` 来识别

Using ``AnonymousParty`` to identify parties in states and commands prevents nodes from learning the identities
of the parties involved in a transaction when they verify the transaction's dependency chain. When preserving the
anonymity of each party is not required (e.g. for internal processing), ``Party`` can be used instead.

使用 ``AnonymousParty`` 在 states 和 commands 中识别 parties 可以避免节点在确认 transaction 的依赖链（dependency chain）的时候知道一个 transaction 涉及到的 parties 的身份。为每一个 party 保留匿名性不是必须的（比如对于一个内部的流程），``Party`` 可以被使用。

The identity service allows flows to resolve ``AnonymousParty`` to ``Party``, but only if the anonymous party's
identity has already been registered with the node (typically handled by ``SwapIdentitiesFlow`` or
``IdentitySyncFlow``, discussed below).

Identity service 允许 flows 将 ``AnonymousParty`` 变为 ``Party``，但是仅仅当这个匿名的 party 的 identity 已经在这个节点中注册过了（通常是被下边要介绍的 ``SwapIdentitiesFlow`` 或者 ``IdentitySyncFlow`` 来处理的）。

Party names use the ``CordaX500Name`` data class, which enforces the structure of names within Corda, as well as
ensuring a consistent rendering of the names in plain text.

Party 的名字使用的是 ``CordaX500Name`` 数据类，这个强制了在 Corda 中的名字的结构，同时也确保了当名字转变为纯文本格式时的一致性。

Support for both ``Party`` and ``AnonymousParty`` classes in Corda enables sophisticated selective disclosure of
identity information. For example, it is possible to construct a transaction using an ``AnonymousParty`` (so nobody can
learn of your involvement by inspection of the transaction), yet prove to specific counterparts that this
``AnonymousParty`` actually corresponds to your well-known identity. This is achieved using the
``PartyAndCertificate`` data class, which contains the X.509 certificate path proving that a given ``AnonymousParty``
corresponds to a given ``Party``. Each ``PartyAndCertificate`` can be propagated to counterparties on a need-to-know
basis.

在 Corda 中同时支持 ``Party`` 和 ``AnonymousParty`` 使精确地有选择地暴露身份信息成为可能。例如，可以使用一个 ``AnonymousParty`` 来构建一个 transaction（所以没有人能够通过拦截这个 transaction 来知道你已经加入了这个 transaction），但是你还是可以指定对于哪个 counterparts 这个 ``AnonymousParty`` 实际对应的是哪个 well-known 的身份。这是通过使用 ``PartyAndCertificate`` 数据类来实现的，其包含了 X.509 证书的路径，可以用来证明一个给定的 ``AnonymousParty`` 对应的是哪个给定的 ``Party``。每个 ``PartyAndCertificate`` 可以基于 need-to-know 的原则发送给相关的 counterparties。

The ``PartyAndCertificate`` class is also used by the network map service to represent well-known identities, with the
certificate path proving the certificate was issued by the doorman service.

``PartyAndCertificate`` 类同样也会被 network map service 使用来代表 well-known identities，包括证书的路径，以此来证明证书是由 doorman service 发行的。

.. _confidential_identities_ref:

Confidential identities
-----------------------

.. warning:: The ``confidential-identities`` module is still not stabilised, so this API may change in future releases.
   See :doc:`corda-api`.

.. warning:: ``confidential-identities`` 模块目前还不是稳定的，所以这个 API 可能在以后会改动。查看 :doc:`corda-api`。

Confidential identities are key pairs where the corresponding X.509 certificate (and path) are not made public, so that
parties who are not involved in the transaction cannot identify the owner. They are owned by a well-known identity,
which must sign the X.509 certificate. Before constructing a new transaction the involved parties must generate and
exchange new confidential identities, a process which is managed using ``SwapIdentitiesFlow`` (discussed below). The
public keys of these confidential identities are then used when generating output states and commands for the
transaction.

Confidential identities 是一个秘钥对，所对应的 X.509 证书（和路径）并不是公开的，所以 transaction 中没有引入的 parties 是没法识别出这个代表的是谁。这个秘钥对是由一个 well-known identity 持有的，它必须要为 X.509 证书签名。当构建一个新的 transaction 之前，被引入的 parties 必须生成并交换新的 confidential identities，这个过程通过使用 ``SwapIdentitiesFlow`` 来进行管理。当为 transaction 生成 output states 和 commands 的时候，这些 confidential identities 的公钥到时会被使用。

Where using outputs from a previous transaction in a new transaction, counterparties may need to know who the involved
parties are. One example is the ``TwoPartyTradeFlow``, where an existing asset is exchanged for cash. If confidential
identities are being used, the buyer will want to ensure that the asset being transferred is owned by the seller, and
the seller will likewise want to ensure that the cash being transferred is owned by the buyer. Verifying this requires
both nodes to have a copy of the confidential identities for the asset and cash input states. ``IdentitySyncFlow``
manages this process. It takes as inputs a transaction and a counterparty, and for every confidential identity involved
in that transaction for which the calling node holds the certificate path, it sends this certificate path to the
counterparty.

当在一个新的 transaction 中使用前一个 transaction 产生的 outputs 的时候，counterparties 可能需要知道都哪些 parties 被引入了。一个例子就是 ``TwoPartyTradeFlow``，其中将一个已经存在的 asset 交换为现金。如果 confidential identities 被使用的话，购买者需要确保要交换的这个 asset 确实是由出售方所有的，出售方也可能想要确保要交换的这笔现金确实是由购买方所有的。为了确认这些，两个节点都需要留有这个 asset 和现金的 input state 的 confidential identities 的副本。``IdentitySyncFlow`` 管理了这个过程。它带有一个 transaction 的 inputs 和一个 counterparty，并且针对于每一个该 transaction 所引入的 confidential identity，调用的节点持有相关的证书的路径，那么调用节点需要将这个证书路径发送给 counterparty。

SwapIdentitiesFlow
~~~~~~~~~~~~~~~~~~
``SwapIdentitiesFlow`` is typically run as a subflow of another flow. It takes as its sole constructor argument the
counterparty we want to exchange confidential identities with. It returns a mapping from the identities of the caller
and the counterparty to their new confidential identities. In the future, this flow will be extended to handle swapping
identities with multiple parties at once.

``SwapIdentitiesFlow`` 通常会作为一个 flow 的一个 subflow 来运行。它只带有唯一的一个构造参数那就是我们想要交换 confidential identities 的 counterparty。它会返回一个调用方的 identities 和对应于他们新的 confidential identities 的 counterparty 的 mapping。在未来，这个 flow 会被执行用来处理同时与多个 parties 之间交换 identities。

You can see an example of using ``SwapIdentitiesFlow`` in ``TwoPartyDealFlow.kt``:

你可以在 ``TwoPartyDealFlow.kt`` 中查看 ``SwapIdentitiesFlow`` 的例子：

.. container:: codeset

    .. literalinclude:: ../../finance/workflows/src/main/kotlin/net/corda/finance/flows/TwoPartyDealFlow.kt
        :language: kotlin
        :start-after: DOCSTART 2
        :end-before: DOCEND 2
        :dedent: 8

``SwapIdentitiesFlow`` goes through the following key steps:

1. Generate a new confidential identity from our well-known identity
2. Create a ``CertificateOwnershipAssertion`` object containing the new confidential identity (X500 name, public key)
3. Sign this object with the confidential identity's private key
4. Send the confidential identity and aforementioned signature to counterparties, while receiving theirs
5. Verify the signatures to ensure that identities were generated by the involved set of parties
6. Verify the confidential identities are owned by the expected well known identities
7. Store the confidential identities and return them to the calling flow

``SwapIdentitiesFlow`` 会经过以下几个主要步骤：

1. 从我们的 well-known identity 生成一个新的 confidential identity
2. 创建一个 ``CertificateOwnershipAssertion`` 对象，该对象包含这个新生成的 confidential identity （X500 名字，公钥）
3. 使用 confidential identity 的私钥来给这个对象提供签名
4. 将 confidential identity 和前面所说的签名发送给所有的 counterparties，同时接收他们的信息
5. 确认签名来确保这些 identities 是由引入的这一些列 parties 所生成的
6. 确认这些 confidential identities 是由期望的 well-known identities 所有
7. 存储这些 confidential identities 并且将他们返回给调用 flow

This ensures not only that the confidential identity X.509 certificates are signed by the correct well-known
identities, but also that the confidential identity private key is held by the counterparty, and that a party cannot
claim ownership of another party's confidential identities.

这样既确保了 confidential identity X.509 证书是由正确的 well-know identities 签过名的，又能够确保 confidential identity 的私钥是由 counterparty 持有的，并且一个 party 是不能够生成自己是另一个 party 的 confidential identities 的所有者。

IdentitySyncFlow
~~~~~~~~~~~~~~~~
When constructing a transaction whose input states reference confidential identities, it is common for counterparties
to require knowledge of which well-known identity each confidential identity maps to. ``IdentitySyncFlow`` handles this
process. You can see an example of its use in ``TwoPartyTradeFlow.kt``.

当构建一个 input states 引用了 confidential identities 的 transaction 的时候，通常会要求 counterpartieis 要知道每一个 confidential identity 应该对应于哪个 well-known identity。``IdentitySyncFlow`` 处理了这个流程。你可以在 ``TwoPartyTradeFlow.kt`` 中看到一个例子。

``IdentitySyncFlow`` is divided into two parts:

``IdentitySyncFlow`` 分为两部分：

* ``IdentitySyncFlow.Send``
* ``IdentitySyncFlow.Receive``

``IdentitySyncFlow.Send`` is invoked by the party initiating the identity synchronization:

IdentitySyncFlow.Send 是由初始这个 identity 同步的 party 来调用的：

.. container:: codeset

    .. literalinclude:: ../../finance/workflows/src/main/kotlin/net/corda/finance/flows/TwoPartyTradeFlow.kt
        :language: kotlin
        :start-after: DOCSTART 6
        :end-before: DOCEND 6
        :dedent: 12

The identity synchronization flow goes through the following key steps:

1. Extract participant identities from all input and output states and remove any well known identities. Required
   signers on commands are currently ignored as they are presumed to be included in the participants on states, or to
   be well-known identities of services (such as an oracle service)
2. For each counterparty node, send a list of the public keys of the confidential identities, and receive back a list
   of those the counterparty needs the certificate path for
3. Verify the requested list of identities contains only confidential identities in the offered list, and abort
   otherwise
4. Send the requested confidential identities as ``PartyAndCertificate`` instances to the counterparty

这个 identity 同步 flow 包括以下几个主要步骤：

1. 从所有的 input 和 output states 中获得参与者的 identities，并且删除任何 well-known identities。Commands 中要求的签名者当前会被忽略因为他们假定会被包含在 states 上的参与者里，或者会作为 well-known identities 服务（比如 oracle service）
2. 对于每个 counterparty 节点，发送一个 confidential identities 的公钥列表，并且接收 counterparty 需要证书的路径的列表
3. 确认在提供的列表中，这些被请求的 identities 列表仅包含了 confidential identities，其他的会被跳过
4. 将请求的 confidential identities 作为 ``PartyAndCertificate`` 实例发送给 counterparty

.. note:: ``IdentitySyncFlow`` works on a push basis. The initiating node can only send confidential identities it has
   the X.509 certificates for, and the remote nodes can only request confidential identities being offered (are
   referenced in the transaction passed to the initiating flow). There is no standard flow for nodes to collect
   confidential identities before assembling a transaction, and this is left for individual flows to manage if
   required.

.. note:: ``IdentitySyncFlow`` 是基于推的方式工作的。发起方节点只能发送它持有的 X.509 证书的 confidential identity，远程节点也仅仅能够请求被提供的 confidential identities（会在传递给初始 flow 中的 transaction 中被引用）。这没有一个标准的流程让节点在安装一个 transaction 之前搜集 confidential identities，并且如果需要的话，这个会被留给每个单独的 flows 来管理。

Meanwhile, ``IdentitySyncFlow.Receive`` is invoked by all the other (non-initiating) parties involved in the identity
synchronization process:

同时，``IdentitySyncFlow.Receive`` 会被这个 identity 同步流程引入的所有其他 parties（非发起方）来调用：

.. container:: codeset

    .. literalinclude:: ../../finance/workflows/src/main/kotlin/net/corda/finance/flows/TwoPartyTradeFlow.kt
        :language: kotlin
        :start-after: DOCSTART 07
        :end-before: DOCEND 07
        :dedent: 12

``IdentitySyncFlow`` will serve all confidential identities in the provided transaction, irrespective of well-known
identity. This is important for more complex transaction cases with 3+ parties, for example:

* Alice is building the transaction, and provides some input state *x* owned by a confidential identity of Alice
* Bob provides some input state *y* owned by a confidential identity of Bob
* Charlie provides some input state *z* owned by a confidential identity of Charlie

``IdentitySyncFlow`` 会在被提供的 transaction 中处理所有的 confidential identities，不管是不是 well-know identity。这个对于多于3个 parties 参与的复杂 transaction 很重要，比如：

* Alice 正在创建一个 transaction，并且提供了一个 input state *x*，这个 state x 是由 Alice 的 confidential identity 所有
* Bob 提供了由 Bob 的一个 confidential identity 所有的一些 input state *y*
* Charlie 提供了一些由 Charlie 的 confidential identity 所有 input state *z*

Alice may know all of the confidential identities ahead of time, but Bob not know about Charlie's and vice-versa.
The assembled transaction therefore has three input states *x*, *y* and *z*, for which only Alice possesses
certificates for all confidential identities. ``IdentitySyncFlow`` must send not just Alice's confidential identity but
also any other identities in the transaction to the Bob and Charlie.

Alice 可能提前就知道了所有的 confidential identities，但是 Bob 和 Charlie 不知道彼此的 confidential identity。这个被组装的 transaction 因此会包3个 input states *x*、*y* 和 *z*，并且只有 Alice 持有所有的 confidential identities 的证书。``IdentitySyncFlow`` 必须不仅要发送 Alice 的 confidential identity，还要将 transaction 中其他的 identities 给 Bob 和 Charlie。