快速搭建 CorDapp 开发环境
======================================

软件需求
---------------------

Corda uses industry-standard tools:

Corda 使用业界标准的工具：

* **Java 8 JVM** - we require at least version **|java_version|**, but do not currently support Java 9 or higher.
  We have tested with Oracle JDK, Amazon Corretto, and Red Hat's OpenJDK builds. Please note that OpenJDK builds
  usually exclude JavaFX, which our GUI tools require.
* **IntelliJ IDEA** - supported versions **2017.x** and **2018.x** (with Kotlin plugin version |kotlin_version|)
* **Gradle** - we use 4.10 and the ``gradlew`` script in the project / samples directories will download it for you.

* **Java 8 JVM** - 我们要求最低版本为 **|java_version|**，但是当前还不支持 Java 9 或更高版本。我们测试过 Orable JDK，Amazon Corretoo 和 Read Hat 的 OpenJDK builds。注意 OpenJDK builds 通常会排除 JavaFX，这个是我们的 GUI 工具必须的。
* **IntelliJ IDEA** - 支持的版本 **2017.x** 和 **2018.x** （Kotlin plugin 版本 |kotlin_version|）
* **Gradle** - 我们使用 4.10 并且在 project / samples 路径下的 ``gradlew`` 脚本会为你下载它。

Please note:

请注意：

* Applications on Corda (CorDapps) can be written in any language targeting the JVM. However, Corda itself and most of
  the samples are written in Kotlin. Kotlin is an
  `official Android language <https://developer.android.com/kotlin/index.html>`_, and you can read more about why
  Kotlin is a strong successor to Java
  `here <https://medium.com/@octskyward/why-kotlin-is-my-next-programming-language-c25c001e26e3>`_. If you're
  unfamiliar with Kotlin, there is an official
  `getting started guide <https://kotlinlang.org/docs/tutorials/>`_, and a series of
  `Kotlin Koans <https://kotlinlang.org/docs/tutorials/koans.html>`_

* IntelliJ IDEA is recommended due to the strength of its Kotlin integration.

* 基于 Corda 的应用程序（CorDapps）能够使用任何能够运行在 JVM 中的语言来编写。但是 Corda 本身以及大多数的样例程序都是用 Kotlin 编写的。Kotlin 是 `Android 官方语言 <https://developer.android.com/kotlin/index.html>`_，你可以 `在这 <https://medium.com/@octskyward/why-kotlin-is-my-next-programming-language-c25c001e26e3>`_ 阅读更多关于为什么 Kotlin 是 Java 的后继者。如果你对 Kotlin 还不熟悉的话，这里有一个官方的 `指南 <https://kotlinlang.org/docs/tutorials/>`_，和一系列的 `Kotlin Koans <https://kotlinlang.org/docs/tutorials/koans.html>`_。

Following these software recommendations will minimize the number of errors you encounter, and make it easier for
others to provide support. However, if you do use other tools, we'd be interested to hear about any issues that arise.

使用这些推荐的软件开发 CorDapp 可以最小化产生问题的几率，也能够让他人提供支持的时候更加容易。然而，如果你在使用其他的工具，我们也希望能够听到你反馈的任何问题。

安装指导
-------------------
The instructions below will allow you to set up your development environment for running Corda and writing CorDapps. If
you have any issues, please reach out on `Stack Overflow <https://stackoverflow.com/questions/tagged/corda>`_ or via
`our Slack channels <http://slack.corda.net/>`_.

下边的指导内容可以告诉你如何配置一个 Corda 开发环境来运行和编写一个基本的 CorDapp。如果你遇到任何问题，可以在 `Stack Overflow <https://stackoverflow.com/questions/tagged/corda>`_ 或者 `我们的 Slack channels <http://slack.corda.net/>`_ 中提问。

The set-up instructions are available for the following platforms:

* :ref:`windows-label` (or `in video form <https://vimeo.com/217462250>`__)

* :ref:`mac-label` (or `in video form <https://vimeo.com/217462230>`__)

