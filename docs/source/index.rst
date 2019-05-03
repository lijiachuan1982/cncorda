欢迎来到 Corda!
==================

`Corda <https://www.corda.net/>`_ is an open-source blockchain platform. If you’d like a quick introduction to blockchains and how Corda is different, then watch this short video:

`Corda <https://www.corda.net/>`_ 是一个开源的区块链平台。如果你想看到一个关于区块链以及 Corda 有哪些特性的简单介绍的话，那么你可以看看下边这个简短的视频：

.. raw:: html

    <embed>
      <iframe src="https://player.youku.com/embed/XMzM5NTY5NjU4OA==" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
    </embed>

Want to see Corda running? Download our demonstration application `DemoBench <https://www.corda.net/downloads/>`_ or
follow our :doc:`quickstart guide </quickstart-index>`.

想看看 Corda 是如何运行的？下载我们的展示应用 `DemoBench <https://www.corda.net/downloads/>`_ 或者参考我们的  :doc:`快速开始指南 </quickstart-index>`。

If you want to start coding on Corda, then familiarise yourself with the :doc:`key concepts </key-concepts>`, then read
our :doc:`Hello, World! tutorial </hello-world-introduction>`. For the background behind Corda, read the non-technical
`platform white paper`_ or for more detail, the `technical white paper`_.

如果你希望开始进行 Corda 开发的话，让你自己先熟悉一下 :doc:`核心概念 </key-concepts>`，并且读一读我们的 :doc:`Hello, World! tutorial </hello-world-introduction>`。关于在 Corda 背后的背景，阅读以下非技术的 `平台白皮书`_ 或者有更多详细内容的 `技术白皮书`_。

If you have questions or comments, then get in touch on `Slack <https://slack.corda.net/>`_ or ask a question on
`Stack Overflow <https://stackoverflow.com/questions/tagged/corda>`_ .

如果你有问题或者是评论的话，可以在 `Slack <https://slack.corda.net/>`_ 上联系，或者在 `Stack Overflow <https://stackoverflow.com/questions/tagged/corda>`_ 上提问题。

We look forward to seeing what you can do with Corda!

我们期待看到你将用 Corda 来实现什么！

.. note:: You can read this site offline. Either `download the PDF`_ or download the Corda source code, run ``gradle buildDocs`` and you will have
   a copy of this site in the ``docs/build/html`` directory.

.. note:: 你可以用离线的方式阅读这个网站。或者 `下载 PDF 文档`_ 或者下载 Corda 的源代码，运行 ``gradle buildDocs`` 然后在  ``docs/build/html`` 你就能得到这个网站的拷贝了。

.. _`平台白皮书`: _static/corda-platform-whitepaper.pdf
.. _`技术白皮书`: _static/corda-technical-whitepaper.pdf
.. _`下载 PDF 文档`: _static/corda-developer-site.pdf

.. toctree::
   :maxdepth: 1

   release-notes
   app-upgrade-notes
   node-upgrade-notes
   corda-api
   cheat-sheet

.. toctree::
   :caption: Development
   :maxdepth: 1

   quickstart-index.rst
   key-concepts.rst
   building-a-cordapp-index.rst
   tutorials-index.rst
   tools-index.rst
   node-internals-index.rst
   component-library-index.rst
   serialization-index.rst
   json.rst
   troubleshooting.rst

.. toctree::
   :caption: Operations
   :maxdepth: 2

   corda-nodes-index.rst
   corda-networks-index.rst
   docker-image.rst
   azure-vm.rst
   aws-vm.rst
   loadtesting.rst
   cli-application-shell-extensions.rst

.. Documentation is not included in the pdf unless it is included in a toctree somewhere

.. conditional-toctree::
   :caption: Corda Network
   :maxdepth: 2
   :if_tag: htmlmode

   corda-network/index.md
   corda-network/UAT.md

.. conditional-toctree::
   :caption: Contents
   :maxdepth: 2
   :if_tag: pdfmode

   deterministic-modules.rst
   release-notes.rst
   changelog.rst

.. conditional-toctree::
   :caption: Participate
   :maxdepth: 2
   :if_tag: htmlmode

   contributing-index.rst
   deterministic-modules.rst
   design/design-docs-index.rst
   changelog

