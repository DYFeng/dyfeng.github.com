---
layout: post
title : "First Post on Github"
description : ""
category : Life 
tags : [github]
---
{% include JB/setup %}

终于出炉了！！这是我第一篇用github做blog的文章！！小生愚钝，在Win7下弄了4个小时才弄出来。遇到了cygwin和gem不兼容、gem源被墙、jekyll在Win7下各种奇怪的症状...得感叹下在Windows下生活的孩子不容易吖。

因为在Windows下操作，入乡随俗地下载了个Github的Windows客户端，那界面很炫，很有Win8 feel，难道这就是传说中的WinForm。可惜就是push的时候太慢了点，感觉要比命令行下要慢半拍。而且他在界面里不叫push，叫sync，这个名词更容易让人理解。

下面是各种华丽的Markdown测试


<pre class="prettyprint">
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
</pre>



<pre class="prettyprint">
##大标题
###小标题
####小小标题
#####小小小标题
######你还能再小点不
#######其实我能一直地小，只是css没有定义到了
</pre>

##大标题
###小标题
####小小标题
#####小小小标题
######你还能再小点不
#######其实我能一直地小，只是css没有定义到了


<pre class="prettyprint">
*强调一下*

**再强调一下**

***我次次强调我容易吗我***
</pre>


*强调一下*

**再强调一下**

***我次次强调我容易吗我***