* :ref:`deb-ubuntu-label`

* :ref:`fedora-label`

.. _windows-label:

安装指导包含以下平台:

* :ref:`windows-label` (or `in video form <https://vimeo.com/217462250>`__)

* :ref:`mac-label` (or `in video form <https://vimeo.com/217462230>`__)

* :ref:`deb-ubuntu-label`

* :ref:`fedora-label`

.. _windows-label:

Windows
-------

.. warning:: If you are using a Mac, Debian/Ubuntu or Fedora machine, please follow the :ref:`mac-label`, :ref:`deb-ubuntu-label` or :ref:`fedora-label` instructions instead.

.. warning:: 如果你在使用 Mac、Debian/Ubuntu 或者 Fedora 机器，请按照 :ref:`mac-label`、 :ref:`deb-ubuntu-label` 或者 :ref:`fedora-label` 的指导来操作.

Java
^^^^
1. Visit http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
2. Scroll down to "Java SE Development Kit 8uXXX" (where "XXX" is the latest minor version number)
3. Toggle "Accept License Agreement"
4. Click the download link for jdk-8uXXX-windows-x64.exe (where "XXX" is the latest minor version number)
5. Download and run the executable to install Java (use the default settings)
6. Add Java to the PATH environment variable by following the instructions at https://docs.oracle.com/javase/7/docs/webnotes/install/windows/jdk-installation-windows.html#path
7. Open a new command prompt and run ``java -version`` to test that Java is installed correctly

1. 访问 http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
2. 到 “Java SE Development Kit 8uXXX”（XXX 表示最新的版本号）
3. 选择 “Accept License Agreement”
4. 点击 jdk-8uXXX-windows-x64.exe 的下载链接（XXX 表示最新的版本号）
5. 下载并运行Java 的安装文件（使用默认设置）
6. 跟着下边的指导将 Java 添加到 PATH 环境变量中 https://docs.oracle.com/javase/7/docs/webnotes/install/windows/jdk-installation-windows.html#path
7. 打开一个新的命令窗口然后运行 ``java -version`` 来测试一下 Java 是否正确安装了

Git
^^^
1. Visit https://git-scm.com/download/win
2. Click the "64-bit Git for Windows Setup" download link.
3. Download and run the executable to install Git (use the default settings)
4. Open a new command prompt and type ``git --version`` to test that git is installed correctly

1. 访问 https://git-scm.com/download/win
2. 点击下载链接 “64-bit Git for Windows Setup”
3. 下载并运行 Git 安装文件（使用默认设置）
4. 打开一个新的命令窗口然后运行 git --version 来测试一下 Git 是否正确安装了

IntelliJ
^^^^^^^^
1. Visit https://www.jetbrains.com/idea/download/download-thanks.html?code=IIC
2. Download and run the executable to install IntelliJ Community Edition (use the default settings)
3. Ensure the Kotlin plugin in Intellij is updated to version |kotlin_version|

1. 访问 https://www.jetbrains.com/idea/download/download-thanks.html?code=IIC
2. 下载并运行 InteliJ Community Edition 安装文件（使用默认设置）
3. 确保 Intellij 中的 Kotlin plugin 版本是 |kotlin_version|

.. _mac-label:

Mac
---

.. warning:: If you are using a Windows, Debian/Ubuntu or Fedora machine, please follow the :ref:`windows-label`, :ref:`deb-ubuntu-label` or :ref:`fedora-label` instructions instead.

.. warning:: 如果你在使用 Windows、Debian/Ubuntu 或者 Fedora 机器，请按照 :ref:`windows-label`、 :ref:`deb-ubuntu-label` 或者 :ref:`fedora-label` 的指导来操作.

Java
^^^^
1. Visit http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
2. Scroll down to "Java SE Development Kit 8uXXX" (where "XXX" is the latest minor version number)
3. Toggle "Accept License Agreement"
4. Click the download link for jdk-8uXXX-macosx-x64.dmg (where "XXX" is the latest minor version number)
5. Download and run the executable to install Java (use the default settings)
6. Open a new terminal window and run ``java -version`` to test that Java is installed correctly

