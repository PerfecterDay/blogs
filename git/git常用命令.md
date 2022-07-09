# git 常用命令
{docsify-updated}

### Github 配置 SSH 登录
1. 打开 git bash，非 windows 下不需要
2. 查看是否已经有 SSH keys： `ls -la ~/.ssh`
3. 生成 SSH keys: `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
4. 启动 ssh-agent：eval `ssh-agent -s`
5. 添加 key 到 ssh-agent: `ssh-add ~/.ssh/id_ed25519`
6. 将生成的 SSH 公钥(id_rsa.pub)粘贴到GitHub上
7. 测试能否连接：`ssh -T git@github.com`

### git 常用命令
<center><img src="pics/git-frame.jpg" width="30%" /></center>

1. `git status [-s]` : 查看 git 状态，-s 代表 --short
2. `git diff`: 比较的是工作目录中当前文件和暂存区域快照之间的差异。 也就是修改之后还没有暂存起来的变化内容。
3. `git diff --staged`: 比对已暂存文件与最后一次提交的文件差异。
4. `git clean`: 删除工作区中未跟踪（不在版本库也未索引的）的文件
5. `git add`: 这是个多功能命令，可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。将这个命令理解为“精确地将内容添加到下一次提交中”而不是“将一个文件添加到项目中”要更加合适精确地将内容添加到下一次提交中
6. `git ls-files --satge`:查看暂存区的文件
7. `git diff` : 只显示尚未暂存的改动，而不是自上次提交以来所做的所有改动
8. `git diff --staged `:比对已暂存文件与最后一次提交的文件差异
9. `git rm filename` : 删除某个文件，提交后会同时删除版本库和工作区的文件
10. `git rm --cached filename`:从暂存区和版本库中删除某个文件，提交后，版本库中会删除这个文件，但是工作区磁盘上会保留
11. `git checkout head filename`:从暂存区恢复工作区中的文件，通常是想放弃对工作区中的修改或者误删了工作区文件
12. `git checkout <hash> <filename>`：恢复文件到某个提交状态
13. `git reset [HEAD|<h></h>ash] filename`:从版本库中恢复某个文件，当你删除了工作区中的文件，又删除了暂存区中的文件，想恢复文件可以用这个命令
14. `git commit`:将暂存区中的文件提交到版本库
15. `git commit --amend`：将本次提交与前一次的提交合并为一次提交
16. `git checkout branchname`:切换分支
17. `git checkout -b branchname`:切换分支,如果分支名不存在则基于当前分支新建一个分支
18. `git reset filename`:从版本库中还原某个文件到暂存区，当想撤销某次git add filename操作时使用，不加filename时，重置整个暂存区，工作区内容不受影响
19. `git reset`:恢复暂存区文件为版本库状态,工作区文件不会改变
20. `git reset --hard head~1`:用上一次提交的版本库状态恢复暂存区和工作区
21. `git log`: 查看提交历史记录
22. `git log filename`: 查看某个文件的历史提交记录
23. `git show <commit_id>`: 查看某次提交的信息信息
24. `git reflog`:查看引用历史记录
25. `git reset ref`:将版本库恢复到某次提交状态，适用于往前reset后又想往后reset
26. `git stash`:将当前工作区中的修改保存起来,并用版本库中的内容恢复暂存区和工作区，当开发到一半时，需要处理另一个bug时使用
27. `git stash pop`:将保存的内容恢复到工作区

### git 分支
0. `git checkout -b 本地分支名x origin/远程分支名x `: 在本地新建分支x，并自动切换到该本地分支x
1. `git branch -a`:查看所有分支
2. `git branch --merged|--no-merged`:查看已经合并到/尚未合并当前分支的分支
3. `git branch <BranchName>`:创建 BranchName 分支
4. `git checkout <BranchName>`:切换到 BranchName 分支
5. `git checkout -b <BranchName>`:在当前分支基础上新建 BranchName 分支，并切换到 BranchName 分支
6. `git brnch -d <BranchName>`:删除 BranchName 分支
7. `git merge <BranchName>`: 将 BranchName 合并到当前分支
8. `git merge --abort`: 中断合并
9. `git rebase <targetBranch> <sourceBranch>`: Rebase 实际上就是取出 sourceBranch 的一系列的提交记录，“复制”它们，然后在targetBranch 的后边逐个的放下去，创造更线性的提交历史。sourceBranch 省略时，表示将当前分支 rebase到 targetBranch。

### git远程命令
1. `git clone`: 从远程库克隆
2. `git remote -vv` :显示远程主机
3. `git remote show <仓库名>` :显示远程主机详细信息
4. `git remote add <仓库名> <仓库地址>`:添加远程主机
5. `git remote rm <仓库名>`：删除远程主机
6. `git remote rename <原仓库名> <新仓库名>`：远程主机的改名
7. `git remote set-url <仓库名> <新的repo-url>`：修改远程仓库的地址
8. `git fetch <仓库名>`：访问远程仓库，从中拉取所有你本地仓库还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看
9. `git fetch origin 远程分支名x:本地分支名x` : 在本地新建分支x，但是不会自动切换到该本地分支x，需要手动checkout
10. `git fetch <仓库名> <分支名>`：取回远程主机指定分支的更新
11. `git pull <仓库名> <远程分支名>:<本地分支名>`：取回远程主机某个分支的更新，再与本地的指定分支合并
12. `git pull <仓库名> <远程分支名>`：取回远程主机某个分支的更新，再与本地的当前分支合并
13. `git branch --set-upstream <本地分支名> <仓库名>/<远程分支名>` ：手动建立追踪关系
14. `git pull origin` : 如果当前分支与远程分支存在追踪关系，git pull就可以省略远程分支名
15. `git pull` :如果当前分支只有一个追踪分支，连远程主机名都可以省略
16. `git push remote localbranch:remotebranch` : 将本地localbranch分支推送到remote的remotebranch分支上
17. `git push remote localbranch` : 将本地localbranch分支推送到remote的同名分支上
18. `git push remote :remotebranch` :删除remote的remotebranch分支
19. `git push remote --delete remotebranch` :删除remote的remotebranch分支
20. `git push` : 如果当前分支只有一个追踪分支，那么主机名都可以省略

#### 常见问题
1. windows乱码：  
    使用git bush时，使用git add XX 添加文件后，git status 发现中文文件名是数字形式，比如"\123\456\789.txt"，点击也无法打开，二使用ls，git log都可以显示中文，最后修改配置:
    `git config --global core.quotepath false ` 解决



