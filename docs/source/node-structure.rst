节点文件夹结构
=====================

A folder containing a Corda node files has the following structure:

一个包含 Corda 节点文件的文件夹具有以下的结构：

.. sourcecode:: none

    .
    ├── additional-node-infos   // Additional node infos to load into the network map cache, beyond what the network map server provides
    ├── artemis                 // Stores buffered P2P messages
    ├── brokers                 // Stores buffered RPC messages
    ├── certificates            // The node's certificates
    ├── corda-webserver.jar     // The built-in node webserver (DEPRECATED)
    ├── corda.jar               // The core Corda libraries (This is the actual Corda node implementation)
    ├── cordapps                // The CorDapp JARs installed on the node
    ├── drivers                 // Contains a Jolokia driver used to export JMX metrics, the node loads any additional JAR files from this directory at startup.
    ├── logs                    // The node's logs
    ├── network-parameters      // The network parameters automatically downloaded from the network map server
    ├── node.conf               // The node's configuration files
    ├── persistence.mv.db       // The node's database
    └── shell-commands          // Custom shell commands defined by the node owner

You install CorDapps on the node by placing CorDapp JARs in the ``cordapps`` folder.

通过将 CorDap JARs 放到 ``cordapps`` 文件夹的方式你可以在节点上安装 CorDapps。

In development mode (i.e. when ``devMode = true``), the ``certificates`` directory is filled with pre-configured
keystores if they do not already exist to ensure that developers can get the nodes working as quickly as
possible.

在开发模式中（当 ``devMode = true``），如果需要的 keystores 不存在的话，``certificates`` 路径里会被放进一个预先配置好的 keystores。这个确保了开发者能够尽快地让节点工作起来。

.. warning:: These pre-configured keystores are not secure and must not used in a production environments.

.. warning:: 这些预先配置好的 keystores 并不是安全的，所以不应该被用在生产环境。

The keystores store the key pairs and certificates under the following aliases:

keystores 存储了秘钥对儿以及证书：

* ``nodekeystore.jks`` uses the aliases ``cordaclientca`` and ``identity-private-key``
* ``sslkeystore.jks`` uses the alias ``cordaclienttls``

All the keystores use the password provided in the node's configuration file using the ``keyStorePassword`` attribute.
If no password is configured, it defaults to ``cordacadevpass``.

所有的 keystores 使用的都是由节点的配置文件中使用 ``keyStorePassword`` 属性所提供的密码。

To learn more, see :doc:`permissioning`.

查看 :doc:`permissioning` 了解更多。

