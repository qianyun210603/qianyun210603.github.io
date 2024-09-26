---
title: Github：解除Fork关系
date: 2024-09-26T16:27:50+08:00
toc: true
tags: 
categories: 
  - Programming
  - Git
---


　　开发者在利用GitHub进行项目开发时，常会遇到与Fork操作相关的一系列挑战，尤其是在项目独立发展后，这些挑战尤为突出。以下是几个关键问题及其有条理的重述：

##### 1. **项目独立发展后的管理难题**

- **深度定制与分化**：最初通过Fork一个仓库开始的项目，随着时间推移，可能经历了从功能扩展到开发语言的变更，导致该仓库与原始父仓库在各个方面都已显著分化，各自独立发展。

##### 2. **Pull Request的误操作风险**

- **默认目标分支问题**：由于项目是从Fork开始的，每次创建Pull Request（PR）时，GitHub默认将父仓库作为目标分支，这增加了不小心将更改提交到父仓库的风险，可能导致不必要的混乱或冲突。

##### 3. **社区与贡献度显示的局限性**

- **贡献者识别缺失**：在Fork的仓库中，即使有人做出了重要贡献，这些贡献在GitHub上可能无法被正确归因，因为系统主要基于原始仓库来追踪贡献者。
- **项目影响力不明**：此外，由于Fork的仓库与父仓库在GitHub上的关联，很难直观地展示该项目被哪些其他项目所依赖或使用，这限制了项目的曝光度和影响力评估。

##### 4. **与父仓库分离的需求**

- **缺乏直接解决方案**：鉴于上述问题，开发者可能希望将Fork的仓库与父仓库正式分离，以便更好地管理项目、展示贡献者及影响力。然而，GitHub当前并未提供直接的“Unfork”或“Detach”功能来实现这一需求。

## 解决方案

#### 删除并重新创建仓库

- 打开Git Bash。

- 创建一个 Fork 的裸克隆。
    ```bash
    git clone --bare https://github.com/EXAMPLE-USER/FORK-NAME.git
    ```
- 删除Fork的仓库。

    警告：删除 Fork 将永久删除所有相关的拉取请求（Pull Requests）和配置。此操作无法撤销。

- 在同一位置使用相同的名称创建一个新仓库。
- 将仓库镜像推送到相同的远程 URL。
    ```bash
        cd FORK-NAME.git
        git push --mirror https://github.com/EXAMPLE-USER/FORK-NAME.git
    ```
- 删除之前创建的临时本地克隆。
    ```bash
    cd ..
    rm -rf FORK-NAME.git
    ```

请注意，上述步骤中的`EXAMPLE-USER`和`FORK-NAME`需要替换为您实际的GitHub用户名和Fork仓库的名称。此外，`git push --mirror`命令会将所有引用（包括分支和标签）以及对象推送到远程仓库，这通常用于镜像或备份目的。在这个场景中，它被用来将本地裸克隆的内容推送到新创建的GitHub仓库中，从而“重建”Fork，但不再与原始仓库有 Fork 关系。

这种方法的缺点是会丢失原有的Issues，Wiki等。

#### 联系Github Support

可以直接点击下面链接：

https://support.github.com/contact?tags=rr-forks&subject=Detach%20Fork&flow=detach_fork

然后根据virtual-assistant的提示一步步进行，之后会自动生成Ticket，等待Github客服处理。一般一个工作日内就能收到解决完毕的邮件了。

## 参考资料
> - [Detaching a fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/detaching-a-fork)
> - [Delete fork dependency of a GitHub repository](https://stackoverflow.com/questions/16052477/delete-fork-dependency-of-a-github-repository)
