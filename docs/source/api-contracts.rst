.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

API: Contracts
==============

.. note:: Before reading this page, you should be familiar with the key concepts of :doc:`key-concepts-contracts`.

.. note:: 当你阅读这里的时候，你应该已经熟悉了核心概念 :doc:`key-concepts-contracts`。

.. contents::

Contract
--------
Contracts are classes that implement the ``Contract`` interface. The ``Contract`` interface is defined as follows:

Contracts 都是实现了 ``Contract`` 接口的类。``Contract`` 接口定义如下：

.. container:: codeset

    .. literalinclude:: ../../core/src/main/kotlin/net/corda/core/contracts/Structures.kt
        :language: kotlin
        :start-after: DOCSTART 5
        :end-before: DOCEND 5

``Contract`` has a single method, ``verify``, which takes a ``LedgerTransaction`` as input and returns
nothing. This function is used to check whether a transaction proposal is valid, as follows:

* We gather together the contracts of each of the transaction's input and output states
* We call each contract's ``verify`` function, passing in the transaction as an input
* The proposal is only valid if none of the ``verify`` calls throw an exception

``Contract`` 只有一个 ``verify`` 方法，它会有一个 ``LedgerTransaction`` 作为 input 参数并且不会返回任何内容。这个方法被用来检验一个交易的提议是否有效，包括下边的验证：

* 我们会搜集这个交易的 input 和 output states 的 contract code
* 我们会调用每个 contract code 的 ``verify`` 方法，将 transaction 作为 input 传进去
* 这个更新账本的提议仅仅在所有的 verify 方法都没有返回 exception 的情况下才算是有效的

``verify`` is executed in a sandbox:

* It does not have access to the enclosing scope
* The libraries available to it are whitelisted to disallow:
  * Network access
  * I/O such as disk or database access
  * Sources of randomness such as the current time or random number generators

``verify`` 是在一个 sandbox 中执行的：

* 它没有权限访问内部的内容
* 针对于它可用的类库被放入白名单来不允许：
  * 网络访问
  * 硬盘或数据库访问的 I/O
  * 随机的资源比如当前的时间或者随机数生成器

This means that ``verify`` only has access to the properties defined on ``LedgerTransaction`` when deciding whether a
transaction is valid.

这意味着 ``verify`` 仅仅能够在决定一个交易是否有效的时候才能够访问 ``LedgerTransaction`` 中定义的属性。

Here are the two simplest ``verify`` functions:

最简单的两个 verify 方法：

* A  ``verify`` that **accepts** all possible transactions:
* 一个 ``verify`` **接受** 所有可能的 transactions：

.. container:: codeset

   .. sourcecode:: kotlin

        override fun verify(tx: LedgerTransaction) {
            // Always accepts!
        }

   .. sourcecode:: java

        @Override
        public void verify(LedgerTransaction tx) {
            // Always accepts!
        }

* A ``verify`` that **rejects** all possible transactions:
* 一个 ``verify`` **拒绝** 所有的 transactions：

.. container:: codeset

   .. sourcecode:: kotlin

        override fun verify(tx: LedgerTransaction) {
            throw IllegalArgumentException("Always rejects!")
        }

   .. sourcecode:: java

        @Override
        public void verify(LedgerTransaction tx) {
            throw new IllegalArgumentException("Always rejects!");
        }

LedgerTransaction
-----------------
The ``LedgerTransaction`` object passed into ``verify`` has the following properties:

被传入 verify 方法中的 ``LedgerTransaction`` 对象具有以下属性：

.. container:: codeset

    .. literalinclude:: ../../core/src/main/kotlin/net/corda/core/transactions/LedgerTransaction.kt
        :language: kotlin
        :start-after: DOCSTART 1
        :end-before: DOCEND 1

Where:

* ``inputs`` are the transaction's inputs as ``List<StateAndRef<ContractState>>``
* ``outputs`` are the transaction's outputs as ``List<TransactionState<ContractState>>``
* ``commands`` are the transaction's commands and associated signers, as ``List<CommandWithParties<CommandData>>``
* ``attachments`` are the transaction's attachments as ``List<Attachment>``
* ``notary`` is the transaction's notary. This must match the notary of all the inputs
* ``timeWindow`` defines the window during which the transaction can be notarised

