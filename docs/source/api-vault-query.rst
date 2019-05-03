.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

API: Vault Query
================

.. contents::

概览
--------
Corda has been architected from the ground up to encourage usage of industry standard, proven query frameworks and
libraries for accessing RDBMS backed transactional stores (including the Vault).

Corda 从最底层的架构开始一直都在推崇使用业界标准的，经过考验的查询框架和类库来访问存储 transaction 数据的 RDBMS 后台（包括 Vault）。

Corda provides a number of flexible query mechanisms for accessing the Vault:

- Vault Query API
- Using a JDBC session (as described in :ref:`Persistence <jdbc_session_ref>`)
- Custom JPA_/JPQL_ queries
- Custom 3rd party Data Access frameworks such as `Spring Data <http://projects.spring.io/spring-data>`_

Corda 提供了一系列的灵活的查询机制来访问 Vault：

- Vault Query API
- 使用 JDBC session（像 :ref:`Persistence <jdbc_session_ref>` 描述的那样）
- 自定义 JPA_/JPQL_ 查询
- 自定义第三方的数据访问框架，比如 `Spring Data <http://projects.spring.io/spring-data>`_

The majority of query requirements can be satisfied by using the Vault Query API, which is exposed via the
``VaultService`` for use directly by flows:

大多数的查询需求都能够通过使用 Vault Query API 来满足，该 API 是通过 ``VaultService`` 暴露出来的，可以被 flow 直接使用：

.. literalinclude:: ../../core/src/main/kotlin/net/corda/core/node/services/VaultService.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryAPI
    :end-before: DOCEND VaultQueryAPI
    :dedent: 4

And via ``CordaRPCOps`` for use by RPC client applications:

并且 RPC 客户端应用程序可以通过 ``CordaRPCOps`` 来使用

.. literalinclude:: ../../core/src/main/kotlin/net/corda/core/messaging/CordaRPCOps.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryByAPI
    :end-before: DOCEND VaultQueryByAPI
    :dedent: 4

.. literalinclude:: ../../core/src/main/kotlin/net/corda/core/messaging/CordaRPCOps.kt
    :language: kotlin
    :start-after: DOCSTART VaultTrackByAPI
    :end-before: DOCEND VaultTrackByAPI
    :dedent: 4

Helper methods are also provided with default values for arguments:

Helper 方法也被提供，包括参数的默认值：

.. literalinclude:: ../../core/src/main/kotlin/net/corda/core/messaging/CordaRPCOps.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryAPIHelpers
    :end-before: DOCEND VaultQueryAPIHelpers
    :dedent: 4

.. literalinclude:: ../../core/src/main/kotlin/net/corda/core/messaging/CordaRPCOps.kt
    :language: kotlin
    :start-after: DOCSTART VaultTrackAPIHelpers
    :end-before: DOCEND VaultTrackAPIHelpers
    :dedent: 4

The API provides both static (snapshot) and dynamic (snapshot with streaming updates) methods for a defined set of
filter criteria:

- Use ``queryBy`` to obtain a current snapshot of data (for a given ``QueryCriteria``)
- Use ``trackBy`` to obtain both a current snapshot and a future stream of updates (for a given ``QueryCriteria``)

API 提供了静态的（snapshot）和动态的（包含 steaming updates 的 snapshot）方法来定义一系列的过滤条件:

- 使用 ``queryBy`` 来获得数据当前的 snapshot（对于一个给定的 ``QueryCriteria``）
- 使用 ``trackBy`` 来获得既包括当前的 snapshot 也包括将来的更新流（对于一个给定的 ``QueryCriteria``）

.. note:: Streaming updates are only filtered based on contract type and state status (UNCONSUMED, CONSUMED, ALL).
          They will not respect any other criteria that the initial query has been filtered by.

.. note:: 更新流只能够基于 contract type 和 state status（UNCONSUMED, CONSUMED, ALL）来进行过滤。除了初始查询时候的过滤条件，他们不能够使用其他的条件。

Simple pagination (page number and size) and sorting (directional ordering using standard or custom property
attributes) is also specifiable. Defaults are defined for paging (pageNumber = 1, pageSize = 200) and sorting
(direction = ASC).

也可以指定简单的分页（页码和每页包含记录数）和排序（使用标准或者自定义的属性值进行排序）。分页的默认值为（pageNumber = 1（显示第一页）, pageSize = 200（每页200条记录）），排序的默认值为（direction = ASC（升序））。

The ``QueryCriteria`` interface provides a flexible mechanism for specifying different filtering criteria, including
and/or composition and a rich set of operators to include:

``QueryCriteria`` 接口提供了灵活的机制来指定不同的过滤条件，包括 and/or 组合和一系列的操作符，包括：

* Binary logical (AND, OR)
* Comparison (LESS_THAN, LESS_THAN_OR_EQUAL, GREATER_THAN, GREATER_THAN_OR_EQUAL)
* Equality (EQUAL, NOT_EQUAL)
* Likeness (LIKE, NOT_LIKE)
* Nullability (IS_NULL, NOT_NULL)
* Collection based (IN, NOT_IN)
* Standard SQL-92 aggregate functions (SUM, AVG, MIN, MAX, COUNT)

There are four implementations of this interface which can be chained together to define advanced filters.

