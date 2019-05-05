节点的配置
==================

.. contents::

配置文件路径
---------------------------
When starting a node, the ``corda.jar`` file defaults to reading the node's configuration from a ``node.conf`` file in the directory from which the command to launch Corda is executed.
There are two command-line options to override this behaviour:

* The ``--config-file`` command line option allows you to specify a configuration file with a different name, or in a different file location.
  Paths are relative to the current working directory

* The ``--base-directory`` command line option allows you to specify the node's workspace location.
  A ``node.conf`` configuration file is then expected in the root of this workspace.

当启动一个节点的时候，``corda.jar`` 默认地会去从加载 Corda 的命令行所在的路径下的一个 ``node.conf`` 中读取节点的配置信息。有两个命令行可选项能够重写这个行为：

* ``--config-file`` 命令行可选项允许你指定一个不同名字或者存放在不同位置的配置文件。路径是当前工作路径的相对路径
* ``--base-directory`` 命令行可选项允许你指定节点的工作路径。一个 ``node.conf`` 配置文件应该在工作空间的根路径下。

If you specify both command line arguments at the same time, the node will fail to start.

如果你同时指定了两个命令行参数，节点将会启动失败。

配置文件格式
-------------------------
The Corda configuration file uses the HOCON format which is a superset of JSON. Please visit
`<https://github.com/typesafehub/config/blob/master/HOCON.md>`_ for further details.

Corda 配置文件使用 HOCON 格式，它是 JSON 的 superset。它有一些特性对于配置文件的格式有很多好处。浏览 `<https://github.com/typesafehub/config/blob/master/HOCON.md>`_ 了解更多。

