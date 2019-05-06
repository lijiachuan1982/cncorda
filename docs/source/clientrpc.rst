.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

与节点互动
=======================

.. contents::

概要
--------
To interact with your node, you need to write a client in a JVM-compatible language using the `CordaRPCClient`_ class.
This class allows you to connect to your node via a message queue protocol and provides a simple RPC interface for
interacting with the node. You make calls on a JVM object as normal, and the marshalling back-and-forth is handled for
you.

为了跟你的节点互动，你需要使用 JVM 兼容语言 和 `CordaRPCClient`_ 类来编写一个客户端。这个类会通过使用一个消息队列协议来连接你的节点并且提供一个简单的 RPC 接口来跟节点互动。你可以像通常那样去调用一个 Java 对象，然后来回的消息交互它会帮你控制。

.. warning:: The built-in Corda webserver is deprecated and unsuitable for production use. If you want to interact with
   your node via HTTP, you will need to stand up your own webserver that connects to your node using the
   `CordaRPCClient`_ class. You can find an example of how to do this using the popular Spring Boot server
   `here <https://github.com/corda/spring-webserver>`_.

.. warning:: 内置的 Corda webserver 已经废弃并且不再适合生产环境使用。如果你想通过 HTTP 跟你的节点互动的话，你需要创建你自己的 webserver，使用 `CordaRPCClient`_ 类来连接到你的节点。你可以在 `这里 <https://github.com/corda/spring-webserver>`_ 找到如何使用流行的 Spring Boot server 的例子。

.. _clientrpc_connect_ref:

通过 RPC 连接到一个节点
----------------------------
To use `CordaRPCClient`_, you must add ``net.corda:corda-rpc:$corda_release_version`` as a ``cordaCompile`` dependency
in your client's ``build.gradle`` file.

为了使用 `CordaRPCClient`_，你必须要将 ``net.corda:corda-rpc:$corda_release_version`` 作为一个 ``cordaCompile`` 的依赖添加到你的客户端的 ``build.gradle`` 文件中。

`CordaRPCClient`_ has a ``start`` method that takes the node's RPC address and returns a `CordaRPCConnection`_.
`CordaRPCConnection`_ has a ``proxy`` method that takes an RPC username and password and returns a `CordaRPCOps`_
object that you can use to interact with the node.

`CordaRPCClient`_ 具有一个 ``start`` 方法会需要节点的 RPC 地址并且会返回一个 `CordaRPCConnection`_。`CordaRPCConnection`_ 具有一个 ``proxy`` 方法会需要一个 RPC 用户名和密码并且返回一个 `CordaRPCOps`_ 对象，你可以用它来跟你的节点互动。

Here is an example of using `CordaRPCClient`_ to connect to a node and log the current time on its internal clock:

下边是一个使用 `CordaRPCClient`_ 来连接到一个节点并且在它的内部时间上 log 当前的时间：

.. container:: codeset

   .. literalinclude:: example-code/src/main/kotlin/net/corda/docs/kotlin/ClientRpcExample.kt
      :language: kotlin
      :start-after: START 1
      :end-before: END 1

   .. literalinclude:: example-code/src/main/java/net/corda/docs/java/ClientRpcExample.java
      :language: java
      :start-after: START 1
      :end-before: END 1

.. warning:: The returned `CordaRPCConnection`_ is somewhat expensive to create and consumes a small amount of
   server side resources. When you're done with it, call ``close`` on it. Alternatively you may use the ``use``
   method on `CordaRPCClient`_ which cleans up automatically after the passed in lambda finishes. Don't create
   a new proxy for every call you make - reuse an existing one.

.. warning:: 返回的 `CordaRPCConnection`_ 会消耗你的服务器端的资源。当你用完它的时候，调用 ``close``。或者你可以使用 `CordaRPCClient`_ 的 ``use`` 方法，它会在传入 lambda 的方法结束后自动清理。不要为你的每次调用创建一个新的代理 - 重用一个已经存在的。

For further information on using the RPC API, see :doc:`tutorial-clientrpc-api`.

