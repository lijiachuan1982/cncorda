Deploying a node to a server
============================

.. contents::

.. note:: These instructions are intended for people who want to deploy a Corda node to a server,
   whether they have developed and tested a CorDapp following the instructions in :doc:`generating-a-node`
   or are deploying a third-party CorDapp.

.. note:: 这个指导是为了需要将一个 Corda 节点部署到 server 上的开发者，他们或是按照在 :doc:`generating-a-node` 的指引来开发并测试了一个 CorDapp，或是在部署一个第三方的 CorDapp。

Linux：安装并且运行 Corda 作为一个系统的服务
-------------------------------------------------------
We recommend creating system services to run a node and the optional webserver. This provides logging and service
handling, and ensures the Corda service is run at boot.

我们建议创建一个系统服务来运行节点，还可以选择用系统服务来运行 webserver。这个会提供 logging 和 service handling，并且确保了 Corda 服务会在 server 启动时自动启动。

**Prerequisites**:

   * A supported Java distribution. The supported versions are listed in :doc:`getting-set-up`

   * 一个支持的 Java distribution。支持的版本在 :doc:`getting-set-up` 中有说明

1. As root/sys admin user - add a system user which will be used to run Corda:

   作为 root/sys 管理员用户 - 添加一个被用来运行 Corda 的系统用户

    ``sudo adduser --system --no-create-home --group corda``


2. Create a directory called ``/opt/corda`` and change its ownership to the user you want to use to run Corda:

   创建一个名为 ``/opt/corda`` 的路径然后将它的 ownership 变为要运行 Corda 的那个用户

   ``mkdir /opt/corda; chown corda:corda /opt/corda``

3. Download the `Corda jar <https://r3.bintray.com/corda/net/corda/corda/>`_
   (under ``/|corda_version|/corda-|corda_version|.jar``) and place it in ``/opt/corda``

   下载 `Corda jar <https://r3.bintray.com/corda/net/corda/corda/>`_ （在 ``/|corda_version|/corda-|corda_version|.jar`` 下）并且把它放在 ``/opt/corda`` 里

4. (Optional) Download the `Corda webserver jar <http://r3.bintray.com/corda/net/corda/corda-webserver/>`_
   (under ``/|corda_version|/corda-|corda_version|.jar``) and place it in ``/opt/corda``

   （可选步骤）下载 `Corda webserver jar <http://r3.bintray.com/corda/net/corda/corda-webserver/>`_（在 ``/|corda_version|/corda-|corda_version|.jar`` 下）并且把它放在 ``/opt/corda`` 里

5. Create a directory called ``cordapps`` in ``/opt/corda`` and save your CorDapp jar file to it. Alternatively, download one of
   our `sample CorDapps <https://www.corda.net/samples/>`_ to the ``cordapps`` directory

   在 ``/opt/corda`` 里创建一个名为 ``cordapps`` 的路径，并且将你的 CorDapp jar 文件放到里边。你也可以下载我们的 `CorDapps 样例 <https://www.corda.net/samples/>`_ 到 ``cordapps`` 路径下

6. Save the below as ``/opt/corda/node.conf``. See :doc:`corda-configuration-file` for a description of these options::

   将以下的 ``node.conf`` 保存到 ``/opt/corda/node.conf``。查看 :doc:`corda-configuration-file` 了解这些配置项的描述::

      p2pAddress = "example.com:10002"
      rpcSettings {
          address: "example.com:10003"
          adminAddress: "example.com:10004"
      }
      h2port = 11000
      emailAddress = "you@example.com"
      myLegalName = "O=Bank of Breakfast Tea, L=London, C=GB"
      keyStorePassword = "cordacadevpass"
      trustStorePassword = "trustpass"
      devMode = false
      rpcUsers= [
          {
              user=corda
              password=portal_password
              permissions=[
                  ALL
              ]
          }
      ]
      custom { jvmArgs = [ '-Xmx2048m', '-XX:+UseG1GC' ] }

7. Make the following changes to ``/opt/corda/node.conf``:

   *  Change the ``p2pAddress``, ``rpcSettings.address`` and ``rpcSettings.adminAddress`` values to match
      your server's hostname or external IP address. These are the addresses other nodes or RPC interfaces will use to
      communicate with your node.
   *  Change the ports if necessary, for example if you are running multiple nodes on one server (see below).
   *  Enter an email address which will be used as an administrative contact during the registration process. This is
      only visible to the permissioning service.
   *  Enter your node's desired legal name (see :ref:`node-naming` for more details).
   *  If required, add RPC users

