# Git常用命令

---

## 配置用户名称

```
git config --global user.name "your name"
```

## 配置电子邮件地址

```
git config --global user.email "your email"
```

## 创建Git版本库

```
git init
```

## 克隆远程仓库

命令格式

**`git clone <repo> <directory>`**

参数说明: 

- repo: Git仓库地址
- directory: 本地目录名称, 可省略

示例: 

```
git clone git@github.com:public-apis/public-apis.git public_apis
```

## 创建文件并提交到版本库

Git本地仓库有三大区域: 工作区、暂存区、版本区

创建、修改文件等操作是在工作区

`git add`命令是将工作区的内容添加到暂存区

git commit命令将临时存区内容添加到本地仓库中

### add命令

添加一个或多个文件到暂存区
**`git add file1 file2 ...`**
添加指定目录到暂存区, 包括子目录
**`git add dir`**
添加当前目录下的所有文件到暂存区
**`git add .`**

### commit命令

#### 提交暂存区到本地仓库中

**`git commit -m 'message'`**

参数说明: 
-m: 添加-m参数表示可以直接输入后面的message

> **如果commit注释写错了, 只想改一下注释, 只需要: `git commit --amend`, 即可进入编辑器更改注释**

#### 提交暂存区的指定文件到仓库区

**`git commit file1 file2 ... -m 'message'`**

### 查看当前仓库文件状态

**`git status`**

## 版本回退

git reset命令用于回退版本, 可以指定退回某一次提交的版本。git reset命令语法如下。

**`git reset [--soft | --mixed | --hard] [HEAD]`**

参数说明: 

- --mixed: 默认, 撤销commit, 并且撤销`git add .`操作, 用于重置暂存区的文件, 工作区文件内容保持不变

- --soft: 撤销commit, 但不撤销`git add .`操作, 工作区文件内容保持不变, 用于回退到某个版本

- --hard: 用于撤销工作区中所有未提交的修改内容, 将暂存区与工作区都回到上一次版本, 并删除之前的所有信息提交

- HEAD: 

    HEAD和HEAD~0表示当前版本

    HEAD^和HEAD~1表示上一个版本

    HEAD^^和HEAD~2表示上上一个版本

    HEAD^^^和HEAD~3表示上上上一个版本

    其他版本以此类推...

## 文件比较

git diff命令用于比较文件的不同, 即比较文件在暂存区和工作区的差异

git diff命令显示已写入暂存区和已经被修改但尚未写入暂存区文件的区别

### 查看尚未缓存的改动

**`git diff`**

### 查看已缓存的改动

**`git diff --cached`**

### 查看已缓存的与未缓存的所有改动

**`git diff HEAD`**

### 显示摘要, 而非整个diff

**`git diff --stat`**

### 查看某个文件的改动

**`git diff HEAD -- file`**

## Git的撤销修改

Git的撤销修改有两种方式

- 文件修改后还没有被放到暂存区, 撤销修改就能回到版本库未修改前状态
- 文件修改后添加到暂存区, 撤销暂存区的修改就回到添加到暂存区后的状态, 需要继续执行撤销文件操作, 才能完全撤销成功

### 未添加到缓存区

**`git checkout file`**

> PS: 可以通过此命令找回误删除的文件

### 已添加到缓存区

**`git reset HEAD file `**

**`git checkout file`**

## 分支管理

### 查看分支

**`git branch`**

### 创建|切换分支

**`git branch branch_name`**

> 分支存在表示切换, 否则表示创建

### 创建并切换分支

**`git checkout -b branch_name`**

### 删除分支

**`git branch -d test`**

> 若想要删除当前分支需要先切换到其他分支

### 合并分支

**`git merge branch_name`**

表示将名为branch_name的分支合并到当前分支

### 重命名分支

**`git branch -m old_name new_name`**

如果该分支已经被推送到远程仓库, 则在重命名本地分支后, 还需删除远程分支, 再上传本地重命名后的分支

删除: **`git push --delete origin old_name`**

推送: **`git push origin new_name`**

某些情况下还可能需要做一次关联操作: `git branch --set-upstream-to origin/new_name`

