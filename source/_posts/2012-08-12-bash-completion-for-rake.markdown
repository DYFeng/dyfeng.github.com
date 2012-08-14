---
layout: post
title: "为rake添加命令补全"
date: 2012-08-12 00:32
comments: true
categories: Linux
tags: [linux, bash, completion, rake]
---
为了用上Octopress，最近我没有少跟rake打交道。不过用起来始终有点不爽–命令补全没有了，没有make用得那么爽。虽然用`rake -T`可以列出所有的任务，但是始终没有一个Tab键补全方便。

幸亏觉得不方便的不止我一个人，Google了下，发现已经有人写出了基于bash-completion的[补全插件](http://turadg.aleahmad.net/2011/02/bash-completion-for-rake-tasks/)，还带缓存的。

<!-- more-->
把下面的文件保存到`completions.d`目录（在ubuntu上是/etc/bash_completion.d）

{% codeblock lang:bash %}
#!/bin/bash
# bash completion for rake
#
# some code from on Jonathan Palardy's http://technotales.wordpress.com/2009/09/18/rake-completion-cache/
# and http://pastie.org/217324 found http://ragonrails.com/post/38905212/rake-bash-completion-ftw
#
# For details and discussion
# http://turadg.aleahmad.net/2011/02/bash-completion-for-rake-tasks/
#
# INSTALL
#
# Place in your bash completions.d and/or source in your .bash_profile
# If on a Mac with Homebrew, try "brew install bash-completion"
#
# USAGE
#
# Type 'rake' and hit tab twice to get completions.
# To clear the cache, run rake_cache_clear() in your shell.
#

function _rake_cache_path() {
  # If in a Rails app, put the cache in the cache dir
  # so version control ignores it
  if [ -e 'tmp/cache' ]; then
prefix='tmp/cache/'
  fi
echo "${prefix}.rake_t_cache"
}

function rake_cache_store() {
  rake --tasks --silent > "$(_rake_cache_path)"
}

function rake_cache_clear() {
  rm -f .rake_t_cache
  rm -f tmp/cache/.rake_t_cache
}

export COMP_WORDBREAKS=${COMP_WORDBREAKS/\:/}

function _rakecomplete() {
  # error if no Rakefile
  if [ ! -e Rakefile ]; then
echo "missing Rakefile"
    return 1
  fi

  # build cache if missing
  if [ ! -e "$(_rake_cache_path)" ]; then
rake_cache_store
  fi

local tasks=`awk '{print $2}' "$(_rake_cache_path)"`
  COMPREPLY=($(compgen -W "${tasks}" -- ${COMP_WORDS[COMP_CWORD]}))
  return 0
}
complete -o default -o nospace -F _rakecomplete rake
{% endcodeblock %}

在当前shell载入补全

{% codeblock lang:bash %}
$ source /etc/bash_completion.d/rake
{% endcodeblock %}
