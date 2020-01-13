---
title: 规范你的Git Commit message
date: 2018-09-08 20:45:11
tags: [Git]
categories: [Git]
description: ""
---

#### 写在前面

关于 Git，大家想必都很熟悉，因为我们几乎每天都在重复着`git add`、`git commit`、`git push`等命令，自然也就留下很多“痕迹”，如果没有好的规范和工具来约束，可能就会出现以下情况：

![](https://github.com/gaoac/images-library/blob/master/blog/git_commit_error.png?raw=true)

因此，规范和工具的重要性就体现出来了：

> 关于 Git Commit message 的写法规社区有多种，本文采用的的 Angular 规范是目前使用最广的写法，比较合理和系统化，并且有配套的工具。<!-- more -->

#### 相关工具

- [commitizen](https://github.com/commitizen/cz-cli)
- [gitmoji](https://github.com/carloscuesta/gitmoji/)
- [cz-conventional-emoji](https://github.com/gaoac/cz-conventional-emoji) - 关于适配器，可以根据团队选择不同适配器，此处就安利该适配器了。

#### 安装

##### 全局

```
yarn global add cz-conventional-emoji
# OR
# npm install --global cz-conventional-emoji

# 设置全局默认适配器
echo '{ "path": "cz-conventional-emoji" }' > ~/.czrc
```

##### 本地

```
yarn add cz-conventional-emoji
# OR
# npm install --save-dev cz-conventional-emoji

# 为你的项目设置默认适配器
"config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-emoji"
    },
  }
```

#### 使用

以后，凡是用到 git commit 命令，一律改为使用 git cz。这时，就会出现选项，用来生成符合格式的 Commit message（commitizen 与 emoji 结合）。如图：

```
Select the type of change that you're committing: (Use arrow keys)
❯ ✨  Feat:              Introducing new features.
  🐛  Bug:               Fixing a bug.
  📝  Docs:              Writing docs.
  🎨  Style:             Improving structure / format of the code.
  💄  UI:                Updating the UI and style files.
  🚑  Quickfix:          Critical hotfix.
  ⚡️  Pref:               Improving performance.
(Move up and down to reveal more choices)
```

再看提交记录，是不是赏心悦目多了：

![](https://github.com/gaoac/images-library/blob/master/blog/git_commit_normal.png?raw=true)
