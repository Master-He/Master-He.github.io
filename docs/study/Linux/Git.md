

# github 常用技巧



搜索内容含有"PackageReference"， 且文件名正则是*.vbproj的文件

```shell
path:/^*.vbproj$/ PackageReference
path:/^*\.xproj$/ PackageReference

path:/^*\.nuspec$/
```







git 常用命令

```shell
git checkout -b dev origin/dev # 切换到dev分支，同时追踪远程分支
git checkout dev # 切换到dev分支，同时追踪远程分支

查看差异

# 查看git 配置
# 查看系统config
git config --system --list
# 查看当前用户（global）配置
git config --global --list
# 查看当前仓库配置信息
git config --local --list
# 修改全部配置
git config --global --edit

# 配置用户名和邮箱
git config --global user.name "Wenji He"
git config --global user.email "hewj3@lenovo.com"
# git config --global user.name "Master-He"
# git config --global user.email "478243824@qq.com"

# 取消合并
git merge --abort

# git stash命令
git stash save 'comment message' # 保存当前进度，并备注信息
git stash list  # 列出所有的进度
git stash pop # 弹出最后一次保存的进度（删除进度）
git stash pop stash@{1} # 弹出指定的进度（删除进度）
git stash apply # 应用最后一次保存的进度，不删除进度
git stash apply stash@{1} # 弹出指定的进度（删除进度）
git stash drop stash@{1}
git stash clear # 清空进度

# 撤销全部修改
git checkout .


# git仓库里面可以嵌套一个git仓库
比如https://github.com/HIT-SCIR/pyltp里面就嵌套着https://github.com/HIT-SCIR/ltp/tree/f414a9b4bbdc44b355376821a3ee849e264bef16

这个时候源码安装就需要这么做：
$ git clone https://github.com/HIT-SCIR/pyltp
$ cd pyltp
$ git submodule init
$ git submodule update
$ python setup.py install


git rebase 

```



### github连接问题

有一天突然git pull没有用， git clone， ssh 连不上github了

```shell
Admin@PS2020TQTPXTCF MINGW64 /d/Document
$ git clone git@github.com:Master-He/Master-He.github.io.git
Cloning into 'Master-He.github.io'...
ssh: connect to host github.com port 22: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

Admin@PS2020TQTPXTCF MINGW64 /d/Document
$ ssh -T git@github.com
ssh: connect to host github.com port 22: Connection timed out

```

解决办法

在C:\Users\Admin\\.ssh加一个config文件，config文件内容为

```
Host github.com
 Hostname ssh.github.com
 Port 443
```

然后问题就解决了



### github拒绝连接

![image-20220524212855978](Git.assets/image-20220524212855978.png)

去gitee 搜索github520项目，然后修改hosts文件就可以解决了



### git  push 报错

![image-20220524220654156](Git.assets/image-20220524220654156.png)

清空之前在C:\Users\Admin\\.ssh加的config文件内容， 让config变成空文件， 就可以正常提交了！ 我被搞晕了。。。





## 忽略不想提交的文件

参考： https://www.cnblogs.com/wt645631686/p/10007328.html



```shell
git update-index --assume-unchanged sp_edaijia/protected/controllers/ApiController.php  //这里忽略ApiController.php 文件
```



```shell
git update-index --no-assume-unchanged   sp_edaijia/protected/controllers/ApiController.php  //恢复跟踪
```

