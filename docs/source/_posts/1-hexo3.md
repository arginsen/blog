---
title: hexo博客 | Hexo博客个别主题添加valine评论
date: 2020-4-7
categories: practice
tags:
 - hexo
photo: 
 - /blog/img/hexo.jpg
---

<br>

<!-- more -->

>针对ejs编写的主题，基本是在整体布局的article.ejs/post.ejs插入评论占位，再在comment.ejs里调用valine

# [hipaper](https://github.com/iTimeTraveler/hexo-theme-hipaper)

1. hipaper/_config.yml
```js
valine:
  enable: true 
  appid:   # your leancloud application appid
  appkey:  # your leancloud application appkey
  notify: false
  verify: false
  placeholder: 写下你的评论吧~
  avatar: retro 
  guest_info: nick,mail,link 
  pageSize: 10 
  language: 
  visitor: true 
  comment_count: true 
```

2. hipaper/layout/_partial/article.ejs
```js
      <% if (!index && post.comments && (theme.valine.enable || theme.duoshuo_shortname || theme.disqus_shortname || theme.uyan_uid || theme.wumii || theme.livere_shortname)){ %>
      <%- partial('comment') %>
      <% } %>
```

3. hipaper/layout/_partial/comment.ejs ，多说往后推一个，将第一个写成valine：
```js
<% if (theme.valine.appid && theme.valine.appkey) {%>
    <script src='//unpkg.com/valine/dist/Valine.min.js'></script>
    <div id="vcomments"></div>
    <script>
        var notify = '<%= theme.valine.notify %>' == true ? true : false;
        var verify = '<%= theme.valine.verify %>' == true ? true : false;
        window.onload = function() {
            new Valine({
                el: '#vcomments',
                notify: notify,
                verify: verify,
                app_id: "<%= theme.valine.appid %>",
                app_key: "<%= theme.valine.appkey %>",
                placeholder: "<%= theme.valine.placeholder %>",
                avatar:"<%= theme.valine.avatar %>"
            });
        }
    </script>
<% } else if (theme.duoshuo_shortname){ %>
...
```

# [Daily](https://github.com/GallenHu/hexo-theme-Daily)

1. Daily/_config.yml
```js
valine:
  enable: true 
  appid:   # your leancloud application appid
  appkey:  # your leancloud application appkey
  notify: false
  verify: false
  placeholder: 写下你的评论吧~
  avatar: retro 
  guest_info: nick,mail,link 
  pageSize: 10 
  language: 
  visitor: true 
  comment_count: true 

2. Daily/layout/post.ejs
```js
    <!-- Comments -->
    <div class="container">
        <% if (page.comments){ %>
            <%- partial('_partial/comment') %>
        <% } %> 
    </div>
```

3. Daily/layout/_partial/comment.ejs 
```js
<% if (theme.valine.appid && theme.valine.appkey) {%>
    <script src="//cdn1.lncld.net/static/js/3.0.4/av-min.js"></script>
    <script src='//unpkg.com/valine/dist/Valine.min.js'></script>
    <div id="vcomments"></div>
    <script>
        var notify = '<%= theme.valine.notify %>' == true ? true : false;
        var verify = '<%= theme.valine.verify %>' == true ? true : false;
        window.onload = function() {
            new Valine({
                el: '#vcomments',
                notify: notify,
                verify: verify,
                app_id: "<%= theme.valine.appid %>",
                app_key: "<%= theme.valine.appkey %>",
                placeholder: "<%= theme.valine.placeholder %>",
                avatar:"<%= theme.valine.avatar %>"
            });
        }
    </script>
<% } else if(config.disqus_shortname) { %>
...
```
<br>