1. 访问 http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
2. 到 “Java SE Development Kit 8uXXX”（XXX 表示最新的版本号）
3. 选择 “Accept License Agreement”
4. 点击 jdk-8uXXX-windows-x64.exe 的下载链接（XXX 表示最新的版本号）
5. 下载并运行Java 的安装文件（使用默认设置）
6. 打开一个新的命令窗口然后运行 ``java -version`` 来测试一下 Java 是否正确安装了

IntelliJ
^^^^^^^^
1. Visit https://www.jetbrains.com/idea/download/download-thanks.html?platform=mac&code=IIC
2. Download and run the executable to install IntelliJ Community Edition (use the default settings)
3. Ensure the Kotlin plugin in Intellij is updated to version |kotlin_version|

1. 访问 https://www.jetbrains.com/idea/download/download-thanks.html?platform=mac&code=IIC
2. 下载并运行 InteliJ Community Edition 安装文件（使用默认设置）
3. 确保 Intellij 中的 Kotlin plugin 版本是 |kotlin_version|

.. _deb-ubuntu-label:

Debian/Ubuntu
-------------

.. warning:: If you are using a Mac, Windows or Fedora machine, please follow the :ref:`mac-label`, :ref:`windows-label` or :ref:`fedora-label` instructions instead.

.. warning:: 如果你在使用 Mac、Windows 或者 Fedora 机器，请按照 :ref:`mac-label`、:ref:`windows-label` 或者 :ref:`fedora-label` 的指导来操作.

These instructions were tested on Ubuntu Desktop 18.04 LTS.

这个指导已经在 Ubuntu Desktop 18.04 LTS 中测试过。

Java
^^^^
1. Open a new terminal and add the Oracle PPA to your repositories by typing ``sudo add-apt-repository ppa:webupd8team/java``. Press ENTER when prompted.
2. Update your packages list with the command ``sudo apt update``
3. Install the Oracle JDK 8 by typing ``sudo apt install oracle-java8-installer``. Press Y when prompted and agree to the licence terms.
4. Verify that the JDK was installed correctly by running ``java -version``

1. 打开一个新的 terminal 并且通过输入 ``sudo add-apt-repository ppa:webupd8team/java`` 来将Oracle PPA 添加到你的 repositories。但弹出提示时，点击 ENTER。
2. 通过命令 ``sudo apt update`` 更新你的包列表。
3. 通过输入 ``sudo apt install oracle-java8-installer`` 安装 Oracle JDK 8。弹出提示时输入 Y 并且同意 licence 条款。
4. 通过运行 ``java -version`` 确认 JDK 已经正确被安装

Git
^^^^
1. From the terminal, Git can be installed using apt with the command ``sudo apt install git``
2. Verify that git was installed correctly by typing ``git --version``

1. 在 terminal 中，Git 可以通过使用 apt 命令 ``sudo apt install git`` 来安装
2. 通过运行 ``git --version`` 确认 git 已经被正确安装

IntelliJ
^^^^^^^^
Jetbrains offers a pre-built snap package that allows for easy, one-step installation of IntelliJ onto Ubuntu.

1. To download the snap, navigate to https://snapcraft.io/intellij-idea-community
2. Click ``Install``, then ``View in Desktop Store``. Choose ``Ubuntu Software`` in the Launch Application window.
3. Ensure the Kotlin plugin in Intellij is updated to version |kotlin_version|

为了在 Ubuntu 上更简单地，一步安装 IntelliJ，Jetbrains 提供了一个预建的 snap package。

1. 浏览 https://snapcraft.io/intellij-idea-community 下载这个 snap
2. 点击 ``Install``，然后 ``View in Desktop Store``。在加载应用程序窗口中选择 ``Ubuntu Software``。
3. 确保在 IntelliJ 中的 Kotlin plugin 已经更新到版本 |kotlin_version|

.. _fedora-label:

Fedora
------

