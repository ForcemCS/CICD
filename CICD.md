



## CI/CD流程

**1.Continuous Integration (Automatically build and test)**

Code Quality 

Performance testing

JUnit Tests

Container Scannining

Dependency Scanning

**2.Package**

Container Registry

NPM Registry

Maven repository

**3.Release**

GitLab Pages

Canary

Feature Flags

**4.Continuous Delivery(Click to deploy to production)**

**5.Continuous Deployment(Automatically deploy to production)**

GitLab Releases

Deploy Boards

Auto Deploy

## Git和版本控制

解决冲突

回滚和撤销对源代码的更改

异地源代码备份

### 基础命令

git配置有三个级别，分别对应三个文件

* --system：系统级，全系统范围有效，对应文件/etc/gitconfig
* --global：全局级，当前用户范围有效，对应文件为$HOME/.gitconfig
* --local：本地级，仅对当前一个项目有效，对应文件为项目目录下的.git/config
* init,clone,config：启动版本库
* add,mv,rm：暂存文件变动
* status, log, diff, grep,show：查看状态
* checkout, branch, merge, push, fetch,pull：分支管理

详细内容参见官网https://git-scm.com/doc

<img src=".\ima\基础命令.PNG" alt="基础命令" style="zoom:66%;" />

查看提交：

git会为每个commit保存一组信息，包括日志、时间戳、作者名称及email信息等

git log命令能够列出commit信息，而且使用--stat选项还可给出文件变动相关的统计数据

git log --oneline：每行一个commit

git log --author="USER.NAME" 

项目中文件状态的变化

+ 新增文件

  <img src=".\ima\文件状态对比1.PNG" alt="基础命令" style="zoom:66%;" />

+ 修改文件

  <img src=".\ima\文件状态对比2.PNG" alt="基础命令" style="zoom:66%;" />

对比文件的变动

git diff 
显示自前一次提交之后尚未暂存的文件变动
比较的是工作区和暂存区
git diff --staged | --cached [<path>...] 
显示暂存但尚未提交的文件变动
比较的是暂存区与前一次的提交
git diff <commit-ID> [<path>...] 
比较指定的提交与工作区
git diff --cached <commit-ID> [<path>...] 
比较暂存区与指定的提交

忽略项目中的某些文件

```shell
# .gitignore
# Java class files
*.class
# vim swap file
*.swp
# Executable files
*.exe
# Object and archive files
# Can use regular expression, e.g., [oa] matches either o or a
*.[oa]
# temp sub-directory (ended with a directory separator)
temp/
.gitignore文件自身应该纳入版本库
#  git add .gitignore && git commit -m "Add .gitignore"
```

### 撤销操作和远程库

<img src=".\ima\提交历史.PNG" alt="提交历史" style="zoom:50%;" />

没有直接指向最新一次的提交，而是指向当前分支名称（又指向了最新一次的提交）

git show HEAD命令可以查看HEAD当前所指向的commit的信息

撤销操作

+ 撤消文件变更

  

  + 撤消已经加入暂存区中的文件变更：git restore --stage <pathspec>

  + 撤消已跟踪但尚未加入暂存区中的文件变更：git restore <pathspec>

  + 从某个提交中恢复文件：git restore -s <source> <pathspec>

    + 例如，从前一次提交中恢复README.md文件：git restore -s HEAD~ READEME.md

      ```shell
      root@slb01:~/gitest# git log  --oneline
      af9b3ed (HEAD -> master) commit 1
      root@slb01:~/gitest# echo telemetry >1.yaml 
      root@slb01:~/gitest# cat 1.yaml 
      telemetry
      root@slb01:~/gitest# git diff  af9b3ed   1.yaml
      diff --git a/1.yaml b/1.yaml
      index ba7f31c..5d39060 100644
      --- a/1.yaml
      +++ b/1.yaml
      @@ -1 +1 @@
      -assignment 作业，任务
      +telemetry
      那么现在工作区的1.yaml可以回到只包含assignment字段
      root@slb01:~/gitest# git restore   -s  af9b3ed  1.yaml
      root@slb01:~/gitest# cat 1.yaml 
      assignment 作业，任务
      
      ```

+ 撤消文件变更

  +  本质上是将HEAD和master引用移到指定的commit上

  + 是否基于指定的commit连同重置暂存区和工作区则取决于使用的选项

    + --soft：仅移动HEAD和master指针的指向，不会重置暂存区或工作区,更改了本地仓库的版本

      此时本地仓回滚到commit的那一刻

    + --mixed： 表示本地仓和暂存区，都回滚到指定的commit。工作区代码不受影响
    + --hard：本地仓、暂存区、工作区，三区都回滚。（高危操作）

### 远程仓库

为本地仓库添加远程库

```shell
root@slb01:~/gitest# git remote add origin https://gitee.com/Agross/gitest.git
root@slb01:~/gitest# git remote -v  #默认推送和拉取的远程仓库是一致的
origin	https://gitee.com/Agross/gitest.git (fetch) 
origin	https://gitee.com/Agross/gitest.git (push)
root@slb01:~/gitest# git push  -u origin   "master" #若远程没有master会新建
可以在本地生成一个密钥对，公钥放在远程仓库上，以后避免认证
root@slb01:~/gitest# cd
root@slb01:~# ssh-keygen -t rsa  -C "test key" -P ''
root@slb01:~/gitest# ssh -T git@gitee.com
root@slb01:~# cd gitest/
root@slb01:~/gitest# ll
total 12
drwxr-xr-x  3 root root   46 Jun  1 17:00 ./
drwx------ 13 root root 4096 Jun  1 17:00 ../
drwxr-xr-x  8 root root  183 Jun  1 16:42 .git/
-rw-r--r--  1 root root   65 May 31 22:31 1.yaml
-rw-r--r--  1 root root   15 Jun  1 17:00 2.yaml
root@slb01:~/gitest# git add  .  
root@slb01:~/gitest# git commit -m "commit -3 "
[master d9b9950] commit -3
 2 files changed, 3 insertions(+), 1 deletion(-)
 create mode 100644 2.yaml
root@slb01:~/gitest# git log  --oneline 
d9b9950 (HEAD -> master) commit -3
5d9a56e (origin/master) commit 2   #远程仓库是指向的commit 2 本地是指向最新的commit -3 
af9b3ed commit 1
当zai
```

### 分支管理

当前分支标识：HEAD,会随着最新的提交而变动

tags

+ 为某个commit添加一个标签，从而对其完成特殊标记，例如标记特定的版本信息（不会自动移动）
  + 格式（主版本号.试出号.修订号）

+ Git支持使用两种标签：轻量标签（lightweight）与附注标签（annotated）

  + 轻量标签：仅为某个commit的引用

  + 附注标签：存储在 Git 数据库中的一个完整对象，其中包含打标签者的名字、电子邮件地址、日期时间， 此外还会

    包含一个message

便签管理

+ 创建标签

  + 轻量标签：git tag <tagName>  默认为HEAD打标签

  +  附注标签：git tag -a <tagName> -m <message>

    ```shell
    root@slb01:~/gitest# git tag -l  
    root@slb01:~/gitest# git log --oneline 
    6466af9 (HEAD -> master, origin/master, origin/HEAD) commit -4
    d9b9950 commit -3
    5d9a56e commit 2
    af9b3ed commit 1
    root@slb01:~/gitest# git tag -a v1.1 -m  "release v1.1"
    root@slb01:~/gitest# git tag -l  
    v1.1
    
    ```

  + 给指定的commit打标签：git tag -a <tagName> -m <message> <commitID>

    ```shell
    root@slb01:~/gitest# git tag -a v1.0 -m  "release v1.1" d9b9950 
    root@slb01:~/gitest# git tag -l  
    v1.0
    v1.1
    root@slb01:~/gitest# git show v1.0 
    tag v1.0
    Tagger: wukui <kuiinative@gmail.com>
    Date:   Thu Jun 1 20:40:19 2023 +0800
    
    release v1.1
    
    commit d9b99503c4e37706d8d897f633f5068242b17ffa (tag: v1.0)
    Author: wukui <kuiinative@gmail.com>
    Date:   Thu Jun 1 17:01:24 2023 +0800
    
        commit -3
    
    diff --git a/1.yaml b/1.yaml
    index bbc96dd..4048410 100644
    --- a/1.yaml
    +++ b/1.yaml
    @@ -1,2 +1,3 @@
     assignment 作业，任务
    -telemetry
    +telemetry  遥测技术
    +census 统计
    diff --git a/2.yaml b/2.yaml
    new file mode 100644
    index 0000000..9cde8e8
    --- /dev/null
    +++ b/2.yaml
    @@ -0,0 +1 @@
    +apiVersion: v1
    
    ```

    

+ 删除标签

  + git tag -d <tagName>

+ 显示标签

  + 列出：git tag -l   显示标签的详细信息：git show <tagName>

+  推送标签至远程仓库

  + 推送特定标签：git push origin <tagName>

  + 推送所有标签：git push --tags

    ```shell
    root@slb01:~/gitest# git  push  origin  tag v1.0 
    Enumerating objects: 1, done.
    Counting objects: 100% (1/1), done.
    Writing objects: 100% (1/1), 160 bytes | 160.00 KiB/s, done.
    Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
    remote: Powered by GITEE.COM [GNK-6.4]
    To gitee.com:Agross/gitest.git
     * [new tag]         v1.0 -> v1.0
    此时点击远程仓库的v1.0只有commit3时候的状态
    ```

+ 删除标签

  + git tag -d <tagName>

+ 删除远程仓库上的标签

  + git push --delete origin <tagName>

### Git的内部数据存储机制

对象数据库主要存储如下类型的对象

+ blob对象 可以理解为文件

+ tree 对象 可以理解为目录,但是下边只包含一级目录

+ commit对象 

  ```shell
  root@slb01:~/gitest# git log  --oneline 
  6466af9 (HEAD -> master, tag: v1.1, origin/master, origin/HEAD) commit -4
  d9b9950 (tag: v1.0) commit -3
  5d9a56e commit 2
  af9b3ed commit 1
  root@slb01:~/gitest# git log --pretty=raw 
  commit 6466af902a15a55234518f5c7969f659f53264bc
  tree df668af590db907e5062071726e3285f2e2f070f
  parent d9b99503c4e37706d8d897f633f5068242b17ffa
  author wukui <kuiinative@gmail.com> 1685613736 +0800
  committer wukui <kuiinative@gmail.com> 1685613736 +0800
  
      commit -4
  
  commit d9b99503c4e37706d8d897f633f5068242b17ffa
  tree ff53e5eed77800ccb2174dd8875ebdc6959f75eb
  parent 5d9a56e0c8149a3449f7e4580c4f709c19ba3c28
  author wukui <kuiinative@gmail.com> 1685610084 +0800
  committer wukui <kuiinative@gmail.com> 1685610084 +0800
  root@slb01:~/gitest# git show   ff53e5eed77800ccb2174dd8875ebdc6959f75eb 
  tree ff53e5eed77800ccb2174dd8875ebdc6959f75eb
  
  1.yaml
  2.yaml
  root@slb01:~/gitest# git ls-tree ff53e5eed77800ccb2174dd8875ebdc6959f75eb #tree内部的内容
  100644 blob 4048410a81e664b9980bbe0f06874febc9a95fc9	1.yaml
  100644 blob 9cde8e8dd9d6d5d927fef48b9acf681addb6ce6a	2.yaml
  
  ```

  git log --pretty=raw 查看每个提交历史，其中每个提交对于当时来说都有自己的tree对象

   那么我们就可以使用treeid来看整个tree的结构（git ls-tree  <treeid>）

  若tree下有blob，我们又可以查看blob的内容（git cat-file  blob  <blobid>）

