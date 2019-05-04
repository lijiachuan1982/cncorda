Debugging a CorDapp
===================

.. contents::

There are several ways to debug your CorDapp.

有多种方式来 debug 你的 CorDapp。

使用 ``MockNetwork``
-----------------------

You can attach the `IntelliJ IDEA debugger <https://www.jetbrains.com/help/idea/debugging-code.html>`_ to a
``MockNetwork`` to debug your CorDapp:

* Define your flow tests as per :doc:`api-testing`

    * In your ``MockNetwork``, ensure that ``threadPerNode`` is set to ``false``

* Set your breakpoints
* Run the flow tests using the debugger. When the tests hit a breakpoint, execution will pause

你可以把 `IntelliJ IDEA debugger <https://www.jetbrains.com/help/idea/debugging-code.html>`_ 附加到一个 ``MockNetwork`` 来 debug 你的 CorDap：

* 根据 :doc:`api-testing` 来定义你的 flow tests

    * 在你的 ``MockNetwork`` 中，确保 ``threadPerNode`` 设置为 ``false``

* 添加断点
* 使用 debugger 运行 flow tests。当测试达到一个断点的时候，执行会被暂停

使用 node driver
---------------------

You can also attach the `IntelliJ IDEA debugger <https://www.jetbrains.com/help/idea/debugging-code.html>`_ to nodes
running via the node driver to debug your CorDapp.

你可以把 `IntelliJ IDEA debugger <https://www.jetbrains.com/help/idea/debugging-code.html>`_ 通过 node driver 附加到节点上来 debug 你的 CorDapp。

对于正在执行的节点
^^^^^^^^^^^^^^^^^^^^^^^^^

1. Define a network using the node driver as per :doc:`tutorial-integration-testing`

    * In your ``DriverParameters``, ensure that ``startNodesInProcess`` is set to ``true``

2. Run the driver using the debugger

3. Set your breakpoints

4. Interact with your nodes. When execution hits a breakpoint, execution will pause

    * The nodes' webservers always run in a separate process, and cannot be attached to by the debugger

1. 按照 :doc:`tutorial-integration-testing` 中使用 node driver 定义一个网络

    * 在 ``DriverParameters`` 中，确保 ``startNodesInProcess`` 设置为 ``true``

2. 使用 debugger 运行 driver
3. 设置断点
4. 跟你的节点进行互动。当执行到断点的位置时，执行会被暂停

    * 节点的 webservers 总会在一个单独的进程中运行，并且不能够被 debugger 附加

使用远程 debugging
^^^^^^^^^^^^^^^^^^^^^

1. Define a network using the node driver as per :doc:`tutorial-integration-testing`

    * In your ``DriverParameters``, ensure that ``startNodesInProcess`` is set to ``false`` and ``isDebug`` is set to
      ``true``

2. Run the driver. The remote debug ports for each node will be automatically generated and printed to the terminal.
   For example:

.. sourcecode:: none

    [INFO ] 11:39:55,471 [driver-pool-thread-0] (DriverDSLImpl.kt:814) internal.DriverDSLImpl.startOutOfProcessNode -
        Starting out-of-process Node PartyA, debug port is 5008, jolokia monitoring port is not enabled {}

3. Attach the debugger to the node of interest on its debug port:

    * In IntelliJ IDEA, create a new run/debug configuration of type ``Remote``
    * Set the run/debug configuration's ``Port`` to the debug port
    * Start the run/debug configuration in debug mode

4. Set your breakpoints

5. Interact with your node. When execution hits a breakpoint, execution will pause

    * The nodes' webservers always run in a separate process, and cannot be attached to by the debugger

1. 像 :doc:`tutorial-integration-testing` 中使用 node driver 定义一个网络

    * 在 ``DriverParameter`` 中，确保 ``startNodesInProcess`` 设置为 ``false`` 并且 ``isDubug`` 设置为 ``true``

2. 运行 driver。每个节点的远程 debug 端口会自动生成并打印到终端中。像下边这样：

.. sourcecode:: none

    [INFO ] 11:39:55,471 [driver-pool-thread-0] (DriverDSLImpl.kt:814) internal.DriverDSLImpl.startOutOfProcessNode -
        Starting out-of-process Node PartyA, debug port is 5008, jolokia monitoring port is not enabled {}

3. 将 debugger 附加到节点的 debug 端口：

    * 在 IntelliJ IDEA，创建一个 类型为 ``Remote`` 的 run/debug 配置
    * 将 run/debug 配置的 ``Port`` 设置为 debug 端口
    * 在 debug 模式启动 run/debug 配置

4. 添加断点
5. 跟你的节点互动，当执行到断点的时候，执行会被暂停

    * 节点的 web servers 会一直在独立的一个进程中运行，不会被 debugger 附带

在节点上开启远程 debugging
--------------------------------------

查看 :ref:`enabling-remote-debugging`.