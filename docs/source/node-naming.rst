.. _node-naming:

节点的命名
=============
A node's name must be a valid X.500 distinguished name. In order to be compatible with other implementations
(particularly TLS implementations), we constrain the allowed X.500 name attribute types to a subset of the minimum
supported set for X.509 certificates (specified in RFC 3280), plus the locality attribute:

一个节点的名字必须是一个有效的 X.500 可区分的名字。为了同其他的实现方法兼容（特别是 TLS 实现），我们约束了允许使用的 X.500 的属性类型为最小被支持的 X.509 证书集合的一个子集（在 RFC 3280 中指定的），外加下边的本地属性：

* Organization (O)
* State (ST)
* Locality (L)
* Country (C)
* Organizational-unit (OU)
* Common name (CN)

Note that the serial number is intentionally excluded from Corda certificates in order to minimise scope for uncertainty in
the distinguished name format. The distinguished name qualifier has been removed due to technical issues; consideration was
given to "Corda" as qualifier, however the qualifier needs to reflect the Corda compatibility zone, not the technology involved.
There may be many Corda namespaces, but only one R3 namespace on Corda. The ordering of attributes is important.

需要注意的是，serial number 在内部从 Corda 证书中排除了出去，为了将 distinguished name 格式中不确定性降到最小。distinguished name qualifier 被移除了，是因为技术的问题；将考量给了 “Corda” 让其作为 qualifier，然而这个 qualifier 需要反映 Corda 的 compatibility zone，而不是引入的技术。这里可能有很多 Corda  的命名空间，但是在 Corda 上只有一个 R3 命名空间。这些属性的顺序是非常重要的。

``State`` should be avoided unless required to differentiate from other ``localities`` with the same or similar names at the
country level. For example, London (GB) would not need a ``state``, but St Ives would (there are two, one in Cornwall, one
in Cambridgeshire). As legal entities in Corda are likely to be located in major cities, this attribute is not expected to be
present in the majority of names, but is an option for the cases which require it.

``State`` 应该避免使用除非是为了跟国家级别其他的相同的或者类似名字的 ``localities`` 加以区分。比如，伦敦（GB）就不需要一个 ``state``，但是 St Ives 需要（这里有两个，一个是在 Cornwall，一个是在 Cambridgeshire）。因为在 Corda 中 legal entities 可能都是在主要城市中，这个属性不被期望体现在主要的名字里，但是在需要的时候可以做为一个可选项。

The name must also obey the following constraints:

* The ``organisation``, ``locality`` and ``country`` attributes are present

    * The ``state``, ``organisational-unit`` and ``common name`` attributes are optional

* The fields of the name have the following maximum character lengths:

    * Common name: 64
    * Organisation: 128
    * Organisation unit: 64
    * Locality: 64
    * State: 64

* The ``country`` attribute is a valid `ISO 3166-1<https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>` two letter code in upper-case

* The ``organisation`` field of the name obeys the following constraints:
    * Has at least two letters
    * Does not include the following characters: ``,`` , ``"``, ``\``
    * Is in NFKC normalization form
    * Does not contain the null character
    * Only the latin, common and inherited unicode scripts are supported
    * No double-spacing

This is to avoid right-to-left issues, debugging issues when we can't pronounce names over the phone, and
character confusability attacks.

命名还必须要遵守下边的约束：

* ``Organisation``、``locality`` 和 ``country`` 属性必须要有

    * ``State``、``organisational-unit`` 和 ``common name`` 属性不是必须要有的

* 名字的字段有以下最大长度的限制：

    * Common name: 64
    * Organisation: 128
    * Organisation unit: 64
    * Locality: 64
    * State: 64

* ``Country`` 属性必须是一个有效的大写的 `ISO 3166-1<https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>` 的两个英文字母

* 名字里的 ``organisation`` 字段需要满足下边的约束:
    * 至少包含两个字符
    * 首部和尾部不能含有空格
    * 不能包含以下字符：``,`` , ``"``, ``\``
    * 是 NFKC 常规化形式
    * 不能包含 null 字符
    * 仅仅支持拉丁文、常见和继承的 unicode 脚本
    * 没有双空格（double-spacing）

这个是为了避免当我们不能够在电话中独处名字的发音的时候，由左向右的问题，debugging 问题，以及字符迷惑攻击（character confusability attacks）

.. note:: The network operator of a Corda Network may put additional constraints on node naming in place.

.. note:: 一个 Corda 网络的网络维护者可能会在节点名字上加入一些额外的约束。

外部的 identifiers
^^^^^^^^^^^^^^^^^^^^
Mappings to external identifiers such as Companies House nos., LEI, BIC, etc. should be stored in custom X.509
certificate extensions. These values may change for operational reasons, without the identity they're associated with
necessarily changing, and their inclusion in the distinguished name would cause significant logistical complications.
The OID and format for these extensions will be described in a further specification.

映射到外部的 identifiers，比如 Companies House nos., LEI, BIC, 等等。因为被存储在 X.509 证书扩展中。这些值可能会因为维护的原因被改动，而不需要改动他们相关的 identity，如果把它们加到了 distinguished name 的话，会造成非常大的逻辑上的复杂性。OID 记忆对这些扩展的格式会在之后的详细说明中描述。
