CorDapp 样例
===============

There are two distinct sets of samples provided with Corda, one introducing new developers to how to write CorDapps, and
more complex worked examples of how solutions to a number of common designs could be implemented in a CorDapp.
The former can be found on `the Corda website <https://www.corda.net/samples/>`_. In particular, new developers
should start with the :doc:`example CorDapp <tutorial-cordapp>`.

Corda 提供了两个系列的样例，一个是向新的开发者介绍如何编写 CorDapps，以及更加复杂的关于如何把一些常用的设计实现在一个 CorDapp 的一些样例。前一种可以在 `Corda 网站 <https://www.corda.net/samples/>`_ 中找到。对于新的开发者而言，应该从 :doc:`CorDapp 样例 <tutorial-cordapp>` 开始。

The advanced samples are contained within the `samples/` folder of the Corda repository. The most generally useful of
these samples are:

1. The `trader-demo`, which shows a delivery-vs-payment atomic swap of commercial paper for cash
2. The `attachment-demo`, which demonstrates uploading attachments to nodes
3. The `bank-of-corda-demo`, which shows a node acting as an issuer of assets (the Bank of Corda) while remote client
   applications request issuance of some cash on behalf of a node called Big Corporation

更高级的样例是包含在 Corda repository 的 `samples/` 文件夹下的。这些样例的一些有帮助的部分包括：

1. `trader-demo`，显示了一个有关商业票据与现金的交付与支付的原子交换
2. `attachment-demo`，演示了如何向节点上传附件
3. `bank-of-corda-demo`，演示了一个座位一个资产发行方（Corda 银行）的节点，远程的客户端应用程序代表一个叫做 Big Corporation 的节点请求发行一些现金

Documentation on running the samples can be found inside the sample directories themselves, in the `README.md` file.

关于如何运行这些样例的文档在这些样子的目录中，在 `README.md` 文件里。

.. note:: If you would like to see flow activity on the nodes type in the node terminal ``flow watch``.

.. note:: 如果你想看到在节点上的 flow 动作，可以在 terminal 里输入 ``flow watch``。

Please report any bugs with the samples on `GitHub <https://github.com/corda/corda/issues>`_.

请在 `GitHub <https://github.com/corda/corda/issues>`_ 里反馈任何关于这些样例的 bugs。
