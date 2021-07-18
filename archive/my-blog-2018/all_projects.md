---
title: "项目集"
landing-title: "优雅"
layout: landing
description: '优雅不是与生俱来的觉悟<br>而是永不放弃的自我追求和永无止息的练习成就的'
image: images/project_title.jpg
nav-menu: true
---

<!-- Main -->
<div id="main">

<!-- One -->
<section id="one">
	<div class="inner">
		<header class="major">
			<h2>代码的优雅和项目的优雅</h2>
		</header>
		<p>
      项目代码的优雅非常重要，健壮性、易维护、可拓展等都算是代码的优雅
      <br>
      但是更重要的，是项目整体的优雅。一个项目，或者说是一个作品，应该体现设计者的思想。这种思想体现在项目的方方面面，上至整体功能和实现的构思，甚至是对项目存在的必要性的考虑，下至每一个交互细节、每一个色块...
      <br>
      该页面主要展示了Keyboard自以为不错的优雅的项目供大家批判。如果有兴趣可以选择浏览
    </p>
	</div>
</section>

<!-- Two -->
<section id="two" class="spotlights">
  {% for post in site.posts %}
  {% if post.layout == 'project' %}
    <section>
      <a href="{{ post.url  | relative_url }}" class="image">
        <img src="{{ site.assets_url }}/{{ post.image }}" alt="" data-position="center center" />
      </a>
      <div class="content">
        <div class="inner">
          <header class="major">
            <h3>{{ post.title }}</h3>
          </header>
          <p>{{ post.excerpt }}</p>
          <ul class="actions">
            <li><a href="{{ post.url  | relative_url }}" class="button">了解更多</a></li>
          </ul>
        </div>
      </div>
    </section>
  {% endif %}
  {% endfor %}
</section>

<!-- Three -->
<section id="three">
	<div class="inner">
		<header class="major">
			<h2>自我批判与欣赏</h2>
		</header>
		<p>
      代码总会非常多的，自我批判除可以提高自己知识水平也能在一定程度上自我安慰。
      <br>
      垃圾的代码肯定越写越多的，除了可以自我批判，也可以挑一些看起来没那么差的东西放到GitHub上给自己欣赏一下，让自己拥有小小的成就感。
      <br>
      当然，如果能恬不知耻地将自己GitHub账号推荐给大家，让大家看看垃圾代码，然后发表一些积极意见，更是能让自己有成就感。
    </p>
		<ul class="actions">
			<li><a href="https://github.com/Keyboard-l" class="button next">我的GitHub主页</a></li>
		</ul>
	</div>
</section>

</div>