关于使用 RPC API 的更多信息，查看 see :doc:`tutorial-clientrpc-api`。

RPC 权限
---------------
For a node's owner to interact with their node via RPC, they must define one or more RPC users. Each user is
authenticated with a username and password, and is assigned a set of permissions that control which RPC operations they
can perform. Permissions are not required to interact with the node via the shell, unless the shell is being accessed via SSH.

如果一个节点的 owner 想跟它的节点通过 RPC 来互动的话（比如读取节点 storage 中的内容），他必须要定义一个或多个 RPC 用户。每个用户会通过一个用户名和密码来进行验证，还会被赋予一系列的RPC 能够使用的权限。使用 shell 来跟节点互动的时候是不需要 RPC 权限的，除非 shell 是通过 SSH 来访问的。

RPC users are created by adding them to the ``rpcUsers`` list in the node's ``node.conf`` file:

RPC 用户信息会被添加到节点的 ``node.conf`` 文件中的 ``rpcUsers`` 列表中：

.. container:: codeset

    .. sourcecode:: groovy

        rpcUsers=[
            {
                username=exampleUser
                password=examplePass
                permissions=[]
            },
            ...
        ]

By default, RPC users are not permissioned to perform any RPC operations.

默认的，RPC 用户不允许执行任何的 RPC 操作。

赋予 flow 权限
~~~~~~~~~~~~~~~~~~~~~~~~~
You provide an RPC user with the permission to start a specific flow using the syntax
``StartFlow.<fully qualified flow name>``:

使用 ``StartFlow.<fully qualified flow name>`` 来给一个 RPC 用户提供开始某个指定的 flow 的权限：

.. container:: codeset

    .. sourcecode:: groovy

        rpcUsers=[
            {
                username=exampleUser
                password=examplePass
                permissions=[
                    "StartFlow.net.corda.flows.ExampleFlow1",
                    "StartFlow.net.corda.flows.ExampleFlow2"
                ]
            },
            ...
        ]

You can also provide an RPC user with the permission to start any flow using the syntax
``InvokeRpc.startFlow``:

你也可以使用 ``InvokeRpc.startFlow`` 来给 RPC 用户提供启动任何 flow 的权限：

.. container:: codeset

    .. sourcecode:: groovy

        rpcUsers=[
            {
                username=exampleUser
                password=examplePass
                permissions=[
                    "InvokeRpc.startFlow"
                ]
            },
            ...
        ]

赋予其他的 RPC 权限
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
You provide an RPC user with the permission to perform a specific RPC operation using the syntax
``InvokeRpc.<rpc method name>``:

可以使用 ``InvokeRpc.<rpc method name>`` 来给 RPC 用户分配执行一个指定的 RPC 操作的权限：

.. container:: codeset

    .. sourcecode:: groovy

        rpcUsers=[
            {
                username=exampleUser
                password=examplePass
                permissions=[
                    "InvokeRpc.nodeInfo",
                    "InvokeRpc.networkMapSnapshot"
                ]
            },
            ...
        ]

赋予所有权限
~~~~~~~~~~~~~~~~~~~~~~~~
You can provide an RPC user with the permission to perform any RPC operation (including starting any flow) using the
``ALL`` permission:

使用 ``ALL`` 将允许 RPC 用户进行所有 RPC 操作（包括启动任何的 flow）：

.. container:: codeset

    .. sourcecode:: groovy

        rpcUsers=[
            {
                username=exampleUser
                password=examplePass
                permissions=[
                    "ALL"
                ]
            },
            ...
        ]

.. _rpc_security_mgmt_ref:

RPC 安全管理
-----------------------

