.. highlight:: kotlin

API: 合约约束
=========================

.. note:: Before reading this page, you should be familiar with the key concepts of :doc:`key-concepts-contracts`.

.. note:: 在阅读这里之前，你应该已经熟悉了核心概念 :doc:`key-concepts-contracts`。

.. contents::

*Contract constraints* solve two problems faced by any decentralised ledger that supports evolution of data and code:

1. Controlling and agreeing upon upgrades
2. Preventing attacks

*合约约束* 能够解决对于支持数据和代码更新的任何的去中心化的账本所遇到的两个问题：

1. 控制和同意更新
2. 防止攻击

Upgrades and security are intimately related because if an attacker can "upgrade" your data to a version of an app that gives them
a back door, they would be able to do things like print money or edit states in any way they want. That's why it's important for
participants of a state to agree on what kind of upgrades will be allowed.

升级和安全总是紧密地联系在一起，因为如果一个黑客能够把你的数据 “升级” 到某个版本来为他们提供一个后门的话，他们就能够去做诸如印钱或者按照任何他们想要的方式来变更 states。这就是为什么对于一个 state 的参与方，他们需要同意什么样的更新是被允许的变得尤为重要。

Every state on the ledger contains the fully qualified class name of a ``Contract`` implementation, and also a *constraint*.
This constraint specifies which versions of an application can be used to provide the named class, when the transaction is built.
New versions released after a transaction is signed and finalised won't affect prior transactions because the old code is attached
to it.

每个账本上的 state 包含了一个完全有效的一个 ``Contract`` 实现的类名，还包括一个 *约束*。这个约束在交易被构建的时候，指定了哪个版本的应用程序可以被用来提供这个已命名的类。在一笔交易被签名并且最终结束后产生的新版本是不会影响之前的交易的，因为旧的代码已经被附加在交易上了。

There are several types of constraint:

1. Hash constraint: exactly one version of the app can be used with this state.
2. Compatibility zone whitelisted (or CZ whitelisted) constraint: the compatibility zone operator lists the hashes of the versions that can be used with this contract class name.
3. Signature constraint: any version of the app signed by the given ``CompositeKey`` can be used.
4. Always accept constraint: any app can be used at all. This is insecure but convenient for testing.

这里有不同类型的约束：

1. Hash 约束：只有一个版本的应用程序可以同这个 state 一起使用
2. Compatibility zone 白名单约束：Compatibility zone 维护者会列出所有可以跟这个合约类名一起使用的版本的 hash
3. 签名约束：任何版本的由给定 ``CompositeKey`` 所签过名的应用程序可以被使用
4. 总是接受的约束：一个永远都可以被使用的应用程序。这个不是安全的，但是对于测试是方便的

The actual app version used is defined by the attachments on a transaction that consumes a state: the JAR containing the state and contract classes, and optionally
its dependencies, are all attached to the transaction. Other nodes will download these JARs from a node if they haven't seen them before,
so they can be used for verification. The ``TransactionBuilder`` will manage the details of constraints for you, by selecting both constraints
and attachments to ensure they line up correctly. Therefore you only need to have a basic understanding of this topic unless you are
doing something sophisticated.

真正被使用的应用程序的版本是由消费一个 state 的一笔交易中的附件来定义的：这个 JAR 包含了 state 和 contract 类，并且还可能包含他们的依赖，他们全部被附加在交易中。其他的节点如果之前没有见过这些 JAR 的话会从这个节点下载这些 JARs，所以他们可以被用来做验证。``TransactionBuilder`` 将会通过选择约束及附件来确保他们正确的相关联，以此来为你管理这些约束的详细。因此你只需要对这个话题有一个大概的理解就够了，除非你在做一些比较复杂的事情。

The best kind of constraint to use is the **signature constraint**. If you sign your application it will be used automatically.
We recommend signature constraints because they let you smoothly migrate existing data to new versions of your application.
Hash and zone whitelist constraints are left over from earlier Corda versions before signature constraints were
implemented. They make it harder to upgrade applications than when using signature constraints, so they're best avoided.
Signature constraints can specify flexible threshold policies, but if you use the automatic support then a state will
require the attached app to be signed by every key that the first attachment was signed by. Thus if the app that was used
to issue the states was signed by Alice and Bob, every transaction must use an attachment signed by Alice and Bob.