这里有四种对于该接口的实现，可以结合使用他们来定义高级的过滤条件。

1. ``VaultQueryCriteria`` provides filterable criteria on attributes within the Vault states table: status (UNCONSUMED,
   CONSUMED), state reference(s), contract state type(s), notaries, soft locked states, timestamps (RECORDED, CONSUMED),
   state constraints (see :ref:`Constraint Types <implicit_constraint_types>`), relevancy (ALL, RELEVANT, NON_RELEVANT).

   ``VaultQueryCriteria`` 提供了针对于 Vault states 表的属性值的可过滤条件：status (UNCONSUMED, CONSUMED), state reference(s), contract state type(s), notaries, soft locked states, timestamps (RECORDED, CONSUMED), state constraints (see :ref:`Constraint Types <implicit_constraint_types>`), relevancy (ALL, RELEVANT, NON_RELEVANT)。

	.. note:: Sensible defaults are defined for frequently used attributes (status = UNCONSUMED, always include soft
	   locked states).

	.. note:: 对于经常使用的属性，已经定义了一些显著的默认值（status = UNCONSUMED, 总会包含 soft locked states）

2. ``FungibleAssetQueryCriteria`` provides filterable criteria on attributes defined in the Corda Core
   ``FungibleAsset`` contract state interface, used to represent assets that are fungible, countable and issued by a
   specific party (eg. ``Cash.State`` and ``CommodityContract.State`` in the Corda finance module). Filterable
   attributes include: participants(s), owner(s), quantity, issuer party(s) and issuer reference(s).

   ``FungibleAssetQueryCriteria`` 提供了针对于在 Corda 核心的 ``FungibleAsset`` contract state 接口中定义的属性的可过滤条件，用来展示资产是可替换的，可计数的并且被指定的机构发行（比如在 Corda finance module 中的 ``Cash.State`` 和 ``CommondityContract.State``）。可过滤条件包括：participants(s), owner(s), quantity, issuer party(s) and issuer reference(s)。
	   
	.. note:: All contract states that extend the ``FungibleAsset`` now automatically persist that interfaces common
	   state attributes to the **vault_fungible_states** table.

	.. note:: 所有扩展了 ``FungibleAsset`` 的 contract states 会自动将该接口中的常规 state 属性存储到 **vault_fungible_states** 表中。

3. ``LinearStateQueryCriteria`` provides filterable criteria on attributes defined in the Corda Core ``LinearState``
   and ``DealState`` contract state interfaces, used to represent entities that continuously supersede themselves, all
   of which share the same ``linearId`` (e.g. trade entity states such as the ``IRSState`` defined in the SIMM
   valuation demo). Filterable attributes include: participant(s), linearId(s), uuid(s), and externalId(s).

   ``LinearStateQueryCriteria`` 提供了针对于在 Corda 核心的 ``LinearState`` 和 ``DealState`` 中定义的属性的可过滤条件，用来展示那些一直在取代/替换自己的实体，所有的实体都包含一个相同的 ``linearId``（比如像 SIMM valuation demo 中的 trade 实体 states，比如 ``IRSState``）。可过滤的条件包括：participant(s), linearId(s), dealRef(s)。
	   
	.. note:: All contract states that extend ``LinearState`` or ``DealState`` now automatically persist those
	   interfaces common state attributes to the **vault_linear_states** table.

	.. note:: 所有扩展自 ``LinearState`` 或者 ``DealState`` 的 contract states 会自动地将该接口中常用的 state 属性存储到 **vault_linear_states** 表中。

4. ``VaultCustomQueryCriteria`` provides the means to specify one or many arbitrary expressions on attributes defined
   by a custom contract state that implements its own schema as described in the :doc:`Persistence </api-persistence>`
   documentation and associated examples. Custom criteria expressions are expressed using one of several type-safe
   ``CriteriaExpression``: BinaryLogical, Not, ColumnPredicateExpression, AggregateFunctionExpression. The
   ``ColumnPredicateExpression`` allows for specification arbitrary criteria using the previously enumerated operator
   types. The ``AggregateFunctionExpression`` allows for the specification of an aggregate function type (sum, avg,
   max, min, count) with optional grouping and sorting. Furthermore, a rich DSL is provided to enable simple
   construction of custom criteria using any combination of ``ColumnPredicate``. See the ``Builder`` object in
   ``QueryCriteriaUtils`` for a complete specification of the DSL.

   ``VaultCustomQeuryCriteria`` 提供了一种方式，来指定一个或者多个基于自定义的 contract state 的属性的任意表达式，这个自定义的 contract state 实现了像 :doc:`持久化 </api-persistence>` 文档中描述的自己的 schema。自定义的条件表达式通过使用一个或者多个 type-safe 的 ``CriteriaExpression``：BinaryLogical, Not, ColumnPredicateExpression。``ColumnPredicateExpression`` 允许使用前边罗列的操作符类型来指定任意的条件。``AggregateFunctionExpression`` 允许指定聚合方法类型（sum, avg, max, min, count）并可以带有可选的 grouping 和 sorting。更进一步地讲，一个丰富的 DSL 被提供来使得用任何的 ``ColumnPredicate`` 组合来简单地构建一个自定义的查询条件成为可能。查看在 ``QueryCriteriaUtils`` 里的 ``Builder`` 对象来看到 DSL 的完整描述。

    .. note:: Custom contract schemas are automatically registered upon node startup for CorDapps. Please refer to
       :doc:`Persistence </api-persistence>` for mechanisms of registering custom schemas for different testing
       purposes.

    .. note:: 自定义的 contract schemas 会在节点启动的时候被自动注册。可以参考 :doc:`持久化 </api-persistence>` 来了解对于不同的测试目的自定义 schemas 注册的机制。