Setting ``rpcUsers`` provides a simple way of granting RPC permissions to a fixed set of users, but has some
obvious shortcomings. To support use cases aiming for higher security and flexibility, Corda offers additional security
features such as:

 * Fetching users credentials and permissions from an external data source (e.g.: a remote RDBMS), with optional in-memory
   caching. In particular, this allows credentials and permissions to be updated externally without requiring nodes to be
   restarted.
 * Password stored in hash-encrypted form. This is regarded as must-have when security is a concern. Corda currently supports
   a flexible password hash format conforming to the Modular Crypt Format provided by the `Apache Shiro framework <https://shiro.apache.org/static/1.2.5/apidocs/org/apache/shiro/crypto/hash/format/Shiro1CryptFormat.html>`_

设置 ``rpcUsers`` 提供了一个简单的方式来为一个固定的一些用户赋予 RPC 权限，但是有一些很明显的不足。为了支持更高安全和灵活性，Corda 提供了额外的安全功能，比如：

 * 从外部的数据源获取用户的验证信息和权限信息（比如从一个远程的 RDBMS），带有可选的在内存中的 caching。特别的，这种方式允许验证信息和权限信息可以在外部进行更新，而不需要重新启动节点
 * 密码以哈希加密过的形式存储。当安全是一个要素的时候这种方式就是必须要有了。Corda 当前支持由 `Apache Shiro framework <https://shiro.apache.org/static/1.2.5/apidocs/org/apache/shiro/crypto/hash/format/Shiro1CryptFormat.html>`_ 提供的 Modular Crypt 格式的灵活地密码哈希格式。

These features are controlled by a set of options nested in the ``security`` field of ``node.conf``.
The following example shows how to configure retrieval of users credentials and permissions from a remote database with
passwords in hash-encrypted format and enable in-memory caching of users data:

这些功能是由 ``node.conf`` 中 ``security`` 字段里的一系列选项来控制的。下边的例子演示了如何配置可从一个远程的数据库取回的用户验证信息和权限信息，密码是以哈希加密过的格式并且开启了用户数据在内存中 caching：

.. container:: codeset

    .. sourcecode:: groovy

        security = {
            authService = {
                dataSource = {
                    type = "DB"
                    passwordEncryption = "SHIRO_1_CRYPT"
                    connection = {
                       jdbcUrl = "<jdbc connection string>"
                       username = "<db username>"
                       password = "<db user password>"
                       driverClassName = "<JDBC driver>"
                    }
                }
                options = {
                     cache = {
                        expireAfterSecs = 120
                        maxEntries = 10000
                     }
                }
            }
        }

It is also possible to have a static list of users embedded in the ``security`` structure by specifying a ``dataSource``
of ``INMEMORY`` type:

也可以通过指定一个 ``INMEMORY`` 类型的 ``dataSource`` 来在 ``security`` 结构中指定一个用户的静态列表：

.. container:: codeset

    .. sourcecode:: groovy

        security = {
            authService = {
                dataSource = {
                    type = "INMEMORY"
                    users = [
                        {
                            username = "<username>"
                            password = "<password>"
                            permissions = ["<permission 1>", "<permission 2>", ...]
                        },
                        ...
                    ]
                }
            }
        }

.. warning:: A valid configuration cannot specify both the ``rpcUsers`` and ``security`` fields. Doing so will trigger
   an exception at node startup.

.. warning:: 一个有效的配置不能够同时指定 ``rpcUsers`` 和 ``security`` 字段。这么做会在启动节点的时候造成异常

为数据鉴权和授权
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``dataSource`` structure defines the data provider supplying credentials and permissions for users. There exist two
supported types of such data source, identified by the ``dataSource.type`` field:

``dataSource`` 结构定义了数据提供者来提供用户的验证信息和权限信息。这里有两种支持的数据源类型，通过 ``dataSource.type`` 字段来定义：

 :INMEMORY: A static list of user credentials and permissions specified by the ``users`` field.

 :INMEMORY: 通过 ``users`` 字段指定的用户验证信息和权限信息的静态列表。

 :DB: An external RDBMS accessed via the JDBC connection described by ``connection``. Note that, unlike the ``INMEMORY``
  case, in a user database permissions are assigned to *roles* rather than individual users. The current implementation
  expects the database to store data according to the following schema:

       - Table ``users`` containing columns ``username`` and ``password``. The ``username`` column *must have unique values*.
       - Table ``user_roles`` containing columns ``username`` and ``role_name`` associating a user to a set of *roles*.
       - Table ``roles_permissions`` containing columns ``role_name`` and ``permission`` associating a role to a set of
         permission strings.

 :DB: 可以通过 ``connection`` 描述的 JDBC 连接的一个外部的 RDBMS。注意：不像 ``INMEMORY`` case，在一个用户的数据库中，权限是被分配给 *角色* 的而不是个人。当前的实现期望数据库根据下边的 schema 来存储数据：

        - ``users`` 表包含 ``username`` 和 ``password`` 列。``username`` *列必须要是唯一的值*
        - ``user_roles`` 表包含 ``username`` 和 ``role_name`` 列，这会把一个用户跟一系列的 *角色* 关联起来
        - ``roles_permissions`` 表包含 ``role_name`` 和 ``permission`` 列，这会把一个角色跟一系列的权限关联起来

  .. note:: There is no prescription on the SQL type of each column (although our tests were conducted on ``username`` and
    ``role_name`` declared of SQL type ``VARCHAR`` and ``password`` of ``TEXT`` type). It is also possible to have extra columns
    in each table alongside the expected ones.

  .. note:: 这里并没有强制每个列的 SQL 类型（尽管我们的测试对于 ``username`` 和 ``role_name`` 使用的是 ``VARCHAR`` SQL 类型，``password`` 是 ``TEXT`` 类型）。在每个表中也可以按照需要增加额外的列。

密码的加密
~~~~~~~~~~~~~~~~~~~

Storing passwords in plain text is discouraged in applications where security is critical. Passwords are assumed
to be in plain format by default, unless a different format is specified by the ``passwordEncryption`` field, like:

当安全性是很重要的时候，将密码以明文的形式存储是不被鼓励的。密码默认是明文的格式，除非对 ``passwordEncryption`` 字段指定了不同的格式，比如：

.. container:: codeset

    .. sourcecode:: groovy

        passwordEncryption = SHIRO_1_CRYPT

``SHIRO_1_CRYPT`` identifies the `Apache Shiro fully reversible
Modular Crypt Format <https://shiro.apache.org/static/1.2.5/apidocs/org/apache/shiro/crypto/hash/format/Shiro1CryptFormat.html>`_,
it is currently the only non-plain password hash-encryption format supported. Hash-encrypted passwords in this
format can be produced by using the `Apache Shiro Hasher command line tool <https://shiro.apache.org/command-line-hasher.html>`_.

``SHIRO_1_CRYPT`` 表示 `Apache Shiro fully reversible Modular Crypt Format <https://shiro.apache.org/static/1.2.5/apidocs/org/apache/shiro/crypto/hash/format/Shiro1CryptFormat.html>`_，这是当前唯一支持的非明文密码的哈希加密格式。可以使用 `Apache Shiro Hasher 命令行工具 <https://shiro.apache.org/command-line-hasher.html>`_ 来生成哈希加密密码。

缓存用户账户数据
~~~~~~~~~~~~~~~~~~~~~~~~~~

A cache layer on top of the external data source of users credentials and permissions can significantly improve
performances in some cases, with the disadvantage of causing a (controllable) delay in picking up updates to the underlying data.
Caching is disabled by default, it can be enabled by defining the ``options.cache`` field in ``security.authService``,
for example:

在用户验证信息和权限信息的外部数据源之上的一个 cache 层在很多情况下会很大程度的改善效率，但是会带来一个可控的对底层的数据的抓取延迟。Caching 默认是被 disabled，可以通过定义 security.authService 中的 ``options.cache`` 字段来开启，比如：

.. container:: codeset

    .. sourcecode:: groovy

        options = {
             cache = {
                expireAfterSecs = 120
                maxEntries = 10000
             }
        }

This will enable a non-persistent cache contained in the node's memory with maximum number of entries set to ``maxEntries``
where entries are expired and refreshed after ``expireAfterSecs`` seconds.

