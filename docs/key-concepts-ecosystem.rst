网络
===========

.. topic:: Summary

   * *A Corda network is made up of nodes running Corda and CorDapps*
   * *Communication between nodes is point-to-point, instead of relying on global broadcasts*
   * *Each node has a certificate mapping their network identity to a real-world legal identity*
   * *The network is permissioned, with access requiring a certificate from the network operator*

   * *一个 Corda 网络是由运行着 Corda 和 CorDapps 的节点构成的*
   * *不同节点间的沟通是点对点的，而不依赖于全局广播*
   * *每个节点都可以使用一个数字证书来将真实世界中的法律身份和网络身份相关联
   * *这个网络是一个需要许可的网络，需要从网络维护者那里申请一个数字证书来获得访问权限*

网络结构
-----------------
A Corda network is a peer-to-peer network of **nodes**. Each node runs the Corda software as well as Corda applications
known as **CorDapps**.

Corda 网络是一个 peer-to-peer 的由 **节点** 组成的网络，每个节点运行着 Corda 的软件以及被称为 CorDapps 的 Corda 应用程序。

.. image:: resources/network.png
   :scale: 25%
   :align: center

All communication between nodes is point-to-point and encrypted using transport-layer security. This means that data is
shared only on a need-to-know basis. There are **no global broadcasts**.

节点间的通信是点对点的并且使用 TLS 进行加密。这意味着数据是基于需要了解的基础来共享的。这里 **没有全局广播**。

身份
--------
Each node has a single well-known identity. The node's identity is used to represent the node in transactions, such as
when purchasing an asset.

每个节点具有一个众所周知的身份。节点的身份被用来在交易中代表这个节点，比如在进行一笔资产的交易的时候。

.. note:: These identities are distinct from the RPC user logins that are able to connect to the node via RPC.

.. note:: 这些身份跟能够通过 RPC 来跟节点进行连接的 RPC 用户登录的身份是不同的。

Each network has a **network map service** that maps each well-known node identity to an IP address. These IP
addresses are used for messaging between nodes.

每个网络具有一个 **network map service** 将每个众所周知的的节点身份同它的 IP 地址进行映射。这个 IP 地址就被用来在节点间进行通信。

Nodes can also generate confidential identities for individual transactions. The certificate chain linking a
confidential identity to a well-known node identity or real-world legal identity is only distributed on a need-to-know
basis. This ensures that even if an attacker gets access to an unencrypted transaction, they cannot identify the
transaction's participants without additional information if confidential identities are being used.

节点也可以为了个人的交易而生成一个保密的身份。一个保密身份关联到一个众所周知的身份或者真实世界中的法律身份的数字证书链只会按照需要知道的基础进行分发。这确保了即使一个黑客能够访问一笔未加密的交易，如果保密的身份被使用的话，如果没有额外的信息，他也无法识别出来交易的参与者。

网络的管理
------------------------
Corda networks are semi-private. To join a network, a node must obtain a certificate from the network operator. This
certificate maps a well-known node identity to:

Corda 网络是半私有化的。想要加入一个网络，一个节点必须要从网络维护者那里获得一个数字证书。这个证书会将一个众所周知的节点身份映射到：

* A real-world legal identity
* A public key

* 一个真实世界中的法律身份
* 一个公钥

The network operator enforces rules regarding the information that nodes must provide and the know-your-customer
processes they must undergo before being granted this certificate.

这个网络的维护者会强制在办法证书之前，关于节点要提供的信息规则必须被遵守，并且了解你的客户的流程必须被执行。