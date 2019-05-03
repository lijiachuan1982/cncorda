.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

API: 持久化
================

.. contents::

Corda offers developers the option to expose all or some parts of a contract state to an *Object Relational Mapping*
(ORM) tool to be persisted in a *Relational Database Management System* (RDBMS).

Corda 为开发者提供了一种方式来将 contract state 的全部或部分暴露给一个 *对象关系映射*(ORM) 工具来将其持久化到一个 *关系型数据库管理系统*（RDBMS） 中。

The purpose of this, is to assist :doc:`key-concepts-vault`
development and allow for the persistence of state data to a custom database table. Persisted states held in the
vault are indexed for the purposes of executing queries. This also allows for relational joins between Corda tables
and the organization's existing data.

这样做的目的是帮助 :doc:`key-concepts-vault` 开发以及允许将 state 数据进行持久化到自定义的数据库表中。对 vault 中存储的持久化的 state 为了执行查询的目的而建立有效的索引。这也允许了在 Corda 表和组织已经存在的数据之间进行关联。

The Object Relational Mapping is specified using `Java Persistence API <https://en.wikipedia.org/wiki/Java_Persistence_API>`_
(JPA) annotations. This mapping is persisted to the database as a table row (a single, implicitly structured data item) by the node
automatically every time a state is recorded in the node's local vault as part of a transaction.

对象关系映射是通过使用 `Java Persistence API <https://en.wikipedia.org/wiki/Java_Persistence_API>`_ (JPA) 的注解来指定的，当每次一个 state 作为 transaction 的一部分被记录到本地的 vault 中的时候，它会被节点自动地转换成数据库表中的记录（一个单独的，有明确结构的数据项）。

.. note:: By default, nodes use an H2 database which is accessed using *Java Database Connectivity* JDBC. Any database
          with a JDBC driver is a candidate and several integrations have been contributed to by the community.
          Please see the info in ":doc:`node-database`" for details.

.. note:: 默认的，节点使用一个 H2 数据库，这可以通过使用 *Java Database Connectivity* JDBC 来访问。任何提供 JDBC driver 的数据库都可以作为备选项并且社区已经贡献了多个集成方式。请浏览 ":doc:`node-database`" 查看详细内容。

Schemas
-------
Every ``ContractState`` may implement the ``QueryableState`` interface if it wishes to be inserted into a custom table in the node's
database and made accessible using SQL.

如果一个 ``ContractState`` 想要被插入到节点数据库中的一个自定义表中并且可以使用 SQL 来访问的话，那么它就可能会实现 ``QueryableState`` 接口。

.. literalinclude:: ../core/src/main/kotlin/net/corda/core/schemas/PersistentTypes.kt
    :language: kotlin
    :start-after: DOCSTART QueryableState
    :end-before: DOCEND QueryableState

The ``QueryableState`` interface requires the state to enumerate the different relational schemas it supports, for
instance in situations where the schema has evolved. Each relational schema is represented as a ``MappedSchema``
object returned by the state's ``supportedSchemas`` method.

``QueryableState`` 接口要求 State 需要遍历它所支持的不同的关系型 schemas，比如在 schema 已经被更新了的情况下。每一个关系型 schema 会以由 state 的 ``supportedSchemas`` 的方法返回的 ``MappedSchema`` 对象的形式被展现。

Nodes have an internal ``SchemaService`` which decides what data to persist by selecting the ``MappedSchema`` to use.
Once a ``MappedSchema`` is selected, the ``SchemaService`` will delegate to the ``QueryableState`` to generate a corresponding
representation (mapped object) via the ``generateMappedObject`` method, the output of which is then passed to the *ORM*.

节点有一个内部的 ``SchemaService``，该服务通过使用选择的 ``MappedSchema`` 来决定过了什么会被持久化，什么不会。一旦一个 ``MappedSchema`` 被选择了之后，``SchemaService`` 将会委派 ``QueryableState`` 使用 ``generateMappedObject`` 方法来生成一个相对应的展示（mapped 对象），它的 output 接下来会被传入 *ORM*。

