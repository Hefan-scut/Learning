#### Commit Message - Angular规范

```json
<type>[optional scope]: <description>
// 空行
[optional body]
// 空行
[optional footer(s)]
```

- Header(必须)、Body(可选)、Footer(可选)
- scope必须用括号()括起来
- <type>[<scope>]后必须紧跟冒号，冒号后必须紧跟空格
- 两个空行也是必须的

```json
fix($compile): couple of unit tests for IE9
# Please enter the Commit Message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
# ...

Older IEs serialize html uppercased, but IE9 does not...
Would be better to expect case insensitive, unfortunately jasmine does
not allow to user regexps for throw expectations.

Closes #392
Breaks foo.bar api, foo.baz should be used instead
```

##### Header

###### Type

- Development: 这类修改一般是项目管理类的变更，不会影响最终用户和生产环境的代码，比如 CI 流程、构建方式等的修改。遇到这类修改，通常也意味着可以免测发布。
    - style: 代码格式变更，例如gofmt格式化代码，删除空行等
    - test：更新测试代码
    - ci: 持续集成和部署相关的改动，比如修改Jenkins、GitLab CI等
    - docs: 文档类更新
    - chore: 其他类型，比如构建流程、依赖管理或者辅助工具变动等
- Production: 这类修改会影响最终的用户和生产环境的代码。所以对于这种改动，我们一定要慎重，并在提交前做好充分的测试。
    - feat: 新增功能
    - fix: Bug修复
    - perf: 改进性能
    - refoctor: 其他代码类变更，这些变更不属于feat、fix、perf和style，例如简化代码、重命名变量、删除冗余代码等

###### Scope

- scope 是用来说明 commit 的影响范围的，它必须是名词。
- scope 不适合设置太具体的值

###### Subject

- commit的简短描述，必须以动词开头，使用现在时
- 动词的第一个字母必须是小写
- subject的结尾不能加英文句号

##### Body

- Header对commit做了高度概括
- 以动词开头，使用现在时
- 必须包括修改的动机，以及和上一版本相比的改动点

##### Footer

- 主要说明本次commit导致的后果
- footer通常用来说明不兼容的改动和关闭的issue列表

##### Revert Commit

Commit Message 还有一种特殊情况：如果当前 commit 还原了先前的 commit，则应以 revert: 开头，后跟还原的 commit 的 Header。而且，在 Body 中必须写成 This reverts commit ，其中 hash 是要还原的 commit 的 SHA 标识。

```json
revert: feat(iam-apiserver): add 'Host' option

This reverts commit 079360c7cfc830ea8a6e13f4c8b8114febc9b48a.
```

为了更好地遵循 Angular 规范，建议你在提交代码时养成不用 git commit -m，即不用 -m 选项的习惯，**而是直接用 git commit 或者 git commit -a 进入交互界面编辑 Commit Message**。这样可以更好地格式化 Commit Message。

##### 提交频率

- 一种情况是，只要我对项目进行了修改，一通过测试就立即 commit。比如修复完一个 bug、开发完一个小功能，或者开发完一个完整的功能，测试通过后就提交。
- 另一种情况是，我们规定一个时间，定期提交。这里我建议代码下班前固定提交一次，并且要确保本地未提交的代码，延期不超过 1 天。这样，如果本地代码丢失，可以尽可能减少丢失的代码量。
- 按照上面 2 种方式提交代码，你可能会觉得代码 commit 比较多，看起来比较随意。或者说，我们想等开发完一个完整的功能之后，放在一个 commit 中一起提交。这时候，我们可以在最后合并代码或者提交 Pull Request 前，执行 git rebase -i 合并之前的所有 commit。

##### 合并提交

合并提交，就是将多个 commit 合并为一个 commit 提交。这里，我建议你把新的 commit 合并到主干时，只保留 2~3 个 commit 记录。