+ tag对象

  ```shell
  root@slb01:~/gitest# git cat-file  tag v1.0  可以查看tag对应的commit
  object d9b99503c4e37706d8d897f633f5068242b17ffa
  type commit
  tag v1.0
  tagger wukui <kuiinative@gmail.com> 1685623219 +0800
  
  release v1.1
  
  ```

### Branch

Branch用于跟踪commit，它就像是指定特定提交链上最近一次commit的轻量级可移动的指针（从开始至最新的提交所

形成的这个链称为一个branch）

为了便于引用版本库上当前正在使用的分支，Git 还使用一个名为HEAD的特殊指针

```shell
root@slb01:~/gitest# git branch devel  在最新一次提交上创建的分支，master和devel分开的位置就是commit -4
root@slb01:~/gitest# git branch  -l  
  devel
* master
root@slb01:~/gitest# git log  --oneline 
6466af9 (HEAD -> master, tag: v1.1, origin/master, origin/HEAD, devel) commit -4
d9b9950 (tag: v1.0) commit -3
5d9a56e commit 2
af9b3ed commit 1

```

```shell
root@slb01:~/gitest# git status  
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
root@slb01:~/gitest# git checkout  devel 
Switched to branch 'devel'
root@slb01:~/gitest# git log  --pretty=oneline  
6466af902a15a55234518f5c7969f659f53264bc (HEAD -> devel, tag: v1.1, origin/master, origin/HEAD, master) commit -4
d9b99503c4e37706d8d897f633f5068242b17ffa (tag: v1.0) commit -3
5d9a56e0c8149a3449f7e4580c4f709c19ba3c28 commit 2

```

```shell
root@slb01:~/gitest# git branch -l  
* devel
  master
root@slb01:~/gitest# vim secret-demoapp.yaml 
root@slb01:~/gitest# git add  . 
root@slb01:~/gitest# git commit -m "commit 5.3"
[devel 60fcf64] commit 5.3
 1 file changed, 1 insertion(+)
 create mode 100644 secret-demoapp.yaml
root@slb01:~/gitest# git log --pretty=oneline
60fcf640fbca8879314c5e86062a435882724951 (HEAD -> devel) commit 5.3
6466af902a15a55234518f5c7969f659f53264bc (tag: v1.1, origin/master, origin/HEAD, master) commit -4
d9b99503c4e37706d8d897f633f5068242b17ffa (tag: v1.0) commit -3
5d9a56e0c8149a3449f7e4580c4f709c19ba3c28 commit 2
af9b3ed81161b617c9a7eec7d44a427b64b95f1e commit 1

```

现在改变master分支,只是修改了3.yaml文件文件，

```shell
root@slb01:~/gitest# git  checkout master 
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
root@slb01:~/gitest# ll
total 16
drwxr-xr-x  3 root root   60 Jun  2 07:11 ./
drwx------ 13 root root 4096 Jun  2 07:09 ../
drwxr-xr-x  8 root root  185 Jun  2 07:11 .git/
-rw-r--r--  1 root root   65 Jun  1 18:04 1.yaml
-rw-r--r--  1 root root   15 Jun  1 18:04 2.yaml
-rw-r--r--  1 root root   15 Jun  2 07:11 3.yaml
root@slb01:~/gitest# vim 3.yaml 
root@slb01:~/gitest# git add . 
root@slb01:~/gitest# git commit -m "commit 5.2"
[master b6b1af3] commit 5.2
 1 file changed, 1 insertion(+)
root@slb01:~/gitest# git log --pretty=oneline 
b6b1af3d40775a59fe27691d43490fd377a6110c (HEAD -> master) commit 5.2
6466af902a15a55234518f5c7969f659f53264bc (tag: v1.1, origin/master, origin/HEAD) commit -4
d9b99503c4e37706d8d897f633f5068242b17ffa (tag: v1.0) commit -3
5d9a56e0c8149a3449f7e4580c4f709c19ba3c28 commit 2
af9b3ed81161b617c9a7eec7d44a427b64b95f1e commit 1

```

#### 分支合并

要合并到哪个分支，必须先切换到这个分支

git merge <branchName>     （将指定的其它分支合并至当前分支）

分支合并存在两种情形

<img src=".\ima\分支合并.PNG" alt="分支合并" style="zoom:38%;" />

+ 快进式合并（Fast-Forward Merge）

  ```shell
  root@slb01:~/gitest# git branch    -l  
    devel
  * master
  root@slb01:~/gitest# git merge devel   相当于将commit-4 5.2 5.3 合并
  Merge made by the 'ort' strategy.
   secret-demoapp.yaml | 1 +
   1 file changed, 1 insertion(+)
   create mode 100644 secret-demoapp.yaml
  root@slb01:~/gitest# git log --pretty=oneline  
  8f61f26adc60c0f9fdf7e70ef8f9f532c9ffc751 (HEAD -> master) Merge branch 'devel'
  b6b1af3d40775a59fe27691d43490fd377a6110c commit 5.2
  60fcf640fbca8879314c5e86062a435882724951 (devel) commit 5.3
  6466af902a15a55234518f5c7969f659f53264bc (tag: v1.1, origin/master, origin/HEAD) commit -4
  d9b99503c4e37706d8d897f633f5068242b17ffa (tag: v1.0) commit -3
  5d9a56e0c8149a3449f7e4580c4f709c19ba3c28 commit 2
  af9b3ed81161b617c9a7eec7d44a427b64b95f1e commit 1
  root@slb01:~/gitest# ls  此时将devel分支创建的文件拿了过来
  1.yaml  2.yaml  3.yaml  secret-demoapp.yaml
  
  ```

+ 3路合并

  ```shell
  当进行三路合并的时候
  root@slb01:~/gitest# git merge devel -m  "merge branch devel, commit 5.8"
  Auto-merging secret-demoapp.yaml
  CONFLICT (content): Merge conflict in secret-demoapp.yaml
  Automatic merge failed; fix conflicts and then commit the result.
  root@slb01:~/gitest# git status  
  On branch master
  Your branch is ahead of 'origin/master' by 5 commits.
    (use "git push" to publish your local commits)
  
  You have unmerged paths.
    (fix conflicts and run "git commit")
    (use "git merge --abort" to abort the merge)
  
  Unmerged paths:
    (use "git add <file>..." to mark resolution)
  	both modified:   secret-demoapp.yaml
  
  no changes added to commit (use "git add" and/or "git commit -a")
  #当有了冲突以后，表明合并操作只是在工作区，并没有提交到暂存区甚至是本地仓库
  root@slb01:~/gitest# git diff secret-demoapp.yaml
  diff --cc secret-demoapp.yaml
  index 10a67bc,0ff40ef..0000000
  --- a/secret-demoapp.yaml
  +++ b/secret-demoapp.yaml
  @@@ -1,2 -1,1 +1,6 @@@
  ++<<<<<<< HEAD
   +apiVersion: v2
   +matadata:
  ++=======
  + apiVersion: v3
  ++>>>>>>> devel
  （接下来需要人为接入，然后再add  commit）
  ```

  在三方合并中存在一个问题：devel分支提交过很多次，当合并到master分支的时候，master分支多出来很多内容(这就需要标识详很多的详细信息)，当devel分支删除掉的时候，我们是没办法获取这些详细信息的

+ ![变基](.\ima\变基.PNG)

​        5 和6的父提交不再是2，而是变为了4，commit5 和commit6看到的基变化了，可以认为是产生了两个新的commit

​        这样就通过一个分支可以保留下来所有分支的变更历史

​        note:变基实际是利用已经有的成果，进行我后续的操作（如果有相同的部分，会丢失自己修改过的数据）（可能加一次快进合并）

### Git中的引用（reference）

引用（reference）是间接指向commit的方式

+ reference也可被认为是commit hash的别名
+ git版本库上的各reference以文本文件的形式存储于.git/refs/目录中下的子目录中
  + head目录：HEAD引用，内部的文件名为其指向的分支的名称，内容为相应分支最新一次提交的hash码
  + tags目录：标签引用，内部的文件名为创建的各Tag的名称
  + remotes/<remoteRepoName>目录： remoteRepoName为远程仓库的名称

+ 带有引用的commit，也可以直接使用“ref/X/Y”路径的形式引用，例如“ref/heads/main”

```shell
root@slb01:~/gitest/.git/refs# tree  -L 3 
.
├── heads
│   ├── devel
│   └── master
├── remotes
│   └── origin
│       └── HEAD
└── tags
    ├── v1.0
    └── v1.1

```

```shell
root@slb01:~/gitest/.git/refs# cat tags/v1.0 
a6c12c13a348e58f515c2302f14127bdf7eb8526
root@slb01:~/gitest/.git/refs# git cat-file tag  a6c12c13a348e58f515c2302f14127bdf7eb8526  tag自己的id
object d9b99503c4e37706d8d897f633f5068242b17ffa
type commit
tag v1.0
tagger wukui <kuiinative@gmail.com> 1685623219 +0800

release v1.1
root@slb01:~/gitest/.git/refs# git log  --pretty=oneline  
89752bc8d8acf78e283cfbdbea8cbecd6c4bc8c1 (HEAD -> master, devel) merge branch devel, commit 5.8
83f66401978a53eaa624bbdfff84b29604395197 commit 5.5
a7bb13ce67f55db0f86795aa68eb838514aae613 commit 5.6
858234d347f79bee5dfa29a71079174a71056564 commit  5.4
8f61f26adc60c0f9fdf7e70ef8f9f532c9ffc751 Merge branch 'devel'
b6b1af3d40775a59fe27691d43490fd377a6110c commit 5.2
60fcf640fbca8879314c5e86062a435882724951 commit 5.3
6466af902a15a55234518f5c7969f659f53264bc (tag: v1.1, origin/master, origin/HEAD) commit -4
d9b99503c4e37706d8d897f633f5068242b17ffa (tag: v1.0) commit -3
5d9a56e0c8149a3449f7e4580c4f709c19ba3c28 commit 2
af9b3ed81161b617c9a7eec7d44a427b64b95f1e commit 1

```

#### 引用规范（refspec）

refspec

+ 全称为Reference Specification，即引用规范
+ git通过这种格式来表示本地分支与远程分支的映射关系
+ 引用规范的格式“+<src>:<dst> ”，其中的+可选
  + <src> 是一个模式（pattern），代表远程版本库中的引用，即远程分支
  + <dst> 是本地跟踪的远程引用的位置，即本地远程分支
  + \+ 号告诉 Git 即使在不能快进的情况下也要（强制）更新引用

+ 本地远程分支就像是远程分支的一个本地镜像，在本地版本库与远程版本库之间起到一个桥梁的作用

```shell
本地的master映射到远程的master
root@slb01:~/gitest/.git/refs# git branch -l 
  devel
* master
root@slb01:~/gitest/.git/refs# git branch -r 
  origin/HEAD -> origin/master
  origin/master
root@slb01:~/gitest/.git/refs# git branch -vv   分支间的关联关系
  devel  89752bc merge branch devel, commit 5.8
* master 89752bc [origin/master: ahead 7] merge branch devel, commit 5.8
```

