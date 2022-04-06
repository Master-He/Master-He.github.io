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
```