7. 对 ``/opt/corda/node.conf`` 进行下边的修改

   *  将 ``p2pAddress``、``rpcSettings.address`` 和 ``rpcSettings.adminAddress`` 的值修改为以你的 server hostname 或者外部的 IP address 开始。这个地址会被其他的节点或 RPC 接口用来和你的节点进行沟通
   *  如果需要的话改变端口号，比如你在同一个 server 上运行了多个节点
   *  输入一个 emial address，会在注册的流程中作为管理员的联系方式。这个只有 permissioning service 能够看到
   *  输入你的节点期望的 legal name（查看 :ref:`node-naming` 了解更多信息）。
   *  如何需要的话，添加 RPC 用户

.. note:: Ubuntu 16.04 and most current Linux distributions use SystemD, so if you are running one of these
          distributions follow the steps marked **SystemD**. 
          If you are running Ubuntu 14.04, follow the instructions for **Upstart**.

.. note:: Ubuntu 16.04 以及大多数当前的 Linux distributions 使用 SystemD，所以如果你在运行着这些 distributions 中的一个，那你需要按照下边标记为 **SystemD** 的步骤。如果你运行的是 Ubuntu 14.04，那么按照下边的标记为 **Upstart** 的步骤。

8. **SystemD**: Create a ``corda.service`` file based on the example below and save it in the ``/etc/systemd/system/``
   directory

   **SystemD**：根据下边的例子创建一个 ``corda.service`` 文件，并且将它保存在 ```/etc/systemd/system/`` 路径下

    .. code-block:: shell

       [Unit]
       Description=Corda Node - Bank of Breakfast Tea
       Requires=network.target

       [Service]
       Type=simple
       User=corda
       WorkingDirectory=/opt/corda
       ExecStart=/usr/bin/java -jar /opt/corda/corda.jar
       Restart=on-failure

       [Install]
       WantedBy=multi-user.target

8. **Upstart**: Create a ``corda.conf`` file based on the example below and save it in the ``/etc/init/`` directory

   **Upstart**：根据下边的例子创建一个 ``corda.conf`` 文件，并且将它保存在 `/etc/init/`` 路径下

    .. code-block:: shell

        description "Corda Node - Bank of Breakfast Tea"

        start on runlevel [2345]
        stop on runlevel [!2345]

        respawn
        setuid corda
        chdir /opt/corda
        exec java -jar /opt/corda/corda.jar

9. Make the following changes to ``corda.service`` or ``corda.conf``:

    * Make sure the service description is informative - particularly if you plan to run multiple nodes.
    * Change the username to the user account you want to use to run Corda. **We recommend that this user account is
      not root**
    * **SystemD**: Make sure the ``corda.service`` file is owned by root with the correct permissions:

        * ``sudo chown root:root /etc/systemd/system/corda.service``
        * ``sudo chmod 644 /etc/systemd/system/corda.service``

    * **Upstart**: Make sure the ``corda.conf`` file is owned by root with the correct permissions:

        * ``sudo chown root:root /etc/init/corda.conf``
        * ``sudo chmod 644 /etc/init/corda.conf``

9. 按照下边修改 ``corda.service`` 或者 ``corda.conf``：

    * 确保 service 描述是有意义的 - 特别是你想要运行多个节点的时候
    * 将 username 修改成你想要用来运行 Corda 的用户账户。**我们建议这个用户账号不是 root**
    * **SystemD*：确保 ``corda.service`` 文件是 root 所有并且有正确的权限：

        * ``sudo chown root:root /etc/systemd/system/corda.service``
        * ``sudo chmod 644 /etc/systemd/system/corda.service``

    * **Upstart**：确保 ``corda.conf`` 是被 root 所有并且有正确的权限：

        * ``sudo chown root:root /etc/init/corda.conf``
        * ``sudo chmod 644 /etc/init/corda.conf``

.. note:: The Corda webserver provides a simple interface for interacting with your installed CorDapps in a browser.
   Running the webserver is optional.

.. note:: Corda webserver 提供了一个在浏览器中能够跟你安装的 CorDapps 进行互动的简单接口。运行 webserver 不是必须的。