我们通常会通过 git rebase -i 使用 git rebase 命令，-i 参数表示交互（interactive），该命令会进入到一个交互界面中，其实就是 Vim 编辑器。在该界面中，我们可以对里面的 commit 做一些操作。![img](https://static001.geekbang.org/resource/image/c6/ac/c63a8682c03862802e5eacf1641b86ac.png?wh=1345*866)

这个交互界面会首先列出给定之前（不包括，越下面越新）的所有 commit，每个 commit 前面有一个操作命令，默认是 pick。我们可以选择不同的 commit，并修改 commit 前面的命令，来对该 commit 执行不同的变更操作。git rebase支持的变更操作如下：![img](https://static001.geekbang.org/resource/image/5f/f2/5f5a79a5d2bde029d4de9d98026ef3f2.png?wh=629*393)

在上面的 7 个命令中，squash 和 fixup 可以用来合并 commit。例如用 squash 来合并，我们只需要把要合并的 commit 前面的动词，改成 squash（或者 s）即可。你可以看看下面的示例：

```json
pick 07c5abd Introduce OpenPGP and teach basic usage
s de9b1eb Fix PostChecker::Post#urls
s 3e7ee36 Hey kids, stop all the highlighting
pick fa20af3 git interactive rebase, squash, amend
```

rebase 后，第 2 行和第 3 行的 commit 都会合并到第 1 行的 commit。这个时候，我们提交的信息会同时包含这三个 commit 的提交信息：

```
# This is a combination of 3 commits.
# The first commit's message is:
Introduce OpenPGP and teach basic usage

# This is the 2ndCommit Message:
Fix PostChecker::Post#urls

# This is the 3rdCommit Message:
Hey kids, stop all the highlighting
```

如果我们将第 3 行的 squash 命令改成 fixup 命令：

```
pick 07c5abd Introduce OpenPGP and teach basic usage
s de9b1eb Fix PostChecker::Post#urls
f 3e7ee36 Hey kids, stop all the highlighting
pick fa20af3 git interactive rebase, squash, amend
```

rebase 后，还是会生成两个 commit，第 2 行和第 3 行的 commit，都合并到第 1 行的 commit。但是，新的提交信息里面，第 3 行 commit 的提交信息会被注释掉：

```
# This is a combination of 3 commits.
# The first commit's message is:
Introduce OpenPGP and teach basic usage

# This is the 2ndCommit Message:
Fix PostChecker::Post#urls

# This is the 3rdCommit Message:
# Hey kids, stop all the highlighting
```

除此之外，我们在使用 git rebase 进行操作的时候，还需要注意以下几点：

- 删除某个 commit 行，则该 commit 会丢失掉。
- 删除所有的 commit 行，则 rebase 会被终止掉。
- 可以对 commits 进行排序，git 会从上到下进行合并。

##### 完整的合并提交

假设我们需要研发一个新的模块：user，用来在平台里进行用户的注册、登录、注销等操作，当模块完成开发和测试后，需要合并到主干分支，具体步骤如下。

首先，我们新建一个分支。我们需要先基于 master 分支新建并切换到 feature 分支：

```
$ git checkout -b feature/user
Switched to a new branch 'feature/user'
```

这是我们的所有 commit 历史：

```
$ git log --oneline
7157e9e docs(docs): append test line 'update3' to README.md
5a26aa2 docs(docs): append test line 'update2' to README.md
55892fa docs(docs): append test line 'update1' to README.md
89651d4 docs(doc): add README.md
```

接着，我们在 feature/user分支进行功能的开发和测试，并遵循规范提交 commit，功能开发并测试完成后，Git 仓库的 commit 记录如下：

```
$ git log --oneline
4ee51d6 docs(user): update user/README.md
176ba5d docs(user): update user/README.md
5e829f8 docs(user): add README.md for user
f40929f feat(user): add delete user function
fc70a21 feat(user): add create user function
7157e9e docs(docs): append test line 'update3' to README.md
5a26aa2 docs(docs): append test line 'update2' to README.md
55892fa docs(docs): append test line 'update1' to README.md
89651d4 docs(doc): add README.md
```

可以看到我们提交了 5 个 commit。接下来，我们需要将 feature/user分支的改动合并到 master 分支，但是 5 个 commit 太多了，我们想将这些 commit 合并后再提交到 master 分支。

接着，我们合并所有 commit。在上一步中，我们知道 fc70a21是 feature/user分支的第一个 commit ID，其父 commit ID 是 7157e9e，我们需要将7157e9e之前的所有分支 进行合并，这时我们可以执行：

```
$ git rebase -i 7157e9e
```

执行命令后，我们会进入到一个交互界面，在该界面中，我们可以将需要合并的 4 个 commit，都执行 squash 操作，如下图所示：

![img](https://static001.geekbang.org/resource/image/6e/4e/6e41e61c27ca2a46e55e8801c47cd04e.png?wh=1097*320)

修改完成后执行:wq 保存，会跳转到一个新的交互页面，在该页面，我们可以编辑 Commit Message，编辑后的内容如下图所示：

![img](https://static001.geekbang.org/resource/image/73/87/73a884bac481236969ba2a219a2e9187.png?wh=1184*399)

这里有 2 个点需要我们注意：

- git rebase -i 这里的一定要是需要合并 commit 中最旧 commit 的父 commit ID。
- 我们希望将 feature/user 分支的 5 个 commit 合并到一个 commit，在 git rebase 时，需要保证其中最新的一个 commit 是 pick 状态，这样我们才可以将其他 4 个 commit 合并进去。

然后，我们用如下命令来检查 commits 是否成功合并。可以看到，我们成功将 5 个 commit 合并成为了一个 commit：d6b17e0。

```
$ git log --oneline
d6b17e0 feat(user): add user module with all function implements
7157e9e docs(docs): append test line 'update3' to README.md
5a26aa2 docs(docs): append test line 'update2' to README.md
55892fa docs(docs): append test line 'update1' to README.md
89651d4 docs(doc): add README.md
```

最后，我们就可以将 feature 分支 feature/user 的改动合并到主干分支，从而完成新功能的开发。

```
$ git checkout master
$ git merge feature/user
$ git log --oneline
d6b17e0 feat(user): add user module with all function implements
7157e9e docs(docs): append test line 'update3' to README.md
5a26aa2 docs(docs): append test line 'update2' to README.md
55892fa docs(docs): append test line 'update1' to README.md
89651d4 docs(doc): add README.md
```

#### 