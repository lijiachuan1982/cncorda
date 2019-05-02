Corda API
=========

The following are the core APIs that are used in the development of CorDapps:

下边是开发 CorDapps 使用的核心 APIs：

.. toctree::
   :maxdepth: 1

   api-states
   api-persistence
   api-contracts
   api-contract-constraints
   api-vault-query
   api-transactions
   api-flows
   api-identity
   api-service-hub
   api-rpc
   api-core-types
   api-testing

Before reading this page, you should be familiar with the :doc:`key concepts of Corda <key-concepts>`.

在阅读这里之前，你应该先熟悉一下 :doc:`Corda 核心概念 <key-concepts>`。

.. _internal-apis-and-stability-guarantees:

API 稳定性保证
--------------------------------------

Corda makes certain commitments about what parts of the API will preserve backwards compatibility as they change and
which will not. Over time, more of the API will fall under the stability guarantees. Thus, APIs can be categorized in the following 2 broad categories:

* **public APIs**, for which API/`ABI <https://en.wikipedia.org/wiki/Application_binary_interface>`_ backwards compatibility guarantees are provided. See: :ref:`public-api`
* **non-public APIs**, for which no backwards compatibility guarantees are provided. See: :ref:`non-public-api`

Corda 提供给了一些承诺来确保有些 API 是能够向后兼容的，有些是不能兼容的。随着时间的推移，越来越多的 API 会慢慢变为可兼容的。因此，APIs 可以分为一下两个类型：

* **公共 APIs**，这些 API/`ABI <https://en.wikipedia.org/wiki/Application_binary_interface>`_ 能够确保向后兼容性。查看 :ref:`public-api`
* **非公共 APIs**，这类 API 不具备向后兼容性。查看 :ref:`non-public-api`

.. _public-api:

公共 API
----------

The following modules form part of Corda's public API and we commit to API/ABI backwards compatibility in following releases, unless an incompatible change is required for security reasons:

* **Core (net.corda.core)**: core Corda libraries such as crypto functions, types for Corda's building blocks: states, contracts, transactions, attachments, etc. and some interfaces for nodes and protocols
* **Client RPC (net.corda.client.rpc)**: client RPC
* **Client Jackson (net.corda.client.jackson)**: JSON support for client applications
* **DSL Test Utils (net.corda.testing.dsl)**: a simple DSL for building pseudo-transactions (not the same as the wire protocol) for testing purposes.
* **Test Node Driver (net.corda.testing.node, net.corda.testing.driver)**: test utilities to run nodes programmatically
* **Test Utils (net.corda.testing.core)**: generic test utilities
* **Http Test Utils (net.corda.testing.http)**: a small set of utilities for making HttpCalls, aimed at demos and tests.
* **Dummy Contracts (net.corda.testing.contracts)**: dummy state and contracts for testing purposes
* **Mock Services (net.corda.testing.services)**: mock service implementations for testing purposes

下边这些模块是来自于部分 Corda 的公共 API，我们保证在以后的 releases 中，这些 API/ABI 是向后兼容的，除非因为安全原因必须要有一个非兼容的改动：

* **Core (net.corda.core)**: Corda 的核心类库，比如加密功能，Corda 的构建模块类型：states, contracts, transactions, attachments 等等，以及关于节点和协议的一些接口
* **Client RPC (net.corda.client.rpc)**: 客户端 RPC
* **Client Jackson (net.corda.client.jackson)**: 对于客户端应用程序的 JSON 支持
* **DSL Test Utils (net.corda.testing.dsl)**: 为了测试的目的，一个为了构建 pseudo-transactions 的简单的 DSL (不同于 wire protocol)
* **Test Node Driver (net.corda.testing.node, net.corda.testing.driver)**: 通过程序来运行节点的测试单元
* **Test Utils (net.corda.testing.core)**: 常规测试单元
* **Http Test Utils (net.corda.testing.http)**: 用来进行 HttpCalls，为了 demos 和测试的目的而需要的一个小的工具
* **Dummy Contracts (net.corda.testing.contracts)**: 为了测试的目的而创建的虚构的 state 和 contracts
* **Mock Services (net.corda.testing.services)**: 为了测试的目的而创建虚构的服务实现