这个会开启一个包含在节点的内存中的非持久化的 cache，最大的输入数量设置为 ``maxEntries``，这个 entries 会在 ``expireAfterSecs`` 秒钟后过期。

Observables
-----------
The RPC system handles observables in a special way. When a method returns an observable, whether directly or
as a sub-object of the response object graph, an observable is created on the client to match the one on the
server. Objects emitted by the server-side observable are pushed onto a queue which is then drained by the client.
The returned observable may even emit object graphs with even more observables in them, and it all works as you
would expect.

RPC 系统使用一种特殊的方式来处理 observables。当一个方法返回 observable 的时候，或者直接返回，或者作为一个 response object graph 的子对象返回，客户端会创建一个跟服务器端匹配的 observable。服务器端 observable 发出的对象会被放进一个队列中，这个队列会被客户端来消费。返回的 observable 设置可能会发出含有更多的 observables 的 object graphs，它会像你期望的那样来工作的。

This feature comes with a cost: the server must queue up objects emitted by the server-side observable until you
download them. Note that the server side observation buffer is bounded, once it fills up the client is considered
slow and will be disconnected. You are expected to subscribe to all the observables returned, otherwise client-side
memory starts filling up as observations come in. If you don't want an observable then subscribe then unsubscribe
immediately to clear the client-side buffers and to stop the server from streaming. For Kotlin users there is a
convenience extension method called ``notUsed()`` which can be called on an observable to automate this step.

这个特性是要有一定的消耗的：server 必须要不断地接收由服务器端发出的对象直到你把他们下载下来。注意，服务器端的 observation buffer 是固定的，一旦被用光，客户端会变得很慢。你应该 subscribe 所有返回的 observables，否则的话随着 observations 的进入，客户端的内存就会不断的被使用。如果你不想要一个 observable，那么你先 subscribe 然后立即 unsubscribe，这样会清理掉客户端的 buffer，也会让 server 停止 streaming。对于 Kotlin 用户，这里有一个方便的称为 ``notUsed()`` 的扩展方法，可以在一个 observable 上调用来自动化这个步骤。

If your app quits then server side resources will be freed automatically.

如果你的 app 退出了的话，那么服务器端的资源会被自动释放。

.. warning:: If you leak an observable on the client side and it gets garbage collected, you will get a warning
   printed to the logs and the observable will be unsubscribed for you. But don't rely on this, as garbage collection
   is non-deterministic. If you set ``-Dnet.corda.client.rpc.trackRpcCallSites=true`` on the JVM command line then
   this warning comes with a stack trace showing where the RPC that returned the forgotten observable was called from.
   This feature is off by default because tracking RPC call sites is moderately slow.

.. warning:: 如果你在客户端泄露了一个 observable，然后它得到了搜集到的垃圾，你会在 log 中看到一个警示提示，并且 observable 会被自动的 unsubscribe。但是不要完全依赖于这个，因为垃圾回收是无法预测（ non-deterministic）的。如果你在 JVM 命令行上设置了 ``-Dnet.corda.client.rpc.trackRpcCallSites=true``，那么这个警告会带有一个 stack trace 显示了是在 RPC 的哪里返回的忘记 observable 是从哪里调用的。这个功能默认是关闭的，因为跟踪 RPC 调用网站会变慢。

.. note:: Observables can only be used as return arguments of an RPC call. It is not currently possible to pass
   Observables as parameters to the RPC methods. In other words the streaming is always server to client and not
   the other way around.

.. note:: Observables 仅仅能够作为一个 RPC 调用返回的参数来被使用。现在还不能够将 observables 作为参数传递给 RPC 方法。换句话说，流总会是从 server 到 client 的，不能是其他的方式。

未来
-------
A method can also return a ``CordaFuture`` in its object graph and it will be treated in a similar manner to
observables. Calling the ``cancel`` method on the future will unsubscribe it from any future value and release
any resources.

一个方法也能够在它的 object graph 中返回一个 ``CordaFuture`` 并且会像对待 observables 的方式来对待它。在未来调用 ``cancel`` 方法会解除对任何未来的值的订阅并释放所有的资源。