.. literalinclude:: ../node/src/main/kotlin/net/corda/node/services/api/SchemaService.kt
    :language: kotlin
    :start-after: DOCSTART SchemaService
    :end-before: DOCEND SchemaService

.. literalinclude:: ../core/src/main/kotlin/net/corda/core/schemas/PersistentTypes.kt
    :language: kotlin
    :start-after: DOCSTART MappedSchema
    :end-before: DOCEND MappedSchema

With this framework, the relational view of ledger states can evolve in a controlled fashion in lock-step with internal systems or other
integration points and is not dependant on changes to the contract code.

通过这个框架，ledger state 的关系型的视图能够通过一种可控的方式进行更新，同内部系统或者其他的集成点进行 lock-step，并且者不依赖于合约代码的变更。

It is expected that multiple contract state implementations might provide mappings within a single schema.
For example an Interest Rate Swap contract and an Equity OTC Option contract might both provide a mapping to
a Derivative contract within the same schema. The schemas should typically not be part of the contract itself and should exist independently
to encourage re-use of a common set within a particular business area or Cordapp.

多个不同的 contract state 实现可能会提供跟一些常规 schema 的映射，这个是被期望出现的。例如一个汇率交换合约和一个 Equity OTC Option contract 可能都提供一个跟常见的 Derivative Schema 的映射。这个 schema 通常不应该是 contract 的一部分，并且应该是独立存在的，这样会鼓励针对于特定的业务领域或者 CorDapp 可以对常用部分进行重用。

.. note:: It's advisable to avoid cross-references between different schemas as this may cause issues when evolving ``MappedSchema``
   or migrating its data. At startup, nodes log such violations as warnings stating that there's a cross-reference between ``MappedSchema``'s.
   The detailed messages incorporate information about what schemas, entities and fields are involved.

.. note:: 建议避免再不同的 schemas 之间进行引用，因为这个可能会在升级 ``MappedSchema`` 或者迁移它的数据的时候造成问题。在最开始的时候，节点会将这类冲突作为 warning log 下来，会说明这里有跨 ``MappedSchema`` 引用。详细的信息中会看到是哪些 schemas、entities 和字段。

``MappedSchema`` offer a family name that is disambiguated using Java package style name-spacing derived from the
class name of a *schema family* class that is constant across versions, allowing the ``SchemaService`` to select a
preferred version of a schema.

``MappedSchema`` 提供了一个 family name，该 family name 通过使用 Java package 形式的 name-spacing 来规避模糊的定义，这个 name-spacing 来自于一个在不同的版本中始终不变的 *schema family* 类的类名，这就允许了 ``SchemaService`` 可以选择一个喜欢的 schema 的版本。

The ``SchemaService`` is also responsible for the ``SchemaOptions`` that can be configured for a particular
``MappedSchema``. These allow the configuration of database schemas or table name prefixes to avoid clashes with
other ``MappedSchema``.

``SchemaService`` 同样也要负责提供一个 ``SchemaOptions``，这对于一个指定的 ``MappedSchema`` 是可以配置的，这就允许了对于一个数据库的 schema 或者表名前缀进行配置，以此来避免同其他的 ``MappedSchema`` 的任何冲突。

.. note:: It is intended that there should be plugin support for the ``SchemaService`` to offer version upgrading, implementation
   of additional schemas, and enable active schemas as being configurable.  The present implementation does not include these features
   and simply results in all versions of all schemas supported by a ``QueryableState`` being persisted.
   This will change in due course. Similarly, the service does not currently support
   configuring ``SchemaOptions`` but will do so in the future.