其中：

* ``inputs`` 是类型为 ``List<StateAndRef<ContractState>>`` 的 transaction 的 inputs
* ``outputs`` 是类型为 ``List<TransactionState<ContractState>>`` 的 transaction 的 outputs
* ``commands`` 是类型为 ``List<CommandWithParties<CommandData>>`` 的 transaction 的 commands 和相关的签名者
* ``attachments`` 是类型为 ``List<Attachment>`` 的 transaction 的 attachments
* ``notary`` 是 transaction 的 notary。这个必须要同所有的 inputs 拥有相同的 notary
* ``timeWindow`` 定义了一笔交易在什么样的时间窗内才会被公正

``LedgerTransaction`` exposes a large number of utility methods to access the transaction's contents:

* ``inputStates`` extracts the input ``ContractState`` objects from the list of ``StateAndRef``
* ``getInput``/``getOutput``/``getCommand``/``getAttachment`` extracts a component by index
* ``getAttachment`` extracts an attachment by ID
* ``inputsOfType``/``inRefsOfType``/``outputsOfType``/``outRefsOfType``/``commandsOfType`` extracts components based on
  their generic type
* ``filterInputs``/``filterInRefs``/``filterOutputs``/``filterOutRefs``/``filterCommands`` extracts components based on
  a predicate
* ``findInput``/``findInRef``/``findOutput``/``findOutRef``/``findCommand`` extracts the single component that matches
  a predicate, or throws an exception if there are multiple matches

``LedgerTransaction`` 暴漏了很多 utility 方法来访问交易的内容：

* ``inputStates`` 从 ``StateAndRef`` 列表中获得 input ``ContractState`` 对象
* ``getInput``/``getOutput``/``getCommand``/``getAttachment`` 通过索引（index）来获得某个组件
* ``getAttachment`` 通过 ID 获得一个附件
* ``inputsOfType``/``inRefsOfType``/``outputsOfType``/``outRefsOfType``/``commandsOfType`` 基于他们的通用类型获得相关组件
* ``filterInputs``/``filterInRefs``/``filterOutputs``/``filterOutRefs``/``filterCommands`` 基于一个前提条件获得相关组件
* ``findInput``/``findInRef``/``findOutput``/``findOutRef``/``findCommand`` 获得满足一定条件的单一组件，或者当有多个满足条件的组件的时候抛出异常

requireThat
-----------
``verify`` can be written to manually throw an exception for each constraint:

``verify`` 能够针对每一个约束手动地抛出异常：

.. container:: codeset

   .. sourcecode:: kotlin

        override fun verify(tx: LedgerTransaction) {
            if (tx.inputs.size > 0)
                throw IllegalArgumentException("No inputs should be consumed when issuing an X.")

            if (tx.outputs.size != 1)
                throw IllegalArgumentException("Only one output state should be created.")
        }

   .. sourcecode:: java

        public void verify(LedgerTransaction tx) {
            if (tx.getInputs().size() > 0)
                throw new IllegalArgumentException("No inputs should be consumed when issuing an X.");

            if (tx.getOutputs().size() != 1)
                throw new IllegalArgumentException("Only one output state should be created.");
        }

However, this is verbose. To impose a series of constraints, we can use ``requireThat`` instead:

但是这个定义有些繁琐。我们可以使用 ``requireThat`` 来定义一系列的约束：