版本
----------
The client RPC protocol is versioned using the node's platform version number (see :doc:`versioning`). When a proxy is created
the server is queried for its version, and you can specify your minimum requirement. Methods added in later versions
are tagged with the ``@RPCSinceVersion`` annotation. If you try to use a method that the server isn't advertising support
of, an ``UnsupportedOperationException`` is thrown. If you want to know the version of the server, just use the
``protocolVersion`` property (i.e. ``getProtocolVersion`` in Java).

客户端 RPC 协议是使用节点的 Platform Version （查看 :doc:`versioning`）来定义版本的。当一个代理（proxy）被创建后，server 会查询它的版本，你可以指定你的最小版本要求。在后期的版本中被添加的方法都会带有 ```@RPCSinceVersion`` 的注解。如果你使用了一个 server 不支持的方法，一个 ``UnsupportedOperationException`` 的异常会被抛出。如果你想知道 server 的版本，只需要使用 ``protocolVersion`` 属性（比如 Java 中的 ``getProtocolVersion``）。

The RPC client library defaults to requiring the platform version it was built with. That means if you use the client
library released as part of Corda N, then the node it connects to must be of version N or above. This is checked when
the client first connects. If you want to override this behaviour, you can alter the ``minimumServerProtocolVersion``
field in the ``CordaRPCClientConfiguration`` object passed to the client. Alternatively, just link your app against
an older version of the library.

线程安全
-------------
A proxy is thread safe, blocking, and allows multiple RPCs to be in flight at once. Any observables that are returned and
you subscribe to will have objects emitted in order on a background thread pool. Each Observable stream is tied to a single
thread, however note that two separate Observables may invoke their respective callbacks on different threads.

一个代理（proxy）是线程安全的、阻塞的（blocking）并且允许多个 RPCs 同时存在。任何返回的和你订阅的 observables 将会有对象会在后台运行的线程池中被有序地 emitted。每个 observable stream 会被绑定到一个单独的线程，但是要注意到的是，两个独立的 observables 可能在不同的线程上调用他们对应的 callbacks。

异常处理
--------------
If something goes wrong with the RPC infrastructure itself, an ``RPCException`` is thrown. If you call a method that
requires a higher version of the protocol than the server supports, ``UnsupportedOperationException`` is thrown.
Otherwise the behaviour depends on the ``devMode`` node configuration option.

如果 RPC 基础架构本身出了问题，一个 ``RPCException`` 会被抛出。如果你调用了一个要求比当前 server 支持的版本更高的一个方法， ``UnsupportedOperationException`` 异常会被抛出。否则的话，这个行为会依赖于 ``devMode`` 的节点配置选项。

In ``devMode``, if the server implementation throws an exception, that exception is serialised and rethrown on the client
side as if it was thrown from inside the called RPC method. These exceptions can be caught as normal.

When not in ``devMode``, the server will mask exceptions not meant for clients and return an ``InternalNodeException`` instead.
This does not expose internal information to clients, strengthening privacy and security. CorDapps can have exceptions implement
``ClientRelevantError`` to allow them to reach RPC clients.

重连 RPC clients
------------------------

In the current version of Corda the RPC connection and all the observervables that are created by a client will just throw exceptions and die
when the node or TCP connection become unavailable.

It is the client's responsibility to handle these errors and reconnect once the node is running again. Running RPC commands against a stopped
node will just throw exceptions. Previously created Observables will not emit any events after the node restarts. The client must explicitly re-run the command and
re-subscribe to receive more events.

RPCs which have a side effect, such as starting flows, may have executed on the node even if the return value is not received by the client.
The only way to confirm is to perform a business-level query and retry accordingly. The sample `runFlowWithLogicalRetry` helps with this.

In case users require such a functionality to write a resilient RPC client we have a sample that showcases how this can be implemented and also
a thorough test that demonstrates it works as expected.

The code that performs the reconnecting logic is: `ReconnectingCordaRPCOps.kt <https://github.com/corda/samples/blob/release-V|platform_version|/net/corda/client/rpc/internal/ReconnectingCordaRPCOps.kt>`_.

