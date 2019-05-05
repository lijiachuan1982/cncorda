版本
==========

As the Corda platform evolves and new features are added it becomes important to have a versioning system which allows
its users to easily compare versions and know what feature are available to them. Each Corda release uses the standard
semantic versioning scheme of ``major.minor``. This is useful when making releases in the public domain but is not
friendly for a developer working on the platform. It first has to be parsed and then they have three separate segments on
which to determine API differences. The release version is still useful and every MQ message the node sends attaches it
to the ``release-version`` header property for debugging purposes.

随着 Corda 平台的演进以及更多的功能被添加，拥有一个版本管理系统变得越来越重要，它需要能够允许用户很容易地在不同版本间进行比较。Corda 的每个 Release 都会使用有语义的版本名称，例如 ``major.minor.patch``。这对于在公共域中发布时很有用的，但对于在这个平台上工作的开发者来说可能不是很友好。首先它需要被解析然后这会被分解成三个部分，以此来确定 API 具有的差异。 Release 版本还是很有用的，为了 debug 的目的，节点所发送的每个 MQ 的消息都会将这个版本信息附加到 ``release-version`` 的 header 属性中。

平台版本
----------------

It is much easier to use a single incrementing integer value to represent the API version of the Corda platform, which
is called the *platform version*. It is similar to Android's `API Level <https://developer.android.com/guide/topics/manifest/uses-sdk-element.html>`_.
It starts at 1 and will increment by exactly 1 for each release which changes any of the publicly exposed APIs in the
entire platform. This includes public APIs on the node itself, the RPC system, messaging, serialisation, etc. API backwards
compatibility will always be maintained, with the use of deprecation to suggest migration away from old APIs. In very rare
situations APIs may have to be changed, for example due to security issues. There is no relationship between the platform version
and the release version - a change in the major or minor values may or may not increase the platform version. However
we do endeavour to keep them synchronised for now, as a convenience.

使用一个自增长的数值来表示 Corda 平台的 API 版本会更容易一些，这被称为 *平台版本*。这个跟 `API Level <https://developer.android.com/guide/topics/manifest/uses-sdk-element.html>`_ 类似。它从 1 开始，对于在整个平台中修改任何的公有 APIs 的每个新的 release 会加1。这包括节点自身的共有 APIs，RPC 系统，消息系统，序列化等。API 的向后兼容性一直都是被考虑和维护的，这样就不要从老的 APIs 向新版本迁移。仅仅在很少的情况下 APIs 可能会被移除，比如为了一些安全问题。在平台版本和 release version 之间是没有关系的，对于 major，minor 或者 patch 值的变化可能会也可能不会增加平台的版本。

The platform version is part of the node's ``NodeInfo`` object, which is available from the ``ServiceHub``. This enables
a CorDapp to find out which version it's running on and determine whether a desired feature is available. When a node
registers with the network map it will check its own version against the minimum version requirement for the network.

平台版本是节点的 ``NodeInfo`` 对象中的一部分，它可以从 ``ServiceHub`` 中获取到。这个允许一个 CorDapp 能够找到它正在运行的是那个版本，并且确定一个指定的功能是不是可用。当一个节点在网络地图服务中注册的时候，它会使用节点的平台版本来强制网络使用一个最低版本的要求。

最小平台版本
------------------------

Applications can advertise a *minimum platform version* they require. If your app uses new APIs that were added in (for example) Corda 5,
you should specify a minimum version of 5. This will ensure the app won't be loaded by older nodes. If you can *optionally* use the new
APIs, you can keep the minimum set to a lower number. Attempting to use new APIs on older nodes can cause ``NoSuchMethodError`` exceptions
and similar problems, so you'd want to check the node version using ``ServiceHub.myInfo``.

目标版本
--------------

Applications can also advertise a *target version*. This is similar to the concept of the same name in Android and iOS.
Apps should advertise the highest version of the platform they have been tested against. This allows the node to activate or deactivate
backwards compatibility codepaths depending on whether they're necessary or not, as workarounds for apps designed for earlier versions.

For example, consider an app that uses new features introduced in Corda 4, but which has passed regression testing on Corda 5. It will
advertise a minimum platform version of 4 and a target version of 5. These numbers are published in the JAR manifest file.

If this app is loaded into a Corda 6 node, that node may implement backwards compatibility workarounds for your app that make it slower,
less secure, or less featureful. You can opt-in to getting the full benefits of the upgrade by changing your target version to 6. By doing
this, you promise that you understood all the changes in Corda 6 and have thoroughly tested your app to prove it works. This testing should
include ensuring that the app exhibits the correct behaviour on a node running at the new target version, and that the app functions
correctly in a network of nodes running at the same target version.

Target versioning is one of the mechanisms we have to keep the platform evolving and improving, without being permanently constrained to
being bug-for-bug compatible with old versions. When no apps are loaded that target old versions, any emulations of older bugs or problems
can be disabled.

在你的 JAR manifests 中发布版本
-----------------------------------------

A well structured CorDapp should be split into two separate modules:

1. A contracts jar, that contains your states and contract logic.
2. A workflows jar, that contains your flows, services and other support libraries.

The reason for this split is that the contract JAR will be attached to transactions and sent around the network, because this code is what
defines the data structures and smart contract logic all nodes will validate. If the rest of your app is a part of that same JAR, it'll get
sent around the network too even though it's not needed and will never be used. By splitting your app into a contracts JAR and a workflows
JAR that depends on the contracts JAR, this problem is avoided.

In the ``build.gradle`` file for your contract module, add a block like this::

    cordapp {
        targetPlatformVersion 5
        minimumPlatformVersion 4
        contract {
            name "MegaApp Contracts"
            vendor "MegaCorp"
            licence "MegaLicence"
            versionId 1
        }
    }

This will put the necessary entries into your JAR manifest to set both platform version numbers. If they aren't specified, both default to 1.
Your app can itself has a version number, which should always increment and must also always be an integer.

And in the ``build.gradle`` file for your workflows jar, add a block like this::

    cordapp {
        targetPlatformVersion 5
        minimumPlatformVersion 4
        workflow {
            name "MegaApp"
            vendor "MegaCorp"
            licence "MegaLicence"
            versionId 1
        }
    }

It's entirely expected and reasonable to have an open source contracts module and a proprietary workflow module - the latter may contain
sophisticated or proprietary business logic, machine learning models, even user interface code. There's nothing that restricts it to just
being Corda flows or services.

.. important:: The ``versionId`` specified for the JAR manifest is checked by the platform and is used for informative purposes only.
 See ":ref:`contract_downgrade_rule_ref`" for more information.

.. note:: You can read the original design doc here: :doc:`design/targetversion/design`.