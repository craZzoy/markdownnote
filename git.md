什么是git：版本管理工具

**git和svn的区别**

svn属于CVCSs Centralized，即集中式的代码管理工具，最新的版本靠远端维护，用户抓取时只抓取最新的记录。

![img](E:\note_files\sina3479701051\bb27f3c3f5c348b885737b41695be64e\clipboard.png)

如图，version1中有三个文件，当改变A文件并同步到远端后，SVN Server会产生一个新的版本Version2，但version2只存储A文件的变化。

git属于DVCSs Distributed，即分布式的代码管理工具，最新的版本的snapshot在每个用户中都有维护，版本的提交记录每个用户都有维护。

![img](E:\note_files\sina3479701051\3ca824db46ca4ee5b5e9634464d4431a\clipboard.png)

如图，version1中有三个文件，当改变A文件并同步到远端后，git Server会产生一个新的版本version2，version2对于A新创建了新的版本A1，没有更改的B和C，只新增ref引用。

区别：

- 架构不一样，git是分布式的，svn是集中式的
- server端维护版本方式不一样，git保存修改的文件和未修改文件的引用。svn只保存文件的修改
- git比较安全，server端宕机了也能继续工作，代码不易丢失。而svn server端宕机后代码遗失的几率较高，要保证svn server的安全可以使用集群。

**git的架构和状态**

**架构**

git包含工作区、暂存区、本地仓库和远端仓库。文件新建时在工作区，执行add命令后加入暂存区，commit后加入本地仓库。

![img](E:\note_files\sina3479701051\9d6dae6b11fe4da999b7b8789b09001a\clipboard.png)

**状态**

如图，新建的文件起始状态为untracked，add后状态为staged，然后commit后为unmodified，

然后修改文件后变为modified，然后commit后变为unmodifined，依次类推

![img](E:\note_files\sina3479701051\3427e487cd8e4701bd2aedf297490449\clipboard.png)

**安装**

a)    git config –-global user.name ‘xx’

b)    git config –-global user.email ‘xx’

c)    ssh-keygen -t rsa -C '1510134048@qq.com'

**常用命令**

- git init： 将文件夹加入git管理
- git status：查看状态
- git fetch --help 查看帮助
- git config 

vim ~/.gitconfig 修改配置

1. git config --list     查看配置
2. 增：git config --global --add configName configValue
3. 删：git config  --global --unset configName  (只针对存在唯一值的情况)
4. 改：git config --global configName configValue
5. 查：git config --global configName

- git clone：从远端下载项目，如： git@github.com:craZzoy/hello-world.git
- remote：把项目推送到远端

如将一个文件加到远端仓库：

git remote add origin git@github.com:craZzoy/test.git

git push -u origin master

git push -u origin dev-1128：将本地创建的分支push

- fetch/pull/push：fetch查看远端快照，pull拉取远端代码，push代码推送远端。
- checkout：切换分支，注意未提交的内容会丢失。

checkout -b name (创建新分支)

- git merge：合并分支内容，如git merge dev
- git log：查看日志
- git stash：出栈，入栈。不建议使用
- merge/rebase

merge合并代码

一个merge操作：

![img](E:\note_files\sina3479701051\9720e0bb61044b34869a9b5edc8c4d8c\clipboard.png)

如图，把master分支的v2版本checkout到dev分支后，dev做了修改，即v3版本merge到master时，master会生成一个新版本v4，它指向v3和v2。

一个rebase操作：

![img](E:\note_files\sina3479701051\3aa6d4c467e9411cb972eb233c87ab02\clipboard.png)

- git tag：打标签，方便识别
- git alias

git config --list

alias.ac=!git add -A && git commit -m

场景一：C回滚到B，修改后再提交到server，产生新的C版本

![img](E:\note_files\sina3479701051\18f6b63d163648cd9339e93c6041969f\clipboard.png)

1. git reset --hard cmmitid（回退到某个版本）
2. git pull -f  origin master (同步到远端，git pull不行，使用force pull。)

场景二：把多次提交只用一个commitid

git stash（出栈，入栈，不建议使用）

git commit --amend(减少commitid，比如你在做一件事做到一半要切换分支时须要commit一次，做完之后再提交一次，但你想只有一个commitid)

**git flow项目管理**

合适才是最好的，根据实际项目去搭建模型，例子：

![img](E:\note_files\sina3479701051\03af816df412405085c9a7c01771c8a4\clipboard.png)

**git hooks**

可用于自动部署



切换分支临时保存修改

- 保存：git stash或git stash save '修改的信息'
- 拉取记录：
  - git stash pop
  - git stash list，然后git stash apply stash@{0}（获取指定保存栈的内容）