.. container:: codeset

   .. sourcecode:: kotlin

        requireThat {
            "No inputs should be consumed when issuing an X." using (tx.inputs.isEmpty())
            "Only one output state should be created." using (tx.outputs.size == 1)
            val out = tx.outputs.single() as XState
            "The sender and the recipient cannot be the same entity." using (out.sender != out.recipient)
            "All of the participants must be signers." using (command.signers.containsAll(out.participants))
            "The X's value must be non-negative." using (out.x.value > 0)
        }

   .. sourcecode:: java

        requireThat(require -> {
            require.using("No inputs should be consumed when issuing an X.",  tx.getInputs().isEmpty());
            require.using("Only one output state should be created.", tx.getOutputs().size() == 1);
            final XState out = (XState) tx.getOutputs().get(0);
            require.using("The sender and the recipient cannot be the same entity.", out.getSender() != out.getRecipient());
            require.using("All of the participants must be signers.", command.getSigners().containsAll(out.getParticipants()));
            require.using("The X's value must be non-negative.", out.getX().getValue() > 0);
            return null;
        });

For each <``String``, ``Boolean``> pair within ``requireThat``, if the boolean condition is false, an
``IllegalArgumentException`` is thrown with the corresponding string as the exception message. In turn, this
exception will cause the transaction to be rejected.

对于 ``requireThat`` 中的每一个 <``String``, ``Boolean``> 对来说，如果 boolean 条件返回的是 false，一个 ``IllegalArgumentException`` 会被抛出，包含对应的错误信息。所以这个错误会造成 transaction 被拒绝。

Commands
--------
``LedgerTransaction`` contains the commands as a list of ``CommandWithParties`` instances. ``CommandWithParties`` pairs
a ``CommandData`` with a list of required signers for the transaction:

``LedgerTransaction`` 包含了作为 ``CommandWithParties`` 实例列表的 commands。``CommandWithParties`` 将一个 ``CommandData`` 和一个所需的签名者列表匹配起来：

.. container:: codeset

    .. literalinclude:: ../../core/src/main/kotlin/net/corda/core/contracts/Structures.kt
        :language: kotlin
        :start-after: DOCSTART 6
        :end-before: DOCEND 6

Where:

* ``signers`` is the list of each signer's ``PublicKey``
* ``signingParties`` is the list of the signer's identities, if known
* ``value`` is the object being signed (a command, in this case)

其中：

* ``signers`` 是每个签名者的 ``PublicKey`` 的一个列表
* ``signingParties`` 签名者 identities 的列表，如果知道的话
* ``value`` 是被签名的对象（在这里指的是这个 command）

使用 commands 来处理 verify 分支
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Generally, we will want to impose different constraints on a transaction based on its commands. For example, we will
want to impose different constraints on a cash issuance transaction to on a cash transfer transaction.

通常来说，我们希望基于交易的 commands 来定义不同的约束条件。比如我们想要为一个现金发行的 transaction 和 一个现金交换的 transaction 定义不同的合约。

We can achieve this by extracting the command and using standard branching logic within ``verify``. Here, we extract
the single command of type ``XContract.Commands`` from the transaction, and branch ``verify`` accordingly:

我们可以通过提取这个 command 并在 ``verify`` 里使用标准的分支逻辑来实现这个功能。这里我们提取了交易中的类型为 ``XContract.Commands`` 的单独的 command，并且相应地对 ``verify`` 进行了分支逻辑判断：

.. container:: codeset

   .. sourcecode:: kotlin

        class XContract : Contract {
            interface Commands : CommandData {
                class Issue : TypeOnlyCommandData(), Commands
                class Transfer : TypeOnlyCommandData(), Commands
            }

            override fun verify(tx: LedgerTransaction) {
                val command = tx.findCommand<Commands> { true }

                when (command.value) {
                    is Commands.Issue -> {
                        // Issuance verification logic.
                    }
                    is Commands.Transfer -> {
                        // Transfer verification logic.
                    }
                }
            }
        }

   .. sourcecode:: java

        public class XContract implements Contract {
            public interface Commands extends CommandData {
                class Issue extends TypeOnlyCommandData implements Commands {}
                class Transfer extends TypeOnlyCommandData implements Commands {}
            }

            @Override
            public void verify(LedgerTransaction tx) {
                final Commands command = tx.findCommand(Commands.class, cmd -> true).getValue();

                if (command instanceof Commands.Issue) {
                    // Issuance verification logic.
                } else if (command instanceof Commands.Transfer) {
                    // Transfer verification logic.
                }
            }
        }
