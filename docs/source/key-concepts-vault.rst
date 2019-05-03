Vault
=====

.. toctree::
   :maxdepth: 1

   soft-locking.rst

The vault contains data extracted from the ledger that is considered relevant to the node’s owner, stored in a relational model
that can be easily queried and worked with.

Vault 中存储的是跟节点的所有者相关的从账本上得到的数据，以关系方式模型存储以方便查询和使用。

The vault keeps track of both unconsumed and consumed states:

 * **Unconsumed** (or unspent) states represent fungible states available for spending (including spend-to-self transactions)
   and linear states available for evolution (eg. in response to a lifecycle event on a deal) or transfer to another party.
 * **Consumed** (or spent) states represent ledger immutable state for the purpose of transaction reporting, audit and archival, including the ability to perform joins with app-private data (like customer notes).

Vault 同时会追踪未消费掉的和已消费掉的 states：

 * **未消费掉的** （或者未使用的） states 代表了可以用来花费的 fungible states （包括 spend-to-self 交易）以及可以用来更新的 linear states （比如对于一笔交易的生命周期）或者从一方转换给另一方。
 * **已消费掉的** （或者已使用的） states 代表了为了交易报表、审计和归档的目的而在账本上存储的不可更改的 state，包括进行同 app-private 数据进行关联的能力（比如客户的 notes）。

By fungible we refer to assets of measurable quantity (eg. a cash currency, units of stock) which can be combined
together to represent a single ledger state.

对于这种可替代性，我们通常是指可计算数量的资产能够被合并在一起来代表一个单独的账本上的 state。

Like with a cryptocurrency wallet, the Corda vault can create transactions that send value (eg. transfer of state) to
someone else by combining fungible states and possibly adding a change output that makes the values balance (this
process is usually referred to as ‘coin selection’). Vault spending ensures that transactions respect the fungibility
rules in order to ensure that the issuer and reference data is preserved as the assets pass from hand to hand.

就像一个加密货币钱包，Corda 中的 vault 能够创建发送给其他人价值的交易（比如交换 state），这可以通过将 fungible states 进行合并，以及还可能添加一个变更 output 来调整余额（这个过程通常是指 “选择 coin”）。Vault 的花费确保了交易遵循了可互换性的规则，来确保发行方以及所参考的数据在资产交易的过程中被正确的存储。

A feature called **soft locking** provides the ability to automatically or explicitly reserve states to prevent
multiple transactions within the same node from trying to use the same output simultaneously. Whilst this scenario would
ultimately be detected by a notary, *soft locking* provides a mechanism of early detection for such unwarranted and
invalid scenarios. :doc:`soft-locking` describes this feature in detail.

一个称为 **soft locking** 的功能提供了自动或者显式地预定 states 而避免同一个节点尝试同时在多笔交易中使用相同的 output 的能力。这种情况最终会被一个 notary 发现，*soft locking* 提供了一种能够在早期就发现这种无根据和不正确的情况的机制。:doc:`soft-locking` 提供了更详细的的关于这个功能的描述。

.. note:: Basic 'coin selection' is currently implemented. Future work includes fungible state optimisation (splitting and
          merging of states in the background), and 'state re-issuance' (sending of states back to the
          issuer for re-issuance, thus pruning long transaction chains and improving privacy).

.. note:: 基础的 “选择 coin” 当前已经被实现了。未来的工作包括 fungible state 优化（在后台拆分和合并资产），以及 “再发行 state”（将 states 发送回给发行方进行再发行，因此可以修剪很长的交易链并且改进隐私性）。

There is also a facility for attaching descriptive textual notes against any transaction stored in the vault.

这里也有能够在任何存储在 vault 里的交易附加一个描述性的文本的功能。

The vault supports the management of data in both authoritative ("on-ledger") form and, where appropriate, shadow ("off-ledger") form:

* "On-ledger" data refers to distributed ledger state (cash, deals, trades) to which a firm is participant.
* "Off-ledger" data refers to a firm's internal reference, static and systems data.

Vault 支持管理需授权的（“on-ledger”）的数据，也可以管理 shadow（“off-ledger”）形式的数据：

* “On-ledger” 数据指的是分发的 ledger state （现金、交易），一个公司会参与其中。
* “Off-ledger” 数据指的是公司内部的参考数据、静态或者系统数据。

The following diagram illustrates the breakdown of the vault into sub-system components:

下边的图表展示了将 vault 拆分为子系统组件：

.. image:: resources/vault.png

Note the following:

* The vault "On Ledger" store tracks unconsumed state and is updated internally by the node upon recording of a transaction on the ledger
  (following successful smart contract verification and signature by all participants).
* The vault "Off Ledger" store refers to additional data added by the node owner subsequent to transaction recording.
* The vault performs fungible state spending (and in future, fungible state optimisation management including merging, splitting and re-issuance).
* Vault extensions represent additional custom plugin code a developer may write to query specific custom contract state attributes.
* Customer "Off Ledger" (private store) represents internal organisational data that may be joined with the vault data to perform additional reporting or processing.
* A :doc:`Vault Query API </api-vault-query>` is exposed to developers using standard Corda RPC and CorDapp plugin mechanisms.
* A vault update API is internally used by transaction recording flows.
* The vault database schemas are directly accessible via JDBC for customer joins and queries.

注意以下几点：

* Vault “On Ledger” 存储并追踪未消费掉的 state，并且在将一笔交易记录到账本的时候由节点内部进行更新（会按照成功执行了智能合约验证以及受到所有参与方的签名）
* Vault “Off Ledger” 存储了交易记录以外节点的所有者添加的额外的数据
* Vault 对 fungible state 进行了花费（并且在将来，fungible state 的优化管理包括合并、拆分以及再发行）。
* Vault 扩展代表了开发者可以编写的额外的自定义 plugin 代码，用来查询指定的自定义 contract state 属性。
* 客户的 “Off Ledger”（私有的存储）代表了内部的组织型数据，可能被用来跟 vault 数据进行关联来进行额外的报表或者处理。
* :doc:`Vault Query API </api-vault-query>` 可以使用标准的 Corda RPC 和 CorDapp plugin 机制暴露给开发者。
* Vault 更新 API 可以被交易记录的 flows 内部使用。
* Vault 数据库 schemas 可以通过 JDBC 和自定义的 joins 和查询进行直接地访问。

Section 8 of the `Technical white paper`_ describes features of the vault yet to be implemented including private key management, state splitting and merging, asset re-issuance and node event scheduling.

`Technical white paper`_ 的第 8 部分描述了 vault 尚未实现的一些功能，包括私钥的管理、state 的拆分及合并、资产的再发行以及节点事件的日程安排。

.. _`Technical white paper`: _static/corda-technical-whitepaper.pdf