最好的约束类型是 **签名约束**。如果你为你的应用程序签了名，那么它会被自动地使用。我们建议使用签名约束，因为它能够让你平缓地将已有的数据迁移到新版本的应用程序上。Hash 和 zone 白名单签名仅仅会遗留在签名约束还没有实现的早期的 Corda 版本中。他们相比较于签名约束，会使升级你的应用程序变得更加困难，所以最好还是避免使用他们。签名约束能够制定灵活的限定策略，但是如果你使用的是自动支持的话，一个 state 将会要求这个附加的应用程序需要这个附件第一次被签名的所有的公钥提供签名。因此，如果一个初始一个 state 的应用程序被 Alice 和 Bob 签过名了，那么每一笔交易都必须要使用被 Alice 和 Bob 签过名的附件。

**Constraint propagation.** Constraints are picked when a state is created for the first time in an issuance transaction. Once created,
the constraint used by equivalent output states (i.e. output states that use the same contract class name) must match the
input state, so it can't be changed and you can't combine states with incompatible constraints together in the same transaction.

**约束的传递** 当一个 state 在一个初始的交易中被第一次创建的时候，约束会被使用。一旦被创建，被同等的 output states（比如使用相同的 contract 类型的 output states） 所使用的约束必须要跟 input state 匹配，所以它就不能被改动了，并且你也不能够在相同的交易中把不兼容的约束的 states 合并到一起。

.. _implicit_vs_explicit_upgrades:

**Implicit vs explicit.** Constraints are not the only way to manage upgrades to transactions. There are two ways of handling
upgrades to a smart contract in Corda:

1. *Implicit:* By pre-authorising multiple implementations of the contract ahead of time, using constraints.
2. *Explicit:* By creating a special *contract upgrade transaction* and getting all participants of a state to sign it using the
   contract upgrade flows.

**隐式和显式** 约束并不是管理交易升级的唯一的方式。在 Corda 中处理智能合约升级有两种方式：

1. *隐式*：使用约束，通过提前预授权关于合约的多个不同的实现。
2. *显式*：使用合约升级 flows，通过创建一个特殊的 *合约升级交易* 并且得到一个 state 的所有参与方的签名

This article focuses on the first approach. To learn about the second please see :doc:`upgrading-cordapps`.

这篇文章主要讨论第一种方式。查看 :doc:`upgrading-cordapps` 来了解第二种方式。

The advantage of pre-authorising upgrades using constraints is that you don't need the heavyweight process of creating
upgrade transactions for every state on the ledger. The disadvantage is that you place more faith in third parties,
who could potentially change the app in ways you did not expect or agree with. The advantage of using the explicit
upgrade approach is that you can upgrade states regardless of their constraint, including in cases where you didn't
anticipate a need to do so. But it requires everyone to sign, requires everyone to manually authorise the upgrade,
consumes notary and ledger resources, and is just in general more complex.

使用约束提前授权升级更新的优势是你不需要走一个非常繁琐的流程来为账本中的每个 state 创建一个升级的 transaction。缺点是你将更多的信任交给了合约开发的第三方，他们可能会按照你不期望的或者不同意的方式来改变这个 application。使用显式更新的好处是你可以不用去考虑他们的约束而去升级 states，也包括你不想参与一次升级的情况。但是这个流程需要每个人为其提供签名，需要每个人手动地为一次升级授权，消耗 notary 和账本资源，大体上来说是更加复杂的做法。

.. _implicit_constraint_types:

Contract/State Agreement
------------------------

Starting with Corda 4, a ``ContractState`` must explicitly indicate which ``Contract`` it belongs to. When a transaction is
verified, the contract bundled with each state in the transaction must be its "owning" contract, otherwise we cannot guarantee that
the transition of the ``ContractState`` will be verified against the business rules that should apply to it.

从 Corda 4.0 开始，一个 ``ContractState`` 必须要显式地说明它属于哪一个 ``Contract``。当一笔交易被验证的时候，contract 绑定的交易中的每个 state 必须要有它们 “自己的” contract，否则我们不能保证 ``ContractState`` 的交换能够按照它应该被使用的业务规则来验证。

There are two mechanisms for indicating ownership. One is to annotate the ``ContractState`` with the ``BelongsToContract`` annotation,
indicating the ``Contract`` class to which it is tied:

这有两种表明所有权的机制。一种是向 ``ContractState`` 添加 ``BelongsToContract`` 的注解，说明它关联的是哪个 ``Contract``：

.. sourcecode:: java

    @BelongToContract(MyContract.class)
    public class MyState implements ContractState {
        // implementation goes here
    }


.. sourcecode:: kotlin

    @BelongsToContract(MyContract::class)
    data class MyState(val value: Int) : ContractState {
        // implementation goes here
    }


