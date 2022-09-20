### git三种状态

已提交（committed）文件已保存在数据库中

已修改（modified）修改了文件，还没有保存到数据库中

已暂存（staged）



cat +文件名  查看文件内容

git --version 查看版本

git --config list查看所有配置

git status 查看暂存区的状态

git log  查看日志

git reflog   查看历史记录

 git log -5 --pretty=oneline  输出5行，1行1行输出

git reset HEAD 撤销将文件加入暂存区



git init 初始化

git add +文件名     提交文件到暂存区

git commit -m '提示信息'   提交文件到git本地库



git reset --hard HEAD^  回退到上一个版本

git reset --hard HEAD^^   回退两个版本

git reset --hard HEAD~1  往回退一个版本

git reset --hard +版本标识符  跳到某一个版本



 git checkout -- +文件名 从本地库拉取文件到工作区

git rm +文件名  将文件从本地仓库中删除

 git ls-files  查看本地仓库的文件



git remote add origin + 远程仓库地址   绑定远程仓库并取别名

git push -u origin master  将本地文件推送到远程仓库



再次上传时

git add

git commit -m '提示信息' 

git push -u origin master



git remote rm origin 删除远程仓库地址





分支操作

git checkout + 分支名

git checkout -b + 分支名  创建并切换到新的分支

git branch-d + 分支名  删除指定的分支

git merge + 分支名

git branch 查看所有分支（带*的为当前所在分支）

git branch -m 旧的分支名  新的分支名





分支push与pull操作

git branch -a    查看本地和远程分支

git push origin 分支名   推送本地分支到远程

git push origin :+远程分支名   删除远程分支（本地分支还在）

git checkout -b 本地分支名 origin/远程分支名  (拉取远程指定分支并在本地创建分支)



git fetch读取远程仓库的最新状态





多人协同操作冲突

每次push之前应该先pull最新版本项目，再push



标签

git tag

git tag 标签名

git tag -a 标签名 -m '描述信息'

git push origin 标签名  将标签推送到远程仓库





