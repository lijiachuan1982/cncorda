.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

JSON
====

Corda provides a module that extends the popular Jackson serialisation engine. Jackson is often used to serialise
to and from JSON, but also supports other formats such as YaML and XML. Jackson is itself very modular and has
a variety of plugins that extend its functionality. You can learn more at the `Jackson home page <https://github.com/FasterXML/jackson>`_.

Corda 提供了一个扩展自流行的 Jackson 序列化引擎的模块。Jackson 通常被用来序列化为 JSON 或者从 JSON 的序列化，当时也支持其他格式，比如 YaML 和 XML。Jackson 自身非常模块化并且具有很多扩展功能的插件。你可以在 `Jackson 首页 <https://github.com/FasterXML/jackson>`_ 学习更多内容。

To gain support for JSON serialisation of common Corda data types, include a dependency on ``net.corda:jackson:XXX``
in your Gradle or Maven build file, where XXX is of course the Corda version you are targeting (0.9 for M9, for instance).
Then you can obtain a Jackson ``ObjectMapper`` instance configured for use using the ``JacksonSupport.createNonRpcMapper()``
method. There are variants of this method for obtaining Jackson's configured in other ways: if you have an RPC
connection to the node (see ":doc:`clientrpc`") then your JSON mapper can resolve identities found in objects.

为了能够得到常规 Corda 数据类型的 JSON 序列化的支持，包括在你的 Gradle 或者 Maven build 文件中的 ``net.corda:jackson:XXX`` 上的一个依赖，XXX 是你的目标的 Corda 版本（比如 0.9 代表 M9）。然后你可以使用 ``JacksonSupport.createNonRpcMapper()`` 方法来获取一个配置好的 Jackson ``ObjectMapper`` 实例来使用。这个方法有很多的参数通过其他方式来得到 Jackson 的配置：如果你跟节点有一个 RPC 链接（查看 :doc:`clientrpc`），那么你的 JSON mapper 能够处理在对象中找到的 identities。

The API is described in detail here:

这个 API 在下边有详细的描述：

* `Kotlin API docs <api/kotlin/corda/net.corda.client.jackson/-jackson-support/index.html>`_
* `JavaDoc <api/javadoc/net/corda/client/jackson/package-summary.html>`_

.. container:: codeset

   .. sourcecode:: kotlin

      import net.corda.jackson.JacksonSupport

      val mapper = JacksonSupport.createNonRpcMapper()
      val json = mapper.writeValueAsString(myCordaState)  // myCordaState can be any object.

   .. sourcecode:: java

      import net.corda.jackson.JacksonSupport

      ObjectMapper mapper = JacksonSupport.createNonRpcMapper()
      String json = mapper.writeValueAsString(myCordaState)  // myCordaState can be any object.


.. note:: The way mappers interact with identity and RPC is likely to change in a future release.

.. note:: mapper 和 identity 和 RPC 进行互动的方式可能在将来的 release 中被改变。
