# Git ：Git Command

## 前提摘要

1. 个人说明：

   - **限于时间紧迫以及作者水平有限，本文错误、疏漏之处恐不在少数，恳请读者批评指正。意见请留言或者发送邮件至：“[Email:noahpanzzz@gmail.com](noahpanzzz@gmail.com)”**。
   - **本博客的工程文件均存放在：[GitHub:https://github.com/panziping](https://github.com/panziping)。**
   - **本博客的地址：[CSDN:https://blog.csdn.net/ZipingPan](https://blog.csdn.net/ZipingPan)**。
2. 参考：

   - [Git 教程 - 廖雪峰](https://www.liaoxuefeng.com/wiki/896043488029600)

---

## 正文

```shell
#GIT命令

#下述是本地仓库操作
#创建learngit文件夹
mkdir learngit 					
#进入learngit文件夹
cd learngit						
#显示当前文件夹路径
pwd								
#将当前目录变成Git可以管理的仓库
git init						
#创建readme.txt文件
touch readme.txt	

#将readme.txt文件添加到仓库
git add readme.txt
#将readme.txt文件提交到仓库
git commit -m "wrote a readme file"
#1.查看仓库当前状态。（长时间未更新仓库）2.也可以用在git add之后，查看是哪些文件将要被提交的修改。3.提交完毕，查看当前仓库是否还有未被提交的。
git status
#显示所有内容
git status -uall
#比较文件差异
git diff
#显示从最近到最远的提交日志
git log
#将log信息省略成一行（oneline）
git log --pretty=oneline
#将当前版本回退到上一个版本
git reset --hard HEAD^
#将当前版本回到指定版本（版本号）
git reset --hard f8ce
#查看文件gittest.txt内容
cat gittest.txt
#查看记录操作的历史命令，可以用来查看版本号
git reflog

#丢弃工作区的修改，把readme.txt文件在工作区的修改全部撤销
#1. readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
#2. readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
git checkout -- readme.txt

#将暂存区的修改撤销掉（unstage），重新放回工作区
git reset HEAD readme.txt
#删除文件
rm test.txt




#下述是远程仓库操作

#创建SSH Key
ssh-keygen -t rsa -C "youremail@example.com"

#把本地仓库的内容推送到GitHub仓库
git remote add origin git@github.com:panziping/BLOG.git
#把本地库的所有内容推送到远程库（第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。）
git push -u origin master
#查看远程库信息
git remote -v
# 添加的时候地址写错了，或者就是想删除远程库（此处的“删除”其实是解除了本地和远程的绑定关系，并不是物理上删除了远程库）。
git remote rm origin


#下述是分支管理

#创建dev分支，然后切换到dev分支
git checkout -b dev
#创建dev分支，然后切换到dev分支,拆解成两条命令
git branch dev
git checkout dev
#查看当前分支
git branch

#切换回master分支
git checkout master

#将dev分支合并到master
git merge dev

#删除dev分支
git branch -d dev


#创建并切换到新的dev分支,区别于checkout
git switch -c dev
#切换到已有的master分支
git switch master
```



```shell
#关闭自动转换，解决:LF将被readme.txt中的CRLF替换。
git config --global core.autocrlf false    
#解决git status不能显示中文
git config --global core.quotepath false
```



## 总结



---

**本文均为原创，欢迎转载，请注明文章出处：[CSDN:https://blog.csdn.net/ZipingPan/编程语言](https://blog.csdn.net/zipingpan/category_12627795.html)。百度和各类采集站皆不可信，搜索请谨慎鉴别。技术类文章一般都有时效性，本人习惯不定期对自己的博文进行修正和更新，因此请访问出处以查看本文的最新版本。**

**非原创博客会在文末标注出处，由于时效原因，可能并不是原创作者地址（已经无法溯源）。**