#### 远程仓库的使用

![远程仓库使用1](.\ima\远程仓库使用1.PNG)

![远程仓库使用2](.\ima\远程仓库使用2.PNG)

### Git项目团队协作的工作流模型

#### Git的分支模型

![Git分支模型1](.\ima\Git分支模型1.PNG)

![Git分支模型1](.\ima\Git分支模型2.PNG)

![Git分支模型1](.\ima\Git分支模型3.PNG)

![Git分支模型1](.\ima\Git分支模型4.PNG)

![Git分支模型1](.\ima\Git分支模型5.PNG)

git-flow模型

![Git-flow模型](.\ima\Git-flow模型1.PNG)



#### 工作流模型

基于团队内部共享的工作流模型

![基于团队内部共享的工作流模型](.\ima\基于团队内部共享的工作流模型.PNG)

基于Fork的工作流

![Git-flow模型2](.\ima\Git-flow模型2.PNG)

![基于fork的工作流](.\ima\基于fork的工作流.PNG)

总结：

```shell
Git工作流
    （1）基于共享库的协同
        授权研发成员，都能push代码至共享库
            每位研发人员，要克隆远程仓库至本地，拉出一个专用的feature分支做研发
            研发任务完成后，推送feature分支至远程库
            发起一个Pull Request、Merge Request，通知相关人员审核代码
            被接受后，才能被合并；一旦合并，feature分支即可被删除； 

        建小组

    （2）基于fork（派生库）的协同
        只授权研发成员，能从上游库pull代码
        每个研发人员基于自己的远程库进行克隆和推送
            每位研发人员，要克隆自己远程仓库至本地，拉出一个专用的feature分支做研发
            研发任务完成后，推送feature分支至自己远程库
            发起一个Pull Request、Merge Request，通知相关人员审核代码
                请求并入的对象：而是上游库的develop分支
            被接受后，才能被合并；一旦合并，feature分支即可被删除；
```

## Gitlab

GitLab:
    Git服务器
        多用户，每个用户都有自己名称空间
        Ruby on Rails 
        有易用，美观的Web界面
        全功能
        重量级

### Gitlab组件

<img src=".\ima\Gitlab组件.PNG" alt="Gitlab组件" style="zoom: 35%;" />

### Gitlab部署

Omnibus GitLab是基于Chef的应用编排工具，它基于Chef的cookbooks和recipes等组件自动化编排GitLab

的各组件，避免了用户复杂的配置过程

项目地址：https://gitlab.com/gitlab-org/omnibus-gitlab

https://packages.gitlab.com/gitlab  下载对应系统的deb包

还可以使用软件仓库镜像通过apt安装

 程序包安装：
        /etc/gitlab/ 配置文件目录
            /etc/gitlab/gitlab.rb （修改web url  和ssh协议的端口）

​            /etc/gitlab/initial_root_password
​        /var/opt/gitlab/  数据目录
​        /var/log/gitlab/  日志目录

        如果需要自设初始密码，配置参数
            gitlab_rails['initial_root_password']
    
        默认的管理员：
            账号：root
            初始密码保存文件：/etc/gitlab/initial_root_password


        下载GitLab
    
            https://packages.gitlab.com/app/gitlab/gitlab-ce/search
    
            https://packages.gitlab.com/gitlab/gitlab-ce


        中文文档
    
        https://docs.gitlab.cn/jh/


        关于GitLab的备份和恢复：
    
            https://docs.gitlab.com/ee/raketasks/backup_restore.html
            https://docs.gitlab.cn/ee/raketasks/backup_restore.html

现在以管理员root登陆，建立用户账号（用来演示Git的工作流模型）

+ 常用设置

  ![常用设置](.\ima\常用设置.PNG)

+ 创建tom jerry两个用户

  ![创建用户1](.\ima\admin管理.PNG)

  ![创建用户1](.\ima\创建用户1.PNG)

  ![创建用户2](.\ima\创建用户2.PNG)

  ![创建用户3](.\ima\创建用户3.PNG)

  以同样的方式创建jerry

+ 创建Groups

  ![创建组1](.\ima\创建组1.PNG)

  ![创建组2](.\ima\创建组2.PNG)

+ 接下来邀请成员

  ![组成员](.\ima\组成员.PNG)

  ![组成员2](.\ima\组成员2.PNG)

### 组中创建项目

以root,tom或jerry都可以创建，在一个组中的所有成员都可以pull push这个项目的

![组中创建项目](.\ima\组中创建项目.PNG)

![组中创建项目2](.\ima\组中创建项目2.PNG)

![组中创建项目3](.\ima\组中创建项目3.PNG)

可以通过如下的方式修改权限

![组中创建项目4](.\ima\组中创建项目4.PNG)

导入站点的项目

![组中创建项目5](.\ima\组中创建项目5.PNG)

接下来登陆tom或者jerry查看是否可以看到这个项目

![组中创建项目6](.\ima\组中创建项目6.PNG)

接下来我们以jerry用户开发代码，root用户进行管理（基于共享库的协同）

```shell
root@slb01:~# mkdir  jerry  
root@slb01:~# cd jerry/
root@slb01:~/jerry# git clone  http://gitlab.wukui.com/devops/spring-boot-helloWorld.git
Cloning into 'spring-boot-helloWorld'...
Username for 'http://gitlab.wukui.com': jerry
Password for 'http://jerry@gitlab.wukui.com': 
remote: Enumerating objects: 256, done.
remote: Counting objects: 100% (256/256), done.
remote: Compressing objects: 100% (135/135), done.
remote: Total 256 (delta 73), reused 256 (delta 73), pack-reused 0
Receiving objects: 100% (256/256), 35.06 KiB | 8.76 MiB/s, done.
Resolving deltas: 100% (73/73), done.
root@slb01:~/jerry/spring-boot-helloWorld# git config  --local  user.name  jerry
root@slb01:~/jerry/spring-boot-helloWorld# git config  --local  user.email  jerry@gmail.com
root@slb01:~/jerry/spring-boot-helloWorld# git remote  -v  
origin	http://gitlab.wukui.com/devops/spring-boot-helloWorld.git (fetch)
origin	http://gitlab.wukui.com/devops/spring-boot-helloWorld.git (push)
root@slb01:~/jerry/spring-boot-helloWorld# git branch    -l 
* main
要进行代码的开发，不应该基于现有的main分支
root@slb01:~/jerry/spring-boot-helloWorld# git checkout  -b feature/001   创建并剪出
Switched to a new branch 'feature/001'
root@slb01:~/jerry/spring-boot-helloWorld# git branch    -l 
* feature/001
  main
root@slb01:~/jerry/spring-boot-helloWorld# echo "new branch feature/001 by jerry" >README.md 
root@slb01:~/jerry/spring-boot-helloWorld# git add  .  
root@slb01:~/jerry/spring-boot-helloWorld# git commit -m "modify README.md by jerry" 
[feature/001 33bec20] modify README.md by jerry
 1 file changed, 1 insertion(+), 27 deletions(-)
 rewrite README.md (100%)
root@slb01:~/jerry/spring-boot-helloWorld# git push origin 
fatal: The current branch feature/001 has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin feature/001   远程分支不存在
root@slb01:~/jerry/spring-boot-helloWorld# git push --set-upstream origin feature/001 
```

此时在自己的账号下会出现一个推送消息

![组中创建项目7](.\ima\组中创建项目7.PNG)

可以更改想要合并到的分支

![组中创建项目8](.\ima\组中创建项目8.PNG)

指派给成员进行审核

![组中创建项目9](.\ima\组中创建项目9.PNG)

接下来回到Administrator账户

![组中创建项目11](.\ima\组中创建项目11.PNG)

![组中创建项目10](.\ima\组中创建项目10.PNG)

![组中创建项目12](.\ima\组中创建项目12.PNG)

接下来创建一个可以登陆的用户，但是不属于Deveops组，但是可以clone这个项目，也就是基于fork的工作流

然后在浏览器输入http://gitlab.wukui.com/devops（虽然不在Deveops组，但是可以clone这个项目，因为我们在定义组的时候，定义为内部的，只要可以登陆这个系统的用户，都可以clone这个项目）

![模拟fork工作流](.\ima\模拟fork工作流.PNG)

![模拟fork工作流2](.\ima\模拟fork工作流2.PNG)

![模拟fork工作流3](.\ima\模拟fork工作流3.PNG)

```shell
root@slb01:~# mkdir george  
root@slb01:~# cd george/
root@slb01:~/george# git clone  http://gitlab.wukui.com/george/spring-boot-helloWorld.git
Cloning into 'spring-boot-helloWorld'...
Username for 'http://gitlab.wukui.com': george
Password for 'http://george@gitlab.wukui.com': 
remote: Enumerating objects: 260, done.
remote: Counting objects: 100% (260/260), done.
remote: Compressing objects: 100% (138/138), done.
Receiving objects: 100% (260/260), 35.55 KiB | 5.08 MiB/s, done.
remote: Total 260 (delta 74), reused 255 (delta 73), pack-reused 0
Resolving deltas: 100% (74/74), done.
root@slb01:~/george# cd spring-boot-helloWorld/
root@slb01:~/george/spring-boot-helloWorld# git branch  -l  
* main
root@slb01:~/george/spring-boot-helloWorld# git checkout   -b  feature/001  
Switched to a new branch 'feature/001'
root@slb01:~/george/spring-boot-helloWorld# git config  --local  name.user george  
root@slb01:~/george/spring-boot-helloWorld# git config  --local  name.email  george@gmail.com  
root@slb01:~/george/spring-boot-helloWorld# echo "new branch feature/001 by george "  >> README.md 
root@slb01:~/george/spring-boot-helloWorld# git add  .  
root@slb01:~/george/spring-boot-helloWorld# git commit  -m "modify README.md by george"
root@slb01:~/george/spring-boot-helloWorld# git push --set-upstream origin feature/001
```

![模拟fork工作流4](.\ima\模拟fork工作流4.PNG)

同时对于george来说，需要添加一个上游库，用来拉取其他组织提交的新代码

```shell
root@slb01:~/george/spring-boot-helloWorld# git  remote  add upstream  http://gitlab.wukui.com/devops/spring-boot-helloWorld.git 
root@slb01:~/george/spring-boot-helloWorld# git remote   -v  
origin	http://gitlab.wukui.com/george/spring-boot-helloWorld.git (fetch)
origin	http://gitlab.wukui.com/george/spring-boot-helloWorld.git (push)
upstream	http://gitlab.wukui.com/devops/spring-boot-helloWorld.git (fetch)
upstream	http://gitlab.wukui.com/devops/spring-boot-helloWorld.git (push)
pull的时候从upstream push到origin 
```

接下来创建合并请求，合到其他组织的devops项目下

![模拟fork工作流5](.\ima\模拟fork工作流5.PNG)

![模拟fork工作流6](.\ima\模拟fork工作流6.PNG)

回到管理员用户

note:合并的时候特别注意分支的删除问题

对于jerry用户来说，版本落后，可以使用 git pull origin/main 来更新

        仓库上的操作会生成event：
            类型：
                push 
                PR/MR  
                tag_push 