.. warning:: If you are using a Mac, Windows or Debian/Ubuntu machine, please follow the :ref:`mac-label`, :ref:`windows-label` or :ref:`deb-ubuntu-label` instructions instead.

.. warning:: 如果你在使用 Mac、Windows 或者 Debian/Ubuntu 机器，请按照 :ref:`mac-label`、:ref:`windows-label` 或者 :ref:`deb-ubuntu-label` 的指导来操作.

These instructions were tested on Fedora 28.

这个指导已经在 Fedora 28 中测试过。

Java
^^^^
1. Download the RPM installation file of Oracle JDK from https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html.
2. Install the package with ``rpm -ivh jdk-<version>-linux-<architecture>.rpm`` or use the default software manager.
3. Choose java version by using the following command ``alternatives --config java``
4. Verify that the JDK was installed correctly by running ``java -version``

1. 从 https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html 上下载 Oracle JDK 的 RPM 安装文件
2. 通过命令 ``rpm -ivh jdk-<version>-linux-<architecture>.rpm`` 安装这个包，或者使用默认的软件管理器
3. 通过使用下边的命令 ``alternatives --config java`` 来选择 Java 版本
4. 通过运行 ``java -version`` 确认 Java 已经正确地安装

Git
^^^^
1. From the terminal, Git can be installed using dnf with the command ``sudo dnf install git``
2. Verify that git was installed correctly by typing ``git --version``

1. 从 terminal 中，通过运行命令 ``sudo dnf install git`` 来安装 Git
2. 通过运行 ``git --version`` 来确认 git 已经被正确安装了

IntelliJ
^^^^^^^^
1. Visit https://www.jetbrains.com/idea/download/download-thanks.html?platform=linux&code=IIC
2. Unpack the ``tar.gz`` file using the following command ``tar xfz ideaIC-<version>.tar.gz -C /opt``
3. Run IntelliJ with ``/opt/ideaIC-<version>/bin/idea.sh``
4. Ensure the Kotlin plugin in IntelliJ is updated to version |kotlin_version|

1. 访问 https://www.jetbrains.com/idea/download/download-thanks.html?platform=linux&code=IIC
2. 通过命令 ``tar xfz ideaIC-<version>.tar.gz -C /opt`` 解压 ``tar.gz`` 文件
3. 通过 ``/opt/ideaIC-<version>/bin/idea.sh`` 运行 IntelliJ
4. 确保在 IntelliJ 中的 Kotlin plugin 已经更新到版本 |kotlin_version|

接下来的步骤
----------
First, run the :doc:`example CorDapp <tutorial-cordapp>`.

首先，运行 :doc:`CorDapp 样例 <tutorial-cordapp>`。

Next, read through the :doc:`Corda Key Concepts <key-concepts>` to understand how Corda works.

接下来，阅读 :doc:`Corda 核心概念 <key-concepts>` 来理解 Corda 是如何工作的。

By then, you'll be ready to start writing your own CorDapps. Learn how to do this in the
:doc:`Hello, World tutorial <hello-world-introduction>`. You may want to refer to the
:doc:`API documentation <corda-api>`, the :doc:`flow cookbook <flow-cookbook>` and the
`samples <https://www.corda.net/samples/>`_ along the way.

然后，你就已经准备好要开始编写你自己的 CorDapps 了。在 :doc:`Hello, World tutorial <hello-world-introduction>` 中学习如何做这些。在这个过程中，你可能想要参考 :doc:`API 文档 <corda-api>`，:doc:`flow cookbook <flow-cookbook>` 以及 `例子 <https://www.corda.net/samples/>`_。

If you encounter any issues, please ask on `Stack Overflow <https://stackoverflow.com/questions/tagged/corda>`_ or via
`our Slack channels <http://slack.corda.net/>`_.

如果你遇到任何的困难，请在 `Stack Overflow <https://stackoverflow.com/questions/tagged/corda>`_ 或者通过 `我们的 Slack channels <http://slack.corda.net/>`_ 提问。
