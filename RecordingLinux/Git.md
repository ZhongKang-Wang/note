# 20250605

Q1：什么是Git？

- Git是xxx用于管理Linux内核开发的==版本控制==软件。
- Git是分布式的。



## 创建新仓库

`mkdir my-project`

- windows命令行，在当前工作目录下创建文件夹

`git init`

- 使用当前目录作为git仓库，该命令执行完会在该文件夹下生成`.git`文件目录

example

`git init newrepo`

- 使用指定目录作为Git仓库，若没有则创建



## 克隆新仓库

`git clone /path/to/repository`

如果是远程服务器上的仓库，则应用下面这条命令

`git clone username@host:/path/to/repository`

example

`git clone https://github.com/username/repo.git E:\Temporary_jobs\GitProject\my-repo`

- 这条命令会把被克隆的仓库放在GitProject文件夹下，并将仓库更名为my-repo
- 被克隆的仓库默认会放在命令行的当前文件夹下



`git`维护三棵“树”。

- 工作目录：在工作目录下进行代码编辑、添加/删除文件
- 暂存区（Index）：缓存区域，保存改动
- HEAD：指向最后一次提交的结果（版本库）



## 添加和提交

`git add filename`

- 将修改过的文件添加到暂存区

`git add .`

- 提交所有文件到暂存区

`git status`

- 查看暂存区有哪些文件



`git commit -m "代码提交信息"`

- 将暂存区文件提交到HEAD，但是还没到远端仓库
- 附带代码提交信息
- *在 Linux 系统中，commit 信息使用单引号* **'**，Windows 系统，commit 信息使用双引号**"**

`git log`

- 查看提交历史

`git diff`

- 查看工作区和暂存区之间的差异

`git diff --cached`

- 查看暂存区和最后一次提交之间的差异



## 推送改动

`git push origin master`

- HEAD是本地仓库，执行上述这条命令以将这些改动提交到远端仓库
- master可以换成你想要推送的任何分支（暂时没理解这句话的意思）



## 分支

创建仓库时，master是默认分支。

我们可以在其他分支上进行开发，完成后再将其合并到主分支上。

`git checkout -b branch_1`

- 创建一个分支并切换过去

`git checkout master`

- 切换回主分支

`git branch -d branch_1`

- 删除新建分支

`git push origin branch`

- 将分支推送到远端仓库



## 更新与合并

`git pull`

- 实际上是两个命令的组合：
  - **`git fetch`**：从远程仓库获取最新的更改，但不会自动合并到本地分支。
  - **`git merge`**：将获取到的远程更改合并到本地当前分支。

example

`git pull origin main`

- `origin` 是默认的远程仓库名称
- `main` 是分支名称，通常是指远程仓库中的主分支
- 从远程仓库 `origin` 的 `new-feature` 分支获取最新的更改，并将其合并到当前分支。



## 替换本地改动

`git checkout -- filename`

- 使用暂存区文件替换掉工作目录中的指定文件
- 同时清除工作区中未添加到暂存区中的改动

`git rm --cached filename`

- 直接从暂存区删除指定文件，工作区不受影响