## Jenkins

Jenkins：
    Web Server:
        http协议接入客户端请求
            调用API
            使用Web UI 
    高度插件化
        插件Hub
        动态管理：Jenkins运行过程中管理插件
        流水线的定义高度依赖于插件
    流水线定义：支持多种风格
        Freestyle Job
        Pipeline Job 
        Maven Job 
        Multibranch Pipeline Job
        ……
    分布式构建
        master/agent 
            master：定义管理流水线 
            agent：执行流水线

    1.x, 2.x
    
    安装部署Jenkins：
        war文件，由Servlet Container（tomcat/jetty）负责运行
    
        docker-compose 
            docker container agent 
        kubernetes编排
            Pod agent 
    
    Jenkins： 
        代码仓库：GitLab 
        代码质量评估：SonarQube Server 
        制品库：Harbor
        部署环境：Kubernetes
            Deployment： 滚动发布
        ArgoCD：GitOps
            Rollouts：GitOPs
            Prometheus：渐进式交付

### Jenkins部署和使用

根据官网手册选择适合的版本进行安装

![jenkins用户](.\ima\jenkins用户.PNG)

![jenkins用户2](.\ima\jenkins用户2.PNG)

对于每一个jenkins job 在本都的jenkins的工作目录下建一个子目录，通常与job name一致，基于这个子目录完成一系列的自动化操作

使用“凭据”保存敏感数据

+ Jenkins的Job中可能会使用到敏感数据
  + 基于ssh连接目标主机时使用的私钥，或者用户名与密码
  + 从私有代码仓库中克隆代码时使用的认证信息
  + 将构建后的Package推送至目标存储服务时使用的认证信息
+ 为确保自动化的Job能自动访问到这些信息，Jenkins则通过“凭据”来保存它们
  + 创建的“凭据”可基于其名称等进行引用

![Jenkins凭据](.\ima\Jenkins凭据.PNG)

Credentials :增加和删除凭据

凭据配置：凭据保存在何处

![凭据1](.\ima\凭据1.PNG)

以jerry用户的身份去访问gitlab中的devops项目

<img src=".\ima\cicd.PNG" alt="cicd" style="zoom:50%;" />

![凭据2](.\ima\凭据2.PNG)

为流水线添加一个input,会将对应的仓库clone到本地，把工作目录切换到项目之下，执行相关的命令

![test1](.\ima\test1.PNG)

![test2](.\ima\test2.PNG)

接下来手动触发（触发方式有很多种）

### 如何使用Jenkins创建CICD任务

![cicd2](.\ima\cicd2.PNG)

![cicd3](.\ima\cicd3.PNG)

### 自由风格的流水线作业

把流水线的定义也纳入到项目的版本库中，然后让jenkins以读取源码的方式，来执行相关的构建步骤，这个文件就是Jenkinsfile

虽然jenkins在其操作系统上的权限不是管理员的权限，但是在其工作目录下可以安装相应的工具（对于流水线风格的任务）（/var/lib/jenkins/）

![工具配置1](.\ima\工具配置1.PNG)

![工具配置2](.\ima\工具配置2.PNG)

但是对于自由风格的项目来说，不会使用全局配置，而是使用的操作系统的工具。但是可以改为**调用顶层目标**

![工具配置3](.\ima\工具配置3.PNG)

![test3](.\ima\test3.PNG)

流水线的定义：从某一位置加载一些数据（代码仓库），经过中间的一些步骤，输出至某一位置。而定义流水线又有不同的风格

    Freestyle作业步骤：
        不同的步骤类型，他们加载所需工具的逻辑有所主别
            执行shell：
                依赖的工具，源自于Jenkins所在的操作系统环境
                这些工具，可能依赖于管理手动部署
    
            其它类型的步骤：
                调用Jenkins定义的全局构建工具：
                    支持自动安装
                    安装位置：$JENKINS_HOME/tools/

#### Maven项目构建简介

![maven构建](.\ima\maven构建.PNG)

居于全局的配置工具，maven是在线安装的，如下修改其存储库（用来完成指定的依赖）

+ 基于插件的方式定义配置文件，将其作为可调用对象

  ![config file provider](.\ima\config file provider.PNG)

  ![config file provider2](.\ima\config file provider2.PNG)

  ![config file provider3](.\ima\config file provider3.PNG)

  ![config file provider4](.\ima\config file provider4.PNG)

  <mirror>
      <id>nexus-aliyun</id>
      <mirrorOf>central</mirrorOf>
      <name>Nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
  </mirror>

+ ![config file provider5](.\ima\config file provider5.PNG)

  

#### 案例

有一个动态的web项目，可以打包为一个war文件，将其部署到tomcat之上（当其在仓库上有代码变动的时候，直接clone来下，执行流水线部署到tomcat上）

+ 创建流水线

  ![案例1](.\ima\案例1.PNG)

  ![案例2](.\ima\案例2.PNG)

+ 准备servlet

  ```
  root@slb01:~# git clone  https://github.com/ForcemCS/learning-jenkins-cicd.git
  Cloning into 'learning-jenkins-cicd'...
  remote: Enumerating objects: 185, done.
  remote: Counting objects: 100% (185/185), done.
  remote: Compressing objects: 100% (122/122), done.
  remote: Total 185 (delta 78), reused 158 (delta 54), pack-reused 0
  Receiving objects: 100% (185/185), 594.29 KiB | 694.00 KiB/s, done.
  Resolving deltas: 100% (78/78), done.
  root@slb01:~/learning-jenkins-cicd/04-tomcat-with-manager# docker compose build
  root@slb01:~/learning-jenkins-cicd/04-tomcat-with-manager# docker compose build
  然后测试 http://12.0.0.5:8088/manager 
  ```

  

+ 在部署到tomcat的时候，可以额外在安装一个插件Deploy to container（支持自动部署应用到tomcat环境中）

  ![案例3](.\ima\案例3.PNG)



+ 访问servlet容器的管理接口时，通常要先完成认证；用到的认证要以“凭据”形式创建于Jenkins之上（不然无法完成部署）

  ```
  root@slb01:~/learning-jenkins-cicd/04-tomcat-with-manager# cat ./tomcat/tomcat-users.xml
  ....
  <role rolename="manager-gui"/>
    <role rolename="manager-script"/>  远程部署需要用到的权限
    <user username="tomcat" password="magedu.com" roles="manager-gui,manager-script"/>
  </tomcat-users>
  
  ```

  ![案例4](.\ima\案例4.PNG)

  ![案例5](.\ima\案例5.PNG)

  ![案例6](.\ima\案例6.PNG)

  ![案例7](.\ima\案例7.PNG)

  ![案例8](.\ima\案例8.PNG)

  ![案例9](.\ima\案例9.PNG)

  ![image-20230604203801827](C:\Users\kuigr\AppData\Roaming\Typora\typora-user-images\image-20230604203801827.PNG)

+ 克隆helloworldJSP项目的代码进行修改

  ```
  root@slb01:~# git clone  http://gitlab.wukui.com/jerry/helloworld-jsp.git
  Cloning into 'helloworld-jsp'...
  remote: Enumerating objects: 29, done.
  remote: Counting objects: 100% (29/29), done.
  remote: Compressing objects: 100% (21/21), done.
  remote: Total 29 (delta 0), reused 29 (delta 0), pack-reused 0
  Receiving objects: 100% (29/29), 13.11 KiB | 13.11 MiB/s, done.
  root@slb01:~/helloworld-jsp/WebContent# cat index.jsp
  <%@ page language="java" contentType="text/html; charset=ISO-8859-1"
      pageEncoding="ISO-8859-1"%>
  <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.or
  /TR/html4/loose.dtd">
  <html>
  <head>
  <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
  <title>Welcome From index.jsp</title></head>
  <body>
  <p>Hello World, Application Version: v0.0.2.</p>
  </body>
  </html>
  #pom.xml中的版本信息也改为0.0.2
  root@slb01:~/helloworld-jsp# git status  
  On branch master
  Your branch is up to date with 'origin/master'.
  
  Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git restore <file>..." to discard changes in working directory)
  	modified:   WebContent/index.jsp
  	modified:   pom.xml
  
  no changes added to commit (use "git add" and/or "git commit -a")
  root@slb01:~/helloworld-jsp# git add  .  
  root@slb01:~/helloworld-jsp# git config   --local user.name jerry 
  root@slb01:~/helloworld-jsp# git config   --local user.email jerry@gmail.com
  root@slb01:~/helloworld-jsp# git commit -m "v0.0.2"
  [master f845c1a] v0.0.2
   2 files changed, 2 insertions(+), 2 deletions(-)
  root@slb01:~/helloworld-jsp# git log  --pretty=oneline 
  f845c1a3b82031f29e0e63e8e7b9899c321e9f0a (HEAD -> master) v0.0.2
  8db5bca64f33dced75d468e0e7bd10147aadc523 (origin/master, origin/HEAD) init commit
  aa234ce5571a2885da5e9bc02c5a6dbd4c799885 Initial commit
  root@slb01:~/helloworld-jsp# git  push origin 
  Username for 'http://gitlab.wukui.com': jerry
  Password for 'http://jerry@gitlab.wukui.com': 
  Enumerating objects: 9, done.
  Counting objects: 100% (9/9), done.
  Delta compression using up to 4 threads
  Compressing objects: 100% (5/5), done.
  Writing objects: 100% (5/5), 435 bytes | 435.00 KiB/s, done.
  Total 5 (delta 4), reused 0 (delta 0), pack-reused 0
  To http://gitlab.wukui.com/jerry/helloworld-jsp.git
     8db5bca..f845c1a  master -> master
  
  ```
  
  正常的流程应该是自动触发的 ，但是目前还没有配置，手动触发
  
     思考：
          1、如何为golang、python、npm提供全局构建环境？
          2、如何Jenkins请求下线tomcat上运行中的web app?

+ 考虑到jenkins所在的系统之上没有可以用的maven的时候，可以利用全局工具自动部署，并完成mvn项目的构建

+ 对于不同的项目来讲，可能都会有响应的插件，接下来安装Maven插件，创建maven风格的项目

  + docker-compose-multi-instances.yaml 启动了多个实例，各自都打开了manager应用

  + 启动这三个应用

  + 然后通过**Deploy war/ear to a container**插件部署war包到这三个实例下

    + 先部署第一个

      ![案例11](.\ima\案例11.PNG)

      ![案例12](.\ima\案例12.PNG)

    + 以同样的方式部署两外两个实例

  + 然后立即构建

    1、freestyle完成helloworldJSP应用的构建和部署；
        （1）shell构建
        （2）调用顶层maven目标
        （3）使用maven job 
    2、部署到单个tomcat环境；部署多个tomcat环境；
    3、扩展：使用ansible插件，调用ansible完成应用部署；

### 环境变量和构建参数

    Jenkins:
        CICD Server 
            帮助用户定制流水线
        工具：依赖系统环境、全局工具配置
        高度插件化：
            社区推荐的插件
    定制流水线：
        多种Job类型：
            Freestyle：自由风格作业
                配置接口：UI
    
                定制构建步骤：
                    执行shell命令
                        mvn clean package test verify
                    调用顶层Maven目标
            Pipeline：流水线风格作业
                配置接口：
                    脚本式流水线
                    声明式流水线
            Maven Job： 
    
    Freestyle:
        流水线： 
            Spring-boot-helloworld, maven工程, ...
    
            克隆 --> 构建 --> 测试 --> [Nexus（推送）]--> 部署 
    
            构建：mvn -B -DskipTests clean package 
            测试：mvn test
    
        Workspace：
            $JENKINS_HOME/workspaces/JOB_NAME/

