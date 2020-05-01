---
layout: post
title: 个人博客搭建-使用Hexo和Travis CI部署Github Pages
Author: tzcqupt
categories: [软件]
tags: [软件安装]
comments: true
toc: true
date: 2020-05-01 00:00:00
---

# 安装Hexo

## 安装前提

- Node.js(版本建议10.0及以上)

- Git

> Windows安装Node.js时，请确保勾选 **Add to PATH** 选项（默认已勾选）

## 安装Hexo

~~~bash
# 安装Hexo
npm install hexo-cli -g
# 初始化博客目录，确保文件夹blog为空
hexo init blog
# 进入该文件夹
cd blog
# 执行npm安装，hexo会去下载最小依赖
npm install
#.. 选择主题，写文章，生成静态文件
# 本地运行hexo
hexo server
~~~

## 选择博客主题

官网[hexo-theme-melody](https://molunerfinn.com/hexo-theme-melody-doc/zh-Hans/)，下方记录自己的步骤

### 安装主题

~~~bash
# 进入博客工作目录
cd blog
# 下载melody主题到工作目录里的themes/melody文件夹下
git clone -b master https://github.com/Molunerfinn/hexo-theme-melody themes/melody
~~~

### 配置

在blog根目录下，修改`_config.yml`文件

~~~yml
theme: melody # 将主题设置成melody
~~~

>如果你没有 pug 以及 stylus 的渲染器，请下载安装： `npm install hexo-renderer-pug hexo-renderer-stylus --save` or `yarn add hexo-renderer-pug hexo-renderer-stylus`

## 博客写作

### 标签页

1. 前往你的Hexo博客的根目录

2. 输入`hexo new page tags`

3. 你会找到`source/tags/index.md`这个文件

4. 修改这个文件：

```yaml
---
title: 标签
date: 2018-01-05 00:00:00
type: "tags"
---
```

### 分类页

1. 前往你的Hexo博客的根目录
2. 输入`hexo new page categories`
3. 你会找到`source/categories/index.md`这个文件
4. 修改这个文件：

```yaml
---
title: 分类
date: 2018-01-05 00:00:00
type: "categories"
---
```

### 文章写作例子

#### hexo-admin

[**hexo-admin**](https://github.com/jaredly/hexo-admin) 是一个Hexo博客引擎的管理用户界面插件。这个插件最初是作为本地编辑器设计的，在本地运行[hexo](https://hexo.io/zh-cn/)使用**hexo-admin**编写文章。

**插件安装**

```bash
 npm install --save hexo-admin
```

#### 其他Markdown写作

推荐 Typora

# 部署

## 本地部署

1. 在`_posts`文件里写好文件。
2. 确认在主题文件夹中执行了`npm install`。
3. 执行`hexo clean`清理旧文件。
4. 执行`hexo g`生成静态文件
5. 执行`hexo server`本地部署

## 自动部署到Github Pages

2. 在github上创建公开的`username.gihub.io`仓库。

3. 创建`blog-source`分支作为博客的源文件。

   > Github Pages只能选择master分支存放博客的静态页面，才能通过`username.github.io`访问。

4. 推送`blog`里的相关文件到该仓库的`blog-source`分支。

   > 检查`.gitignore`文件是否忽略了`public`文件夹。
   >
   > `hexo g`生成的静态页面会存放在这个文件夹里。

5. 将 [Travis CI](https://github.com/marketplace/travis-ci) 添加到你的 GitHub 账户中。

6. 前往 GitHub 的 [Applications settings](https://github.com/settings/installations)，配置 Travis CI 权限，使其能够访问你的 repository。

7. 进入重定向的Travis CI页面，或者访问https://travis-ci.com/。

8. 在浏览器新建一个标签页，前往 GitHub 新建[Personal Access Token](https://github.com/settings/tokens)，只勾选 `repo` 的权限并生成一个新的 Token。Token 生成后请复制并保存好。

9. 回到 Travis CI，前往你的 repository 的设置页面，在 **Environment Variables** 下新建一个环境变量，**Name** 为 `GH_TOKEN`，**Value** 为刚才你在 GitHub 生成的 Token。确保 **DISPLAY VALUE IN BUILD LOG** 保持 **不被勾选** 避免你的 Token 泄漏。点击 **Add** 保存。

10. 在hexo站点文件夹中(blog文件夹根目录)新建`.travis.yml`文件。

    1. `.travis.yml`

        ~~~yml
        language: node_js
        node_js:
          - 10 # use nodejs v10 LTS
        branches:
          only:
            - blog-source # 源文件分支
        # cache this directory
        cache:
          directories:
            - node_modules
        before_install:
          - npm install -g hexo-cli

        # Start: Build Lifecycle
        install:
          - npm install
          - npm install hexo-deployer-git --save

        # 执行清缓存，生成网页操作
        script:
          - hexo clean
          - hexo generate

        # 设置git提交名，邮箱；替换真实token到_config.yml文件，最后depoy部署
        after_script:
          - git config user.name "tzcqupt"
          - git config user.email "tzcqupt@163.com"
          # 替换同目录下的_config.yml文件中gh_token字符串为travis后台刚才配置的变量，注意此处sed命令用了双引号。单引号无效！
          - sed -i "s/gh_token/${GH_TOKEN}/g" ./_config.yml
          - hexo deploy
        # End: Build LifeCycle
        ~~~

    2. `_config.yml`
    
       ~~~yml
       # Hexo Configuration
       ## Docs: https://hexo.io/docs/configuration.html
       ## Source: https://github.com/hexojs/hexo/
       
       # Site
       title: 个人学习博客
       subtitle: '博客'
       description: '一个努力的Java程序员小白'
       keywords:
       author: tzcqupt
       language: zh-Hans
       timezone: ''
       
       # URL
       ## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
       url: https://github.com/tzcqupt/tzcqupt.github.io
       root: /
       permalink: :year/:month/:day/:title/
       permalink_defaults:
       pretty_urls:
         trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
         trailing_html: true # Set to false to remove trailing '.html' from permalinks
       
       # Directory
       source_dir: source
       public_dir: public
       tag_dir: tags
       archive_dir: archives
       category_dir: categories
       code_dir: downloads/code
       i18n_dir: :lang
       skip_render:
       
       # Writing
       new_post_name: :title.md # File name of new posts
       default_layout: post
       titlecase: false # Transform title into titlecase
       external_link:
         enable: true # Open external links in new tab
         field: site # Apply to the whole site
         exclude: ''
       filename_case: 0
       render_drafts: false
       post_asset_folder: false
       relative_link: false
       future: true
       highlight:
         enable: true
         line_number: true
         auto_detect: false
         tab_replace: ''
         wrap: true
         hljs: false
       
       # Home page setting
       # path: Root path for your blogs index page. (default = '')
       # per_page: Posts displayed per page. (0 = disable pagination)
       # order_by: Posts order. (Order by date descending by default)
       index_generator:
         path: ''
         per_page: 10
         order_by: -date
       
       # Category & Tag
       default_category: Java
       category_map:
       tag_map:
       
       # Metadata elements
       ## https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta
       meta_generator: true
       
       # Date / Time format
       ## Hexo uses Moment.js to parse and display date
       ## You can customize the date format as defined in
       ## http://momentjs.com/docs/#/displaying/format/
       date_format: YYYY-MM-DD
       time_format: HH:mm:ss
       ## Use post's date for updated date unless set in front-matter
       use_date_for_updated: false
       
       # Pagination
       ## Set per_page to 0 to disable pagination
       per_page: 10
       pagination_dir: page
       
       # Include / Exclude file(s)
       ## include:/exclude: options only apply to the 'source/' folder
       include:
       exclude:
       ignore:
       
       # Extensions
       ## Plugins: https://hexo.io/plugins/
       ## Themes: https://hexo.io/themes/
       theme: melody
       search:
         path: search.xml
         field: post
         content: true
       # Deployment
       ## Docs: https://hexo.io/docs/deployment.html
       deploy:
         type: git
         # 主要修改这里，配合travis文件加上gh_token，方便其识别并自动部署到master分支
         repo: https://gh_token@github.com/tzcqupt/tzcqupt.github.io
         branch: master
       ~~~
    
       

# 页面访问失败排查

## Hexo启动页面异常

### 错误信息

Hexo启动页面显示extends includes/layout.pug block content include includes/recent-posts.pug include。

### 解决办法

执行`npm install --save hexo-renderer-jade hexo-generator-feed hexo-generator-sitemap hexo-browsersync hexo-generator-archive`

## 其他错误

缺少相关插件，如配置了字数统计，需要安装wordcount插件

`npm i --save hexo-wordcount`

执行`npm ls --depth=0`查看缺少的npm依赖，并安装即可。

# 参考链接

- [hexo-theme-melody](https://molunerfinn.com/hexo-theme-melody-doc/zh-Hans/)

- [Hexo官方文档部署到GithubPages](https://hexo.io/zh-cn/docs/github-pages)

- [Hexo遇上Travis-CI：可能是最通俗易懂的自动发布博客图文教程](https://blog.csdn.net/Xiong_IT/article/details/78675874)