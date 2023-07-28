# hugo 博客搭建






# hugo 安装

## windows

### hugo

下载地址：`https://github.com/gohugoio/hugo/releases`

下载二进制文件（官方推荐下载扩展版），解压到一个目录，并配置 hugo 的环境变量。

![image-20230613191605156](./note%20picture/hugo%20%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA.assets/image-20230613191605156.png " ")

**注意：使用 `hugo` 一定要配置好 `git` 和 `go`  的环境**

> 配置环境变量

- 右键此电脑，打开属性

![image-20230613214806614](./note%20picture/hugo%20%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA.assets/image-20230613214806614.png " ")

- 在关于页面中往下拉，找到 **高级系统设置** ，并打开。

![image-20230613215315561](./note%20picture/hugo%20%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA.assets/image-20230613215315561.png " ")

- 打开环境变量

![image-20230613220819155](./note%20picture/hugo%20%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA.assets/image-20230613220819155.png " ")

- 双击打开

![image-20230613220855745](./note%20picture/hugo%20%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA.assets/image-20230613220855745.png " ")

- 点击新建，并把 `hugo` 解压的目录添加进去

![image-20230613220953657](./note%20picture/hugo%20%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA.assets/image-20230613220953657.png " ")

- 运行 `hugo version`，看是否配置成功

![image-20230613221146810](./note%20picture/hugo%20%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA.assets/image-20230613221146810.png " ")



### git

下载地址：`https://git-scm.com/download/win`

- 根据自己的操作系统下载对应的安装包

![image-20230613221341358](./note%20picture/hugo%20%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA.assets/image-20230613221341358.png " ")

安装程序包，无脑安装点下一步就行



### go

下载地址：`https://go.dev/dl/`

这个也是和 git 安装一样，无脑安装点下一步。





## linux

我是使用的 Arch Linux 系统，可以直接使用 `sudo pacman -S hugo` 来下载。使用命令下载安装的就是扩展版的。

其他 linux 操作系统，可以使用和 windows 的安装方法一样。

> 配置环境变量

```bash
# 编辑 /etc/profile 文件
export PATH=hugo文件解压路径:$PATH
# 加载 profile
source /etc/profile
```

![image-20230613192442234](./note%20picture/hugo%20%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA.assets/image-20230613192442234.png " ")

**注意： linux 下的 path 变量使用英文冒号**





# 创建一个博客

## 创建网站

```bash
# 创建一个博客目录
hugo new site blog
cd my_website
```



## 安装主题

```bash
git init
git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt
```



## 添加一个博客文章

```bash
hugo new posts/first_post.md
```

**注意：默认情况下，所有文章和页面均作为草稿创建。如果想要渲染这些页面，请从元数据中删除属性 `draft: true`, 设置属性 `draft: false` 或者在以下步骤中为 `hugo` 命令添加 `-D` 或 `--buildDrafts` 参数。**



## 启动

```bash
hugo server
```

**注意： 由于 ixIt 使用了 Hugo 中的 `.Scratch` 来实现一些特性， 非常建议你为 `hugo server` 命令添加 `--disableFastRender` 参数来实时预览你正在编辑的文章页面。**





# 博客内容管理

- **title**: 文章标题

- **subtitle**:文章副标题

- **date**: 这篇文章创建的日期时间它通常是从文章的前置参数中的 `date` 字段获取的，但是也可以在 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中设置

- **lastmod**: 上次修改内容的日期时间

- **draft**: 如果设为 `true`, 除非 `hugo` 命令使用了 `--buildDrafts`/`-D` 参数，这篇文章不会被渲染

- **author**:文章作者配置，和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.author` 部分相同

  ```yaml
  author:
    name: "" # 文章作者
    link: "" # 文章作者的链接
    email: "" # 文章作者的邮箱，用于设置 Gravatar 头像，优先于 `author.avatar`
    avatar: "" # 文章作者的头像
  ```

- **authorAvatar**: 是否启用文章作者头像

- **description**: 文章内容的描述

- **keywords**: 文章内容的关键词

- **license**: 这篇文章特殊的许可

- **images**: 页面图片，用于 Open Graph 和 Twitter Cards

- **tags**: 文章的标签

- **categories**: 文章所属的类别

- **featuredImage**: 文章的特色图片

- **featuredImagePreview**: 用在主页预览的文章特色图片

- **hiddenFromHomePage**: 如果设为 `true`, 这篇文章将不会显示在主页上

- **hiddenFromSearch**:  如果设为 `true`, 这篇文章将不会显示在搜索结果中

- **twemoji**: 如果设为 `true`, 这篇文章会使用 twemoji

- **lightgallery**:  和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.lightgallery` 部分相同

- **ruby**: 如果设为 `true`, 这篇文章会使用 [上标注释扩展语法](https://fixit.lruihao.cn/zh-cn/documentation/content-management/markdown-syntax/extended/#ruby)

- **fraction**:  如果设为 `true`, 这篇文章会使用 [分数扩展语法](https://fixit.lruihao.cn/zh-cn/documentation/content-management/markdown-syntax/extended/#fraction)

- **fontawesome**:  如果设为 `true`, 这篇文章会使用 [Font Awesome 扩展语法](https://fixit.lruihao.cn/zh-cn/documentation/content-management/markdown-syntax/extended/#fontawesome)

- **linkToMarkdown**: 如果设为 `true`, 内容的页脚将显示指向原始 Markdown 文件的链接

- **rssFullText**: 如果设为 `true`, 在 RSS 中将会显示全文内容

- **pageStyle**:  页面样式，详见 [页面宽度](https://fixit.lruihao.cn/zh-cn/documentation/advanced/#page-style)

- **toc**:  和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.toc` 部分相同

- **expirationReminder**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.expirationReminder` 部分相同

- **code**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.code` 部分相同

- **edit**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.edit` 部分相同

- **math**:  和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.math` 部分相同

- **mapbox**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.mapbox` 部分相同

- **share**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.share` 部分相同

- **comment**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.comment` 部分相同

- **library**:和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.library` 部分相同

- **seo**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.seo` 部分相同

- **type**: 页面渲染模板，详见 [页面模板](https://fixit.lruihao.cn/zh-cn/documentation/content-management/introduction/#templates)

- **menu**: 详见 [添加内容到菜单](https://fixit.lruihao.cn/zh-cn/documentation/basics/#content-to-menu)

- **password**: 加密页面内容的密码，详见 [内容加密](https://fixit.lruihao.cn/zh-cn/documentation/content-management/introduction/#content-encryption)

- **message**: 加密提示信息，详见 [内容加密](https://fixit.lruihao.cn/zh-cn/documentation/content-management/introduction/#content-encryption)

- **repost**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.repost` 部分相同

- **autoBookmark**:和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.autoBookmark` 部分相同

- **wordCount**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.wordCount` 部分相同

- **readingTime**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.readingTime` 部分相同

- **endFlag**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.endFlag` 部分相同

- **reward**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.reward` 部分相同

- **instantPage**: 和 [主题配置](https://fixit.lruihao.cn/zh-cn/documentation/basics/#theme-configuration) 中的 `params.page.instantPage` 部分相同


# github action 自动化部署

---

> 作者: pankx  
> URL: https://roukaixin.github.io/hugo-%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/  