All ``QueryCriteria`` implementations are composable using ``and`` and ``or`` operators.

所有的 ``QueryCriteria`` 实现都可以使用 ``and`` 和 ``or`` 操作符。

All ``QueryCriteria`` implementations provide an explicitly specifiable set of common attributes:

1. State status attribute (``Vault.StateStatus``), which defaults to filtering on UNCONSUMED states.
   When chaining several criteria using AND / OR, the last value of this attribute will override any previous
2. Contract state types (``<Set<Class<out ContractState>>``), which will contain at minimum one type (by default this
   will be ``ContractState`` which resolves to all state types). When chaining several criteria using ``and`` and
   ``or`` operators, all specified contract state types are combined into a single set

所有的 ``QueryCriteria`` 实现提供了一套可显式指定的常用属性：

1. State 状态属性（``Vault.StateStatus``），默认值是 UNCONSUMED states。当使用 AND/OR 定义多个条件的时候，这个属性的最后一个值会覆盖之前的所有值
2. Contract state 类型（``<Set<Class<out ContractState>>``），它至少包含一个类型（默认的会是满足所有 state 类型的 ``ContractState``）。当使用 ``and`` 和 ``or`` 定义多个条件的时候，所有指定的 contract state types 会被合并为一套

An example of a custom query is illustrated here:

下边是一个自定义查询的演示实例:

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample20
    :end-before: DOCEND VaultQueryExample20
    :dedent: 12

.. note:: Custom contract states that implement the ``Queryable`` interface may now extend common schemas types
   ``FungiblePersistentState`` or, ``LinearPersistentState``.  Previously, all custom contracts extended the root
   ``PersistentState`` class and defined repeated mappings of ``FungibleAsset`` and ``LinearState`` attributes. See
   ``SampleCashSchemaV2`` and ``DummyLinearStateSchemaV2`` as examples.

.. note:: 自定义的实现了 ``Queryable`` 接口的 contract states 现在可能扩展了常规的 schemas types ``FungiblePersistentState`` 或者 ``LinearPersistentState``。以前，所有的自定义 contracts 扩展了根 ``PersistentState`` 类并且定义了对于 ``FungibleAsset`` 和 ``LinearState`` 属性的重复 mappings。参考例子 ``SampleCashSchemaV2`` 和 ``DummyLinearStateSchemaV2``。

Examples of these ``QueryCriteria`` objects are presented below for Kotlin and Java.

下边会有这些 ``QueryCriteria`` 对象的使用例子。

.. note:: When specifying the ``ContractType`` as a parameterised type to the ``QueryCriteria`` in Kotlin, queries now
   include all concrete implementations of that type if this is an interface. Previously, it was only possible to query
   on concrete types (or the universe of all ``ContractState``).

.. note:: 当在 Kotlin 中将指定的 ``ContractType`` 作为传给`` QueryCriteria`` 的参数类型的时候，如果这是一个 interface 的话，那么查询会包括所有的 concrete 实现。在以前，只能够基于 concrete 类型进行查询（或者所有的 ``ContractState``）。

The Vault Query API leverages the rich semantics of the underlying JPA Hibernate_ based
:doc:`Persistence </api-persistence>` framework adopted by Corda.

Vault Query API 使用了底层 JPA Hibernate_ 的丰富的语义，JPA Hibernate 是基于 Corda 所采用的基于 :doc:`持久化 </api-persistence>` 的框架的。

.. _Hibernate: https://docs.jboss.org/hibernate/jpa/2.1/api/

.. note:: Permissioning at the database level will be enforced at a later date to ensure authenticated, role-based,
   read-only access to underlying Corda tables.

.. note:: 对于数据库 level 的权限控制会在今后被强制执行，确保经过验证的，基于角色的和只读的访问权限来控制 Corda 的表。

.. note:: API's now provide ease of use calling semantics from both Java and Kotlin. However, it should be noted that
   Java custom queries are significantly more verbose due to the use of reflection fields to reference schema attribute
   types.

.. note:: 现在 API 提供了非常简单的方式通过 Java 和 Kotlin 来使用 semantics。然而，我们应该注意到 Java 的 查询更加的繁琐因为使用了反射字段来引用 schema 属性类型。

An example of a custom query in Java is illustrated here:

在 Java 中创建一个自定义查询的例子：

.. literalinclude:: ../../node/src/test/java/net/corda/node/services/vault/VaultQueryJavaTests.java
    :language: java
    :start-after: DOCSTART VaultJavaQueryExample3
    :end-before: DOCEND VaultJavaQueryExample3
    :dedent: 16