10. **SystemD**: Create a ``corda-webserver.service`` file based on the example below and save it in the ``/etc/systemd/system/``
    directory

    **SystemD**：根据下边的例子创建一个 ``corda-webserver.service`` 文件并把它存在 ``/etc/systemd/system/`` 路径下

    .. code-block:: shell

       [Unit]
       Description=Webserver for Corda Node - Bank of Breakfast Tea
       Requires=network.target

       [Service]
       Type=simple
       User=corda
       WorkingDirectory=/opt/corda
       ExecStart=/usr/bin/java -jar /opt/corda/corda-webserver.jar
       Restart=on-failure

       [Install]
       WantedBy=multi-user.target

10. **Upstart**: Create a ``corda-webserver.conf`` file based on the example below and save it in the ``/etc/init/``
    directory

    **Upstart**：基于下边的例子创建一个 ``corda-webserver.conf`` 的文件并将它放在 ``/etc/init/`` 路径下

    .. code-block:: shell

        description "Webserver for Corda Node - Bank of Breakfast Tea"

        start on runlevel [2345]
        stop on runlevel [!2345]

        respawn
        setuid corda
        chdir /opt/corda
        exec java -jar /opt/corda/corda-webserver.jar

11. Provision the required certificates to your node. Contact the network permissioning service or see
    :doc:`permissioning`

    为你的节点生成证书。联系 network permissioning service 或者查看 :doc:`permissioning`

12. **SystemD**: You can now start a node and its webserver and set the services to start on boot by running the
    following ``systemctl`` commands:

    **SystemD**：现在你就可以启动一个节点和它的 webserver，通过运行下边的 ``systemctl`` 命令来将 service 设置为同系统启动一起运行：

   * ``sudo systemctl daemon-reload``
   * ``sudo systemctl enable --now corda``
   * ``sudo systemctl enable --now corda-webserver``

12. **Upstart**: You can now start a node and its webserver by running the following commands:

   **Upstart**：现在你就可以通过运行下边的命令启动一个节点和它的 webserver：

   * ``sudo start corda``
   * ``sudo start corda-webserver``

The Upstart configuration files created above tell Upstart to start the Corda services on boot so there is no need to explicitly enable them.

上边创建的 Upstart 配置文件会告诉 Upstart 在 server 重启的时候要运行 Corda services，所以这里不需要显式地开启他们。

You can run multiple nodes by creating multiple directories and Corda services, modifying the ``node.conf`` and
SystemD or Upstart configuration files so they are unique.

你可以通过创建多个路径和 Corda services 来运行多个节点，修改 ``node.conf`` 和 SystemD 或者 Upstart 配置文件，这样他们就都是唯一的了。

Windows：作为 Windows service 来安装和运行 Corda
----------------------------------------------------------
We recommend running Corda as a Windows service. This provides service handling, ensures the Corda service is run
at boot, and means the Corda service stays running with no users connected to the server.

我们建议将 Corda 作为一个 Windows service 来运行。这提供了 service handling，确保了 Corda 能够在系统重启后自动运行，这意味着我们不需要有人去连接到 server， Corda 就能够始终保持运行状态。

**Prerequisites**:

   * A supported Java distribution. The supported versions are listed in :doc:`getting-set-up`

   * 一个支持的 Java destribution。支持的版本在 :doc:`getting-set-up` 中有说明

1. Create a Corda directory and download the Corda jar. Here's an
   example using PowerShell::

   创建一个 Corda 目录，然后下载 Corda jar。下边是一个使用 powershell 的一个例子

        mkdir C:\Corda
        wget http://jcenter.bintray.com/net/corda/corda/|corda_version|/corda-|corda_version|.jar -OutFile C:\Corda\corda.jar

2. Create a directory called ``cordapps`` in ``C:\Corda\`` and save your CorDapp jar file to it. Alternatively,
   download one of our `sample CorDapps <https://www.corda.net/samples/>`_ to the ``cordapps`` directory

   在 ``C:\Corda\`` 下创建一个名为 ``cordapps`` 的目录，然后将你的 CorDapp jar 文件存储到这里。或者也可以从我们的 `CorDapps 样例 <https://www.corda.net/samples/>`_ 中下载一个放到 ``cordapps`` 目录下。