The other is to define the ``ContractState`` class as an inner class of the ``Contract`` class

另外一种方式是在 ``Contract`` 类的内部定义 ``ContractState`` 类

.. sourcecode:: java

    public class MyContract implements Contract {
    
        public static class MyState implements ContractState {
            // state implementation goes here
        }

        // contract implementation goes here
    }


.. sourcecode:: kotlin

    class MyContract : Contract {
        data class MyState(val value: Int) : ContractState
    }
    

If a ``ContractState``'s owning ``Contract`` cannot be identified by either of these mechanisms, and the ``targetVersion`` of the
CorDapp is 4 or greater, then transaction verification will fail with a ``TransactionRequiredContractUnspecifiedException``. If
the owning ``Contract`` *can* be identified, but the ``ContractState`` has been bundled with a different contract, then
transaction verification will fail with a ``TransactionContractConflictException``.

如果一个 ``ContractState`` 所关联的 ``Contract`` 不能够通过这两种机制被识别出来，并且 CorDapp 的 ``targetVersion`` 是 4 或者更高的话，那么交易的验证就会失败，带有一个 ``TransactionRequiredContractUnspecifiedException``。如果所关联的 ``Contract`` *能够* 被识别出来，但是 ``ContractState`` 已经被绑定到一个不同的 contract 的话，那么交易的验证会失败，带有一个 ``TransactionContractConflictException``。

.. _contract_downgrade_rule_ref:

带有签名约束的应用版本
-----------------------------------------

Signed apps require a version number to be provided, see :doc:`versioning`. You can't import two different
JARs that claim to be the same version, provide the same contract classes and which are both signed. At runtime
the node will throw a ``DuplicateContractClassException`` exception if this condition is violated.

被签过名的应用需要提供一个版本编号，查看 :doc:`versioning`。你不能够引用使用相同版本的不同的 JARs，提供相同的 contract 类并且他们都已经被签名了。如果这个条件没有满足的话，在运行时，节点会抛出 ``DuplicateContractClassException``。

当时用 HashAttachmentConstraint 的问题
----------------------------------------------

When setting up a new network, it is possible to encounter errors when states are issued with the ``HashAttachmentConstraint``,
but not all nodes have that same version of the CorDapp installed locally.

当设置一个新的网络的时候，当 states 是由 ``HashAttachmentConstraint`` 来初始出来的话，是可能会遇到错误的，但是并不是所有的节点都在本地安装了相同版本的 CorDapp。

In this case, flows will fail with a ``ContractConstraintRejection``, and the failed flow will be sent to the flow hospital.
From there it's suspended waiting to be retried on node restart.
This gives the node operator the opportunity to recover from those errors, which in the case of constraint violations means
adding the right cordapp jar to the ``cordapps`` folder.

在这种情况下，flows 会失败并返回 ``ContractConstraintRejection``，失败的 flow 会被发送到 flow 意愿。在那里，它会被挂起并等待节点重启的时候被重试。这就给了节点的维护者机会来解决这些错误，如果是约束冲突的话，那么可以把正确的 CorDapp JAR 添加到 ``cordapps`` 文件夹。

.. _relax_hash_constraints_checking_ref:

在私有网络中的 Hash 约束的 states
-------------------------------------------

Where private networks started life using CorDapps with hash constrained states, we have introduced a mechanism to relax the checking of
these hash constrained states when upgrading to signed CorDapps using signature constraints.

当使用带有 hash 约束的 states 的 CorDapps 开始一个私有网络的时候，当升级使用签名约束的签过名的 CorDapps 的时候，我们引入了一个机制来把这些 hash 约束的 states 的检查变得更轻松。

The Java system property ``-Dnet.corda.node.disableHashConstraints="true"`` may be set to relax the hash constraint checking behaviour.

可以通过设置 Java 的系统属性 ``-Dnet.corda.node.disableHashConstraints="true"`` 来把检查 hash 约束的行为变得简单。

This mode should only be used upon "out of band" agreement by all participants in a network.

这个模式应该仅仅在一个网络中的所有参与者都同意的情况下才被使用。

Please also beware that this flag should remain enabled until every hash constrained state is exited from the ledger.

也要注意这个标记应该保持开启，知道每个 hash 约束的 state 都已经从账本上消除掉。

CorDapps 作为附件
-----------------------

CorDapp JARs (see :doc:`cordapp-overview`) that contain classes implementing the ``Contract`` interface are automatically
loaded into the ``AttachmentStorage`` of a node, and made available as ``ContractAttachments``.

