# Git

**SSH生成密钥**

```shell
ssh-keygen -t rsa
```

**初始化Git仓库**

```
git init
```

**把文件添加到仓库**

```
git add filename
```

**把文件提交到仓库**

```
git commit -m “本次提交到说明”
```

**查看仓库当前的状态，还有没有准备提交的修改**

```
git status
```

**对比文件，查看修改内通**

```
git diff filename
```

**提交修改和提交新文件是一样 的两步**

```
第一步 git add
第二步 git commit
```

**显示从近到最远大提交日志**

```
git log —pretty=oneline
```

**回退到上一个版本**

```
git reset —hard HEAD^
```

**回退到上上一个版本**

```
git reset —hard HEAD^^
```

**回退到往上**100个版本

```
git reset —hard HEAD~100
```

**回退到指定版本号**

```
git reset —hard commit id
```

**查看历史命令**，以便确定回到未来的哪个版本

```
git reflog
```

**场景一  修改了工作区某个文件的内容，直接丢弃工作区的修改**

```shell
git checkout — <file>
```

**场景二  当你不但乱改了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步**

```
第一步用命令，回到场景一
git reset HEAD <file>
第二步按场景一操作
```

**删除文件(小提示：先手动删除文件，然后使用git rm <file>和git add <file>)**

```shell
git rm <file>
```

**另外一种情况是删错了，因为版本库还有呢，所以可以轻松地把误删的文件恢复到最新版本**

```shell
git checkout — <file>
```

**关联一个远程库**

```shell
git remote add origin [git@github.com](mailto:git@github.com):cqzhongxc/learngit.git
```

**使用命令，第一次推送master分支的所有内容**

```shell
git push -u origin master
```

由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

**克隆仓库**

```shell
git clone <url>
```