.. note:: Queries by ``Party`` specify the ``AbstractParty`` which may be concrete or anonymous. In the later case,
   where an anonymous party does not resolve to an X500 name via the ``IdentityService``, no query results will ever be
   produced. For performance reasons, queries do not use ``PublicKey`` as search criteria.

.. note:: 当前这个根据 ``Party`` 的查询指定了 ``AbstractParty``，该 AbstractParty 可能是具体的或者是匿名的。如果是匿名的，当一个匿名的参与方无法通过 ``IdentityService`` 来获得一个指定的 X500Name 的话，将不会返回任何结果。基于效率的原因，查询不会使用 ``PublicKey`` 作为查询条件。

Custom queries can be either case sensitive or case insensitive. They are defined via a ``Boolean`` as one of the function parameters of each operator function. By default each operator is case sensitive.

自定义查询可以是区分大小写的，也可以是不区分的。这个是通过每个操作符的一个 ``Boolean`` 类型的方法参数来定义的。默认的，每个操作符是区分大小写的。

An example of a case sensitive custom query operator is illustrated here:

下边是一个区分大小写的自定义查询的例子：

.. container:: codeset

    .. sourcecode:: kotlin

        val currencyIndex = PersistentCashState::currency.equal(USD.currencyCode, true)

.. note:: The ``Boolean`` input of ``true`` in this example could be removed since the function will default to ``true`` when not provided.

.. note:: 这个例子中的值为 ``true`` 的 ``Boolean`` 类型的 input 可以被移除，因为如果没有提供这个值的话，这个方法默认就是 ``true``。

An example of a case insensitive custom query operator is illustrated here:

下边是一个不区分大小写的自定义查询的例子：

.. container:: codeset

    .. sourcecode:: kotlin

        val currencyIndex = PersistentCashState::currency.equal(USD.currencyCode, false)

An example of a case sensitive custom query operator in Java is illustrated here:

下边是一个 Java 中区分大小写的自定义查询的例子：

.. container:: codeset

    .. sourcecode:: java

        FieldInfo attributeCurrency = getField("currency", CashSchemaV1.PersistentCashState.class);
        CriteriaExpression currencyIndex = Builder.equal(attributeCurrency, "USD", true);

An example of a case insensitive custom query operator in Java is illustrated here:

下边是 Java 中一个不区分大小写的自定义查询的例子：

.. container:: codeset

    .. sourcecode:: java

        FieldInfo attributeCurrency = getField("currency", CashSchemaV1.PersistentCashState.class);
        CriteriaExpression currencyIndex = Builder.equal(attributeCurrency, "USD", false);

分页
----------
The API provides support for paging where large numbers of results are expected (by default, a page size is set to 200
results). Defining a sensible default page size enables efficient checkpointing within flows, and frees the developer
from worrying about pagination where result sets are expected to be constrained to 200 or fewer entries. Where large
result sets are expected (such as using the RPC API for reporting and/or UI display), it is strongly recommended to
define a ``PageSpecification`` to correctly process results with efficient memory utilisation. A fail-fast mode is in
place to alert API users to the need for pagination where a single query returns more than 200 results and no
``PageSpecification`` has been supplied.

API 提供了分页的支持来应对可能返回大批量数据的情况（默认会返回200条记录）。定义一个合理的分页数量的默认值能够让 flows 中的 checkpointing 更有效率，并且开发人员不用再去担心当结果集包含200条或者更少的记录的时候要怎么去进行分页。当大批量的结果可能返回的时候（比如使用 RPC API 做报表并且要现在 UI 上的情况），我们强烈地建议定义一个 ``PageSpecification`` 来有效使用内存来正确的执行查询并获得结果。当返回结果超过 200 条记录但是 ``PageSpecification`` 没有被指定的时候，这里已经存在一个 fail-fast 模型会来提醒 API 用户需要去进行分页。

Here's a query that extracts every unconsumed ``ContractState`` from the vault in pages of size 200, starting from the
default page number (page one):

下边的查询会从账本中查询所有 unconsumed ``ContractState``， 每页会包含 200 条记录，返回的是默认的页数（第一页）

.. container:: codeset

    .. sourcecode:: kotlin

        val vaultSnapshot = proxy.vaultQueryBy<ContractState>(
            QueryCriteria.VaultQueryCriteria(Vault.StateStatus.UNCONSUMED),
            PageSpecification(DEFAULT_PAGE_NUM, 200))

.. note:: A pages maximum size ``MAX_PAGE_SIZE`` is defined as ``Int.MAX_VALUE`` and should be used with extreme
   caution as results returned may exceed your JVM's memory footprint.

.. note:: 每页最多显示多少条记录 ``MAX_PAGE_SIZE`` 会被定义为 ``Int.MAX_VALUE``，并且你需要很小心的使用它，因为返回的结果可能会超出你的 JVM 的内存 footprint。

使用样例
-------------

Kotlin
^^^^^^

**General snapshot queries using** ``VaultQueryCriteria``:

使用 ``VaultQueryCriteria`` 的 **常规 snapshot 查询**

Query for all unconsumed states (simplest query possible):

查询所有的 unconsumed states（可能是最简单的一个查询）：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample1
    :end-before: DOCEND VaultQueryExample1
    :dedent: 12

Query for unconsumed states for some state references:

对于一些 state 引用的 unconsumed states 查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample2
    :end-before: DOCEND VaultQueryExample2
    :dedent: 12