包含实现了 ``Contract`` 接口的类的 CorDapps JARs 文件（查看 :doc:`cordapp-overview`）会被自动加载到一个节点的 ``AttachmentStorage``，并且作为 ``ContractAttachments`` 变得可用。

They are retrievable by hash using ``AttachmentStorage.openAttachment``. These JARs can either be installed on the
node or will be automatically fetched over the network when receiving a transaction.

通过使用 ``AttachmentStorage.openAttachment`` 能够根据 hash 把他们取回来。这些 JARs 能够被安装在节点上，或者在收到一个交易的时候被自动在网络上获取回来。

.. warning:: The obvious way to write a CorDapp is to put all you states, contracts, flows and support code into a single
   Java module. This will work but it will effectively publish your entire app onto the ledger. That has two problems:
   (1) it is inefficient, and (2) it means changes to your flows or other parts of the app will be seen by the ledger
   as a "new app", which may end up requiring essentially unnecessary upgrade procedures. It's better to split your
   app into multiple modules: one which contains just states, contracts and core data types. And another which contains
   the rest of the app. See :ref:`cordapp-structure`.

.. warning:: 一种简单的编写一个 CorDapp 的方式是将所有的 states，contracts，flows 和支持的代码都放在同一个 Java module 中。这个可以工作但是它也会将你整个 app 发布到账本上去。这会有两个问题：(1) 它不是有效率的，并且(2) 它意味着对于 flows 或者 app 其他部分的改动会在账本中作为一个“新 app”被看到，这个可能会以要求一个没有必要的升级流程而终止。将你的 app 分别放到多个 modules 中是一个更好的方式：一个 module 仅仅包含 states，contracts 和核心的数据类型。另一个 module 包含 app 剩下的部分。查看 :ref:`cordapp-structure`。


约束传递
-----------------------

As was mentioned above, the ``TransactionBuilder`` API gives the CorDapp developer or even malicious node owner the possibility
to construct output states with a constraint of his choosing.

向上边讲到的，``TransactionBuilder`` API 为 CorDapp 开发者以及不同的节点所有者一个可能性来使用他们选择的约束来构建 output states。

For the ledger to remain in a consistent state, the expected behavior is for output state to inherit the constraints of input states.
This guarantees that for example, a transaction can't output a state with the ``AlwaysAcceptAttachmentConstraint`` when the
corresponding input state was the ``SignatureAttachmentConstraint``. Translated, this means that if this rule is enforced, it ensures
that the output state will be spent under similar conditions as it was created.

为了使账本能够保持在一个一致的 state，期望的行为是对于 output state，应该继承 input states 的约束。这个能够保证比如，当对应的 input state 是 ``SignatureAttachmentConstraint`` 的时候，一个交易是不能够产生一个 ``AlwaysAcceptAttachmentConstraint`` 的 output state 的。也就是说，如果这个规则被强制，它就能够确保 output state 将会按照它被创建的时候相同的条件来被消费掉。

Before version 4, the constraint propagation logic was expected to be enforced in the contract verify code, as it has access to the entire Transaction.

在 4.0 版本之前，约束的传递逻辑是在 contract verify 代码中被强制执行的，因为它能够访问整个交易。

Starting with version 4 of Corda the constraint propagation logic has been implemented and enforced directly by the platform,
unless disabled by putting ``@NoConstraintPropagation`` on the ``Contract`` class which reverts to the previous behavior of expecting
apps to do this.

从 4.0 版本开始，约束的传递逻辑被平台实现并且强制执行，除非通过将 ``@NoConstraintPropagation`` 添加到 ``Contract`` 类上来把它变为无效，这就像恢复到了以前所期待的那样的行为。

For contracts that are not annotated with ``@NoConstraintPropagation``, the platform implements a fairly simple constraint transition policy
to ensure security and also allow the possibility to transition to the new ``SignatureAttachmentConstraint``.

对于没有 ``@NoConstraintPropagation`` 标签的 contracts，平台实现了一个非常简单的约束交易策略来确保安全并且也能够过度到新的 ``SignatureAttachmentConstraint``。

During transaction building the ``AutomaticPlaceholderConstraint`` for output states will be resolved and the best contract attachment versions
will be selected based on a variety of factors so that the above holds true. If it can't find attachments in storage or there are no
possible constraints, the ``TransactionBuilder`` will throw an exception.

