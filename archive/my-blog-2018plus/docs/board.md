# 无聊的留言板

<span id="comments_switch" hidden>False</span>

你可能以为这就是一篇没有内容的文章，这种想法是有理由的，因为默认每篇文章都有一个留言板。

但是他们是不一样的，你会发现，这里不像文章下面的留言板那样默认只显示最新的五条留言。

其实在这里单独放一个留言板主要是为了让那些对Keyboard本人的或者对整个站点，而不是对某一篇文章的留言，能有一个比较好的归宿。

---

不管怎样，还是那样

如果你对Keyboard本人有什么看法，随便什么东西，甚至是“hhh”，都可以在下面留言。随便吐槽，反正我不会看的...

这是完全匿名的，当然你也可以留个名，甚至留个邮箱，因为如果你有什么有趣的留言我可能会好奇想认识一下你

<b>快写吧，说实话，我会看的，真的！</b>

<div id="vcomments"></div>

<script>
    new Valine({
        el: '#vcomments',
        appId: '9nBhQjarudnTltm9uQgqT04z-gzGzoHsz',
        appKey: 'jpCWh05kBHWtjvxg6pjbPiMP',
        placeholder: 'e.g. hhh',
        meta: ['nick', 'mail'],
        pageSize: 5,
        visitor: true
    })
</script>