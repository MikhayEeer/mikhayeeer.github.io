

<!--
  <<< Author notes: Finish >>>
  Review what we learned, ask for feedback, provide next steps.
-->

# 恭喜，您已开启了GitHub Pages！🎉
minimal主题 ： https://github.com/pages-themes/minimal/tree/master

github-pages用法： https://www.github-zh.com/getting-started/github-pages

我们将在我为您创建的一个分支my-pages中工作，以使该网站看起来很棒。 ❇️

Jekyll 使用名为 _config.yml 的文件来保存站点配置、如主题、网站名称、等设置。

我们需要使用博客主题，这里我们选择 "minima"。

## 动手练习：开始配置您的站点
切换到my-pages分支，点击_config.yml文件

右上角有个编辑按钮，打开文件编辑器

为了使用minima主题，我们在_config.yml文件里添加下面的内容

theme: minima
（可选）您还可以添加其他配置变量，例如站名title:, 作者author:、站点描述description:

提交修改

（可选）创建一个拉取请求以便查看您在本课程中将进行的所有更改。 单击Pull Requests tab，单击New pull request，设置base: main和compare:my-pages

等待大约20秒，然后刷新页面。GitHub Actions 将自动更新到下一步。

## 步骤3：自定义你的网站首页
干的漂亮，您已设置好了网站主题✨

想要自定义首页，您可以修改 index.md 或者 README.md 文件，向其添加内容。GitHub Pages 会首先查找是否有 index.md 文件。 我们的仓库里已经有一个 index.md 文件，因此我们可以修改它。

⌨️ 动手练习：创建您的首页
切换到my-pages分支，点击index.md文件
右上角有个编辑按钮，打开文件编辑器
在编辑框内输入你想要在首页上展示的内容，注意使用Markdown格式
（可选）您还可以修改 title: 或暂时不管，我们将在下一步中讨论
提交my-pages分支中的修改
等待大约20秒，然后刷新页面。GitHub Actions 将自动更新到下一步。
步骤4：发布一篇文章
您的首页看起来很棒！🤠

GitHub Pages 使用 Jekyll 工具来生成网站。在 Jekyll 中，博客需要遵守特定的文件命名格式和frontmatter。文件必须以 _posts/YYYY-MM-DD-title.md 方式命名， frontmatter中必须包括title和date。

什么是frontmatter？ Jekyll 文件使用的语法称为 YAML frontmatter，它位于文件开头，示例如下：
```
---
title: "Welcome to my blog"
date: 2019-01-20
---
```
更多关于front matter配置说明请参考 Jekyll frontmatter 文档

## ⌨️ 动手练习：创建一篇博客文章
切换到 my-pages 分支

点击 Add file 点击 Create new file 创建文件

文件取名为 _posts/YYYY-MM-DD-title.md

将 YYYY-MM-DD 替换为今天的日期，同时可以修改title

如果您确实要编辑标题，请确保单词之间有连字符。 如果您的博客文章日期不遵循正确的日期约定，您将收到错误消息，并且您的网站将无法构建。 有关详细信息，请参阅“排查 GitHub Pages 站点的 Jekyll 构建错误”。

将下面的内容放于文件顶部
```
---
title: "YOUR-TITLE"
date: YYYY-MM-DD
---
```
将 YOUR-TITLE 替换为你真实的文章标题

将 YYYY-MM-DD 替换为今天的日期，例如 2023-09-04

快速编写一段文章草稿，后面可以随时对其修改

提交修改

等待大约20秒，然后刷新页面。GitHub Actions 将自动更新到下一步。

## 步骤5：合并您的拉取请求
干的漂亮，朋友 ❤️！大家很快就能阅读您的博客！

您现在可以合并您的拉取请求！

动手
将my-pages分支的修改合并到main。如果您已经在步骤2中创建了Pull Request，那么现在只需打开那个PR，然后点击Merge pull request完成合并。 如果您之前没有创建，现在可以按照步骤2 中的说明进行操作。

(可选) 删除my-pages分支

等待大约20秒，然后刷新页面。GitHub Actions 将自动更新到下一步。

## Finish

_Congratulations friend, you've completed this course!_

<img src=https://octodex.github.com/images/constructocat2.jpg alt=celebrate width=300 align=right>

Your blog is now live and has been deployed!

Here's a recap of all the tasks you've accomplished in your repository:

- You enabled GitHub Pages.
- You selected a theme using the config file.
- You learned about proper directory format and file naming conventions in Jekyll.
- You created your first blog post with Jekyll!

### What's next?

- Keep working on your GitHub Pages site... we love seeing what you come up with!
- We'd love to hear what you thought of this course [in our discussion board](https://github.com/orgs/skills/discussions/categories/github-pages).
- [Take another GitHub Skills course](https://github.com/skills).
- [Read the GitHub Getting Started docs](https://docs.github.com/en/get-started).
- To find projects to contribute to, check out [GitHub Explore](https://github.com/explore).