Query for unconsumed states for several contract state types:

对于一些 contract state 类型的 unconsumed states 查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample3
    :end-before: DOCEND VaultQueryExample3
    :dedent: 12

Query for unconsumed states for specified contract state constraint types and sorted in ascending alphabetical order:

对于一些指定的 contract state 约束类型的 unconsumed states 的查询，按照字母升序排序：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample30
    :end-before: DOCEND VaultQueryExample30
    :dedent: 12

Query for unconsumed states for specified contract state constraints (type and data):

对于一些指定的 contract state 约束（类型和数据）的 unconsumed states 的查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample31
    :end-before: DOCEND VaultQueryExample31
    :dedent: 12

Query for unconsumed states for a given notary:

对于指定的一个 notary 的 unconsumed states 查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample4
    :end-before: DOCEND VaultQueryExample4
    :dedent: 12

Query for unconsumed states for a given set of participants:

对于指定的一系列 participants 的 unconsumed states 的查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample5
    :end-before: DOCEND VaultQueryExample5
    :dedent: 12

Query for unconsumed states recorded between two time intervals:

对于指定时间区间内的 unconsumed states 记录的查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample6
    :end-before: DOCEND VaultQueryExample6
    :dedent: 12

.. note:: This example illustrates usage of a ``Between`` ``ColumnPredicate``.

.. note:: 上边的例子演示了如何使用 ``Between`` ``ColumnPredicate``。

Query for all states with pagination specification (10 results per page):

查询所有的 states 并且使用指定的分页条件（每页 10 条记录）：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample7
    :end-before: DOCEND VaultQueryExample7
    :dedent: 12

.. note:: The result set metadata field `totalStatesAvailable` allows you to further paginate accordingly as
   demonstrated in the following example.

.. note:: 结果集的 `totalStatesAvailable` metadata 字段允许你可以像下边的例子那样进行进一步的分页。

Query for all states using a pagination specification and iterate using the `totalStatesAvailable` field until no further
pages available:

使用 pagination specification 和 iterate，使用 `totalStatesAvailable` 字段查询所有的 states 直到最后一页：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample24
    :end-before: DOCEND VaultQueryExample24
    :dedent: 8

Query for only relevant states in the vault:

查询 vault 中的相关的 states：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample25
    :end-before: DOCEND VaultQueryExample25
    :dedent: 8

**LinearState and DealState queries using** ``LinearStateQueryCriteria``:

**LinearState 和 DealState 查询应该使用** ``LinearStateQueryCriteria``：

Query for unconsumed linear states for given linear ids:

对于指定的 linear ids 的 unconsumed linear states 的查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample8
    :end-before: DOCEND VaultQueryExample8
    :dedent: 12

Query for all linear states associated with a linear id:

对于指定的 linear id 的所有 linear states 的查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample9
    :end-before: DOCEND VaultQueryExample9
    :dedent: 12

Query for unconsumed deal states with deals references:

对于指定的 deal references 的 unconsumed deal states 的查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample10
    :end-before: DOCEND VaultQueryExample10
    :dedent: 12

Query for unconsumed deal states with deals parties:

对于指定的 deals parties 的 unconsumed deal states 的查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample11
    :end-before: DOCEND VaultQueryExample11
    :dedent: 12

Query for only relevant linear states in the vault:

仅对于 vault 中的相关的 linear states 进行查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample26
    :end-before: DOCEND VaultQueryExample26
    :dedent: 8

**FungibleAsset and DealState queries using** ``FungibleAssetQueryCriteria``:

**FungibleAsset 和 DealAsset 查询应该使用** ``FungibleAssetQueryCriteria``。

Query for fungible assets for a given currency:

对于指定的 currency 的 fungible assets 的查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample12
    :end-before: DOCEND VaultQueryExample12
    :dedent: 12

Query for fungible assets for a minimum quantity:

对于最小数量的 fungible assets 的查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample13
    :end-before: DOCEND VaultQueryExample13
    :dedent: 12

.. note:: This example uses the builder DSL.

.. note:: 这个例子使用了 builder DSL。

Query for fungible assets for a specific issuer party:

对于指定的 issuer party 的 fungible assets 的查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample14
    :end-before: DOCEND VaultQueryExample14
    :dedent: 12

Query for only relevant fungible states in the vault:

仅对于 vault 中的相关的 fungible states 进行查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample27
    :end-before: DOCEND VaultQueryExample27
    :dedent: 12

**Aggregate Function queries using** ``VaultCustomQueryCriteria``:

使用 ``VaultCustomQueryCriteria`` 来实现**聚合查询**：

.. note:: Query results for aggregate functions are contained in the ``otherResults`` attribute of a results Page.

.. note:: 聚合查询的查询结果会被包含在结果页的 ``otherResults`` 属性中。

Aggregations on cash using various functions:

使用多种方法对 cash 进行聚合查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample21
    :end-before: DOCEND VaultQueryExample21
    :dedent: 12

.. note:: ``otherResults`` will contain 5 items, one per calculated aggregate function.

.. note:: ``otherResults`` 会包含5个 items，每个聚合方法一个。

Aggregations on cash grouped by currency for various functions:

使用多种方法按照 currency 进行分组对 cash 进行聚合查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample22
    :end-before: DOCEND VaultQueryExample22
    :dedent: 12

.. note:: ``otherResults`` will contain 24 items, one result per calculated aggregate function per currency (the
   grouping attribute - currency in this case - is returned per aggregate result).

.. note:: ``otherResults`` 将会包含 24 个 items，每个 currency 的每个局和方法会有一个结果（这里的分组属性 currency 会和每一个聚合结果返回回来）。

Sum aggregation on cash grouped by issuer party and currency and sorted by sum:

对 cash 按照 issuer party 进行分组并按照 sum 排序的 sum 聚合查询：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample23
    :end-before: DOCEND VaultQueryExample23
    :dedent: 12

.. note:: ``otherResults`` will contain 12 items sorted from largest summed cash amount to smallest, one result per
   calculated aggregate function per issuer party and currency (grouping attributes are returned per aggregate result).

.. note:: ``otherResults`` 会包含从加和最大到最小的 cash 数量共 12个 items，每个 issuer party 和 currency 的聚合方法会有一个结果。

Dynamic queries (also using ``VaultQueryCriteria``) are an extension to the snapshot queries by returning an
additional ``QueryResults`` return type in the form of an ``Observable<Vault.Update>``. Refer to
`ReactiveX Observable <http://reactivex.io/documentation/observable.html>`_ for a detailed understanding and usage of
this type.

动态查询（同样使用 ``VaultQueryCriteria``）是对于 snapshot 查询的一个扩展，其返回了一个额外的 ``QueryResults`` 返回类型，作为一个 ``Observable<Vault.Update>`` 形式返回。参考 `ReactiveX Observable <http://reactivex.io/documentation/observable.html>`_ 来了解详细内容和理解怎么使用这种类型。

Track unconsumed cash states:

跟踪 unconsumed cash states：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample15
    :end-before: DOCEND VaultQueryExample15
    :dedent: 12

Track unconsumed linear states:

跟踪 unconsumed linear states：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample16
    :end-before: DOCEND VaultQueryExample16
    :dedent: 12

.. note:: This will return both ``DealState`` and ``LinearState`` states.

.. note:: 这会返回 ``DealState`` 和 ``LinearState`` states。

Track unconsumed deal states:

跟踪 unconsumed deal states：

.. literalinclude:: ../../node/src/test/kotlin/net/corda/node/services/vault/VaultQueryTests.kt
    :language: kotlin
    :start-after: DOCSTART VaultQueryExample17
    :end-before: DOCEND VaultQueryExample17
    :dedent: 12

.. note:: This will return only ``DealState`` states.

.. note:: 这个会只返回 ``DealState`` states。

Java
^^^^

Query for all unconsumed linear states:

查询所有 unconsumed linear states：

.. literalinclude:: ../../node/src/test/java/net/corda/node/services/vault/VaultQueryJavaTests.java
    :language: java
    :start-after: DOCSTART VaultJavaQueryExample0
    :end-before: DOCEND VaultJavaQueryExample0
    :dedent: 12

Query for all consumed cash states:

查询所有 consumed cash states：

.. literalinclude:: ../../node/src/test/java/net/corda/node/services/vault/VaultQueryJavaTests.java
    :language: java
    :start-after: DOCSTART VaultJavaQueryExample1
    :end-before: DOCEND VaultJavaQueryExample1
    :dedent: 12

Query for consumed deal states or linear ids, specify a paging specification and sort by unique identifier:

查询 consumed deal states 或者 linear ids，指定了一个 paging specification 并且按照 unique identifier 排序：

.. literalinclude:: ../../node/src/test/java/net/corda/node/services/vault/VaultQueryJavaTests.java
    :language: java
    :start-after: DOCSTART VaultJavaQueryExample2
    :end-before: DOCEND VaultJavaQueryExample2
    :dedent: 12

Query for all states using a pagination specification and iterate using the `totalStatesAvailable` field until no further pages available:

使用一个分页说明来查询所有的 states，并且反复使用 `totalStatesAvailable` 字段直到最后一页：

.. literalinclude:: ../../node/src/test/java/net/corda/node/services/vault/VaultQueryJavaTests.java
    :language: java
    :start-after: DOCSTART VaultQueryExample24
    :end-before: DOCEND VaultQueryExample24
    :dedent: 8

**Aggregate Function queries using** ``VaultCustomQueryCriteria``:

使用 ``VaultCustomQueryCriteria`` 来实现**聚合查询**：

Aggregations on cash using various functions:

使用多种方法对 cash 进行聚合查询：

.. literalinclude:: ../../node/src/test/java/net/corda/node/services/vault/VaultQueryJavaTests.java
    :language: java
    :start-after: DOCSTART VaultJavaQueryExample21
    :end-before: DOCEND VaultJavaQueryExample21
    :dedent: 16

Aggregations on cash grouped by currency for various functions:

使用多种方法按照 currency 进行分组对 cash 进行聚合查询：

.. literalinclude:: ../../node/src/test/java/net/corda/node/services/vault/VaultQueryJavaTests.java
    :language: java
    :start-after: DOCSTART VaultJavaQueryExample22
    :end-before: DOCEND VaultJavaQueryExample22
    :dedent: 16

