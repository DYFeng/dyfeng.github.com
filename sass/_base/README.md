# [eDoctor前端团队基础样式库](http://edoctor.github.com/f2e/ji-chu-yang-shi-ku.html)
---

你也许看出来了，这几个文件的名称跟[Alice](https://github.com/alipay/Alice/tree/master/sass)的Sass版很像。没错，我就是基于我们团队的基础样式库为Alice的`base.css`弄的Sass版。只是实际代码略有不同而已～

具体的代码实现，我们同时借鉴了Twitter的[Bootstrap](https://github.com/twitter/bootstrap)，可悲的是我们还得比他们多兼容一个该死的IE6～

我们的基础样式库，就是集大家之所长，感谢所有对我们有帮助的开源项目！

## 文件说明
    ---
     |--- _variables_.scss    全局变量
     |--- _mixins_.scss       自定义的Mixins，同时引入Compass的CSS3 Mixins
     |--- _reset_.scss        代码按需重置
     |--- _utilities_.scss    实用的Class们
     |--- _typo_.scss         源自@sofish的`typo.css`项目
     |--- ui/                 模块化后的UI组件
     |--- base.scss           @import上面的文件，同时等着被具体项目@import

## Compass的CSS3 Mixins

1. [Site](http://beta.compass-style.org/reference/compass/css3/)
2. [GitHub](https://github.com/chriseppstein/compass/tree/master/frameworks/compass/stylesheets/compass/css3)

为什么只使用Compass的CSS3部分？因为我们不想用其他部分。


## 安装Compass

先：

    group :assets do
      gem 'compass-rails'
    end

然后：

    $ bundle exec compass init

## 版权说明

©2011~世界末日 &nbsp; eDoctor

Licensed under the [MIT License](http://www.opensource.org/licenses/mit-license.php).