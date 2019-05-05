Blob 查看器
==============

There are many benefits to having a custom binary serialisation format (see :doc:`serialization` for details) but one
disadvantage is the inability to view the contents in a human-friendly manner. The Corda Blob Inspector tool alleviates
this issue by allowing the contents of a binary blob file (or URL end-point) to be output in either YAML or JSON. It
uses ``JacksonSupport`` to do this (see :doc:`json`).

对于自定义的二进制序列化格式（查看 :doc:`serialization` 更多详细信息）是有很多好处的，但是一个不好的地方是没办法用自然人的方式来查看里边的内容。Blob 查看器工具通过允许将一个二进制 blob 文件（或者一个 URL end-point）的内容输出为 YAML 或者 JSON 的方式解决了这个问题。它使用 ``JacksonSupport`` 实现了这个功能（查看 :doc:`json`）。

The tool can be downloaded from `here <https://corda.net/resources>`_.

该工具的最新版本可以从 `这里 <https://corda.net/resources>`_ 下载。

To run simply pass in the file or URL as the first parameter::

想要运行它，只需要简单地将文件或者 URL 作为第一个参数传给它：

    java -jar blob-inspector.jar <file or URL>

Use the ``--help`` flag for a full list of command line options.

使用 ``--help`` 标记查看关于这个命令的所有选项。

When inspecting your custom data structures, there's no need to include the jars containing the class definitions for them
in the classpath. The blob inspector (or rather the serialization framework) is able to synthesize any classes found in the
blob that aren't on the classpath.

当查看你自定义的数据结构的时候，不需要在 classpath 中包含这些类定义的 jars。Blob 查看器（或者更像是一个序列化框架） 能够将任何没有在 classpath 上的 blob 中的类分析出来。

支持的格式
~~~~~~~~~~~~~~~~~

The inspector can read **input data** in three formats: raw binary, hex encoded text and base64 encoded text. For instance
if you have retrieved your binary data and it looks like this::

这个查看器能够以三种格式阅读 **input data**，hex 编码文字和 base64 编码文字。比如你获取回来的二进制数据像下边这样::

    636f7264610100000080c562000000000001d0000030720000000300a3226e65742e636f7264613a38674f537471464b414a5055...

then you have hex encoded data. If it looks like this it's base64 encoded::

接着你有 hex 编码的数据。如果它看起来像这样的话，那应该是 base64 编码::

    Y29yZGEBAAAAgMViAAAAAAAB0AAAMHIAAAADAKMibmV0LmNvcmRhOjhnT1N0cUZLQUpQVWVvY2Z2M1NlU1E9PdAAACc1AAAAAgCjIm5l...

And if it looks like something vomited over your screen it's raw binary. You don't normally need to care about these
differences because the tool will try every format until it works.

通常你不需要关心这些不同的格式，因为这个工具会尝试每种格式，直到成功。

Something that's useful to know about Corda's format is that it always starts with the word "corda" in binary. Try
hex decoding 636f726461 using the `online hex decoder tool here <https://convertstring.com/EncodeDecode/HexDecode>`_
to see for yourself.

关于 Corda 的格式很有用的需要了解的东西是在二进制中，它总是会以单词 “corda” 来开头。试着使用 `在线 hex 解码工具 <https://convertstring.com/EncodeDecode/HexDecode>`_ 来解码 hex 636f726461。

**Output data** can be in either a slightly extended form of YaML or JSON. YaML (Yet another markup language) is a bit
easier to read for humans and is the default. JSON can of course be parsed by any JSON library in any language.

**输出的数据** 可以是一个稍微扩展自 YaML 或者 JSON 形式。YaML（Yet another markup language）更容易被人类阅读并且是默认的。JSON 当然也可以被任何的语言中的 JSON 类库来解析。

.. note:: One thing to note is that the binary blob may contain embedded ``SerializedBytes`` objects. Rather than printing these
   out as a Base64 string, the blob inspector will first materialise them into Java objects and then output those. You will
   see this when dealing with classes such as ``SignedData`` or other structures that attach a signature, such as the
   ``nodeInfo-*`` files or the ``network-parameters`` file in the node's directory.

