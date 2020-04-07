title: hexo+github搭建自己的个人博客
date: 2016-02-24 14:54:05
tags: [hexo,github]
categories: 个人心得

---
　　在之前我一直都想找个地方记录自己慢慢前进路上的一些感想和心得，也锻炼一下自己的思考能力和写作能力，但是一直苦于没有找到合适的工具（其实实际上还是懒），后来工作了一段时间之后觉得自己虽然工作能力有提高，但是自己在学习上却落下了很多。之后我就希望能找一个能记录自己平时所学所想的地方，以此督促自己不断学习进步，于是在网上搜了很久，终于找到了hexo这个非常好用的工具，方便快捷，非常适合我这样的程序员。这篇文章主要记录的就是我在安装配置hexo和github中所学到的和遇到的一些问题，如果哪天有人看见了我的这篇文章也想用hexo搭建个人博客时，我希望我的这篇文章能给他一些帮助。这也是我写博客的尝试，也是我决心的开始，希望自己能一直坚持下去。
### 一，安装git和node.js
　　1.安装git,主要是因为[github pages](https://pages.github.com/),github提供了一个专门展示自己以及自己项目的平台，我们的博客代码和生成的静态页面最后就是托管在github上，我们需要安装git来push代码和静态页面到github。  
　　2.安装[node.js](https://nodejs.org/),hexo就是一个基于node.js的静态博客框架，所以安装node.js是必须的，主要是用于生成博客的静态页面。  
　　安装git和node.js都是只用到相应的网站下载软件然后一直下一步就可以了。申请github账号和搭建仓库稍后配置的时候一并说。  
<!-- more -->
### 二，安装hexo 
　　node.js安装好了之后，直接在命令行(windows下在cmd命令行或者git bash,建议以管理员身份打开cmd命令行)执行  
　　```npm install -g hexo ```  

　　等待命令执行完成之后，可以用  
　　```hexo -v ```  

　　查看hexo的版本以及一些其他的信息,我这里显示的版本信息是hexo-3.1.1  
　　之后cd到对应的你想初始化hexo程序的目录下面执行  
　　```hexo init ```  

　　hexo初始化完成之后，还需要安装hexo所依赖的一些其他组件，还需要执行  
　　```npm install ```  

　　等安装完成之后，就可以生成静态页面了，也可以在本地测试一下hexo，因为hexo初始化的时候就会生成一个默认的helloworld的页面，所以先生成静态页面  
　　```hexo generate ```  

　　然后启动服务  
　　```hexo server ```  

　　之后，当命令行出现  
　　```INFO Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop ```  

　　就表示服务器已经启动了,在浏览器中访问  
　　```http://localhost:4000/ ```  

　　就能出现hexo的helloworld页面了。  
　　- 这里记录了一些在安装hexo中我遇到过的问题：  
　　1.```hexo server ```  执行失败,显示缺少server命令。可能原因是hexo缺少server模块，如果没有执行```npm install ```  就有可能会出现这种情况。  
　　解决办法：执行```npm install ```  ,如果之后还是出现缺少server命令时候，也可以单独安装hexo-server模块：在目录下```npm install hexo-server --save ```  。
　　2.服务启动后在浏览器访问没有出现相应的页面。可能原因是4000端口被别的程序占用了,解决办法：自行搜索**端口占用**的解决办法,这里我就不赘述了。

### 三，配置hexo部署github  
　　本地node.js，和hexo配置好了之后，我们就可以开始登录github了  
　　没有账号的话就申请一个吧,附[github](https://github.com/),github在网上有一些教程,可以自己去阅读入门，[廖雪峰git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/)。登录github之后,就是创建仓库了，创建仓库时要填写仓库的名字，描述等一些信息，这里注意在填写仓库名字的时候，[github pages](https://pages.github.com/)明确规定了名字需要符合一定的格式要求(注意这里的名字要求主要是针对User Pages site的，如果想使用Project Pages site的话名字就没有要求，关于两种pages的不同在后面有介绍),就是名字必须是```username.github.io```,按照要求完成之后就得到了自己的代码仓库,这里要记下自己的仓库地址就行了。   
　　然后回到我们刚才的hexo初始化的目录下面，在当中有一个文件_config.yml，这个文件是hexo的配置文件，里面的一些配置的意义可以到[hexo官网](https://hexo.io/)找的到,这里还是附一个[配置文档的链接](https://hexo.io/docs/configuration.html),其中的一些配置都可以自定义,这里我说明几个比较常用的,  

参数  | 描述
----- | -----
title  | 网站标题
subtitle  | 网站副标题
description  | 网站描述
author  | 您的名字
language  | 网站使用的语言(简体中文是zh-Hans)
timezone  | 网站时区。Hexo 默认使用您电脑的时区。
url  | 您的网址
root  | 网站根目录
permalink  | 文章的链接格式(比如2016/02/25/文章标题)
permalink_default  | 链接中各部分的默认值
theme  | 当前主题名称。值为false时禁用主题
deploy  | 部署部分的设置  
　　其中重点说一下deploy参数，deploy参数表示当hexo生成静态页面后部署到github上的一些参数设置,格式是  
　　deploy:  
　　　　type: git  
　　　　repo: git@github.com:username/projectname.git  
　　　　branch: gh-pages  
　　repo后面写自己的项目地址,这里的branch我填写的是gh-pages主要是因为github pages提供了两种类型的pages模式，一种是User or Organization site,另外一个是Project site,其中这[两种pages](https://help.github.com/articles/user-organization-and-project-pages/)还是有一些不同的  

pages网站类型  | pages默认域名&github主机地址 | 发布分支
----  | ----
User Pages site  | username.github.io  | master
Organization Pages site  | orgname.github.io  | master
Project Pages site owned by a user account  | username.github.io/projectname  | gh-pages
Project Pages site owned by an organization  | orgname.github.io/projectname  | gh-pages  
　　这些不同主要就表现在访问域名和发布分支不同，另外在我看来还有一点就是user pages site只能创建一个，而project pages site的话就可以创建多个。从project pages site字面上理解的话，这不仅可以用作个人博客，也可以用作自己github项目的介绍网站，而User Pages site我自己想的话还是放别的网站比较好。当配置了hexo，github仓库也建好了之后，就可以在命令行下部署博客静态页面到github仓库上了。  
　　首先测试一下将helloworld部署到github pages访问,依次执行  
　　```hexo g```  
　　```hexo d```  
　　正常的话在github仓库上gh-pages分支中就能看见部署的博客静态页面，同时访问http://username.github.io/projectname 就能看见刚才在本地测试的helloworld页面,如果是User Pages site的话就直接访问http://username.github.io 就可以看到了。  
　　- 一些可能遇到的问题：  
　　1.```hexo d```Error。这主要是因为缺少hexo-deployer-git模块，执行  
　　```npm install hexo-deployer-git --save```一般就可以解决了。  
　　2.部署时缺少权限(Please make sure you have the correct access rights)。这个主要是因为没有配置deploy key也就是ssh key,github添加ssh key网上有很多的教程，我这里就不多少了,可以自己搜索  

### 四，hexo主题和自定义  
　　hexo官网上有很多主题可供选择,这里我也附一个链接直接链接到[hexo主题](https://hexo.io/themes/),这里的主题一般都有安装向导,按照上面的指导基本上就都能完成配置了,另外hexo目录下的配置文件_config.yml里有theme一项，就是用来填写对应的主题的。另外一般的主题上都会有相应的评论,统计，搜索一类的配置，这里我也不再详细说了。  
　　这些配置都完成了，也能部署博客文件到github上访问之后，我们终于可以开始写文章了，使用  
　　```hexo new new_article ```  
　　命令可以新建一篇文章,然后在/source/_post/文件夹下面会生成对应的.md文件,这里生成的文章文件名是可以在_config.yml的new_post_name项做更改的，在新建的文章中有Front-matter，Front-matter 是文件最上方以 --- 分隔的区域，用于指定个别文件的变量属性  

属性  | 描述  | 实例
---  | ---  | ---
title  | 文章标题  | article
date  | 文章创建时间  | 2016-02-25 17:26:25
layout  | 布局  | post
comments  | 是否创建评论  | true
tags  | 文章标签  | tags: [hexo,github]
categories  | 文章分类  | categories: hexo
permalink  | 覆盖文章网址  | http://username.github.io/2016-05-25/article (这里设置的值会替代article)  
　　-这里列举一些可能会遇到的问题:  
　　1.最常见的更改配置文件属性之后再```hexo g```结果会报错,或者没有反应，最大的可能就是配置文件中更改属性冒号后面忘记加空格。  
　　2.我当时遇到一个问题就是在更换了电脑之后如何继续之前的写作,因为hexo只是生成了静态的博客页面然后deploy到github上保存，这样保存的只是静态页面,而原始的hexo程序和md文章文件以及一些自定义的hexo主题和配置都保存在本地,解决这个问题办法也很简单，就是将hexo程序整个都部署到githb上,对于我来说,gh-pages分支用来存放博客的静态页面文件，而master分支就可以存放hexo的程序配置文件和主题文件,这样的话不论到哪里,都可以从github上clone代码下来继续以前的博客写作,完成之后再将代码保存到github,在hexo的.gitignore文件中会自动忽略一些比如node_modules等文件。  

### 五，配置个人域名  
　　等一切都配置完成之后,也能自己开始轻轻松松写博客之后，其实配置自己的域名这一步就完全看个人的要求了。其实配置域名指向github pages网上也有很多教程，一般的步骤就是购买域名之后将域名解析到对应的github pages地址，一般主要要注意的就是填写解析类型为CNAME，记录值写对应的访问github pages的域名，这里比如User Pages site是 http://username.github.io ，那么记录值就填写 username.github.io ，而对于Project Pages site例如 http://username.github.io/projectname ,那么记录值就是填写projectname了。域名解析完了还要在github上新增CNAME文件，里面就填写你自己的域名然后部署到github对应的仓库分支上保存就行了。  
　　-可能会出现的问题：  
　　1.创建CNAME文件保存到仓库后，```hexo d```之后CNAME文件又没了。这个问题解决办法就是将CNAME文件保存到source文件夹下面，这样在部署的时候就不会消失了，具体的原因就是hexo本身有一个存放除文章以外的资源的文件夹，这个文件夹可供存放一些图片、CSS、JS 文件等供部署静态页面后使用，CNAME文件也是存放在这个文件夹里面。在hexo的官方文档里有解释[hexo资源文件夹](https://hexo.io/docs/asset-folders.html)