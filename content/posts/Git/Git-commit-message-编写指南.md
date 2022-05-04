---
title: "Git-commit message 编写指南"
tags:
  - git
  - commit
categories: ["Git"]
date: 2021-09-14 16:34:15
draft: false
toc: false
images:
math: true
---

commit message 不要乱写

<!--more-->

## 提交信息规范

我们对项目的 git 提交信息格式进行统一格式约定，这将提升项目日志的可读性。

共包含三个部分：

- Header（必须）
- Body（可省略）
- Footer（可省略）

```
<type>[scope]: <subject>
// 空一行
<body>
// 空一行
<footer>
```

> 其中，Header 是必需的，Body 和 Footer 可以省略(默认忽略)，一般我们在 `git commit` 提交时指定的 `-m` 参数，就相当于默认指定 Header。

> 不管是哪一个部分，任何一行都不得超过72个字符（或100个字符）。这是为了避免自动换行影响美观。

### Header

Header 部分只有一行，包括三个字段：type（必需）、scope（可选）和 subject（必需），可以用如下格式表示它的结构：

```
<type>[scope]: <subject>
```



#### type

用于表述此次提交信息的意义，首写字母大写，包括但不局限于如下类型：
- 功能相关
  - `Feat`：新功能（feature）
  - `Fix`：Bug 修复
  - `Test`：增加测试
- 文档相关
  - `Docs`：文档内容变化（documentation）
  - `Style`：格式（不影响代码运行的变动）
- 优化相关
  - `Perf`：性能优化
  - `Refactor`：重构（即不是新增功能，也不是修改 Bug 的代码变动）
- 版本相关
  - `Revert`：代码回滚
  - `Release`：版本发布
- 构建相关
  - `Chore`：构建过程或辅助工具的变动
  - `Build`：基础构建系统或依赖库的变化
  - `Ci`：CI 构建系统及其脚本变化
#### scope
scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视仓库不同而不同

#### subject
subject是 commit 目的的简短描述，用于简要描述修改变更的内容，不超过50个字符。如 `Update code highlighting in readme.md`。
- 句尾不要使用符号。
- 使用第一人称、现在时、祈使句语气。
- 以动词开头，比如change，而不是changed或changes
- 第一个字母小写

### Body

Body 部分是对本次 commit 的详细描述，可以分成多行。下面是一个范例。

>   More detailed explanatory text, if necessary. Wrap it to about 72  characters or so. Further paragraphs come after blank lines.- Bullet  points are okay, too- Use a hanging indent

有两个注意点。

- 使用第一人称现在时，比如使用change而不是changed或changes。
- 应该说明代码变动的动机，以及与以前行为的对比。

### Footer

Footer 部分只用于两种情况。

#### 1、不兼容变动

如果当前代码与上一个版本不兼容，则 Footer 部分以BREAKING CHANGE开头，后面是对变动的描述、以及变动理由和迁移方法。

```
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

#### 2、关闭 Issue

如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个 issue 。

> Closes #234

也可以一次关闭多个 issue 。

> Closes #123, #245, #992

### Revert

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以revert:开头，后面跟着被撤销 Commit 的 Header。

> revert: feat(pencil): add 'graphiteWidth' option
>
> This reverts commit 667ecc1654a317a13331b17617d973392f415f02.

Body部分的格式是固定的，必须写成This reverts commit .，其中的hash是被撤销 commit 的 SHA 标识符。

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的Reverts小标题下面。

## 标签规范

为了方便维护人员和用户能够快速找到他们想要查看的问题，我们使用“标签”功能对 Pull requests 和 Issues 进行分类。

如果您不确定某个标签的含义，或者不知道将哪些标签应用于 PR 或 issue，千万别错过这个。

Issue 的标签：

- 类型
  - `Bug`: 检测到需要进行确认的 Bug
  - `Feature Request`: 提出了新功能请求的 Issue
  - `Question`: 提出疑问的 Issue
  - `Meta`: 表明使用条款变更的 Issue
  - `Support`: 被标记为支持请求的 Issue
  - `Polls`: 发起投票的 Issue
- 结果
  - `Duplicate`: 重复提及的 Issue
  - `Irrelevant`: 与 NexT 主题无关的 Issue
  - `Expected Behavior`: 与预期行为相符的 Issue
  - `Need More Info`: 需要更多信息的 Issue
  - `Need Verify`: 需要开发人员或用户确认 Bug 或解决方法的 Issue
  - `Verified`: 已经被确认的 Issue
  - `Can't Reproduce`: 无法复现的 Issue
  - `Solved`: 已经解决的 Issue
  - `Stale`: 由于长期无人回应被封存的 Issue

Pull Request 的标签：

- `Breaking Change`: 产生重大变动的 Pull Request
- `Bug Fix`: 修复相关 Bug 的 Pull Request
- `New Feature`: 添加了新功能的 Pull Request
- `Feature`: 为现有功能提供选项或加成的 Pull Request
- `i18n`: 更新了翻译的 Pull Request
- `Work in Progress`: 仍在进行改动和完善的 Pull Request
- `Skip Release`: 无需在 Release Note 中展现的 Pull Request

两者兼有：

- `Roadmap`: 与 NexT 主题发展相关的 Issue 或者 Pull Request
- `Help Wanted`: 需要帮助的 Issue 或者 Pull Request
- `Discussion`: 需要进行讨论的 Issue 或者 Pull Request
- `Improvement`: 需要改进的 Issue 或者改进了 NexT 主题的 Pull Request
- `Performance`: 提出性能问题的 Issue 或者提高了 NexT 主题性能的 Pull Request
- `Hexo`: 与 Hexo 和 Hexo 插件相关的 Issue 或者 Pull Request
- `Template Engine`: 与模版引擎相关的 Issue 或者 Pull Request
- `CSS`: 与 NexT 主题 CSS 文件相关的 Issue 或者 Pull Request
- `Fonts`: 与 NexT 主题字体相关的 Issue 或者 Pull Request
- `PJAX`: 与 PJAX 相关的 Issue 或者 Pull Request
- `3rd Party Plugin`: 与第三方插件和服务相关的 Issue 或者 Pull Request
- `Docs`: 与文档说明相关的 Issue 或者 Pull Request
- `Configurations`: 与 NexT 主题设置相关的 Issue 或者 Pull Request