.. _non-public-api:

非公共 APIs (探索性的)
-----------------------------

The following modules are not part of the Corda's public API and no backwards compatibility guarantees are provided. They are further categorized in 2 classes:

* the incubating modules, for which we will do our best to minimise disruption to developers using them until we are able to graduate them into the public API
* the internal modules, which are not to be used, and will change without notice

下边的模块不是 Corda 公共 API，并且不具备向后兼容性。他们可以分为两种类型：

* 正在孵化的模块，我们会进我们最大的努力减小开发者使用他们时造成的影响，知道我们能够保证他们成为公共 API
* 内部的模块，这些是没有被使用的，并且可能会改动而没有任何通知

Corda 孵化中的模块
~~~~~~~~~~~~~~~~~~~~~~~~

* **net.corda.confidential**: experimental support for confidential identities on the ledger
* **net.corda.finance**: a range of elementary contracts (and associated schemas) and protocols, such as abstract fungible assets, cash, obligation and commercial paper
* **net.corda.client.jfx**: support for Java FX UI
* **net.corda.client.mock**: client mock utilities
* **Cordformation**: Gradle integration plugins

* **net.corda.confidential**: 对于账本上保密的身份的探索
* **net.corda.finance**: 一系列的基本的合约（和相关的 schemas）以及协议，比如抽象的 fungible assets、现金、债务以及商业票据
* **net.corda.client.jfx**: 对于 Java FX UI 的支持
* **net.corda.client.mock**: 客户端的模拟工具
* **Cordformation**: Gradle 集成插件

Corda 内部模块
~~~~~~~~~~~~~~~~~~~~~~

Everything else is internal and will change without notice, even deleted, and should not be used. This also includes any package that has
``.internal`` in it. So for example, ``net.corda.core.internal`` and sub-packages should not be used.

其他的模块就是内部模块，可能会被改动而没有任何的通知，甚至是删除，所以这些是不应该被使用的。这也包括了任何其中包含 ``.internal`` 的模块。比如 ``net.corda.core.internal`` 和它的子开发包都不应该被使用。

Some of the public modules may depend on internal modules, so be careful to not rely on these transitive dependencies. In particular, the
testing modules depend on the node module and so you may end having the node in your test classpath.

有些公共模块可能会依赖于一些内部的模块，所以要小心不要使用这些有过度性依赖的模块。特别要指出的是，测试的模块依赖于节点模块，所以在你的测试 classpath 中可能会没有它。

.. warning:: The web server module will be removed in future. You should call Corda nodes through RPC from your web server of choice e.g., Spring Boot, Vertx, Undertow.

.. warning:: Web server 模块在将来会被移除。你应该使用自己的 web server（Spring Boot、Vertx、Undertow） 使用 RPC 来调用 Corda 节点。

``@DoNotImplement`` 注解
----------------------------------

Certain interfaces and abstract classes within the Corda API have been annotated
as ``@DoNotImplement``. While we undertake not to remove or modify any of these classes' existing
functionality, the annotation is a warning that we may need to extend them in future versions of Corda.
Cordapp developers should therefore just use these classes "as is", and *not* attempt to extend or implement any of them themselves.

一些在 Corda API 中的接口和抽象类含有 ``@DoNotImplement`` 的注解。我们不会移除或者修改这些类已经存在的这些功能，这个注解表示我们会在以后版本的 Corda 中需要对其进行扩展。CorDapp 开发者应该仅仅按照现在这样去使用他们，并且 *不要* 自己尝试去扩展或实现这些类。

This annotation is inherited by subclasses and sub-interfaces.

这个注解会被子类或者子接口继承。