3. Save the below as ``C:\Corda\node.conf``. See :doc:`corda-configuration-file` for a description of these options::

   把下边的内容存储为 ``C:\Corda\node.conf``。查看 :doc:`corda-configuration-file` 来了解这些选项的详细介绍::

        p2pAddress = "example.com:10002"
        rpcSettings {
            address = "example.com:10003"
            adminAddress = "example.com:10004"
        }
        h2port = 11000
        emailAddress = "you@example.com"
        myLegalName = "O=Bank of Breakfast Tea, L=London, C=GB"
        keyStorePassword = "cordacadevpass"
        trustStorePassword = "trustpass"
        devMode = false
        rpcSettings {
           useSsl = false
           standAloneBroker = false
           address = "example.com:10003"
           adminAddress = "example.com:10004"
       }
       custom { jvmArgs = [ '-Xmx2048m', '-XX:+UseG1GC' ] }

4. Make the following changes to ``C:\Corda\node.conf``:

   *  Change the ``p2pAddress``, ``rpcSettings.address`` and ``rpcSettings.adminAddress`` values to match
      your server's hostname or external IP address. These are the addresses other nodes or RPC interfaces will use to
      communicate with your node.
   *  Change the ports if necessary, for example if you are running multiple nodes on one server (see below).
   *  Enter an email address which will be used as an administrative contact during the registration process. This is
      only visible to the permissioning service.
   *  Enter your node's desired legal name (see :ref:`node-naming` for more details).
   *  If required, add RPC users

4. 对 ``C:\Corda\node.conf`` 做以下的修改：

   *  将 ``p2pAddress``、``rpcSettings.address`` 和 ``rpcSettings.adminAddress`` 的值修改为以你的 server hostname 或者外部的 IP address 开始。这个地址会被其他的节点或 RPC 接口用来和你的节点进行沟通
   *  如果需要的话改变端口号，比如你在同一个 server 上运行了多个节点
   *  输入一个 emial address，会在注册的流程中作为管理员的联系方式。这个只有 permissioning service 能够看到
   *  输入你的节点期望的 legal name（查看 :ref:`node-naming` 了解更多信息）。
   *  如何需要的话，添加 RPC 用户

5. Copy the required Java keystores to the node. See :doc:`permissioning`
   将要求的 Java keystores 拷贝到节点。查看 :doc:`permissioning`

6. Download the `NSSM service manager <nssm.cc>`_
   下载 `NSSM service manager <nssm.cc>`_

7. Unzip ``nssm-2.24\win64\nssm.exe`` to ``C:\Corda``
   Upzip ``nssm-2.24\win64\nssm.exe`` 到 ``C:\Corda``

8. Save the following as ``C:\Corda\nssm.bat``:
   将下边的代码存储为 ``C:\Corda\nssm.bat``：

   .. code-block:: batch

      nssm install cordanode1 C:\ProgramData\Oracle\Java\javapath\java.exe
      nssm set cordanode1 AppDirectory C:\Corda
      nssm set cordanode1 AppStdout C:\Corda\service.log
      nssm set cordanode1 AppStderr C:\Corda\service.log
      nssm set cordanode1 Description Corda Node - Bank of Breakfast Tea
      nssm set cordanode1 Start SERVICE_AUTO_START
      sc start cordanode1

9. Modify the batch file:

    * If you are installing multiple nodes, use a different service name (``cordanode1``) for each node
    * Set an informative description

9. 修改这个 batch 文件：

    * 如果你安装了多个节点，对每个节点要使用不同的 service name（``cordanode1``）
    * 设置一个有意义的描述

10. Provision the required certificates to your node. Contact the network permissioning service or see
    :doc:`permissioning`

    为你的节点生成证书。联系网络权限服务或者查看 :doc:`permissioning`

11. Run the batch file by clicking on it or from a command prompt
   双击或者从命令行运行这个 batch file

12. Run ``services.msc`` and verify that a service called ``cordanode1`` is present and running
   运行 ``services.msc`` 并确认一个名为 ``cordanode1`` 的 service 显示并运行着

13. Run ``netstat -ano`` and check for the ports you configured in ``node.conf``
   运行 ``netstat -ano`` 并确认你在 ``node.conf`` 中设置的端口是否在运行

    * You may need to open the ports on the Windows firewall
       你可能需要在防火墙中打开这个端口

测试你的安装
-------------------------
You can verify Corda is running by connecting to your RPC port from another host, e.g.:

你可以通过另外的 host 来链接到你的 RPC 端口来确认 Corda 是否在运行：

        ``telnet your-hostname.example.com 10002``

If you receive the message "Escape character is ^]", Corda is running and accessible. Press Ctrl-] and Ctrl-D to exit
telnet.

如果你收到的消息是 “Escape character is ^]”，Corda 已经在运行并且可以访问了。按 Ctrl-] 和 Ctrl-D 退出 telnet。
