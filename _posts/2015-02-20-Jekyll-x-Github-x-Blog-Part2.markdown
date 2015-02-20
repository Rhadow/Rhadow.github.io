---
layout     : post
title      : "Jekyll x Github x Blog (Part 2)"
subtitle   : "分享如何在 Jekyll 部落格內加入評論、標籤、分析等功能"
date       : 2015-02-20 14:11:37
author     : "Rhadow"
tags       : Jekyll Github
comments   : true
signature  : true
---

這篇是 [Jekyll x Github x Blog (Part1)](http://rhadow.github.io/2015/02/18/Jekyll-x-Github-x-Blog-Part1/) 的接續文章。在上一篇文章中，我大略的介紹了Jekyll 部落格的架構，這篇將主要探討如何在已建立的 Jekyll 部落格內加入一些簡易功能。

***注意：如果要使用文章內的程式碼內容，請將 `{.%` 與 `{.{` 中間的 `.` 去掉再用。***

## _config.yml 基本設定
`config.yml` 的檔案內容決定了 Jekyll 部落格內的所有設定，包括網址路徑，網站描述，markdown 的寫法等，以下是我的部落格的一些 `_config.yml` 內容：

```
# Dependencies
markdown            : redcarpet
redcarpet:
  extensions        : ["tables", "autolink", "strikethrough", "space_after_headers", "with_toc_data", "fenced_code_blocks"]
highlighter         : pygments

# Permalinks
permalink           : pretty
```

Jekyll 支援多種不同的 markdown 語法例如 `kramdown` 、`redcarpet` 等。這些語法都各有優缺點，也有各自的設定方法，這部分可到 [Jekyll 官網的設定頁面] (http://jekyllrb.com/docs/configuration/) 查詢。這個部落格使用的是 `redcarpet`，最主要的原因在於它支援所謂的 `Github Flavored Markdown`。讓我們可以使用與 Github 相同的 markdown 寫法來寫部落格內容，其中的 `fenced_code_block` 更是能讓分享程式碼的過程更為簡易。 `fenced_code_block` 能夠依據你所提供的程式語言並使用 highlighter 自動幫程式碼做色彩的調整，分隔的寫法也使 markdown 檔案內容更易讀。以下是一些使用 `fenced_code_block` 寫法的例子：

Ruby:

    ```ruby
    require 'redcarpet'
    markdown = Redcarpet.new("Hello World!")
    puts markdown.to_html
    ```

JavaScript:

    ```javascript
    function fancyAlert(arg) {
      if(arg) {
        $.facebox({div:'#foo'})
      }
    }
    ```

更詳細的說明可至 [Github Markdown 說明頁](https://guides.github.com/features/mastering-markdown/) 閱讀，順帶一提，這個部落格使用的 highlighter 是 [pygments](http://pygments.org/)。

`Permalink` 影響著部落格文章網址的顯示方式，Jekyll 提供三種選擇：

|Permalink  | URL 樣式|
------------ | -------------
|date | /:categories/:year/:month/:day/:title.html|
|pretty | /:categories/:year/:month/:day/:title/|
|none | /:categories/:title.html|

`Permalink` 這部分其實比較偏個人喜好去做設定的。
剩餘的一些設定算是相當直觀，比較需要注意的是，所有在 `_config.yml` 內部的屬性都能夠在其他使用 liquid template 的頁面以 `site.屬性名稱` 的方式來取得值。這部分可用來連結至你個人的社群網站如 Facebook 或 Github。

## 加入討論串功能
在文章中加入討論串功能其實相當容易，我們將會使用 [Disqus](https://disqus.com/) 做為討論串的平台。首先第一步是創建一個 Disqus 帳號並連結至我們的部落格，Disqus 會提供一段專屬的程式碼讓你加入到網站中。加入的方法也很簡單，在 `_layouts/default.html ` 的檔案中加入

```javascript
{.% include comments.html %}
```

接著新建一個 `comments.html` 檔案到 `_include` 資料夾中並將 Disqus 提供的程式碼貼上：

```html
{.% if page.comments %}
<div id="disqus_thread"></div>
    <script type="text/javascript">
    /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
    var disqus_shortname = 'rhadowtechnote'; // required: replace example with your forum shortname
    var disqus_identifier = '{.{ page.url }}';
    var disqus_url = 'http://rhadow.github.com{.{ page.url }}';

    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
        var dsq = document.createElement('script');
        dsq.type = 'text/javascript';
        dsq.async = true;
        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
    </script>
    <noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a>
    </noscript>
    <a href="http://disqus.com" class="dsq-brlink">comments powered by <span class="logo-disqus">Disqus</span></a>
{.% endif %}
```

我在上面的程式碼中加入了 `{.% if page.comments %}` ，這樣的設計可以讓使用者決定哪些文章可以開放討論，你只需要在文章的 header 設定中加入 "comments: True" 就能啟用討論串功能了。

## 加入標籤功能
實作標籤功能算是比較複雜的，網路上有許多使用其他外掛的方法來製作標籤功能，但在 Github Host 的網頁是無法執行這些外掛的。於是我在[網路](http://erjjones.github.io/blog/Part-two-how-I-built-my-blog/)上找到另一種方法，在這邊分享給大家。首先先新建兩個檔案：

* 在 `_plugins` 資料夾 (如果沒有請自己建) 中新建 `tag_gen.rb`
* 在 `_layouts` 資料夾中新建 `tag_index.html`

`tag_gen.rb` 的程式碼如下，它的主要功能是將部落格中的文章依照標籤做出獨立的標籤分頁。

```ruby
module Jekyll
 
  class TagIndex < Page    
    def initialize(site, base, dir, tag)
      @site = site
      @base = base
      @dir = dir
      @name = 'index.html'
 
      self.process(@name)
      self.read_yaml(File.join(base, '_layouts'), 'tag_index.html')
      self.data['tag'] = tag
      self.data['title'] = "Posts Tagged &ldquo;"+tag+"&rdquo;"
    end
  end
 
  class TagGenerator < Generator
    safe true
    
    def generate(site)
      if site.layouts.key? 'tag_index'
        dir = 'tags'
        site.tags.keys.each do |tag|
          write_tag_index(site, File.join(dir, tag), tag)
        end
      end
    end
  
    def write_tag_index(site, dir, tag)
      index = TagIndex.new(site, site.source, dir, tag)
      index.render(site.layouts, site.site_payload)
      index.write(site.dest)
      site.pages << index
    end
  end
 
end
```

`tag_index.html` 則是每個標籤分頁的骨架：

```html
<h2 class="tag-summary-title">{.{page.title}}</h2>
<table class="table table-striped tag-summary">
  <tbody>
	{.% for post in site.posts %}	
	{.% for tag in post.tags %}
	{.% if tag == page.tag %}
	<tr>
	    <td>
		    <h3>
		        <a href="{.{ post.url }}">
		            {.{ post.title }}
		            <small class="subtitle">
		                <span>{.{ post.subtitle }}</span>
		            </small>
		        </a>
		    </h3>
			<p>
				<span class="post-date">Posted by {.{ post.author }} on {.{ post.date | date: "%B %-d, %Y" }}</span>
				<small>
				    <span class="fa-stack fa-sm">
		                <i class="fa fa-tags fa-stack-1x"></i>
		            </span> : 
		            {.% for tag in post.tags %} <a class="tag-wrapper" href="/tags/{.{ tag }}" title="View posts tagged with &quot;{.{ tag }}&quot;"><i class="tags">{.{ tag }}</i></a>  {.% if forloop.last != true %} {.% endif %} {.% endfor %} 
		        </small>
		        <small class="comment-count">
			        <span class="fa-stack fa-sm">
			            <i class="fa fa-comments fa-stack-1x"></i>
			        </span>
			        <a href="http://rhadow.github.com{.{ post.url }}#disqus_thread" data-disqus-identifier="{.{ post.url }}"></a>
			    </small> 
	        </p>
	    </td>
	</tr>
	{.% endif %}
	{.% endfor %}
	{.% endfor %}			
  </tbody>
</table>
```

在使用 `jekyll build` 和 `jekyll serve` 後，`tag_gen.rb` 會在 `_site` 的資料夾內產生一個叫做 `tags` 的資料夾。在 `tags` 資料夾內會有多個依照標籤命名的資料夾，裡面的內容則是已經經過標籤分類後的文章總覽靜態頁面。這方法雖然不使用其他的外掛，但每次有新的標籤時必須得在本地先跑一次 'jekyll serve' 並把新產生的 `_site` 內的 `tags` 資料夾拉到根目錄取代原本的 `tags` 資料夾再 push，Github 才會抓到這些頁面。可以參考 [我的專案頁面](https://github.com/Rhadow/Rhadow.github.io) 來深入了解。

## 目錄頁與標籤雲
目錄頁的製作其實只是利用 liquid template 的語法來將所有文章依照你喜歡的格式來做輸出，我的範例如下：

```html
<ul class="archive">
  {.% for post in site.posts %}

    {.% unless post.next %}
      <h3>{.{ post.date | date: '%Y' }}</h3>
    {.% else %}
      {.% capture year %}{.{ post.date | date: '%Y' }}{.% endcapture %}
      {.% capture nyear %}{.{ post.next.date | date: '%Y' }}{.% endcapture %}
      {.% if year != nyear %}
        <h3>{.{ post.date | date: '%Y' }}</h3>
      {.% endif %}
    {.% endunless %}

    <li>    
        <div class="month">{.{ post.date | date:"%b" }}</div>
        <div class="archive-post-title"><a href="{.{ post.url }}">{.{ post.title }}</a></div>
    </li>
  {.% endfor %}
</ul>
```

標籤雲的部分也是相同道理：

```html
<ul class="tag-cloud">
{.% assign sorted_tags = (site.tags | sort: 0) %}
{.% for tag in sorted_tags %}
  <li style="font-size: {.{ tag | last | size | times: 100 | divided_by: site.tags.size | plus: 35  }}%">
    <a href="/tags/{.{ tag[0] }}">
      {.{ tag | first }} ({.{ tag | last | size }})
    </a>
  </li>
{.% endfor %}
</ul>
```

## Google 分析
最後一個要分享的就是加入 Google 分析功能。首先前往 [Google 分析](http://www.google.com/analytics/) 並註冊。在註冊完成並成功連結你的網站後，Google 會提供一段追蹤程式碼給你：

```html
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-59285222-1', 'auto');
  ga('send', 'pageview');

</script>
```

接下來只需要將這段程式碼加入你頁面的 `<head>` 就完成囉。有關 Google 分析的詳細教學請參考 [這裡](http://www.google.com.hk/analytics/learn/)

## 總結
其實部落格能夠有的功能非常多，這邊也只介紹了冰山一角而已。這篇除了分享給其他想自己做部落格的人之外也算是為自己曾做過的留下紀錄，以防未來忘記當初是如何製作的。完成以上分享的這些功能的方法有很多種，因此，如各位有其他更好的做法，也歡迎留言提供分享。