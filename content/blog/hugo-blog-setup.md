+++

author = "vicever"
categories = ["hugo"]
date = "2019-08-14"
description = "用 HUGO 搭建一个 github.io 的站点 "
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
title = "站点搭建手册"
type = "post"

+++

---------
###  如何用 HUGO 搭建个人站点，并挂载到 github pages 上

 会用到的资源链接：   
 [HUGO](https://gohugo.io)   
 [Themes:Hugo Future Imperfect Slim](https://themes.gohugo.io/hugo-future-imperfect-slim/)  
 [github pags](https://pages.github.com/)


## Step 1. 安装 hugo
在 mac 上，安装 hugo 很方便，直接用 Homebrew 就可以了。没有的自行搜索操作步骤。  
```
$brew install hugo
```
验证下安装是否成功
```
$hugo version
Hugo Static Site Generator v0.56.3/extended darwin/amd64 BuildDate: unknown
```


## Step 2. 新建一个站点
来到自己的项目目录下，新建一个 hugo 的站点  
不需要自行新建目录，new site 的时候会新增一个目录  
```
$hugo new site myblog  
Congratulations! Your new Hugo site is created in /Users/Document/myblog.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/ or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

```
可以试试能不能跑起来
```
$hugo server

Building sites … WARN 2019/08/14 16:05:48 found no layout file for "HTML" for "home": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
WARN 2019/08/14 16:05:48 found no layout file for "HTML" for "taxonomyTerm": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
WARN 2019/08/14 16:05:48 found no layout file for "HTML" for "taxonomyTerm": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.

                   | EN
+------------------+----+
  Pages            |  3
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  0
  Processed images |  0
  Aliases          |  0
  Sitemaps         |  1
  Cleaned          |  0

Total in 7 ms
Watching for changes in /Users/zhangguangsheng/Documents/sourcecode/tmp/temp/{archetypes,content,data,layouts,static}
Watching for config changes in /Users/zhangguangsheng/Documents/sourcecode/tmp/temp/config.toml
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```
打开连接看看 [http://localhost:1313/](http://localhost:1313/)

hugo就这点不好，它的默认配置是啥也没有的，只有个简单的空页面，都需要自行配置的。看到个大白页面也不用着急，正常现象。

## Step 3. 加载一个主题
不能就这么大白板放着啊，加个主题吧！  
可以去它的主题站点去找一个[https://themes.gohugo.io/](https://themes.gohugo.io/)，还是有不少能看的。我看上了[Hugo Future Imperfect Slim](https://themes.gohugo.io/hugo-future-imperfect-slim/)
下载到本地先
```
$mkdir themes // Creates a Themes Folder
$cd themes		 // Points to the Themes Folder
$git clone https://github.com/pacollins/hugo-future-imperfect-slim.git
```
这个时候如果直接去 ```hugo server```一下，你会发现页面乱套了，淡定！！！   
现在需要把配置文件给配上，如果简单点的话，直接上 copy 大法，把 主题下examplesite 的内容都复制到站点目录里面。  
```
$cp -R themes/hugo-future-imperfect-slim/exampleSite/ ./
$hugo server
```
这下在看下 [http://localhost:1313/](http://localhost:1313/)， 干得漂亮。

然后就需要根据自己的需要，改 config.toml 的内容。以及 content 里面的文章内容（都删了，一会儿自己写）

## Step 4. 提交到 github.io上
需要在 github 上建立两个项目，一个用来放这个站点的源代码，包括各种配置、主题、资源等，就是 hugo 建的这个。还有一个项目是github pages 的项目，这个就是站点的静态资源信息。
如何建个普通项目就不说了，到处都是，找一个就行。特别注意建立的 github pages 的项目，项目名称必须是<用户名>.github.io，项目地址是：https://github.com/<用户名>/<用户名>.github.io，而后就能通过 http://<用户名>.github.io 的地址来访问。

#### 提交项目源码
举例：项目源代码的项目目录为 myblog，github用户名为 hugo
```shell
$cd myblog
$git init
$git add .
$git remote add origin https://github.com/hugo/myblog.git
$git commit -m"xxxxx"
$git push origin master
```
现在到[https://github.com/hugo/myblog](https://github.com/hugo/myblog) 看看代码提交情况。  

#### 提交 pages 站点代码  

  public 目录里面保存的都是最终站点需要展示的静态文件。之前一顿折腾总是留下些乱七八糟的东西，斩断重来吧！！  

```shell  
$rm -rf public
$git submodule add -b master https://github.com/hugo/hugo.github.io.git public
```  

然后就要提交内容了，鉴于每次提交一下流程都要操作一次，所以可以写成脚本。  
新建一个```deploy.sh```的文件，并把它设为可执行```$chmod +x deploy.sh```。然后我们编辑这个文件成下面这样。
```shell
#!/bin/sh

# If a command fails then the deploy stops
set -e

printf "\033[0;32mDeploying updates to GitHub...\033[0m\n"

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public

# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site $(date)"
if [ -n "$*" ]; then
	msg="$*"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

```
然后就可以执行```./deploy.sh "你的提交注释"```，一顿操作猛如虎，然后等一会可以上[https://hugo.github.io](https://hugo.github.io)看站点了。  

## Step 5. 绑定自己的域名
虽然 github.io 的站点和域名是免费的，但是不够个性，站点养不起，域名还是可以买一个的。怎么买就不在这里介绍了。
现在假如你已经有了自己的域名hugo.io，需要做两件事：

#### 1. 让这个域名指向 github.io   
具体操作可以看 github 的[官方说明](https://help.github.com/cn/articles/quick-start-setting-up-a-custom-domain)
#### 2. 让这个站点知道自己叫这个并响应  
在```public```目录中新建一个文本文件，取名为```CNAME```，这个文件的内容就一个，你的域名，比如```hugo.io```。完全按照这个格式来，别的都不用。
有了这个站点就知道原来自己就是```hugo.io```了。

## Step 6. 写一个文章并发布试试
为了保证格式的有效性，最好以现有的内容为模板进行修改，包括很多的配置要求，都沿用 examplesite 里面的内容。
```shell
$hugo new blog/my-first-blog.md
```
然后用文本编辑工具编辑这个 md 文件就行了，它是遵循的 markdown 的文件格式。  

完成编辑之后，运行发布脚本```./deploy.sh```

**ENJOY IT**
## 