Please do NOT use double quotes (``"``) in configuration keys.

在配置 keys 中请不要使用双引号 (``"``)。

Node setup will log ``Config files should not contain " in property names. Please fix: [key]`` as an error when it finds double quotes around keys.
This prevents configuration errors when mixing keys containing ``.`` wrapped with double quotes and without them e.g.: The property
``"dataSourceProperties.dataSourceClassName" = "val"`` in `Reference.conf`_ would be not overwritten by the property
``dataSourceProperties.dataSourceClassName = "val2"`` in *node.conf*.

当发现在 keys 中含有双引号的时候，节点启动将会 log ``Config files should not contain " in property names. Please fix: [key]`` 的错误。这个可以避免当 keys 包含使用或者不适用双引号来包括 ``.``的混合 keys 的时候的错误，比如在 `Reference.conf`_ 中的 ``"dataSourceProperties.dataSourceClassName" = "val"`` 属性不会被在 *node.conf* 中的 ``dataSourceProperties.dataSourceClassName = "val2"`` 属性所重写。

By default the node will fail to start in presence of unknown property keys.
To alter this behaviour, the ``on-unknown-config-keys`` command-line argument can be set to ``IGNORE`` (default is ``FAIL``).

如果出现了一些未知的属性 keys 的话，默认地，节点会启动失败。

重写 node.conf 里的值
--------------------------------

Environment variables
  For example: ``${NODE_TRUST_STORE_PASSWORD}`` would be replaced by the contents of environment variable ``NODE_TRUST_STORE_PASSWORD`` (see: :ref:`hiding-sensitive-data` section).

环境变量
  比如 ``${NODE_TRUST_STORE_PASSWORD}`` 将会被环境变量 ``NODE_TRUST_STORE_PASSWORD`` 的内容所替换（查看 :ref:`hiding-sensitive-data` 部分）。

JVM options
  JVM options or environmental variables prefixed with ``corda.`` can override ``node.conf`` fields.
  Provided system properties can also set values for absent fields in ``node.conf``.
  This is an example of adding/overriding the keyStore password :

JVM 选项
  JVM 选项或者以 ``corda`` 开始的环境变量能够重载 ``node.conf`` 中的字段。提供的系统属性同样也可以设置在 ``node.conf`` 中缺少的字段。下边是一个添加/重载 keyStore 密码的例子：

  .. sourcecode:: shell

    java -Dcorda.rpcSettings.ssl.keyStorePassword=mypassword -jar node.jar

配置文件字段
-------------------------

.. note :: The available configuration fields are listed below in alphabetic order.

.. note :: 可用的配置字段按照字母顺序排列：

.. _corda-configuration-file-fields:

additionalP2PAddresses
  An array of additional host:port values, which will be included in the advertised NodeInfo in the network map in addition to the :ref:`p2pAddress <corda_configuration_file_p2pAddress>`.
  Nodes can use this configuration option to advertise HA endpoints and aliases to external parties.

  一个额外的 host:port 值的数组，这些会被包含在网络地图中除了 :ref:`p2pAddress <corda_configuration_file_p2pAddress>` 以外的 advertised NodeInfo 中。

  *Default:* empty list

attachmentContentCacheSizeMegaBytes
  Optionally specify how much memory should be used to cache attachment contents in memory.

  可选的设置使用多少内存来 cache attachment 内容。

  *Default:* 10MB

attachmentCacheBound
  Optionally specify how many attachments should be cached locally. Note that this includes only the key and metadata, the content is cached separately and can be loaded lazily.

  可选的设置应该有多少 attachments 应该被 cache 到本地。注意这个仅仅包含 key 和 metadata，内容是被单独 cache 并且可以被懒加载。

  *Default:* 1024


compatibilityZoneURL (deprecated)
  The root address of the Corda compatibility zone network management services, it is used by the Corda node to register with the network and obtain a Corda node certificate, (See :doc:`permissioning` for more information.) and also is used by the node to obtain network map information.
  Cannot be set at the same time as the :ref:`networkServices <corda_configuration_file_networkServices>` option.

  **Important:  old configuration value, please use networkServices**

  **重要：这是一个旧的配置值，请使用 networkServices**

  *Default:* not defined

.. _corda_configuration_file_signer_blacklist:

cordappSignerKeyFingerprintBlacklist
  List of the public keys fingerprints (SHA-256 of public key hash) not allowed as Cordapp JARs signers.
  The node will not load Cordapps signed by those keys.
  The option takes effect only in production mode and defaults to Corda development keys (``["56CA54E803CB87C8472EBD3FBC6A2F1876E814CEEBF74860BD46997F40729367", "83088052AF16700457AE2C978A7D8AC38DD6A7C713539D00B897CD03A5E5D31D"]``), in development mode any key is allowed to sign Cordpapp JARs.

  *Default:* not defined

crlCheckSoftFail
  This is a boolean flag that when enabled (i.e. ``true`` value is set) causes certificate revocation list (CRL) checking to use soft fail mode.
  Soft fail mode allows the revocation check to succeed if the revocation status cannot be determined because of a network error.
  If this parameter is set to ``false`` rigorous CRL checking takes place. This involves each certificate in the certificate path being checked for a CRL distribution point extension, and that this extension points to a URL serving a valid CRL.
  This means that if any CRL URL in the certificate path is inaccessible, the connection with the other party will fail and be marked as bad.
  Additionally, if any certificate in the hierarchy, including the self-generated node SSL certificate, is missing a valid CRL URL, then the certificate path will be marked as invalid.

  *Default:* true

custom
  Set custom command line attributes (e.g. Java system properties) on the node process via the capsule launcher

  jvmArgs:
      A list of JVM arguments to apply to the node process. This removes any defaults specified from ``corda.jar``, but can be overridden from the command line.
      See :ref:`setting_jvm_args` for examples and details on the precedence of the different approaches to settings arguments.

      *Default:* not defined

.. _database_properties_ref:

database
  Database configuration

  transactionIsolationLevel:
    Transaction isolation level as defined by the ``TRANSACTION_`` constants in ``java.sql.Connection``, but without the ``TRANSACTION_`` prefix.

    就像在 ``java.sql.Connection`` 中由 ``TRANSACTION_`` 常量定义的事务隔离级别（transaction isolation level），但是没有 ``TRANSACTION_`` 前缀。

    *Default:* ``REPEATABLE_READ``

  exportHibernateJMXStatistics:
    Whether to export Hibernate JMX statistics.

    是否导出 Hibernate JMX statistics

    **Caution: enabling this option causes expensive run-time overhead**

    **注意：开启这个选项会造成昂贵的 run-time overhead**

    *Default:* false

  initialiseSchema
    Boolean which indicates whether to update the database schema at startup (or create the schema when node starts for the first time).
    If set to ``false`` on startup, the node will validate if it's running against a compatible database schema.

    *Default:* true

  initialiseAppSchema
    The property allows to override ``database.initialiseSchema`` for the Hibernate DDL generation for CorDapp schemas.
    ``UPDATE`` performs an update of CorDapp schemas, while ``VALID`` only verifies their integrity and ``NONE`` performs no check.
    When ``initialiseSchema`` is set to ``false``, then ``initialiseAppSchema`` may be set as ``VALID`` or ``NONE`` only.

    *Default:* CorDapp schema creation is controlled with ``initialiseSchema``.

dataSourceProperties
  This section is used to configure the JDBC connection and database driver used for the node's persistence.
  :ref:`Node database <standalone_database_config_examples_ref>` contains example configurations for other database providers.
  To add additional data source properties (for a specific JDBC driver) use the ``dataSource.`` prefix with the property name (e.g. `dataSource.customProperty = value`).

  这部分是用来配置 JDBC 连接和数据库驱动的，用来对节点数据进行持久化处理。:ref:`Node database <standalone_database_config_examples_ref>` 包含了对于其他的数据库 driver 的例子。使用 ``dataSource.`` 前缀加上属性名字（比如 `dataSource.customProperty = value`）来添加额外的数据源属性（对于一个指定的 JDBC driver）。

  dataSourceClassName
    JDBC Data Source class name.

  dataSource.url
    JDBC database URL.

  dataSource.user
    Database user.

  dataSource.password
    Database password.

  *Default:*

  .. parsed-literal::

    dataSourceClassName = org.h2.jdbcx.JdbcDataSource
    dataSource.url = "jdbc:h2:file:"${baseDirectory}"/persistence;DB_CLOSE_ON_EXIT=FALSE;WRITE_DELAY=0;LOCK_TIMEOUT=10000"
    dataSource.user = sa
    dataSource.password = ""

detectPublicIp
  This flag toggles the auto IP detection behaviour.
  If enabled, on startup the node will attempt to discover its externally visible IP address first by looking for any public addresses on its network interfaces, and then by sending an IP discovery request to the network map service.
  Set to ``true`` to enable.

  这个标志值开启/关闭了是否自动发现 IP 的功能，默认是开启的。当节点启动的时候，它会尝试通过在它的网络接口上查找公用地址的方式去发现自己的外部可见的 IP 地址，然后会向 network map service 发送一个 IP 发现请求。将它设置为 ``true`` 来开启这个功能。

  *Default:* false

devMode
  This flag sets the node to run in development mode.
  On startup, if the keystore ``<workspace>/certificates/sslkeystore.jks``
  does not exist, a developer keystore will be used if ``devMode`` is true.
  The node will exit if ``devMode`` is false and the keystore does not exist.
  ``devMode`` also turns on background checking of flow checkpoints to shake out any bugs in the checkpointing process.
  Also, if ``devMode`` is true, Hibernate will try to automatically create the schema required by Corda or update an existing schema in the SQL database; if ``devMode`` is false, Hibernate will simply validate the existing schema, failing on node start if the schema is either not present or not compatible.
  If no value is specified in the node configuration file, the node will attempt to detect if it's running on a developer machine and set ``devMode=true`` in that case.
  This value can be overridden from the command line using the ``--dev-mode`` option.

  这个标志设定了节点是否是在开发模式下运行。在节点启动的时候，如果 ``<workspace>/certificates/sslkeystore.jks`` 的 keystore 文件不存在的话，如果 ``devMode`` 是 true 的话，那么一个开发者 keystore 会被使用。如果 ``devMode`` 设置为 false 并且 keystore 不存在的话，那么节点就会退出。 ``devMode`` 同样也会打开后台对 flow checkpoints 的检查，来找到在 checkpointing 流程中存在的 bugs。并且，如果 ``devMode`` 是 true 的话，Hibernate 会在 SQL 数据库中尝试自动地创建 Corda 要求的 schema 或者更新一个已经存在的 schema。如果 ``devMode`` 是 false 的话，Hibernate 会简单地验证一个已经存在的 schema，如果这个 schema 不存在或者不兼容的话，那么节点就会启动失败。如果在节点配置文件中没有指定值的话，节点就会尝试发现节点是否运行在一个开发者的机器上，如果是的话会设置 ``devMode=true``。

  *Default:* Corda will try to establish based on OS environment

devModeOptions
  Allows modification of certain ``devMode`` features

  **Important: This is an unsupported configuration.**

  allowCompatibilityZone
    Allows a node configured to operate in development mode to connect to a compatibility zone.

    *Default:* not defined


emailAddress
  The email address responsible for node administration, used by the Compatibility Zone administrator.

  *Default:* company@example.com

extraNetworkMapKeys
  An optional list of private network map UUIDs. Your node will fetch the public network and private network maps based on these keys.
  Private network UUID should be provided by network operator and lets you see nodes not visible on public network.

  **Important: This is a temporary feature for onboarding network participants that limits their visibility for privacy reasons.**

  *Default:* not defined


flowMonitorPeriodMillis
  Duration of the period suspended flows waiting for IO are logged.

  *Default:* 60 seconds

flowMonitorSuspensionLoggingThresholdMillis
  Threshold duration suspended flows waiting for IO need to exceed before they are logged.

  *Default:* 60 seconds

flowTimeout
  When a flow implementing the ``TimedFlow`` interface and setting the ``isTimeoutEnabled`` flag does not complete within a defined elapsed time, it is restarted from the initial checkpoint.
  Currently only used for notarisation requests with clustered notaries: if a notary cluster member dies while processing a notarisation request, the client flow eventually times out and gets restarted.
  On restart the request is resent to a different notary cluster member in a round-robin fashion. Note that the flow will keep retrying forever.

  timeout
    The initial flow timeout period.

    *Default:* 30 seconds

  maxRestartCount
    The number of retries the back-off time keeps growing for.
    For subsequent retries, the timeout value will remain constant.

    *Default:* 6

  backoffBase
    The base of the exponential backoff, `t_{wait} = timeout * backoffBase^{retryCount}`

    *Default:* 1.8

h2Port (deprecated)
  Defines port for h2 DB.

  **Important: Deprecated please use h2Setting instead**

h2Settings
  Sets the H2 JDBC server host and port.
  See :doc:`node-database-access-h2`.
  For non-localhost address the database password needs to be set in ``dataSourceProperties``.

  *Default:* NULL

jarDirs
  An optional list of file system directories containing JARs to include in the classpath when launching via ``corda.jar`` only.
  Each should be a string.
  Only the JARs in the directories are added, not the directories themselves.
  This is useful for including JDBC drivers and the like. e.g. ``jarDirs = [ ${baseDirectory}"/libs" ]``.
  (Note that you have to use the ``baseDirectory`` substitution value when pointing to a relative path).

  一个可选的包含 JARs 的文件系统路径列表，仅仅在通过 ``corda.jar`` 加载的时候会被包含在 classpath 中。每个应该是字符串。只有在路径下的 JARs 才会被加载，而不是路径本身。这个对于包括 JDBC drivers 的时候会很有用。比如： ``jarDirs = [ ${baseDirectory}"/libs" ]``。（注意当指向一个相对路径的时候，你需要使用 ``baseDirectory`` 的替代值）

  *Default:* not defined

jmxMonitoringHttpPort
  If set, will enable JMX metrics reporting via the Jolokia HTTP/JSON agent on the corresponding port.
  Default Jolokia access url is http://127.0.0.1:port/jolokia/

  *Default:* not defined

jmxReporterType
  Provides an option for registering an alternative JMX reporter.
  Available options are ``JOLOKIA`` and ``NEW_RELIC``.

  The Jolokia configuration is provided by default.
  The New Relic configuration leverages the Dropwizard_ NewRelicReporter solution.
  See `Introduction to New Relic for Java`_ for details on how to get started and how to install the New Relic Java agent.

  *Default:* ``JOLOKIA``

  .. _Dropwizard: https://metrics.dropwizard.io/3.2.3/manual/third-party.html
  .. _Introduction to New Relic for Java: https://docs.newrelic.com/docs/agents/java-agent/getting-started/introduction-new-relic-java

keyStorePassword
  The password to unlock the KeyStore file (``<workspace>/certificates/sslkeystore.jks``) containing the node certificate and private key.

  解锁 KeyStore 文件（``<workspace>/certificates/sslkeystore.jks``）的密码，KeyStore 文件中包含了节点的证书（certificate）和私钥（private key）。

  **Important: This is the non-secret value for the development certificates automatically generated during the first node run.
  Longer term these keys will be managed in secure hardware devices.**

  **Important: 这个非安全的值是用于开发目的的，在节点第一次运行的时候自动生成。长期来说，这些秘钥应该在安全硬件设备（secure hardware devices）中进行管理。**

  *Default:* cordacadevpass

lazyBridgeStart
  Internal option.

  **Important: Please do not change.**

  *Default:* true

messagingServerAddress
  The address of the ArtemisMQ broker instance.
  If not provided the node will run one locally.

  ArtemisMA broker 实例的地址。如果没有指定的话，节点会在本地运行一个。

  *Default:* not defined

messagingServerExternal
  If ``messagingServerAddress`` is specified the default assumption is that the artemis broker is running externally.
  Setting this to ``false`` overrides this behaviour and runs the artemis internally to the node, but bound to the address specified in ``messagingServerAddress``.
  This allows the address and port advertised in ``p2pAddress`` to differ from the local binding, especially if there is external remapping by firewalls, load balancers , or routing rules. Note that ``detectPublicIp`` should be set to ``false`` to ensure that no translation of the ``p2pAddress`` occurs before it is sent to the network map.

  *Default:* not defined

myLegalName
  The legal identity of the node.
  This acts as a human-readable alias to the node's public key and can be used with the network map to look up the node's info.
  This is the name that is used in the node's certificates (either when requesting them from the doorman, or when auto-generating them in dev mode).
  At runtime, Corda checks whether this name matches the name in the node's certificates.
  For more details please read :ref:`node-naming` chapter.

  节点的法律标识（legal identity）。

  *Default:* not defined

notary
  Optional configuration object which if present configures the node to run as a notary. If part of a Raft or BFT-SMaRt
  cluster then specify ``raft`` or ``bftSMaRt`` respectively as described below. If a single node notary then omit both.

  这是一个可选的配置对象，如果添加了这个配置项那么该节点就会作为 notary 来运行。如果是一个 Raft 或者 BFT-SMaRt 集群的一部分，那么像下边描述的那样去指定 ``raft`` 或者 ``bftSMaRt``。如果是单一的一个 notary，那么请忽略他们。

  validating
    Boolean to determine whether the notary is a validating or non-validating one.

    Boolean 值，确定一个 notary 节点是否是一个 validating notary

    *Default:* false

  serviceLegalName
    If the node is part of a distributed cluster, specify the legal name of the cluster.
    At runtime, Corda checks whether this name matches the name of the certificate of the notary cluster.

    *Default:* not defined

  raft
    *(Experimental)* If part of a distributed Raft cluster, specify this configuration object with the following settings:

    *（探索性的）* 如果该节点是一个分布式 Raft 集群的一部分的话，那么使用下边的配置指定这个配置对象：

      nodeAddress
        The host and port to which to bind the embedded Raft server. Note that the Raft cluster uses a
        separate transport layer for communication that does not integrate with ArtemisMQ messaging services.

        绑定到内置的 Raft server 的 host 和 port。注意：Raft 集群使用一个独立的 transport 层来进行沟通，这个并没有跟 ArtemisMQ 消息服务集成

        *Default:* not defined

      clusterAddresses
        Must list the addresses of all the members in the cluster. At least one of the members must
        be active and be able to communicate with the cluster leader for the node to join the cluster. If empty, a
        new cluster will be bootstrapped.

        必须要列出这个 Raft 集群中所有成员的地址。如果节点想要加入一个集群，这些成员中至少要有一个是运行的状态并且能够跟集群的 leader 进行沟通。如果是空的，一个新的集群会被启动

        *Default:* not defined

  bftSMaRt
    *(Experimental)* If part of a distributed BFT-SMaRt cluster, specify this configuration object with the following settings:

    *（探索性的）* 如果该节点是一个分布式的 BFT-SMaRt 集群的一部分的话，那么使用下边的配置指定这个配置对象：

      replicaId
        The zero-based index of the current replica. All replicas must specify a unique replica id.

        当前的 replica 的从 0 开始的 index 值。所有的 replicas 必须要指定一个唯一的 replica id

        *Default:* not defined

      clusterAddresses
        Must list the addresses of all the members in the cluster. At least one of the members must
        be active and be able to communicate with the cluster leader for the node to join the cluster. If empty, a
        new cluster will be bootstrapped.

        必须要列出这个 Raft 集群中所有成员的地址。如果节点想要加入一个集群，这些成员中至少要有一个是运行的状态并且能够跟集群的 leader 进行沟通。如果是空的，一个新的集群会被启动

        *Default:* not defined

networkParameterAcceptanceSettings
  Optional settings for managing the network parameter auto-acceptance behaviour.
  If not provided then the defined defaults below are used.

  autoAcceptEnabled
    This flag toggles auto accepting of network parameter changes.
    If a network operator issues a network parameter change which modifies only auto-acceptable options and this behaviour is enabled then the changes will be accepted without any manual intervention from the node operator.
    See :doc:`network-map` for more information on the update process and current auto-acceptable parameters.
    Set to ``false`` to disable.

    *Default:* true

  excludedAutoAcceptableParameters
    List of auto-acceptable parameter names to explicitly exclude from auto-accepting.
    Allows a node operator to control the behaviour at a more granular level.

    *Default:* empty list

.. _corda_configuration_file_networkServices:

networkServices
  If the Corda compatibility zone services, both network map and registration (doorman), are not running on the same endpoint
  and thus have different URLs then this option should be used in place of the ``compatibilityZoneURL`` setting.

  **Important: Only one of ``compatibilityZoneURL`` or ``networkServices`` should be used.**

  doormanURL
    Root address of the network registration service.

    *Default:* not defined

  networkMapURL
    Root address of the network map service.

    *Default:* not defined

  pnm
    Optional UUID of the private network operating within the compatibility zone this node should be joining.

    *Default:* not defined

.. _corda_configuration_file_p2pAddress:

p2pAddress
  The host and port on which the node is available for protocol operations over ArtemisMQ.

  通过 ArtemisMQ 对节点进行协议操作时的可用的主机（host）和端口号

  In practice the ArtemisMQ messaging services bind to **all local addresses** on the specified port.
  However, note that the host is the included as the advertised entry in the network map.
  As a result the value listed here must be **externally accessible when running nodes across a cluster of machines.**
  If the provided host is unreachable, the node will try to auto-discover its public one.

  常规来说，ArtemisMQ 消息服务会绑定到 **所有的本地地址** 的指定的端口号。但是，要注意的是主机（host）会在 NetworkMapService 中作为 advertised entry 被包含进来。所以当在机器集群上运行节点的时候，这里所列出的值都应该是可以被外边访问的。如果这个提供的 host 无法访问，节点会尝试自动发现它的共有 host。

  *Default:* not defined

rpcAddress (deprecated)
  The address of the RPC system on which RPC requests can be made to the node.
  If not provided then the node will run without RPC.

  RPC 系统的地址，RPC 请求可以通过它来发送给节点。如果没有指定的话，节点就会不使用 RPC 并运行。

  **Important: Deprecated. Use rpcSettings instead.**

  **重要: 已废弃，请使用 rpcSettings**

  *Default:* not defined

rpcSettings
  Options for the RPC server exposed by the Node.

  节点暴露的 RPC server 的选项

  **Important: The RPC SSL certificate is used by RPC clients to authenticate the connection.  The Node operator must provide RPC clients with a truststore containing the certificate they can trust.  We advise Node operators to not use the P2P keystore for RPC.  The node can be run with the "generate-rpc-ssl-settings" command, which generates a secure keystore and truststore that can be used to secure the RPC connection. You can use this if you have no special requirements.**

    address
      host and port for the RPC server binding.

      RPC server 要绑定的 host 和 port

      *Default:* not defined

    adminAddress
      host and port for the RPC admin binding (this is the endpoint that the node process will connect to).

      RPC admin 绑定的 host 和 port（这是节点进程将会连接到的 endpoint）

      *Default:* not defined

    standAloneBroker
      boolean, indicates whether the node will connect to a standalone broker for RPC.

      boolean 值，指定节点是否为 RPC 连接到一个独立的 broker

      *Default:* false

    useSsl
      boolean, indicates whether or not the node should require clients to use SSL for RPC connections.

      boolean 值，指定节点是否要求 clients 使用 SSL 来进行 RPC 连接

      *Default:* false

    ssl
      (mandatory if ``useSsl=true``) SSL settings for the RPC server.

      RPC server 的 SSL 配置，如果 ``useSsl=true`` 就是必须的。

      keyStorePath
        Absolute path to the key store containing the RPC SSL certificate.

      *Default:* not defined

      keyStorePassword
        Password for the key store.

        key store 的密码

        *Default:* not defined

rpcUsers
  A list of users who are authorised to access the RPC system.
  Each user in the list is a configuration object with the following fields:

  一个有权限访问 RPC 系统的用户列表。列表中的每个用户都是一个包含下列字段的配置对象：

  username
    Username consisting only of word characters (a-z, A-Z, 0-9 and _)

    用户名只能包含英文字母（a-z，A-Z，0-9 和 _）

    *Default:* not defined

  password
    The password

    *Default:* not defined

  permissions
    A list of permissions for starting flows via RPC.
    To give the user the permission to start the flow ``foo.bar.FlowClass``, add the string ``StartFlow.foo.bar.FlowClass`` to the list.
    If the list contains the string ``ALL``, the user can start any flow via RPC.
    This value is intended for administrator users and for development.

    一个通过 RPC 可以启动的 flows 的权限列表。为了给一个用户能够启动 ``foo.bar.FlowClass`` 这个 flow 的权限，需要将字符串 ``StartFlow.foo.bar.FlowClass`` 加到列表中。如果列表包含了字符串 ``ALL``，用户就可以通过 RPC 启动任何的 flow。这个值主要是针对于 administrator 用户以及部署时候使用。

    *Default:* not defined

security
  Contains various nested fields controlling user authentication/authorization, in particular for RPC accesses.
  See :doc:`clientrpc` for details.

  包含了多个嵌套的字段来管理用户的 authentication/authorization，具体就是对于 RPC 的访问控制。查看 :doc:`clientrpc` 了解详细内容。

sshd
  If provided, node will start internal SSH server which will provide a management shell.
  It uses the same credentials and permissions as RPC subsystem.
  It has one required parameter.

  如果提供了这个选项，节点会启动内部的 SSH server，这会提供一个管理 shell。这个会跟 RPC subsystem 使用相同的用户信息和权限。它包含一个必须的参数。

  port
    The port to start SSH server on e.g. ``sshd { port = 2222 }``.

    启动 SSH server 的 port，比如 ``sshd { port = 2222 }``

    *Default:* not defined

systemProperties
  An optional map of additional system properties to be set when launching via ``corda.jar`` only.
  Keys and values of the map should be strings. e.g. ``systemProperties = { visualvm.display.name = FooBar }``

  一个可选的额外的系统属性的 map，仅仅在通过 ``corda.jar`` 加载的时候会设置这些属性。这个 map 中的 Keys 和 values 应该是字符串。比如：``systemProperties = { visualvm.display.name = FooBar }``

  *Default:* not defined

transactionCacheSizeMegaBytes
  Optionally specify how much memory should be used for caching of ledger transactions in memory.

  可选的来配置应该使用多少内存来 cache ledger transactions

  *Default:* 8 MB plus 5% of all heap memory above 300MB.

tlsCertCrlDistPoint
  CRL distribution point (i.e. URL) for the TLS certificate.
  Default value is NULL, which indicates no CRL availability for the TLS certificate.

  **Important: This needs to be set if crlCheckSoftFail is false (i.e. strict CRL checking is on).**

  *Default:* NULL


tlsCertCrlIssuer
  CRL issuer (given in the X500 name format) for the TLS certificate.
  Default value is NULL, which indicates that the issuer of the TLS certificate is also the issuer of the CRL.

  **Important: If this parameter is set then `tlsCertCrlDistPoint` needs to be set as well.**

  *Default:* NULL


trustStorePassword
  The password to unlock the Trust store file (``<workspace>/certificates/truststore.jks``) containing the Corda network root certificate.
  This is the non-secret value for the development certificates automatically generated during the first node run.

  解锁 Trust store 文件（``<workspace>/certificates/truststore.jks``）的密码，Trust store 文件包含了 Corda 网络的根证书（root certificate）。这个非安全的值是用于开发目的的，在节点第一次运行的时候自动生成。

  *Default:* trustpass


useTestClock
  Internal option.

  **Important: Please do not change.**

  *Default:* false


verfierType
  Internal option.

  **Important: Please do not change.**

  *Default:* InMemory

Reference.conf
--------------
A set of default configuration options are loaded from the built-in resource file ``/node/src/main/resources/reference.conf``.
This file can be found in the ``:node`` gradle module of the `Corda repository <https://github.com/corda/corda>`_.
Any options you do not specify in your own ``node.conf`` file will use these defaults.

一系列的默认配置选项从内置的源文件 ``/node/src/main/resources/reference.conf`` 加载进来。这个文件可以在 `Corda repository <https://github.com/corda/corda>`_ 的 ``:node`` gradle module 中找到。任何你没有在你的 ``node.conf`` 指定的选项都会使用这些默认值。

Here are the contents of the ``reference.conf`` file:

下边是  ``reference.conf`` 的内容：

.. literalinclude:: ../../node/src/main/resources/reference.conf


配置样例
----------------------

Node configuration hosting the IRSDemo services
````````````````````````````````````````````````````````````````
General node configuration file for hosting the IRSDemo services

.. literalinclude:: example-code/src/main/resources/example-node.conf

Simple notary configuration file
`````````````````````````````````

.. parsed-literal::

    myLegalName = "O=Notary Service,OU=corda,L=London,C=GB"
    keyStorePassword = "cordacadevpass"
    trustStorePassword = "trustpass"
    p2pAddress = "localhost:12345"
    rpcSettings {
        useSsl = false
        standAloneBroker = false
        address = "my-corda-node:10003"
        adminAddress = "my-corda-node:10004"
    }
    notary {
        validating = false
    }
    devMode = false
    networkServices {
        doormanURL = "https://cz.example.com"
        networkMapURL = "https://cz.example.com"
    }

Node configuration with diffrent URL for NetworkMap and Doorman
```````````````````````````````````````````````````````````````

Configuring a node where the Corda Compatibility Zone's registration and Network Map services exist on different URLs

.. literalinclude:: example-code/src/main/resources/example-node-with-networkservices.conf


