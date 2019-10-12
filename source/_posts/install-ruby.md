---
title: 安装ruby
date: 2019-10-12 15:05:29
tags:
---

## ruby

不熟悉ruby，只是因为要搭建code.org的开发环境，所以要开始了解学习下了。 

### 安装ruby

最简单的方法应该是使用rbenv. 具体方法如下：

#### 安装rbenv

```bash
git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
```

#### 安装ruby-build

ruby-build 用来编译安装 Ruby 源码，如果选择手动编译，可不使用这个工具。

```bash
git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
```

#### 安装需要的版本

查看可用的 ruby版本

  >rbenv install --list

这里从列表中选择 2.5.0 进行安装：

  >rbenv install 2.5.0

等待一会儿，安装完毕后可以查看已经安装的所有 Ruby 版本：

  >rbenv versions

设置全局版本

  ```bash
  rbenv global 2.1.2
  rbenv rehash
  ```
  
### 问题

国内的环境你懂的，那会这么简单呢。实际上我在执行 *rbenv install 2.5.0* 就一直没反应了，看控制台输出知道是在下载[ruby](https://cache.ruby-lang.org/pub/ruby/2.5/ruby-2.5.0.tar.bz2),用wget看下，果然速度惊人（在广州节点的腾讯云上<10KB/s)。

### 国内镜像

现在看来，基本所有国外包管理都有相应的国内镜像了。例如NPM有淘宝的[CNPM](http://npm.taobao.org/),docker也有淘宝的，ruby的话也曾有过淘宝的，但貌似目前挂了。代替的有[ruby-china](https://gems.ruby-china.com/),替换方法如下：

```bash
git clone https://github.com/andorchen/rbenv-china-mirror.git ~/.rbenv/plugins/rbenv-china-mirror
```

gem 替换源

```bash
gem sources --remove https://rubygems.org/
gem sources -a https://gems.ruby-china.com
```