#### 环境变量

环境变量可分为Jenkins内置变量和用户自定义变量两类

+ 自定义全局环境变量

  ![参数变量1](.\ima\参数变量1.PNG)

  ![参数变量2](.\ima\参数变量2.PNG)

  

#### 构建参数

+ 自定义构建过程中的参数

  ![构建参数1](.\ima\构建参数1.PNG)

  ![构建参数2](.\ima\构建参数2.PNG)

  ![构建参数3](.\ima\构建参数3.PNG)

  

+ 利用插件

  ![构建参数4](.\ima\构建参数4.PNG)

### 构建和推送Docker Image

在这个项目中http://gitlab.wukui.com/devops/spring-boot-helloWorld.git

在deploy目录下，基于Dockerfile配置的应用程序构建成镜像，并推送到目标镜像服务器之后（镜像推送到哪和tag是不确定的），基于镜像文件完成部署

Jenkins的多款插件都能实现Image构建和推送

+ docker-build-step Plugin

  + 要构建的分支参数化配置

    ![镜像推送测试4](.\ima\镜像推送测试4.PNG)

  + clone项目源代码

    ![推送镜像测试1](.\ima\推送镜像测试1.PNG)

  + 修改maven的构建环境

    ![推送镜像测试2](.\ima\推送镜像测试2.PNG)

  + 构建步骤

    ![推送镜像测试3](.\ima\推送镜像测试3.PNG)

  + 接下来使用jenkins所在的主机的docker环境来构建镜像（如果要跨主机调用docker.sock需要打开一个监听的远程的套接字）

    ```
    root@slb01:/var/run# ll /var/run/docker.sock 
    srw-rw---- 1 root docker 0 Jun  5 08:48 /var/run/docker.sock=
    让jenkins加入docker组
    root@slb01:/var/run# usermod  -G docker  jenkins 
    然后重启jenkins
    ```

    ![推送镜像测试5](.\ima\推送镜像测试5.PNG)

    另外的解决办法（假设docker运行在主机上，）是修改[systemd unit file](/lib/systemd/system/docker.service),然后执行systemctl  daemon-reload && systemctl  restart docker.service，指明jenkins用户连接docker sock的[地址](系统管理---Docker Builder---tcp://12.0.0.1:2376)

    ```
    [Service]
    Type=notify
    # the default is not to use systemd for cgroups because the delegate issues still
    # exists and systemd currently does not support the cgroup feature set required
    # for containers run by docker
    ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376 --containerd=/run/containerd/containerd.sock
    ExecReload=/bin/kill -s HUP $MAINPID
    TimeoutStartSec=0
    RestartSec=2
    Restart=always
    
    ```

    

  + 思考：修改harbor的监听端口80-->30080,当我们访问80的时候，为什么仍然可以实现跳转到443

    ​            项目的名字不要有大写字母，不然在构建镜像的时候无法引用{JOB_NAME}

  + 增加构建步骤

    ![推送镜像测试6](.\ima\推送镜像测试6.PNG)

    ![推送镜像测试7](.\ima\推送镜像测试7.PNG)

  + 将构建出来的镜像推送到harbor中

    ![推送镜像1](.\ima\推送镜像1.PNG)

    ![推送镜像2](.\ima\推送镜像2.PNG)

  

+ Docker Plugin

  插件的名字为Docker

  + 删除上一个插件的内容

    ![替换1](.\ima\替换1.PNG)

  + 添加系统配置

    ![docker云1](.\ima\docker云1.PNG)

    ![docker云2](.\ima\docker云2.PNG)

    ![docker云3](.\ima\docker云3.PNG)

    ![docker云4](.\ima\docker云4.PNG)

  + 回到项目中
  + 

  

+ CloudBees Docker Build and Publish Plugin

### 部署应用到kubernetes

推送镜像完成之后，就可以将镜像应用部署到k8s集群中去（假设源码仓库下写好了适配的yaml文件）

但是为了完成应用的部署，首先的确保jenkins主机的有kubectl命令2，基于jenkins主机访问k8s集群的时候，可以认证到看看k8s集群上并具相关的构建应用的权限，所以我们需要准备

+ jenkins主机可以使用kubectl命令

  ```
  root@slb01:~# curl -LO https://dl.k8s.io/v1.26.1/kubernetes-client-linux-amd64.tar.gz
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                   Dload  Upload   Total   Spent    Left  Speed
  100   138  100   138    0     0    273      0 --:--:-- --:--:-- --:--:--   273
  100 31.4M  100 31.4M    0     0  1256k      0  0:00:25  0:00:25 --:--:-- 1704k
  root@slb01:~# tar xf  kubernetes-client-linux-amd64.tar.gz 
  root@slb01:~# cd  kubernetes/client/bin/
  root@slb01:~/kubernetes/client/bin# ls
  kubectl  kubectl-convert
  root@slb01:~/kubernetes/client/bin# cp kubectl /usr/bin/
  
  ```

  

+ kubectl命令在执行远程配置的时候认证到k8s集群上

  ```shell
  root@master01:~# kubectl  create   namespace hello 
  namespace/hello created
  root@master01:~# kubectl   create  serviceaccount hello-admin -n hello
  serviceaccount/hello-admin created
  root@master01:~# kubectl  create rolebinding hello-admin  --clusterrole=cluster-admin --serviceaccount=hello:hello-admin   -n hello
  rolebinding.rbac.authorization.k8s.io/hello-admin created
  创建一个pod，取出其中的token信息，用来认证到k8s集群
  root@master01:~# cat hello-pod.yaml 
  apiVersion: v1
  kind: Pod
  metadata:
    name: test
    namespace: hello
  spec:
    containers:
    - image: ikubernetes/admin-box:v1.2
      name: sleep
      command: ["/bin/sh","-c","sleep 99999"]
    serviceAccountName: hello-admin
  root@master01:~# kubectl  apply -f  hello-pod.yaml
  root@master01:~# TOKEN=$(kubectl exec -it test -n hello -- \
  > cat /var/run/secrets/kubernetes.io/serviceaccount/token) &&echo $TOKEN
  *************(TOKEN信息)
  ```

+ 将认证信息保存为认证凭据

  ```
  保存上边的信息到jenkins上,可以拿着认证到apiserver上
  ```

  ![认证凭据1](.\ima\认证凭据1.PNG)

  ![认证凭据2](.\ima\认证凭据2.PNG)

  ![认证凭据3](.\ima\认证凭据3.PNG)

+ 在构建环境中使用token的文本信息

  ![利用凭据](.\ima\利用凭据.PNG)

  

  ![参数2](.\ima\参数2.PNG)

+ 执行shell步骤，来完成部署

  ```
  #!/bin/bash
  sed -i "s@__IMAGE\__@${registry}/kubernetes/spring-boot-helloworld:${imageTag}@" deploy/all-in-one.yaml
  kubectl --server ${APISERVER} --insecure-skip-tls-verify=${skipTLSCertVerify} --token=${SA-TOKEN} apply -f  deploy/all-in-one.yaml
  其中需要注意的是APISERVER的值必须是https协议
  ```

+ 但是还要解决各个node节点认证到私有仓库的问题

  参考https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

  ```
  root@slb02:~# scp   /root/.docker/config.json root@12.0.0.15:/root/
  root@master01:~# kubectl create secret generic containerd-harbor     --from-file=.dockerconfigjson=/root/config.json     --type=kubernetes.io/dockerconfigjson -n hello
  secret/cantainerd-harbor created
  root@master01:~# kubectl get secret  -n hello
  NAME                TYPE                                  DATA   AGE
  build-bot-secret    kubernetes.io/service-account-token   3      77d
  cantainerd-harbor   kubernetes.io/dockerconfigjson        1      19s
  
  ```

  所以还需要修改项目的源代码，有关提交源码或分支合并的操作参考前边

  ```
  root@slb01:/var/lib/jenkins/workspace# ls
  'HelloWorld JSP'  'HelloWorld JSP_Maven'   spring-boot-helloWorld   spring-boot-helloWorld@tmp   test-freestyl2   test-freestyl3   test-freestyle   test-freestyle@tmp
  root@slb01:/var/lib/jenkins/workspace# cd spring-boot-helloWorld
  直接修改yaml文件（其实可以调用ansible工具在配置JOB的时候，直接修改）
  ```

### 触发任务

    分支： 
        master、develop 
            master：发布历史 
                hotfix 
            develop：变更历史 
                变更：由feature分支合并生成
                feature
                    feature/x, feature/y, feature/z
                release

其实k8s的认证信息是不应该放在jenkins上的，而是在k8s集群中运行一个组件（配置仓库），配置文件一但发生变动，会拖到本地

并完成部署

触发器：

+ 轮询SCM：每隔一定的时间到代码仓库中请求代码，本地与远程的比较

+ 触发远程构建：在作业之上打开一个钩子，允许别人push信息，只要得到了想要的信息就触发

#### Gitlab通知触发

##### Gitlab触发jenkins

在每一个仓库上一但有相关的操作，自动生成一个事件，并且通过webhook调用push给另外一个系统（在jenkins之上打开一个webhook,等待允许别人推送事件，以此触发流水线的执行）并且jenkins将JOB执行的结果推回给gitlab（jenkins调用gitlab的webhook）

插件：Gitlab  Generic Webhook Trigger

+ gitlab配置（管理员登陆）

  + 打开本地的Webhook和服务对本地网络的请求（因为我们的jenkins和gitlab处于同一主机，若不是则可以忽略）

    ![通知触发1](.\ima\通知触发1.PNG)

    ![通知触发2 - 副本](.\ima\通知触发2 - 副本.PNG)

    ![通知触发3](.\ima\通知触发3.PNG)

    

  + 待插件安装完成之后，查看是否支持gitlab触发（允许仓库之上出现什么事件才触发）

    允许别人向 http://jenkins.wukui.com:8080/project/spring-boot-helloWorld这个钩子推送事件需要为这个

    接口设立一个token（点击高级--Generate(e28f4d67b9a26210959fed963d5528c1)）

    这样在jenkins就完成了接受事件push

    ![通知触发4](.\ima\通知触发4.PNG)

    ![触发通知5](.\ima\触发通知5.PNG)

  + 再对代码仓库进行设定（spring-boot-helloWorld--Settings--Webhooks）

    这个Webhooks连接了代码仓库与jenkins,填写webhooks URL与secret token 事件类型（与kenkins保持一致）

    ![触发通知7](.\ima\触发通知7.PNG)

    ![触发通知8 (2)](.\ima\触发通知8 (2).PNG)

​              ![触发通知9](.\ima\触发通知9.PNG)

+ 接下来做一次测试

  ```shell
  克隆代码，修改版本信息
  root@slb01:~/tom/spring-boot-helloWorld# vim src/main/java/com/neo/controller/HelloWorldController.java
  root@slb01:~/tom/spring-boot-helloWorld# vim pom.xml
  ```

  

+ 现在假设gitlab上push的是develop分支，而JOB任务里默认的是Main分支，这样在自动触发job执行是报错的，我们可以利用一个插件（[Git Parameter Plug-In](https://plugins.jenkins.io/git-parameter)）来灵活解决这个问题

##### Jenkins-->gitlab

配置步骤

+ 在GitLab上，以仓库所关联的用户生成Access Token，并选择可访问的API

  ![Gtoken1](.\ima\Gtoken1.PNG)

  ![Gtoken2](.\ima\Gtoken2.PNG)

+ 在Jenkins上，将GitLab生成的Access Token保存为Credential

  ![Gtoken3](.\ima\Gtoken3.PNG)

+ 在Jenkins的系统配置，添加GitLab Connection的定义 （高级--测试）

  ![Gtoken4](.\ima\Gtoken4.PNG)

+ 在FreeStyle类型的流水线上，选择要使用的GitLab Connection

  ![Gtoken5](.\ima\Gtoken5.PNG)

+ 在FreeStyle类型的流水线上，添加“构建后操作”

  ![Gtoken6](.\ima\Gtoken6.PNG)

#### Generic Webhook Trigger触发

+ 提供的是一种通用的触发机制，它允许直接经由该插件暴露的API（<JENKINS_URL>/generic-webhooktrigger/invoke）接收JSON或XML格式的HTTP POST请求进行触发

+ 触发的Jenkins项目则由配置的规则和POST请求进行定义

  + 支持从POST请求的JSONPath和Xpath中提取数据

    + 例如请求方法[curl -X POST -H "Content-Type: application/json" -d '{ "ref": "refs/heads/develop" }' -vs http://jenkins.wukui.com:8080/generic-webhook-trigger/invoke?token=YWTVmiT1eU2cDhzTiVIf2jvaVNzBMP2dJ2FEUykyvw8]中

      将ref字段的值取出来，来拉取分支

  + 支持从query parameters中提取数据

  + 支持从headers中提取数据

### 通知与报告

Jenkins通知：将任务的执行状态、事件或信息推送给相关用户，这些通常发生在pipeline的“构建后处理（post-processing）”时期

Jenkins使用的许多插件或工具都会为其支持的多种任务生成HTML报告，相关的任务有如代码分析、代码覆盖率和单元测试等；

+ 其中一些工具（如SonarQube和JaCoCo等）甚至可以和Jenkins任务输出做定制集成，这些工具甚至会采用直观的图标给出相关报告的链接

#### Jenkins的邮件通知

在基础的邮件通知完成之后，使用扩展的插件来丰富通知的内容[参考](https://blog.csdn.net/GLYX5717/article/details/119747860)

企业微信通知设置（[Qy Wechat Notification ](https://plugins.jenkins.io/qy-wechat-notification)）

## SonarQube

SonarQube是一款开源的代码质量管理系统，支持代码自动审查，用于检测代码中的错误、漏洞，它集成到现有的工作流程，以便在项目分支和拉取（PR）请求之间进行连续的代码检查

SonarQube有四个关键组件

+ SonarQube Server运行有三个组件
  + Web Server：UI
  + Search Server：为UI提供搜索功能，基于ElasticSearch
  + Compute Engine Server：处理代码分析报告并将之存储到SonarQube Database中
+ SonarQube Database：负责存储SonarQube的配置，以及项目的质量快照等
+ SonarQube Plugin
+ Code analysis Scanners：代码扫描器，扫描后将报告提交给SonarQube Server

![SONAR1](.\ima\SONAR1.PNG)

![sonar2](.\ima\sonar2.PNG)

### Install

[Requirement](https://docs.sonarqube.org/latest/requirements/prerequisites-and-overview/)

[二进制包位置](https://binaries.sonarsource.com)

```
root@node01:/lib/systemd/system# cat /lib/systemd/system/sonarqube.service
# postgresql.service is the meta unit for managing all PostgreSQL clusters on
# the system at once. Conceptually, this unit is more like a systemd target,
# but we are using a service since targets cannot be reloaded.
#
# The unit actually managing PostgreSQL clusters is postgresql@.service,
# instantiated as postgresql@15-main.service for individual clusters.

[Unit]
Description=SonarQube service

[Service]
Type=forking
User=sonarqube
Group=sonarqube
PermissionsStartOnly=true
ExecStart=/usr/local/sonarqube/bin/linux-x86-64/sonar.sh  start
ExecStop=/usr/local/sonarqube/bin/linux-x86-64/sonar.sh  stop
StandardOutput=syslog
LimitNOFILE=131072
LimitNPROC=8192
TimeoutStartSec=5
Restart=always
SuccessExitStatus=143
[Install]
WantedBy=multi-user.target

```

Code analysis Scanners的安装

```
curl -LO https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip
unzip sonar-scanner-cli-4.8.0.2856-linux.zip  -d /usr/local/
ln -sv /usr/local/sonar-scanner-cli-4.8.0.2856-linux /usr/local/sonar-scanner

```

+ 配置sonar-scanner的全局属性定义[参考](https://docs.sonarqube.org/latest/analysis/analysis-parameters/)

  ```shell
  root@node01:/usr/local/sonar-scanner/conf# cat sonar-scanner.properties
  #Configure here general information about the environment, such as SonarQube server connection details for example
  #No information about specific project should appear here
  
  #----- Default SonarQube server
  sonar.host.url=http://localhost:9988
  
  #----- Default source code encoding
  sonar.sourceEncoding=UTF-8
  
  sonar.login=jenkins
  sonar.password=wukui123456
  
  sonar.log.level=INFO
  sonar.verbose=false
  
  ```

+ 手动启动代码扫描

  在需要代码扫描的项目根目录下，为sonar-scanner建立必要的配置文件sonar-project.properties，需要提供的参数类似如下所示

  ```shell
  root@node01:~# git clone   http://gitlab.wukui.com/devops/spring-boot-helloWorld.git  
  Cloning into 'spring-boot-helloWorld'...
  Username for 'http://gitlab.wukui.com': root
  Password for 'http://root@gitlab.wukui.com': 
  remote: Enumerating objects: 279, done.
  remote: Counting objects: 100% (20/20), done.
  remote: Compressing objects: 100% (16/16), done.
  remote: Total 279 (delta 2), reused 19 (delta 2), pack-reused 259
  Receiving objects: 100% (279/279), 38.69 KiB | 7.74 MiB/s, done.
  Resolving deltas: 100% (76/76), done.
  root@node01:~# cd spring-boot-helloWorld/
  root@node01:~/spring-boot-helloWorld# cat sonar-project.properties
  # 项目的惟一标识
  sonar.projectKey= spring-boot-helloWorld
  # 项目的名称，用于显示在sonar web中
  sonar.projectName=spring-boot-helloWorld
  # 项目的版本
  sonar.projectVersion=v0.9.6
  # 需要分析的源码所在的目录
  sonar.sources=.
  # targe的位置
  sonar.java.binaries=.
  # 指定项目中使用的编程语言
  sonar.language=java
  #编码格式
  sonar.sourceEncoding=UTF-8
  root@node01:~/spring-boot-helloWorld# /usr/local/sonar-scanner/bin/sonar-scanner  执行扫描
  ```


### Jenkins集成SonarQube

在jenkins上执行代码扫描，将扫描的结果发给sonarqube server,由server进行分析，将分析的结果回送给流水线

Jenkins借助于SonarQube Scanner插件将SonarQube提供的代码质量检查能力集成到pipeline上，从而确保质量阈检查失败时，能

够避免继续进行后续的操作，例如发布等

配置Jenkins使用sonar-scanner进行代码扫描，并将扫描结果报告给SonarQube Server的主要步骤如下

+ 在Jenkins上安装**SonarQube Scanner**插件

+ 配置Jenkins对接到SonarQube Server

  + 在SonarQube Server创建一个用户（添加相应的权限）新建用户的身份生成令牌，该令牌将被Jenkins用于通过相应的URL打开SonarQube 

    ![sona1](.\ima\sona1.PNG)

    ![sona2](.\ima\sona2.PNG)

    ![sona3](.\ima\sona3.PNG)

    ![sona7](.\ima\sona7.PNG)

+ 配置Jenkins的全局工具sonar-scanner

  ![sona8](.\ima\sona8.PNG)

+ 在SonarQube上添加回调Jenkins的Webhook

  ![sona4](.\ima\sona4.PNG)

  ![sona5](.\ima\sona5.PNG)

  ![sona6](.\ima\sona6.PNG)

+ 在Jenkins项目上调用sonar-scanner进行代码质量扫描

  ![sona10](.\ima\sona10.PNG)

  ![sona11](.\ima\sona11.PNG)

  

## Jenkins系统架构

### 静态Agent

在Agent执行的时候，可以调用全局配置工具（Master）完成相关的动作，尽量不用agent的本地shell环境

#### JNLP Agent

Master节点执行

![agent5](.\ima\agent5.PNG)

配置JNLP Agent创建分布式构建环境的简要步骤

+ 准备Agent节点，部署好Jenkins

  ```
  root@node01:~# apt-get  install  -y  openjdk-17-jdk
  在华为云的镜像仓库下找到debian的包
  root@node01:~# curl -LO https://repo.huaweicloud.com/jenkins/debian/jenkins_2.407_all.deb
  调整插件更新站点，以加速插件部署
  root@node01:/var/lib/jenkins# sed -i 's@https://updates.jenkins.io/download/@https://mirrors.tuna.tsinghua.edu.cn/jenkins/@g' updates/default.json
  root@node01:/var/lib/jenkins# sed -i 's@www.google.com@www.baidu.com@gi' updates/default.json
  root@node01:/var/lib/jenkins# sed -i 's@https://updates.jenkins.io/update-center.json@https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/current/update-center.json@gi' hudson.model.UpdateCenter.xml
  接下来访问jenkins的服务 12.0.0.25:8080/restart
  ```

+ 在Master节点上，添加Agent节点的定义

  ![agent1](.\ima\agent1.PNG)

  ![agent2](.\ima\agent2.PNG)

  ![agent3](.\ima\agent3.PNG)

  ![agent4](.\ima\agent4.PNG)

+ 在Master节点上，获取Agent节点连接至Master时需要执行的命令（在agent执行）

  ![agent6](.\ima\agent6.PNG)

  ```
  root@node01:~# mkdir /home/jenkins/agent/   -p
  root@node01:~# chown -R  jenkins.jenkins /home/jenkins/
  jenkins@node01:/home/jenkins$ curl -sO http://jenkins.wukui.com:8080/jnlpJars/agent.jar
  jenkins@node01:/home/jenkins$ nohup   java -jar agent.jar -jnlpUrl .....   & 
  ```

+ 项目使用agent

  ![agent7](.\ima\agent7.PNG)

#### SSH Agent

此时master是客户端

为angent的jenkins用户设置一个密码

```
root@node01:~# passwd jenkins  
New password: 
Retype new password: 
passwd: password updated successfully

```

![agent08](.\ima\agent08.PNG)

![agent9](.\ima\agent9.PNG)

![agent10](.\ima\agent10.PNG)

#### Container

在一台docker主机之上打开TCP连接

确保镜像文件启动成容器之后能够和Master节点建立连接

+ JNLP  (agent连接master)

  ![agent11](.\ima\agent11.PNG)

  ![agent02](.\ima\agent02.PNG)

  ![agent13](.\ima\agent13.PNG)

  复制Master的密钥信息，然后修改agentjnlp的docker compose[文件](.\ima\docker-compose-inbound-agent.yml)

  ```
  root@slb01:~/learning-jenkins-cicd/07-jenkins-agents# docker compose  -f  ./docker-compose-inbound-agent.yml  up   -d
  ```

+ SSH   (master连接agent,确保容器有SSH Server,jenkins提供了相关的镜像来完成)

  ```
  生成一对密钥
  root@slb01:~/learning-jenkins-cicd/07-jenkins-agents# ssh-keygen  -t rsa -b 2048 -P ''
  Generating public/private rsa key pair.
  Enter file in which to save the key (/root/.ssh/id_rsa): ./jenkins_agent_rsa
  Your identification has been saved in ./jenkins_agent_rsa
  Your public key has been saved in ./jenkins_agent_rsa.pub
  The key fingerprint is:
  SHA256:/DypAqCY+5pTLX0d1xzt6iUtMvuZWj0IocbgGGnV9Jc root@slb01
  The key's randomart image is:
  +---[RSA 2048]----+
  |      .o.    .   |
  |     o  ..  ...  |
  |    + .   ooEo   |
  |  .. + +...oo .  |
  |.o +. ..So.  o   |
  |+ o + ...ooo+oo  |
  | o . o    ==o+o  |
  |o.    .  ..o.o . |
  |o+.    .. .o+    |
  +----[SHA256]-----+
  公钥信息保存在agent端，私钥信息保存在Master上（Crenditials）
  ```

  然后系统管理--->节点管理-->添加节点

### 动态Agent

常用的集群管理和分布式构建插件示例

+ Docker插件：在Docker Engine上，基于容器模板，动态创建并运行容器作为Agent节点
+ Kubernetes：在Kubernetes集群上，基于Pod模板，动态创建并运行Pod作为Agent节点
+ OpenStack Cloud：基于OpenStack运行动态Agent
+ Amazon Elastic Container Service (ECS) / Fargate：基于Amazon EC2 Container服务运行动态Agent

#### 基于Docker容器的动态Agent

配置步骤

+ 找一台运行docker的agent主机，修改Docker的Unit File，在其ExecStart中添加监听的地址和端口

  ```shell
  root@slb01:~/harbor/harbor# cat /lib/systemd/system/docker.service
  [Unit]
  Description=Docker Application Container Engine
  Documentation=https://docs.docker.com
  After=network-online.target docker.socket firewalld.service containerd.service time-set.target
  Wants=network-online.target containerd.service
  Requires=docker.socket
  
  [Service]
  Type=notify
  # the default is not to use systemd for cgroups because the delegate issues still
  # exists and systemd currently does not support the cgroup feature set required
  # for containers run by docker
  ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376 --containerd=/run/containerd/containerd.sock
  root@slb01:~/harbor/harbor# systemctl  daemon-reload 
  root@slb01:~/harbor/harbor# systemctl  restart  docker.service
  测试
  root@slb01:~/harbor/harbor# curl http://docker.agent.com:2376/version  
  {"Platform":{"Name":"Docker Engine - Community"},"Components":[{"Name":"Engine","Version":"23.0.1","Details":{"ApiVersion":"1.42","Arch":"amd64","BuildTime":"2023-02-09T19:47:01.000000000+00:00","Experimental":"false","GitCommit":"bc3805a","GoVersion":"go1.19.5","KernelVersion":"5.15.0-73-generic","MinAPIVersion":"1.12","Os":"linux"}},{"Name":"containerd","Version":"1.6.18","Details":{"GitCommit":"2456e983eb9e37e47538f59ea18f2043c9a73640"}},{"Name":"runc","Version":"1.1.4","Details":{"GitCommit":"v1.1.4-0-g5fd4c4d"}},{"Name":"docker-init","Version":"0.19.0","Details":{"GitCommit":"de40ad0"}}],"Version":"23.0.1","ApiVersion":"1.42","MinAPIVersion":"1.12","GitCommit":"bc3805a","GoVersion":"go1.19.5","Os":"linux","Arch":"amd64","KernelVersion":"5.15.0-73-generic","BuildTime":"2023-02-09T19:47:01.000000000+00:00"}
  ```

+ 在Master安装插件（Docker plugin ）,然后进行相关配置（也就是对agent的配置）

  ![docker1](.\ima\docker1.PNG)

  ![docker2](.\ima\docker2.PNG)

+ 选择连接方式（SSH或者JNLP）的模板

  + JNLP

    ![docker4](.\ima\docker4.PNG)

    ![DOCKER5](.\ima\DOCKER5.PNG)

    ![DOCKER6](.\ima\DOCKER6.PNG)

    

    ![DOCKER7](.\ima\DOCKER7.PNG)

    然后在指定的项目上选取响应的Agent节点

  + SSH

    在新加一个模板，用标签来区别

## K8S上的Jenkins分布式构建

各Pod Agent以Inbound Agent形式运行，inbound-agent容器会自动连接到Jenkins Master

无论Master是在集群外部还是内部

+ 在每个Pod Agent中，始终有一个自动配置的名为jnlp的特殊容器（可以运行多个容器实现不同的功能）freestyle风格的作业只能在

  JNLP容器中运行

+ JNLP容器会被注入一些以下几个环境变量，以便它了解Jenkins Master的信息

  + JENKINS_URL
  + JENKINS_SECRET
  + JENKINS_AGENT_NAME
  + JENKINS_NAME

### Jenkins Controller部署在 K8S集群上

+ 部署Ingress-Controller

  在部署好之后，默认是LoadBalancer类型，需要添加一个EXTERNAL-IP,以便于访问

  ```yaml
  root@master01:~# kubectl  get svc  -n ingress-nginx 
  NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
  ingress-nginx-controller             NodePort    10.100.43.208   <none>        80:62684/TCP,443:39316/TCP   87d
  ingress-nginx-controller-admission   ClusterIP   10.100.77.92    <none>        443/TCP                      87d
  root@master01:~# kubectl  edit  svc ingress-nginx-controller    -n ingress-nginx 
  service/ingress-nginx-controller edited
  root@master01:~# kubectl  get svc  -n ingress-nginx 
  NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
  ingress-nginx-controller             LoadBalancer   10.100.43.208   12.0.0.27     80:62684/TCP,443:39316/TCP   87d
  ingress-nginx-controller-admission   ClusterIP      10.100.77.92    <none>        443/TCP                      87d
  
  spec:
    allocateLoadBalancerNodePorts: true
    clusterIP: 10.100.43.208
    clusterIPs:
    - 10.100.43.208
    externalIPs:
    - 12.0.0.27
    externalTrafficPolicy: Cluster
    internalTrafficPolicy: Cluster
    ipFamilies:
    - IPv4
    ipFamilyPolicy: SingleStack
    ports:
    - appProtocol: http
      name: http
      nodePort: 62684
      port: 80
      protocol: TCP
      targetPort: http
  
  ```

+ 使用持久持久存储（longhorn）

  部署步骤参见官网

+ 部署Jenkins

  尤其没有充足的外部IP，所以没有使用EXTERNAL-IP

  ```shell
  #提前下载镜像，因为ctr下载速度很慢
  root@node01:~# ctr -n  k8s.io images  pull  jenkins/jenkins:jdk17
  root@master01:~/CICD-Test# kubectl apply  -f  ./deploy/
  root@master01:~/CICD-Test# kubectl get svc   -n   ingress-nginx 
  NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
  ingress-nginx-controller             NodePort    10.100.43.208   <none>        80:62684/TCP,443:39316/TCP   88d
  ingress-nginx-controller-admission   ClusterIP   10.100.77.92    <none>        443/TCP                      88d
  root@master01:~/CICD-Test# kubectl get pods   -n ingress-nginx  -o wide  
  NAME                                       READY   STATUS      RESTARTS        AGE   IP               NODE        NOMINATED NODE   READINESS GATES
  ingress-nginx-admission-create-9s2vr       0/1     Completed   0               88d   10.200.140.73    worker-02   <none>           <none>
  ingress-nginx-controller-c69664497-5tq7m   1/1     Running     32 (152m ago)   82d   10.200.196.176   worker-01   <none>           <none>
  root@master01:~/CICD-Test# kubectl  get  ingress  -n  jenkins 
  NAME      CLASS   HOSTS                              ADDRESS         PORTS   AGE
  jenkins   nginx   cicd.wukui.com,jenkins.wukui.com   10.100.43.208   80      10m
  #将cicd.wukui.com解析到worker-01
  ```

#### 利用Kubernetes进行分布式构建

+ Jenkins上安装好Kubernetes插件
+ 确认Jenkins开启了jnlp-agent端口
+ 在Kubernetes创建专用的ServiceAccount
+ 在Kubernetes上为该ServiceAccount授予在特定名称空间（如jenkins）下创建和管理Pod等资源的权限
+ 在Jenkins上添加Cloud，配置要接入的Kubernetes集群
+ 在Jenkins上添加Pod Template，以定义如何运行Pod Agent





### Jenkins Controller部署在外部的Linux主机

```shell
root@master01:/etc/kubernetes/ssl# cat ca.pem 
-----BEGIN CERTIFICATE-----
MIIDmjCCAoKgAwIBAgIUMV4maWyzIDumkK8EWLj4Ov9AmNEwDQYJKoZIhvcNAQEL
BQAwZDELMAkGA1UEBhMCQ04xETAPBgNVBAgTCEhhbmdaaG91MQswCQYDVQQHEwJY
UzEMMAoGA1UEChMDazhzMQ8wDQYDVQQLEwZTeXN0ZW0xFjAUBgNVBAMTDWt1YmVy
bmV0ZXMtY2EwIBcNMjMwMzA2MDkwNTAwWhgPMjEyMzAyMTAwOTA1MDBaMGQxCzAJ
BgNVBAYTAkNOMREwDwYDVQQIEwhIYW5nWmhvdTELMAkGA1UEBxMCWFMxDDAKBgNV
BAoTA2s4czEPMA0GA1UECxMGU3lzdGVtMRYwFAYDVQQDEw1rdWJlcm5ldGVzLWNh
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA5K8l2hgkeDZ2ECoA5aCK
FkJ/C71sDn+OT7Sxn7VuoUFKKNXY3ef31kKpbnAwr69hXVUTGM3btXByaLVMf6FE
lMUBUbEJU7+s7bFVbLhJBISCLe7S8I5Ur3PIXQJFummVKr8409gnlcPNQibQpfza
jD2hay1GKkNX031hvSxJrHf8+35jn8Nt20EhVoS6ESs2+Hw24+pMkRiUgZMd0HN0
Hq0S/L22Sih8/l1xVDe4yM9F59IyppXM04l3khqqf1NbRN1GWc1raL20KD/Tp2UF
AnLVr9jdDMdDP7vC6pkjc3aMxKEjjvx0R6bAIBvZP8y9WUdVRphNEA7fQgz4Y0PJ
owIDAQABo0IwQDAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUwAwEB/zAdBgNV
HQ4EFgQUCMlzyn/3AQ1nAywkOf/1kuIDaTMwDQYJKoZIhvcNAQELBQADggEBAM19
Q6lPNG02TdiOLW3yj0OA2re0Feank23ChWj3vzrvvfG+HNz+xR8hw/fDudwtMvJV
xtZ8YsU6YNXt/C1Ag0Gpm5xdF8JL4jFUKDs10m+GL5c83/sJsS9M6mc0u/d4dSH0
Pzg3SRcDzU+oEtk3dEQdRPcDkEFrT7RNysQth+kJtTYZpSJBt3RU3M95wBlWyPTb
LBRfPrVWeEA6JKiTAT23l0j7cGfvzwuxrZ8i2Upyymy3Jr8RERcGvaMniD7xBUr1
eBN2PVZN5RElPWY8JMCyafEkT/tpz1wRIp+S0Z4NnzFOv4uIx0N4K0lT5vt2BsDU
ZNkRtwroSRY6BHDvlcY=
-----END CERTIFICATE-----
#直接系统中已经存在的serviceaccount账号（否则自己创建，然后利用认证授权的相关内容进行操作）
root@master01:~#  kubectl  get  pods -n jenkins  
NAME                      READY   STATUS    RESTARTS      AGE
jenkins-5c84664d4-5km8c   1/1     Running   2 (10m ago)   13h
kube-agent-pod-lwv8r      0/1     Unknown   0             9h
root@master01:~# kubectl  exec  -it jenkins-5c84664d4-5km8c   -n jenkins   -- /bin/bash 
jenkins@jenkins-5c84664d4-5km8c:/$ cat  /var/run/secrets/kubernetes.io/serviceaccount/token  && echo 
eyJhbGciOiJSUzI1NiIsImtpZCI6ImVERm1IejdEVnlwMXRiQmpTRjcyUFBvSnhNNUk2ZjJHS1gtUjRjSUNhYTgifQ.eyJhdWQiOlsiYXBpIiwiaXN0aW8tY2EiXSwiZXhwIjoxNzE4MjM5OTUzLCJpYXQiOjE2ODY3MDM5NTMsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2YyIsImt1YmVybmV0ZXMuaW8iOnsibmFtZXNwYWNlIjoiamVua2lucyIsInBvZCI6eyJuYW1lIjoiamVua2lucy01Yzg0NjY0ZDQtNWttOGMiLCJ1aWQiOiIzMGVhY2IxOS03ODhmLTQ2NmMtYmI1Ny0yZTk0MGFkNGI3MTMifSwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImplbmtpbnMtbWFzdGVyIiwidWlkIjoiZDQwMDAxYmQtNWNmNC00ZTNhLWFjOTEtZTNkZWYzOTkwZTY1In0sIndhcm5hZnRlciI6MTY4NjcwNzU2MH0sIm5iZiI6MTY4NjcwMzk1Mywic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmplbmtpbnM6amVua2lucy1tYXN0ZXIifQ.c8KoX_te5gggtWtLJly_PDb1sMPd3PrhbRJOO6upYqhPS2W_bjVlI9HJFwwJo5SxlL0g6bJUOtL14gn4AVit7MVzjHTpL2wI8Q7uXsr6KD4fIC-P6j8v6FQR6sY0nZhVb-y4FnWFfSHtu1LUDZpZ1vIBTPwzVqLkBtguLfoL1lbabAWbl_Mu8adqIpJDXC0JSOEfet2U5kTUMDTZiboGFwy63pf9M79JLq32BY3meDQ8iktFnRJzHgWF_r_ep85whLWMYmvafGnjYiJ5J_7HxPWrNee9BmqoUfdkYhxRnOub5vD-1FmVw4c8_gl_We8a6dqljeE0UGaeVldpF0xIAQ
```

![ks1](.\ima\ks1.PNG)

![ks2](.\ima\ks2.PNG)

![ks3](.\ima\ks3.PNG)

![ks4](.\ima\ks4.PNG)

 因为Pod Agent要与集群外部的服务建立联系，必须做dns的解析工作，可以借助于coredns实现,修改其配置文件，将解析记录传递给每个Pod

```yaml
root@master01:~/CICD-Test/coredns# cat coredns-cm.yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
            lameduck 5s
        }
        ready
        hosts {
          12.0.0.5   ks.harbor.com   gitlab.wukui.com jenkins.wukui.com 
          12.0.0.25  sonarqube.wukui.com
          fallthrough
        }
        kubernetes wukui.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
            ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
            max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system

```

Note: 当作业运行完成之后，Pod下载的内容会被删除，我们之前创建了一个pvc（基于**longhorn**）

```shell
root@master01:~/CICD-Test/deploy# cat 06-pvc-maven-cache.yaml 
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-maven-cache
  namespace: jenkins
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: longhorn
root@master01:~/CICD-Test/deploy# kubectl  get pvc  -n jenkins 
NAME              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
jenkins-pvc       Bound    pvc-dba54383-1fa9-4f66-b4d2-074bf4884084   10Gi       RWX            longhorn       14h
pvc-maven-cache   Bound    pvc-33c957b8-7985-4c98-8692-52be3b4f2d9f   10Gi       RWX            longhorn       14H
```

![ks5](.\ima\ks5.PNG)



## Argo Rollouts

blue-green:  在更新新的应用时，首先部署出新的版本，待完成之后，将旧版的流量切换至新的版本，若通过监控系统没有发现异常，这部署通过

canary: 在现有应用的pod，再增加少量的pod，并引入少量的流量，等待引入的流量看是否有错误发生

通过控制器及其CRD资源完成应用的发布

### 架构组件

![argo1](.\ima\argo1.PNG)

+ Rollout Controller

  + 负责管理Rollout CRD资源对象

+ Rollout CRD

  + 由Argo Rollout引入的自定义资源类型，与Kubernetes Deployment兼容，但具有控制高级部署方法的阶段、阈值和方法的额外字段

  + 并不会对Kubernetes Deployment施加任何影响，或要使用Rollout的功能，用户需要手动将资源从Deployment迁移至Rollout

+ Ingress/Service

  + Argo Rollouts使用标准的Kubernetes Service，但需要一些额外的元数据
  + 针对Canary部署，Rollouts支持多种不同的ServiceMesh和Ingress Controller，实现精细化的流量分割和迁移

+ AnalysisTemplate和AnalysisRun

  + Analysis是将Rollout连接至特定的Metric Provider，并为其支持的某些指标定义特定的阈值的能力，于是，这些指标的具体值将决定更新操作是否成功进行
  + 若指标查询结果满足阈值，则继续进行；若不能满足，则执行回滚；若查询结果不确定，则暂停
  + 为了执行Analysis，Argo Rollouts提供了AnalysisTemplate和AnalysisRun两个CRD

### 部署

+ 部署Argo Rollouts
  + 会生成一个services/ argo-rollouts-metrics、一个deployments/argo-rollouts，以及一组CRD

+ 部署Dashboard
+ 部署kubectl argo rollouts插件

参见官网

```shell
root@master01:~/CICD-Test# kubectl   api-versions |grep argo 
argoproj.io/v1alpha1
root@master01:~/CICD-Test# kubectl api-resources     --api-group=argoproj.io
NAME                       SHORTNAMES   APIVERSION             NAMESPACED   KIND
analysisruns               ar           argoproj.io/v1alpha1   true         AnalysisRun
analysistemplates          at           argoproj.io/v1alpha1   true         AnalysisTemplate
clusteranalysistemplates   cat          argoproj.io/v1alpha1   false        ClusterAnalysisTemplate
experiments                exp          argoproj.io/v1alpha1   true         Experiment
rollouts                   ro           argoproj.io/v1alpha1   true         Rollout

```

### 请求分析和渐进式交付

Argo Rollouts中的分析（Analysis）是用于根据阶段性交付效果的测量结果来推动渐进式交付的机制

+ 分析机制通过分析模板（AnalysisTemplate CRD）定义，而后在Rollout中调用它
+ 运行某次特定的交付过程时，Argo Rollouts会将该Rollout调用的AnalysisTemplate实例化为AnalysisRun（CRD）

具体示例的操作步骤

+ 在Ingress controller的svc添加注解

  ```
    annotations:
      prometheus.io/scrape: "true"
      prometheus.io/port: "10254"
  
  ```

+ 借助已有的prometheus监控系统，获取指标数据
+ 创建ingress service rollout AnalysisTemplate资源

### Blue-Green部署

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
  - name: service-name
  - name: namespace
  metrics:
  - name: success-rate
    # NOTE: prometheus queries return results in the form of a vector.
    # So it is common to access the index 0 of the returned array to obtain the value
    successCondition: result[0] >= 0.95
    interval: 20s
    count: 3
    failureLimit: 3
    provider:
      prometheus:
        address: http://prometheus-k8s.monitoring.svc.cluster.local:9090
        query: |monitoring
          sum(irate(nginx_ingress_controller_requests{service=~"{{args.service-name}}",namespace=~"{{args.namespace}}",status!~"[4-5].*"}[1m])) /
          sum(irate(nginx_ingress_controller_requests{service=~"{{args.service-name}}",namespace=~"{{args.namespace}}"}[1m]))
---
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: rollout-helloworld-bluegreen-with-analysis
spec:
  replicas: 3
  revisionHistoryLimit: 5
  selector:
    matchLabels:
      app: rollout-helloworld-bluegreen
  template:
    metadata:
      labels:
        app: rollout-helloworld-bluegreen
    spec:
      containers:
      - name: spring-boot-helloworld
        image: ikubernetes/spring-boot-helloworld:v0.9.2
        ports:
        - containerPort: 80
  strategy:
    blueGreen: 
      activeService: spring-boot-helloworld
      previewService: spring-boot-helloworld-preview
      prePromotionAnalysis: 
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: spring-boot-helloworld
        - name: namespace
          value: default
      postPromotionAnalysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: spring-boot-helloworld
        - name: namespace
          value: default
      autoPromotionEnabled: true
---
kind: Service
apiVersion: v1
metadata:
  name: spring-boot-helloworld
spec:
  selector:
    app: rollout-helloworld-bluegreen
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
kind: Service
apiVersion: v1
metadata:
  name: spring-boot-helloworld-preview
spec:
  selector:
    app: rollout-helloworld-bluegreen
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: spring-boot-helloworld
spec:
  ingressClassName: "nginx"
  rules:
    - host: xxxxx.xxx.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: spring-boot-helloworld
                port:
                  number: 80
---

```

## Gitops

不管是代码仓库中的应用程序代码还时部署代码，都是在同一仓库中，而Weave Cloud的DevOps方法在CD阶段使用Pull机制

+ 将配置仓库当作声明式用户期望状态的定义，把应用部署的结构当作真实状态（是利用operator来完成的）
+ 将Config Repo视为唯一可信源

### Gitops工具

#### ArgoCD

**核心概念**

Application CRD

+ 定义由ArgoCD管理的应用程序

ApplicationSet CRD

+ 以模板化形式自动生成由ArgoCD管理的应用程序
+ 支持从多个不同的角度构建模板，例如不同的Git Repo，或者不同的Kubernetes Cluster等

AppProject CRD

+ 为Application提供逻辑分组，同时提供如下功能
  + 限制可用部署的内容，例如指定受信任的Git Repository白名单
  + 限制应用程序可以部署到目标位置，包括目标集群和目标名称空间
  + 限制可以部署或者不能部署的资源类型，例如RBAC、CRD、DaemonSet等
+ 每个Application都必须隶属某个AppProject，未指定时，则隶属于名为“default”的默认项目
+ default项目可以被修改，但不能删除





