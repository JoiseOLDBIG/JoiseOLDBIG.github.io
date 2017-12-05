---
layout: post
title:  "用Jekyll + Github构建博客"
date:   2017-12-05 10:57:11 +0800
categories: post
---

我终于有自己的GitHub博客了，分享下我是怎么来这的吧。

运行环境：windows

以下便是你基本需要的东西：
# Ruby
windows环境下直接用[Rubyinstaller](https://rubyinstaller.org/downloads/)安装

*安装地址界面↓*

![png1]

*安装步骤↓*

![png2]

# RubyDevKit
在安装RubyDevKit时，仅仅需要注意的是要与你安装的Ruby版本一致。

![png3]

这是去往[RubyDevKit](https://rubyinstaller.org/downloads/)的链接，下载后直接解压到你想让它在的文件夹下，或者自己创建一个文件夹用以存放。

# 运行命令行
首先打开刚才安装好的Ruby命令行，由于我的是windows，所以我直接查看新增，直接就可以看到，并且打开，新的命令行名为‘Start Command Prompt with Ruby’
接着进入刚才解压的RubyDevKit文件夹下配置东西。

* 输入 ‘ruby dk.rb init’ 将会生成一个’config.yml’它的作用是是检测系统安装的ruby的位置并记录在这个文件中。

![png4]

* 在config.yml加上ruby的路径。

![png5]

* 接着 输入 ‘ruby dk.rb install’ 等待一会便将这些配置好了。

![png6]

* 安装’jekyll‘。。。由于现在的Ruby自带gem，所以不用安装gem，可以直接安装jekyll，命令如下 ‘gem install jekyll -v 3.1.6’，安装指定版本，因为3.1.6之后版本jekyll new出来的目录少了很多东西。

![png7]

* 安装bundler。

![png8]

* 新建博客，运行命令‘yekyll new blogName’。

![png9]

* 运行‘tree /f’，查看完整的目录。

![png10]

| 文件/目录           | 说明                                                                                                                                |
|:----------          | :---------                                                                                                                          |
| _config.yml         | Jekyll配置文件                                                                                                                      |
| _drafts             | 草稿目录                                                                                                                            |
| _includes           | 页头、页脚之类的放于这个目录，可以使用{ % include header.html % }这样的标签在别的地方引用                                             |
| _layouts            | 这里是存放页面模板的地方，可以在模板中使用{ { content } }来引用页面内容                                                               |
| _posts              | 这里就是存放动态的博客内容，标题必须为YEAR-MONTH-DAY-title.MARKUP格式                                                               |
| _data               | 存放yaml格式的数据文件，比如存了一个members.yml的文件，那么在别的地方可以使用site.data.members引用相关数据                          |
| _site               | 使用Jekyll编译后的静态站点将存放于这个目录下，这个目录不需要push到github，所以要在.gitignore文件中加入这个目录                      |
| index.html and other HTML,Markdown, Textile files | 首页内容                                                                                              |
| Other Files/Folders | 其它文件和目录会当做静态内容处理                                                                                                    |

* 进入新建文件夹内 启动测试 ‘jekyll server’。此时你在浏览器中输入http://localhost:4000便可初试自己的博客，关于你自己的东西你可以在文件夹下的‘_config.yml’中配置。

![png11]

![png12]

* 本地终于搭建好了

# GitHub

* GitBub新建仓库，命名规则是‘username.github.io’。

* 将本地代码上传到github上，然后在游览器输入‘username.github.io’，就ok啦。



[png1]: {{ site.baseurl }}/assets/images/20171204-1.png
[png2]: {{ site.baseurl }}/assets/images/20171204-2.png
[png3]: {{ site.baseurl }}/assets/images/20171204-3.png
[png4]: {{ site.baseurl }}/assets/images/20171204-4.png
[png5]: {{ site.baseurl }}/assets/images/20171204-5.png
[png6]: {{ site.baseurl }}/assets/images/20171204-6.png
[png7]: {{ site.baseurl }}/assets/images/20171204-7.png
[png8]: {{ site.baseurl }}/assets/images/20171204-8.png
[png9]: {{ site.baseurl }}/assets/images/20171204-9.png
[png10]: {{ site.baseurl }}/assets/images/20171204-10.png
[png11]: {{ site.baseurl }}/assets/images/20171204-11.png
[png12]: {{ site.baseurl }}/assets/images/20171204-12.png