Sum aggregation on cash grouped by issuer party and currency and sorted by sum:

对 cash 按照 issuer party 进行分组并按照 sum 排序的 sum 聚合查询：

.. literalinclude:: ../../node/src/test/java/net/corda/node/services/vault/VaultQueryJavaTests.java
    :language: java
    :start-after: DOCSTART VaultJavaQueryExample23
    :end-before: DOCEND VaultJavaQueryExample23
    :dedent: 16

Track unconsumed cash states:

跟踪 unconsumed cash states：

.. literalinclude:: ../../node/src/test/java/net/corda/node/services/vault/VaultQueryJavaTests.java
    :language: java
    :start-after: DOCSTART VaultJavaQueryExample4
    :end-before: DOCEND VaultJavaQueryExample4
    :dedent: 12

Track unconsumed deal states or linear states (with snapshot including specification of paging and sorting by unique
identifier):

跟踪 unconsumed deal states 或者 linear states（带有包括分页说明以及按照 unique identifier 排序的 snapshot）：

.. literalinclude:: ../../node/src/test/java/net/corda/node/services/vault/VaultQueryJavaTests.java
    :language: java
    :start-after: DOCSTART VaultJavaQueryExample5
    :end-before: DOCEND VaultJavaQueryExample5
    :dedent: 12

Troubleshooting
---------------
If the results your were expecting do not match actual returned query results we recommend you add an entry to your
``log4j2.xml`` configuration file to enable display of executed SQL statements::

如果你期望的结果同真实返回的结果不匹配的话，我们建议你向你的 ``log4j2.xml`` 配置文件中添加一个配置项来开启所执行的 SQL 语句：

        <Logger name="org.hibernate.SQL" level="debug" additivity="false"> 
            <AppenderRef ref="Console-Appender"/> 
        </Logger>

行为笔记
-----------------
1. ``TrackBy`` updates do not take into account the full criteria specification due to different and more restrictive
   syntax in `observables <https://github.com/ReactiveX/RxJava/wiki>`_ filtering (vs full SQL-92 JDBC filtering as used
   in snapshot views). Specifically, dynamic updates are filtered by ``contractStateType`` and ``stateType``
   (UNCONSUMED, CONSUMED, ALL) only

   ``TrackBy`` 更新不会考虑全部的条件说明，因为 `observables <https://github.com/ReactiveX/RxJava/wiki>`_ 过滤更严格的语法和不同（对比像 在 snapshot 视图中使用的全部的 SQL-92 JDBC）。特别的，动态的更新仅仅能够被 ``contractStateType`` 和 ``stateType``（UNCONSUMED, CONSUMED, ALL）过滤。

2. ``QueryBy`` and ``TrackBy`` snapshot views using pagination may return different result sets as each paging request
   is a separate SQL query on the underlying database, and it is entirely conceivable that state modifications are
   taking place in between and/or in parallel to paging requests. When using pagination, always check the value of the
   ``totalStatesAvailable`` (from the ``Vault.Page`` result) and adjust further paging requests appropriately.

   使用分页的 ``QueryBy`` 和 ``TrackBy`` snapshot 视图可能会返回不同的结果集，因为每个分页的请求在底层数据库看来都属于一个独立的 SQL 查询，并且在分页请求的同事对于 state 的更新也极有可能正在发生。当使用分页的时候，经常检查 ``totalStatesAvailable`` 的值（从 ``Vault.Page`` 结构）并且适当地调整更进一步的分页请求。

其他的使用场景
------------------------

For advanced use cases that require sophisticated pagination, sorting, grouping, and aggregation functions, it is
recommended that the CorDapp developer utilise one of the many proven frameworks that ship with this capability out of
the box. Namely, implementations of JPQL (JPA Query Language) such as Hibernate for advanced SQL access, and
Spring Data for advanced pagination and ordering constructs.

对于需要复杂的分页、排序、分组和聚合方法的情况，我们建议 CorDapp 开发者使用一些具有这些功能的已经经过验证的框架。比如 JPQL (JPA Query Language) 的一些实现，比如为了高级 SQL 访问的 Hibernate，以及为了高级分页和排序构建的 Sprint Data。

The Corda Tutorials provide examples satisfying these additional Use Cases:

     1. Example CorDapp service using Vault API Custom Query to access attributes of IOU State
     2. Example CorDapp service query extension executing Named Queries via JPQL_
     3. `Advanced pagination <https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html>`_ queries using Spring Data JPA_

Corda 教程提供了满足这些情况的例子：

     1. 使用 Vault API 自定义查询来访问 IOU state 属性的 CorDapp 服务样例
     2. 通过 JPQL_ 执行命名的查询的 CorDapp 服务查询扩展的样例
     3. 使用 Spring Data JPA_ 进行 `高级分页 <https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html>`_ 查询
        
 .. _JPQL: http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#hql
 .. _JPA: https://docs.spring.io/spring-data/jpa/docs/current/reference/html

将所有者的秘钥跟外部 IDs 映射
-----------------------------------

When creating new public keys via the ``KeyManagementService``, it is possible to create an association between the newly created public
key and an external ID. This, in effect, allows CorDapp developers to group state ownership/participation keys by an account ID.