.. note:: 我们希望这里应该有对 ``SchemaService`` 的 plugin 支持，来提供版本的更新、额外的 schemas 的实现，并且能够让 active schemas 能够按照配置变为可用。但是当前的实现并没有提供这些，造成了 ``QueryableState`` 所支持的所有 schemas 的所有版本都被持久化。这会在将来被改变。类似的，当前也不支持配置 ``SchemaOptions`` 但是将来是会支持的。

注册自定义的 schema
--------------------------
Custom contract schemas are automatically registered at startup time for CorDapps. The node bootstrap process will scan for states that implement
the Queryable state interface. Tables are then created as specified by the ``MappedSchema`` identified by each state's ``supportedSchemas`` method.

自定义的 contract schemas 会在 CorDapps 启动的时候被自动注册。节点的启动过程会扫描实现了 Queryable state 接口的 states。接下来会按照 ``MappedSchema`` 指明的那样创建表，这些表会根据每个 state 的 ``supportedSchemas`` 方法被识别。

For testing purposes it is necessary to manually register the packages containing custom schemas as follows:

- Tests using ``MockNetwork`` and ``MockNode`` must explicitly register packages using the `cordappPackages` parameter of ``MockNetwork``
- Tests using ``MockServices`` must explicitly register packages using the `cordappPackages` parameter of the ``MockServices`` `makeTestDatabaseAndMockServices()` helper method.

为了测试的目的，像下边这样手动地注册包含自定义的 schemas 的包是有必要的：