.. note:: This sample code is not exposed as an official Corda API, and must be included directly in the client codebase and adjusted.

The usage is showcased in the: `RpcReconnectTests.kt <https://github.com/corda/samples/blob/release-V|platform_version|/node/src/integration-test/kotlin/net/corda/node/services/rpc/RpcReconnectTests.kt>`_.
In case resiliency is a requirement, then it is recommended that users will write a similar test.

How to initialize the `ReconnectingCordaRPCOps`:

.. literalinclude:: ../../node/src/integration-test/kotlin/net/corda/node/services/rpc/RpcReconnectTests.kt
   :language: kotlin
   :start-after: DOCSTART rpcReconnectingRPC
   :end-before: DOCEND rpcReconnectingRPC


How to track the vault :

.. literalinclude:: ../../node/src/integration-test/kotlin/net/corda/node/services/rpc/RpcReconnectTests.kt
   :language: kotlin
   :start-after: DOCSTART rpcReconnectingRPCVaultTracking
   :end-before: DOCEND rpcReconnectingRPCVaultTracking


How to start a flow with a logical retry function that checks for the side effects of the flow:

.. literalinclude:: ../../node/src/integration-test/kotlin/net/corda/node/services/rpc/RpcReconnectTests.kt
   :language: kotlin
   :start-after: DOCSTART rpcReconnectingRPCFlowStarting
   :end-before: DOCEND rpcReconnectingRPCFlowStarting


Note that, as shown by the test, during reconnecting some events might be lost.

.. literalinclude:: ../../node/src/integration-test/kotlin/net/corda/node/services/rpc/RpcReconnectTests.kt
   :language: kotlin
   :start-after: DOCSTART missingVaultEvents
   :end-before: DOCEND missingVaultEvents


Wire 安全
-------------
If TLS communications to the RPC endpoint are required the node should be configured with ``rpcSettings.useSSL=true`` see :doc:`corda-configuration-file`.
The node admin should then create a node specific RPC certificate and key, by running the node once with ``generate-rpc-ssl-settings`` command specified (see :doc:`node-commandline`).
The generated RPC TLS trust root certificate will be exported to a ``certificates/export/rpcssltruststore.jks`` file which should be distributed to the authorised RPC clients.

The connecting ``CordaRPCClient`` code must then use one of the constructors with a parameter of type ``ClientRpcSslOptions`` (`JavaDoc <api/javadoc/net/corda/client/rpc/CordaRPCClient.html>`_) and set this constructor
argument with the appropriate path for the ``rpcssltruststore.jks`` file. The client connection will then use this to validate the RPC server handshake.

Note that RPC TLS does not use mutual authentication, and delegates fine grained user authentication and authorisation to the RPC security features detailed above.

Corda 节点的白名单类
----------------------------------------
CorDapps must whitelist any classes used over RPC with Corda's serialization framework, unless they are whitelisted by
default in ``DefaultWhitelist``. The whitelisting is done either via the plugin architecture or by using the
``@CordaSerializable`` annotation.  See :doc:`serialization`. An example is shown in :doc:`tutorial-clientrpc-api`.

CorDapps 对于通过 RPC 所使用的任何类，都要使用 Corda 的序列化（serialization） framework 将他们添加至白名单，除非这些类已经在 ``DefaultWhitelist`` 中默认地被添加到白名单中了。添加白名单的操作既可以通过 plugin architecture 或者使用 ``@CordaSerializable`` 注解来实现。查看 :doc:`serialization`。在 :doc:`tutorial-clientrpc-api` 中也显示了一个例子。

.. _CordaRPCClient: api/javadoc/net/corda/client/rpc/CordaRPCClient.html
.. _CordaRPCOps: api/javadoc/net/corda/core/messaging/CordaRPCOps.html
.. _CordaRPCConnection: api/javadoc/net/corda/client/rpc/CordaRPCConnection.html