当通过 ``KeyManagementService`` 创建新的公钥的时候，在新创建的公钥和一个外部的 ID 之间创建一个关联是可能的。这就允许了 CorDapp 的开发者能够根据一个账户 ID 对 state ownership/participation 秘钥进行分组。

.. note:: This only works with freshly generated public keys and *not* the node's legal identity key. If you require that the freshly
          generated keys be for the node's identity then use ``PersistentKeyManagementService.freshKeyAndCert`` instead of ``freshKey``.
          Currently, the generation of keys for other identities is not supported.

.. note:: 这个仅仅适用于新生成的公钥而 *不是* 节点的 legal identity key。如果你需要这个新生成的秘钥被用作节点的 identity 的话，那么使用 ``PersistentKeyManagementService.freshKeyAndCert`` 而不是 ``freshKey``。当前，生成其他的 identities 的秘钥是不支持的。

The code snippet below show how keys can be associated with an external ID by using the exposed JPA functionality:

表现的代码显示了通过使用暴露的 JPA 方法如何把一个秘钥同一个外部 ID 关联在一起：

.. container:: codeset

    .. sourcecode:: java

        public AnonymousParty freshKeyForExternalId(UUID externalId, ServiceHub services) {
            // Create a fresh key pair and return the public key.
            AnonymousParty anonymousParty = freshKey();
            // Associate the fresh key to an external ID.
            services.withEntityManager(entityManager -> {
                PersistentKeyManagementService.PublicKeyHashToExternalId mapping = PersistentKeyManagementService.PublicKeyHashToExternalId(externalId, anonymousParty.owningKey);
                entityManager.persist(mapping);
                return null;
            });
            return anonymousParty;
        }

    .. sourcecode:: kotlin

        fun freshKeyForExternalId(externalId: UUID, services: ServiceHub): AnonymousParty {
            // Create a fresh key pair and return the public key.
            val anonymousParty = freshKey()
            // Associate the fresh key to an external ID.
            services.withEntityManager {
                val mapping = PersistentKeyManagementService.PublicKeyHashToExternalId(externalId, anonymousParty.owningKey)
                persist(mapping)
            }
            return anonymousParty
        }

As can be seen in the code snippet above, the ``PublicKeyHashToExternalId`` entity has been added to ``PersistentKeyManagementService``,
which allows you to associate your public keys with external IDs. So far, so good.

像上边的代码中看到的那样，``PublicKeyHashToExternalId`` entity 被添加到了 ``PersistentKeyManagementService``，这就允许你能够把你的公钥跟一个外部的 IDs 关联起来。

.. note:: Here, it is worth noting that we must map **owning keys** to external IDs, as opposed to **state objects**. This is because it
          might be the case that a ``LinearState`` is owned by two public keys generated by the same node.

.. note:: 这里，我们必须要把 **owning keys** 映射到 外部的 IDs 是没有意义的，因为它与 **state objects** 相反。这是因为可能一个 ``LinearState`` 会被同一节点生成的两个公钥同时拥有。

The intuition here is that when these public keys are used to own or participate in a state object, it is trivial to then associate those
states with a particular external ID. Behind the scenes, when states are persisted to the vault, the owning keys for each state are
persisted to a ``PersistentParty`` table. The ``PersistentParty`` table can be joined with the ``PublicKeyHashToExternalId`` table to create
a view which maps each state to one or more external IDs. The entity relationship diagram below helps to explain how this works.

很直观地能看到，当这些公钥被用来拥有或者参与一个 state 对象的时候，将这些 states 关联到一个特定的外部 ID 就没有太大的意义了。在这些场景的背后，当 states 被持久化到 vault 的时候，每个 state 的 owning keys 都会被持久化到 ``PersistentParty`` 表，这就创建了一个将每个 state 映射到一个或多个外部 IDs 的视图。下边的 Entity 关系图能够帮助解释这些是怎么工作的。

.. image:: resources/state-to-external-id.png

When performing a vault query, it is now possible to query for states by external ID using a custom query criteria.

当进行一次 vault 查询的时候，现在使用一个自定义的查询条件来通过外部 ID 查询 states 是可能的。

.. container:: codeset

   .. sourcecode:: java

        UUID id = someExternalId;
        FieldInfo externalIdField = getField("externalId", VaultSchemaV1.StateToExternalId.class);
        CriteriaExpression externalId = Builder.equal(externalIdField, id);
        QueryCriteria query = new VaultCustomQueryCriteria(externalId);
        Vault.Page<StateType> results = vaultService.queryBy(StateType.class, query);

   .. sourcecode:: kotlin

        val id: UUID = someExternalId
        val externalId = builder { VaultSchemaV1.StateToExternalId::externalId.equal(id) }
        val queryCriteria = QueryCriteria.VaultCustomQueryCriteria(externalId)
        val results = vaultService.queryBy<StateType>(queryCriteria).states

The ``VaultCustomQueryCriteria`` can also be combined with other query criteria, like custom schemas, for instance. See the vault query API
examples above for how to combine ``QueryCriteria``.

``VaultCustomQueryCriteria`` 也能够跟其他的查询条件合并使用，比如像自定义 schemas。查看上边的 vault 查询 API 例子来了解如何合并 ``QueryCriteria``。