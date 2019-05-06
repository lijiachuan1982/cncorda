访问 H2 数据库
===============================

.. contents::

配置用户名和密码
-------------------------------------

The database (a file called ``persistence.mv.db``) is created when the node first starts up. By default, it has an
administrator user ``sa`` and a blank password. The node requires the user with administrator permissions in order to
creates tables upon the first startup or after deploying new CorDapps with their own tables. The database password is
required only when the H2 database is exposed on non-localhost address (which is disabled by default).

当节点第一次启动的时候，数据库（一个名字为 ``persistence.mv.db`` 的文件）会被创建。默认的它会有一个 ``sa`` 用户和一个空密码。节点需要这个用户有管理员权限，这样才能够在第一次启动或者是在部署带有他们自己的 tables 的新的 CorDapps 之后创建这些表。数据库的密码只有在 H2 数据库暴露在非本地的地址的时候才是必须要有值的（默认的是被 disable 的）。

This username and password can be changed in node configuration:

用户名和密码可以在节点配置中进行改动：

 .. sourcecode:: groovy

     dataSourceProperties = {
        dataSource.user = [USER]
        dataSource.password = [PASSWORD]
    }

Note that changing the user/password for the existing node in ``node.conf`` will not update them in the H2 database.
You need to log into the database first to create a new user or change a user's password.

注意，对于已经存在的节点，在 ``node.conf`` 中改变用户名和密码是不会在 H2 数据库中改变他们的。你需要先登录到数据库，创建一个新的用户，或者改变用户的密码。

在一个运行的节点上通过一个 socket 连接
-----------------------------------------

配置端口
^^^^^^^^^^^^^^^^^^^^

Nodes backed by an H2 database will not expose this database by default. To configure the node to expose its internal
database over a socket which can be browsed using any tool that can use JDBC drivers, you must specify the full network
address (interface and port) using the ``h2Settings`` syntax in the node configuration.

使用 H2 数据库的节点默认是不会暴露这个数据库的。为了配置节点通过一个 socket 暴露它的内部的数据库，以便用任何能够使用 JDBC Driver 的工具浏览，你必须要在节点的配置中使用 ``h2Settings`` 语法指定完整的网络地址（接口和端口）

The configuration below will restrict the H2 service to run on ``localhost``:

下边的配置将会限制 H2 服务运行在 ``localhost`` 上：

.. sourcecode:: groovy

  h2Settings {
      address: "localhost:12345"
  }

If you want H2 to auto-select a port (mimicking the old ``h2Port`` behaviour), you can use:

如果你希望 H2 自动选择一个端口（模仿旧的 ``h2Port`` 行为），你可以使用：

.. sourcecode:: groovy

  h2Settings {
      address: "localhost:0"
  }

If remote access is required, the address can be changed to ``0.0.0.0`` to listen on all interfaces. A password must be
set for the database user before doing so.

如果需要远程访问，地址可以被改成 ``0.0.0.0`` 来监听所有的接口。在做这个之前，一个密码必须要为数据库用户设置好。

.. sourcecode:: groovy

  h2Settings {
      address: "0.0.0.0:12345"
  }
  dataSourceProperties {
      dataSource.password : "strongpassword"
  }

.. note:: The previous ``h2Port`` syntax is now deprecated. ``h2Port`` will continue to work but the database will only
   be accessible on localhost.

.. note:: 以前的 ``h2Port`` 语法已经废弃了。``h2Port`` 还会继续工作，但是数据库仅仅可以从 localhost 访问了。

连接到数据库
^^^^^^^^^^^^^^^^^^^^^^^^^^
The JDBC URL is printed during node startup to the log and will typically look like this:

JDBC URL 会在节点启动的时候被打印到 log，并且通常会像下边这样：

     ``jdbc:h2:tcp://localhost:31339/node``

Any database browsing tool that supports JDBC can be used.

任何支持 JDBC 的数据库浏览工具都能够用来浏览数据库。

通过 H2 Console 连接
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Download the **last stable** `h2 platform-independent zip <http://www.h2database.com/html/download.html>`_, unzip the
  zip, and navigate in a terminal window to the unzipped folder
  下载 **最新稳定版本** `h2 platform-independent zip <http://www.h2database.com/html/download.html>`_，解压 zip，在一个 terminal 窗口浏览至解压的文件夹

* Change directories to the bin folder: ``cd h2/bin``
  将路径改变到 bin 文件夹：``cd h2/bin``

* Run the following command to open the h2 web console in a web browser tab:
  运行下边的命令在一个 web 浏览器的 tab 里打开 h2 web console：

  * Unix: ``sh h2.sh``
  * Windows: ``h2.bat``

* Paste the node's JDBC URL into the JDBC URL field and click ``Connect``, using the default username (``sa``) and no
  password (unless configured otherwise)
  将节点的 JDBC URL 粘贴到 JDBC URL 字段并且点击 ``Connect``，使用默认的用户名（``sa``）并且不需要密码（除非你配置了密码）

You will be presented with a web interface that shows the contents of your node's storage and vault, and provides an
interface for you to query them using SQL.

你会看到一个 web 接口，显示了你的节点的存储和 vault 的内容，并且提供给你一个接口来使用 SQL 查询他们。

.. _h2_relative_path:

直接连接到节点的 ``persistence.mv.db`` 文件
------------------------------------------------------------

You can also use the H2 Console to connect directly to the node's ``persistence.mv.db`` file. Ensure the node is off
before doing so, as access to the database file requires exclusive access. If the node is still running, the H2 Console
will return the following error:
你也可以使用 H2 Console 直接连到节点的 ``persistence.mv.db`` 文件。确保在做这个之前节点是关闭的，因为访问数据库需要一个独占的访问。如果节点还是在运行的话，H2 console 会返回下边的错误：

``Database may be already in use: null. Possible solutions: close all other connection(s); use the server mode [90020-196]``.

    ``jdbc:h2:~/path/to/file/persistence``