- 使用 ``MockNetwork`` 和 ``MockNode`` 的测试必须要显式地使用 ``MockNode`` 的 `cordappPackages` 参数来注册包
- 使用 ``MockServices`` 的测试必须要显式地使用 ``MockServices`` `makeTestDatabaseAndMockServices()`` helper 方法的 `cordappPackages` 参数注册包

.. note:: Tests using the `DriverDSL` will automatically register your custom schemas if they are in the same project structure as the driver call.

.. note:: 使用 `DriverDSL` 的测试，如果他们在相同的项目结构下的话，当 driver 在调用的时候，他们会自动注册你的自定义的 schemas。

对象关系映射
-------------------------
To facilitate the ORM, the persisted representation of a ``QueryableState`` should be an instance of a ``PersistentState`` subclass,
constructed either by the state itself or a plugin to the ``SchemaService``. This allows the ORM layer to always
associate a ``StateRef`` with a persisted representation of a ``ContractState`` and allows joining with the set of
unconsumed states in the vault.

为了协助 ORM，一个 ``QueryableState`` 的被持久化的表述应该是一个 ``PersistentState`` 的子类的一个实例，可以通过 state 本身或者 ``SchemaService`` 的 plugin 来构建。这就允许了 ORM 层永远会将一个 ``ContractState`` 的持久化表现和一个 ``StateRef`` 相关联，并且允许把 vault 中的不同的未消费的 states 进行 joining。

The ``PersistentState`` subclass should be marked up as a JPA 2.1 *Entity* with a defined table name and having
properties (in Kotlin, getters/setters in Java) annotated to map to the appropriate columns and SQL types. Additional
entities can be included to model these properties where they are more complex, for example collections (:ref:`Persisting Hierarchical Data<persisting-hierarchical-data>`), so
the mapping does not have to be *flat*. The ``MappedSchema`` constructor accepts a list of all JPA entity classes for that schema in
the ``MappedTypes`` parameter. It must provide this list in order to initialise the ORM layer.

``PersistentState`` 子类应该被定义为一个 JPA 2.1 *Entity*，这个 entity 应该有一个定义好的 table name 并且还应该有一些属性（Kotlin 中是属性，Java 中是 getters/setters）来映射成为合适的列和 SQL 类型。其他的 entities 可以被包含来形成一些复杂的属性，比如集合（:ref:`持久化结构化数据<persisting-hierarchical-data>`），所以这个映射可以不是 *扁平* 的。``MappedSchema`` 构造函数接收一个在 ``MappedTypes`` 参数中的 schema 的所有 JPA entity 类的列表。必须要提供跟这个列表，以此来初始化这个 ORM 层。

Several examples of entities and mappings are provided in the codebase, including ``Cash.State`` and
``CommercialPaper.State``. For example, here's the first version of the cash schema.

基础代码中提供了一些 entities 和映射的例子，包括 ``Cash.State`` 和 ``CommercialPaper.State``。例如，下边是 cash schema 的第一个版本。

.. container:: codeset

    .. literalinclude:: ../finance/contracts/src/main/kotlin/net/corda/finance/schemas/CashSchemaV1.kt
        :language: kotlin

.. note:: If Cordapp needs to be portable between Corda OS (running against H2) and Corda Enterprise (running against a standalone database),
          consider database vendors specific requirements.
          Ensure that table and column names are compatible with the naming convention of the database vendors for which the Cordapp will be deployed,
          e.g. for Oracle database, prior to version 12.2 the maximum length of table/column name is 30 bytes (the exact number of characters depends on the database encoding).

.. note:: 如果 CorDapp 需要在 Corda 开源版本（运行着 H2）和 Corda 企业版（运行着一个独立的数据库）之间可导入导出的话，需要考虑数据库提供商特殊的需求。确保表和列的名字能够兼容 CorDapp 将要被部署的数据库的 vendors 的命名规则，比如对于 Oracle 数据库，在 12.2 之前的版本，表/列名字的最大长度是 30 字节（确切的字符数量依赖于数据库的编码方式）。

持久化结构化数据
----------------------------

You may wish to persist hierarchical relationships within state data using multiple database tables. In order to facillitate this, multiple ``PersistentState``
subclasses may be implemented. The relationship between these classes is defined using JPA annotations. It is important to note that the ``MappedSchema``
constructor requires a list of *all* of these subclasses.

你可能想使用多个数据库表来持久化 state 数据中的结构化的关系。为了协助这个，多个 ``PersistentState`` 子类可能会被实现。这些类之间的关系是通过使用 JPA 注解的方式来定义的。需要注意的是 ``MappedSchema`` 构造函数需要 *所有* 这些子类的列表。

An example Schema implementing hierarchical relationships with JPA annotations has been implemented below. This Schema will cause ``parent_data`` and ``child_data`` tables to be
created.

下边是一个使用 JPA 注解来实现结构化关系的 Schema 实现。这个 Schema 将会创建 ``parent_data`` 和 ``child_data`` 表。

.. container:: codeset

    .. sourcecode:: java

        @CordaSerializable
        public class SchemaV1 extends MappedSchema {

            /**
             * This class must extend the MappedSchema class. Its name is based on the SchemaFamily name and the associated version number abbreviation (V1, V2... Vn).
             * In the constructor, use the super keyword to call the constructor of MappedSchema with the following arguments: a class literal representing the schema family,
             * a version number and a collection of mappedTypes (class literals) which represent JPA entity classes that the ORM layer needs to be configured with for this schema.
             */

            public SchemaV1() {
                super(Schema.class, 1, ImmutableList.of(PersistentParentToken.class, PersistentChildToken.class));
            }

            /**
             * The @entity annotation signifies that the specified POJO class' non-transient fields should be persisted to a relational database using the services
             * of an entity manager. The @table annotation specifies properties of the table that will be created to contain the persisted data, in this case we have
             * specified a name argument which will be used the table's title.
             */

            @Entity
            @Table(name = "parent_data")
            public static class PersistentParentToken extends PersistentState {

                /**
                 * The @Column annotations specify the columns that will comprise the inserted table and specify the shape of the fields and associated
                 * data types of each database entry.
                 */

                @Column(name = "owner") private final String owner;
                @Column(name = "issuer") private final String issuer;
                @Column(name = "amount") private final int amount;
                @Column(name = "linear_id") public final UUID linearId;

                /**
                 * The @OneToMany annotation specifies a one-to-many relationship between this class and a collection included as a field.
                 * The @JoinColumn and @JoinColumns annotations specify on which columns these tables will be joined on.
                 */

                @OneToMany(cascade = CascadeType.PERSIST)
                @JoinColumns({
                        @JoinColumn(name = "output_index", referencedColumnName = "output_index"),
                        @JoinColumn(name = "transaction_id", referencedColumnName = "transaction_id"),
                })
                private final List<PersistentChildToken> listOfPersistentChildTokens;

                public PersistentParentToken(String owner, String issuer, int amount, UUID linearId, List<PersistentChildToken> listOfPersistentChildTokens) {
                    this.owner = owner;
                    this.issuer = issuer;
                    this.amount = amount;
                    this.linearId = linearId;
                    this.listOfPersistentChildTokens = listOfPersistentChildTokens;
                }

                // Default constructor required by hibernate.
                public PersistentParentToken() {
                    this.owner = "";
                    this.issuer = "";
                    this.amount = 0;
                    this.linearId = UUID.randomUUID();
                    this.listOfPersistentChildTokens = null;
                }

                public String getOwner() {
                    return owner;
                }

                public String getIssuer() {
                    return issuer;
                }

                public int getAmount() {
                    return amount;
                }

                public UUID getLinearId() {
                    return linearId;
                }

                public List<PersistentChildToken> getChildTokens() { return listOfPersistentChildTokens; }
            }

            @Entity
            @CordaSerializable
            @Table(name = "child_data")
            public static class PersistentChildToken {
                // The @Id annotation marks this field as the primary key of the persisted entity.
                @Id
                private final UUID Id;
                @Column(name = "owner")
                private final String owner;
                @Column(name = "issuer")
                private final String issuer;
                @Column(name = "amount")
                private final int amount;

                /**
                 * The @ManyToOne annotation specifies that this class will be present as a member of a collection on a parent class and that it should
                 * be persisted with the joining columns specified in the parent class. It is important to note the targetEntity parameter which should correspond
                 * to a class literal of the parent class.
                 */

                @ManyToOne(targetEntity = PersistentParentToken.class)
                private final TokenState persistentParentToken;


                public PersistentChildToken(String owner, String issuer, int amount) {
                    this.Id = UUID.randomUUID();
                    this.owner = owner;
                    this.issuer = issuer;
                    this.amount = amount;
                    this.persistentParentToken = null;
                }

                // Default constructor required by hibernate.
                public PersistentChildToken() {
                    this.Id = UUID.randomUUID();
                    this.owner = "";
                    this.issuer = "";
                    this.amount = 0;
                    this.persistentParentToken = null;
                }

                public UUID getId() {
                    return Id;
                }

                public String getOwner() {
                    return owner;
                }

                public String getIssuer() {
                    return issuer;
                }

                public int getAmount() {
                    return amount;
                }

                public TokenState getPersistentToken() {
                    return persistentToken;
                }

            }

        }

    .. sourcecode:: kotlin

        @CordaSerializable
        object SchemaV1 : MappedSchema(schemaFamily = Schema::class.java, version = 1, mappedTypes = listOf(PersistentParentToken::class.java, PersistentChildToken::class.java)) {

            @Entity
            @Table(name = "parent_data")
            class PersistentParentToken(
                    @Column(name = "owner")
                    var owner: String,

                    @Column(name = "issuer")
                    var issuer: String,

                    @Column(name = "amount")
                    var currency: Int,

                    @Column(name = "linear_id")
                    var linear_id: UUID,

                     @JoinColumns(JoinColumn(name = "transaction_id", referencedColumnName = "transaction_id"), JoinColumn(name = "output_index", referencedColumnName = "output_index"))

                    var listOfPersistentChildTokens: MutableList<PersistentChildToken>
            ) : PersistentState()

            @Entity
            @CordaSerializable
            @Table(name = "child_data")
            class PersistentChildToken(
                    @Id
                    var Id: UUID = UUID.randomUUID(),

                    @Column(name = "owner")
                    var owner: String,

                    @Column(name = "issuer")
                    var issuer: String,

                    @Column(name = "amount")
                    var currency: Int,

                    @Column(name = "linear_id")
                    var linear_id: UUID,

                    @ManyToOne(targetEntity = PersistentParentToken::class)
                    var persistentParentToken: TokenState

            ) : PersistentState()


身份信息映射
----------------
Schema entity attributes defined by identity types (``AbstractParty``, ``Party``, ``AnonymousParty``) are automatically
processed to ensure only the ``X500Name`` of the identity is persisted where an identity is well known, otherwise a null
value is stored in the associated column. To preserve privacy, identity keys are never persisted. Developers should use
the ``IdentityService`` to resolve keys from well know X500 identity names.

由 identity 类型定义的 schema entity 属性（``AbstractParty``，``Party``，``AnonymousParty``）会被自动处理来确保当一个 identity 是 well known 的时候，只有 ``X500Name`` 的身份信息会被持久化，否则一个 null 值会被存储到相关的 column 中。为了保持隐私性，identity keys 从来不会被持久化。开发者应该使用 ··IdentityService·· 来从 well know X500 identity names 中找到 keys。

.. _jdbc_session_ref:

JDBC session
------------
Apps may also interact directly with the underlying Node's database by using a standard
JDBC connection (session) as described by the `Java SQL Connection API <https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html>`_

Apps 也有可能会使用一个像 `Java SQL Connection API <https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html>`_ 中描述的那种标准的 JDBC 连接（session）来直接地跟节点底层的数据库进行交互。

Use the ``ServiceHub`` ``jdbcSession`` function to obtain a JDBC connection as illustrated in the following example:

使用 ``ServiceHub`` ``jdbcSession`` 方法像下边这样获得一个 JDBC 连接：

.. literalinclude:: ../node/src/test/kotlin/net/corda/node/services/persistence/HibernateConfigurationTest.kt
  :language: kotlin
  :start-after: DOCSTART JdbcSession
  :end-before: DOCEND JdbcSession

JDBC sessions can be used in flows and services (see ":doc:`flow-state-machines`").

JDBC session 可以在 Flows 和 Service Plugins 中被使用（查看 :doc:`flow-state-machines`）。

The following example illustrates the creation of a custom Corda service using a ``jdbcSession``:

下边的例子展示了使用一个 ``jdbcSession`` 来创建一个自定义的 corda service：

.. literalinclude:: ./example-code/src/main/kotlin/net/corda/docs/kotlin/vault/CustomVaultQuery.kt
  :language: kotlin
  :start-after: DOCSTART CustomVaultQuery
  :end-before: DOCEND CustomVaultQuery

which is then referenced within a custom flow:

它会像下边这样在一个自定义的 flow 里被引用：

.. literalinclude:: ./example-code/src/main/kotlin/net/corda/docs/kotlin/vault/CustomVaultQuery.kt
  :language: kotlin
  :start-after: DOCSTART TopupIssuer
  :end-before: DOCEND TopupIssuer

For examples on testing ``@CordaService`` implementations, see the oracle example :doc:`here <oracles>`.

例如，当测试 ``@CordaService`` 实现的时候，查看 oracle 的例子 :doc:`here <oracles>`。

JPA 支持
-----------
In addition to ``jdbcSession``, ``ServiceHub`` also exposes the Java Persistence API to flows via the ``withEntityManager``
method. This method can be used to persist and query entities which inherit from ``MappedSchema``. This is particularly
useful if off-ledger data must be maintained in conjunction with on-ledger state data.

除了 ``jdbcSession``，``ServiceHub`` 也通过 ``withEntityManager`` 方法向 flows 暴露了 Java 持久化 API。这个方法可以用来持久化和查询继承自 ``MappedSchema`` 的 entities。这对于 off-ledger 数据必须同 on-ledger state 数据共同维护的情况是非常有用的。

    .. note:: Your entity must be included as a mappedType as part of a ``MappedSchema`` for it to be added to Hibernate
              as a custom schema. If it's not included as a mappedType, a corresponding table will not be created. See Samples below.

.. note:: 你的 entity 必须以一个 mappedType 的方式被包含并作为一个 ``MappedSchema`` 的一部分，以此它会作为一个自定义的 schema 被添加到 Hibernate。如果它没有作为一个 mappedType 被包含的话，一个相对应的表示不会被创建的。看看下边的例子。

The code snippet below defines a ``PersistentFoo`` type inside ``FooSchemaV1``. Note that ``PersistentFoo`` is added to
a list of mapped types which is passed to ``MappedSchema``. This is exactly how state schemas are defined, except that
the entity in this case should not subclass ``PersistentState`` (as it is not a state object). See examples:

下边的代码在 ``FooSchemaV1`` 里定义了一个 ``PersistentFoo`` 类型。需要注意的是，``PersistentFoo`` 被添加到了一个会被发送给 ``MappedSchema`` 的 mapped 类型的列表中。这就是 state schemas 是如何被定义的，除了这个情况里的 entity 不应该作为 ``PersistentState`` 的子类（因为它不是一个 state 对象）。

.. container:: codeset

    .. sourcecode:: java

        public class FooSchema {}

        public class FooSchemaV1 extends MappedSchema {
            FooSchemaV1() {
                super(FooSchema.class, 1, ImmutableList.of(PersistentFoo.class));
            }

            @Entity
            @Table(name = "foos")
            class PersistentFoo implements Serializable {
                @Id
                @Column(name = "foo_id")
                String fooId;

                @Column(name = "foo_data")
                String fooData;
            }
        }

    .. sourcecode:: kotlin

        object FooSchema

        object FooSchemaV1 : MappedSchema(schemaFamily = FooSchema.javaClass, version = 1, mappedTypes = listOf(PersistentFoo::class.java)) {
            @Entity
            @Table(name = "foos")
            class PersistentFoo(@Id @Column(name = "foo_id") var fooId: String, @Column(name = "foo_data") var fooData: String) : Serializable
        }

Instances of ``PersistentFoo`` can be manually persisted inside a flow as follows:

.. container:: codeset

    .. sourcecode:: java

        PersistentFoo foo = new PersistentFoo(new UniqueIdentifier().getId().toString(), "Bar");
        serviceHub.withEntityManager(entityManager -> {
            entityManager.persist(foo);
            return null;
        });

    .. sourcecode:: kotlin

        val foo = FooSchemaV1.PersistentFoo(UniqueIdentifier().id.toString(), "Bar")
        serviceHub.withEntityManager {
            persist(foo)
        }

And retrieved via a query, as follows:

.. container:: codeset

    .. sourcecode:: java

        node.getServices().withEntityManager((EntityManager entityManager) -> {
            CriteriaQuery<PersistentFoo> query = entityManager.getCriteriaBuilder().createQuery(PersistentFoo.class);
            Root<PersistentFoo> type = query.from(PersistentFoo.class);
            query.select(type);
            return entityManager.createQuery(query).getResultList();
        });

    .. sourcecode:: kotlin

        val result: MutableList<FooSchemaV1.PersistentFoo> = services.withEntityManager {
            val query = criteriaBuilder.createQuery(FooSchemaV1.PersistentFoo::class.java)
            val type = query.from(FooSchemaV1.PersistentFoo::class.java)
            query.select(type)
            createQuery(query).resultList
        }

Please note that suspendable flow operations such as:

注意可挂起的 flow 操作，比如：

* ``FlowSession.send``
* ``FlowSession.receive``
* ``FlowLogic.receiveAll``
* ``FlowLogic.sleep``
* ``FlowLogic.subFlow``

Cannot be used within the lambda function passed to ``withEntityManager``.

不能够在传递给 ``withEntityManager`` 的 lambda 方法中使用。
