Time-windows
============

.. topic:: 概要

   * *If a transaction includes a time-window, it can only be committed during that window*
   * *The notary is the timestamping authority, refusing to commit transactions outside of that window*
   * *Time-windows can have a start and end time, or be open at either end*

   * *如果一个 transaction 包含了一个 time-window，那么这个 transaction 只能在这个 time-window 里被提交*
   * *Notary 具有控制发生的时间的权利，当在 time-window 之外的时候，notary 可以拒绝提交 transaction*
   * *Time-window 可以有开始和结束时间，或者只有两者之中的一个*

.. only:: htmlmode

    Video
    -----
    .. raw:: html
    
        <iframe src="https://player.vimeo.com/video/213879314" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
        <p></p>


分布式系统中的时间
----------------------------
A notary also act as the *timestamping authority*, verifying that a transaction occurred during a specific time-window
before notarising it.

Notary 也可以作为 *时间戳的验证者*，在它确认一笔交易前，需要确保这笔交易是发生在指定的时间窗里。

For a time-window to be meaningful, its implications must be binding on the party requesting it. A party can obtain a
time-window signature in order to prove that some event happened *before*, *on*, or *after* a particular point in time.
However, if the party is not also compelled to commit to the associated transaction, it has a choice of whether or not
to reveal this fact until some point in the future. As a result, we need to ensure that the notary either has to also
sign the transaction within some time tolerance, or perform timestamping *and* notarisation at the same time. The
latter is the chosen behaviour for this model.

为了让一个时间窗有意义，它必须要在一方请求它的时候被绑定。一方可以获得一个 time-window 的签名，以此来证明有些事件是在特定时间点 *之前*、*当时* 或者 *之后* 发生的。然而，如果交易参与者不能够在指定的 time-window 内提交到相关的交易，它可以选择是否在未来的某个时间点将这个事实暴露出去。因此，我们需要确保 notary 或者能够在一些可容错的时间范围内对交易进行签名，或者同时进行打时间戳 *和* 对交易进行公证。后边的这种方式是这个模型中使用的方式。

There will never be exact clock synchronisation between the party creating the transaction and the notary.
This is not only due to issues of physics and network latency, but also because between inserting the command and
getting the notary to sign there may be many other steps (e.g. sending the transaction to other parties involved in the
trade, requesting human sign-off...). Thus the time at which the transaction is sent for notarisation may be quite
different to the time at which the transaction was created.

在创建交易的一方和 notary 之间是无法实现时间的同步的。这并不仅仅是因为物理或者网络的延迟，还会因为在插入命令和获得 notary 签名之间可能会发生很多其他的步骤（比如发送交易到涉及到的其他节点，请求人工的审批等）。所以交易被发送到 notary 的时间和交易创建的时间可能会不同。

Time-windows
------------
For this reason, times in transactions are specified as time *windows*, not absolute times. In a distributed system
there can never be "true time", only an approximation of it. Time windows can be open-ended (i.e. specify only one of
"before" and "after") or they can be fully bounded.

因为这个原因，交易中涉及到的时间会被制定为一个时间 *窗*，而不是一个绝对的时间。在一个分布式系统中是永远不会有 “真实的时间” 的，只有一个大概的时间。时间窗可以是开放的（比如在某个时间点后，或者某个时间点之前）或者是一个闭合的范围。

.. only:: htmlmode

   .. image:: resources/time-window.gif
      :scale: 25%
      :align: center


.. only:: pdfmode

   .. image:: resources/time-window.png
      :scale: 25%
      :align: center


In this way, we express the idea that the *true value* of the fact "the current time" is actually unknowable. Even when
both a before and an after time are included, the transaction could have occurred at any point within that time-window.

通过这种方式，我们表达了我们的想法，就是 “当前的时间” 永远都是未知的。甚至当在某个时间之前和之后都被包含的时候，交易也可能会在那个时间窗中的任何时间发生。

By creating a range that can be either closed or open at one end, we allow all of the following situations to be
modelled:

* A transaction occurring at some point after the given time (e.g. after a maturity event)
* A transaction occurring at any time before the given time (e.g. before a bankruptcy event)
* A transaction occurring at some point roughly around the given time (e.g. on a specific day)

通过在一端创建一个关闭或者开放的范围，我们允许用以下的方式生成时间窗模型：

* 一笔交易在指定时间之后的某个时间发生（比如在一个终止事件之后）
* 一笔交易在指定时间之前的任何时间发生（比如破产事件之前）
* 一笔交易在指定时间前后的某个时间发生（比如在指定的某一天）

If a time window needs to be converted to an absolute time (e.g. for display purposes), there is a utility method to
calculate the mid point.

如果一个时间窗需要被转换成一个绝对的时间（比如为了显示的原因），这里会有一个 utility 的方法来计算一个中间点时间。

.. note:: It is assumed that the time feed for a notary is GPS/NaviStar time as defined by the atomic
   clocks at the US Naval Observatory. This time feed is extremely accurate and available globally for free.

.. note:: 我们假设对于一个 notary 的 time feed 是由在 US Naval Observatory 的原子时钟所定义的 GPS/NaviStart 时间。这个 time feed 是完全准确的并且可以免费地在全球使用。