当交易在为 output states 构建 ``AutomaticPlaceholderConstraint`` 的过程中，最适合的 contract 附件版本会根据不同的考虑被选择已达到上边所说的。如果它无法在存储中找到附件，或者这里没有可用的约束，``TransactionBuilder`` 将会抛出一个异常。

迁移约束到 Corda 4
--------------------------------

Please read :doc:`cordapp-constraint-migration` to understand how to consume and evolve pre-Corda 4 issued hash or CZ whitelisted constrained states
using a Corda 4 signed CorDapp (using signature constraints).

请阅读 :doc:`cordapp-constraint-migration` 来理解如何使用一个 Corda 4 签过名的 CorDapp（使用签名约束）来消费并且更新 4.0 之前版本的 Corda 生成的 hash 或者 CZ 白名单 约束过的 states。

Debugging
---------
If an attachment constraint cannot be resolved, a ``MissingContractAttachments`` exception is thrown. There are three common sources of
``MissingContractAttachments`` exceptions:

如果一个附件的约束无法解决的话，一个 ``MissingContractAttachments`` 的异常会被抛出。有三种常见的 ``MissingContractAttachments`` 异常的 source：

在测试中没有设置 CorDapp 包
*************************************

You are running a test and have not specified the CorDapp packages to scan.
When using ``MockNetwork`` ensure you have provided a package containing the contract class in ``MockNetworkParameters``. See :doc:`api-testing`.

你在运行一个测试并且没有指定要扫描的 CorDapp 的包。当使用 ``MockNetwork`` 的时候，确保你提供了一个在 ``MockNetworkParameters`` 中包含 contract 类的包。查看 :doc:`api-testing`。

Similarly package names need to be provided when testing using ``DriverDSl``. ``DriverParameters`` has a property ``cordappsForAllNodes`` (Kotlin)
or method ``withCordappsForAllNodes`` in Java. Pass the collection of ``TestCordapp`` created by utility method ``TestCordapp.findCordapp(String)``.

当使用 ``DriverDSl`` 进行测试的时候，类似的包名也需要被提供。``DriverParameters`` 有一个 ``cordappsForAllNodes`` 的属性 (Kotlin) 或者在 Java 中是 ``withCordappsForAllNodes`` 方法。将由 utility 方法 ``TestCordapp.findCordapp(String)`` 创建的 ``TestCordapp`` 集合传递过去。

Example of creation of two Cordapps with Finance App Flows and Finance App Contracts in Kotlin:

使用 Kotlin 创建两个带有 Finance App Flows 和 Finance App Contracts 的 CorDapps 的例子：

   .. sourcecode:: kotlin

        Driver.driver(DriverParameters(cordappsForAllNodes = listOf(TestCordapp.findCordapp("net.corda.finance.schemas"),
                TestCordapp.findCordapp("net.corda.finance.flows"))) {
            // Your test code goes here
        })

The same example in Java:

在 Java 中相同的例子：

   .. sourcecode:: java

        Driver.driver(new DriverParameters()
                .withCordappsForAllNodes(Arrays.asList(TestCordapp.findCordapp("net.corda.finance.schemas"),
                TestCordapp.findCordapp("net.corda.finance.flows"))), dsl -> {
            // Your test code goes here
        });


没有 CorDapp(s) 启动节点
**********************************

When running the Corda node ensure all CordDapp JARs are placed in ``cordapps`` directory of each node.
By default Gradle Cordform task ``deployNodes`` copies all JARs if CorDapps to deploy are specified.
See :doc:`generating-a-node` for detailed instructions.

当运行 Corda 节点的时候，要确保所有的 CorDapp JARs 被放在每个节点的 ``cordapps`` 路径下。默认地，如果将要部署的 CorDapps 被指定，Gradle Cordform 任务 ``deployNodes`` 会拷贝所有的 JARs。查看 :doc:`generating-a-node` 了解详细信息。

错误的 full-qualified 的 contract 名
***********************************

You are specifying the fully-qualified name of the contract incorrectly. For example, you've defined ``MyContract`` in
the package ``com.mycompany.myapp.contracts``, but the fully-qualified contract name you pass to the
``TransactionBuilder`` is ``com.mycompany.myapp.MyContract`` (instead of ``com.mycompany.myapp.contracts.MyContract``).

你没有为 contract 指定一个 full-qualified 的名字。例如，你在 ``com.mycompany.myapp.contracts`` 这个包中定义了 ``MyContract``，但是你给 ``TransactionBuilder`` 传递的 fully-qualified 合约名称是 ``com.mycompany.myapp.MyContract``（而不是 ``com.mycompany.myapp.contracts.MyContract``）。