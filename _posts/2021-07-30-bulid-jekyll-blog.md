---
layout: post
title: 架設自己的Jekyll Blog
tags:
- blog
- Jekyll
---

### 為什麼要寫Blog？

一方面是紀錄自己的學習，另一方面在debug時，常常受益於網路上其他Blog的解法，又在`jyt0532`大大的blog看到
[十分鐘在AWS架好個人部落格](https://www.jyt0532.com/2017/12/11/launch-ec2-in-ten-minutes/)
終於下定決心來架自己的Blog。

### 為什麼選Jekyll

Jekyll是一個簡單的靜態網站生成器，不使用資料庫，可以用Markdown寫作，讓文章轉成靜態的HTML網頁，也可以放上Github Pages做免費的Host。
Jekyll的作者也是Github的共同創辦人Thomas Preston-Werner，使用Ruby也是我比較熟悉的語言。


### 安裝Jekyll

[Jekyll官網](https://jekyllrb.com/)

照指示安裝

[QuickStart](https://jekyllrb.com/docs/)

[Installation](https://jekyllrb.com/docs/installation/)

我的版本如下
```shell
$ ruby -v
2.7.3

$ gem -v
3.1.6

$ make -v
GNU Make 3.81

$ gem install jekyll bundler

$ jekyll -v

jekyll 4.2.0
```


## Jekyll with Hydeout

不想花太多時間在微調Jekyll設定，查到大家推薦Hyde這個Jekyll套件，照指示安裝完發現跑不起來，照官方指示也不會Work，看Github Issue發現[作者已經沒有在Maintain了](https://github.com/poole/hyde/issues/282)，也僅支援Jekll版本到2.X。

把Jekyll退回2.X，好像不太符合工程師的精神。

沮喪之於又看到了有人推薦一個同樣版面的套件[Hydeout](https://github.com/fongandrew/hydeout)，A refreshed version of Hyde for Jekyll 3.x and 4.x

好，就是他了。

先Fork Repository到自己的帳戶，因為要使用GitHub Page來當做Host，最簡單的方法就是把Repository命名改爲`YOUR_ACCOUNT.github.io`，`YOURACCOUNT`的部分記得改成自己GitHub帳號
現在訪問[https://pampas23.github.io/](https://pampas23.github.io/)，應該就會看到預設的首頁了。

要發新文章、更改設定等等的功能，就需要`git clone`到本地端，用`jekyll build` 和 `jekyll serve`跑起來，打開`localhost:4000`就看得到了


## Config
最主要的設定就在`_config.yml`，可以裝套件、改標題等等功能，詳細內容可以去[Configuration](https://jekyllrb.com/docs/configuration/)查看
```yaml
# Dependencies
  markdown:         kramdown
  highlighter:      rouge
# Setup
  title:            HaoYi's Notes
  tagline:          'Tech notes'
  description:      'Take a Tech notes'
  baseurl:          '/'
```


## 標籤功能
我們會希望文章可以照日期、或是分類來分，但因為Jekyll主體沒有這種功能，所以需要靠Plugin來補充，偏偏[jekyll-archives](https://github.com/jekyll/jekyll-archives)套件不支援Github Page

找到幾種方法，有網友用[Github Action](https://aneejian.com/automated-jekyll-archives-github-pages/)來處理，也有網友每多一個Category，[自己手動新增](https://bingdoal.github.io/blog/2020/10/category-archive-in-blog/)

覺得都有點麻煩，最後決定用`Rhadow大大`的[方法](https://rhadow.github.io/2015/02/20/Jekyll-x-Github-x-Blog-Part2/)

1. 在 `_plugins` 資料夾中新增 `tag_gen.rb`
2. 在 `_layouts` 資料夾中新建 `tag_index.html`

`tag_gen.rb`程式碼如下

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

`tag_index.html` 的code，基本上直接照搬Rhadow的範例，需要微調Style再去修改內容

```html
h2 class="tag-summary-title">{.{page.title}}</h2>
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
          <a href="https://rhadow.github.com{.{ post.url }}#disqus_thread" data-disqus-identifier="{.{ post.url }}"></a>
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

執行`jekyll build` 和 `jekyll serve`後，在`_site`會產生一個`tag`資料夾，裡面包含各分類的檔案，但因為`_site`資料夾在`.gitignore`中，，所以有新分類的時候，要把這裡的`tag`資料夾拖到根目錄取代原本的`tag`，做這個動作Github Page才會發現有變更。

這樣就完成標籤雲的新建了

## 結語

Jekyll還有很多功能可以調教，這篇只是紀錄自己建立的經過，提供一點更新的資訊，希望能幫助到其他人。


## Reference
[十分鐘在AWS架好個人部落格](https://www.jyt0532.com/2017/12/11/launch-ec2-in-ten-minutes/)

[在 github page 上實現 category archive](https://bingdoal.github.io/blog/2020/10/category-archive-in-blog/)

[Automated Jekyll Archives for GitHub Pages](https://aneejian.com/automated-jekyll-archives-github-pages/)

[Jekyll x Github x Blog (Part 1)](https://rhadow.github.io/2015/02/18/Jekyll-x-Github-x-Blog-Part1/)