.. note:: 一点需要注意的是二进制 blob 可能会包含内置的 ``SerializedBytes`` 对象。Blob 查看器不会将他们输出为 Base64 的字符串，而是将他们转换到 Java 对象中，然后再输出。当你在处理像 ``SignedData`` 或者其他的带有签名的结构，比如节点路径下的 ``nodeInfo-*`` 文件或者 ``network-parameters`` 文件的时候你会看到它。


例子
~~~~~~~

Here's what a node-info file from the node's data directory may look like:

下边是来自于节点的数据路径的一个 node-info 文件：

* YAML:

.. sourcecode:: none

    net.corda.nodeapi.internal.SignedNodeInfo
    ---
    raw:
      class: "net.corda.core.node.NodeInfo"
      deserialized:
        addresses:
        - "localhost:10005"
        legalIdentitiesAndCerts:
        - "O=BankOfCorda, L=London, C=GB"
        platformVersion: 4
        serial: 1527851068715
    signatures:
    - !!binary |-
      VFRy4frbgRDbCpK1Vo88PyUoj01vbRnMR3ROR2abTFk7yJ14901aeScX/CiEP+CDGiMRsdw01cXt\nhKSobAY7Dw==

* JSON:

.. sourcecode:: none

    net.corda.nodeapi.internal.SignedNodeInfo
    {
      "raw" : {
        "class" : "net.corda.core.node.NodeInfo",
        "deserialized" : {
          "addresses" : [ "localhost:10005" ],
          "legalIdentitiesAndCerts" : [ "O=BankOfCorda, L=London, C=GB" ],
          "platformVersion" : 4,
          "serial" : 1527851068715
        }
      },
      "signatures" : [ "VFRy4frbgRDbCpK1Vo88PyUoj01vbRnMR3ROR2abTFk7yJ14901aeScX/CiEP+CDGiMRsdw01cXthKSobAY7Dw==" ]
    }

Notice the file is actually a serialised ``SignedNodeInfo`` object, which has a ``raw`` property of type ``SerializedBytes<NodeInfo>``.
This property is materialised into a ``NodeInfo`` and is output under the ``deserialized`` field.

我们会注意到这个文件其实是一个被序列化的 ``SignedNodeInfo`` 对象，它含有一个类型为 ``SerializedBytes<NodeInfo>`` 的 ``raw`` 属性。这个属性被转换成了 ``NodeInfo`` 并且被输出到 ``deserialized`` 字段的下边。

命令行选项
~~~~~~~~~~~~~~~~~~~~

The blob inspector can be started with the following command-line options:

blob 查看器可以带有下边的命令行选项来被启动：

.. code-block:: shell

    blob-inspector [-hvV] [--full-parties] [--schema] [--format=type]
                   [--input-format=type] [--logging-level=<loggingLevel>] SOURCE
                   [COMMAND]

* ``--format=type``: Output format. Possible values: [YAML, JSON]. Default: YAML.
* ``--input-format=type``: Input format. If the file can't be decoded with the given value it's auto-detected, so you should
  never normally need to specify this. Possible values [BINARY, HEX, BASE64]. Default: BINARY.
* ``--full-parties``: Display the owningKey and certPath properties of Party and PartyAndReference objects respectively.
* ``--schema``: Print the blob's schema first.
* ``--verbose``, ``--log-to-console``, ``-v``: If set, prints logging to the console as well as to a file.
* ``--logging-level=<loggingLevel>``: Enable logging at this level and higher. Possible values: ERROR, WARN, INFO, DEBUG, TRACE. Default: INFO.
* ``--help``, ``-h``: Show this help message and exit.
* ``--version``, ``-V``: Print version information and exit.

子命令
^^^^^^^^^^^^

``install-shell-extensions``: Install ``blob-inspector`` alias and auto completion for bash and zsh. See :doc:`cli-application-shell-extensions` for more info.