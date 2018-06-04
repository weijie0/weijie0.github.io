---
layout: post
title: "Docker部署GitLab"
date: 2018-05-25 
description: "Docker部署GitLab"
tag: 博客 
---   
<div align="center">
	<img src="/images/posts/cmd/docker.png" height="300" width="500">  
</div> 
> GitLab是类似于Github的开源Git版本管理平台，功能强大且界面美观，通常用于搭建私有代码仓库，是目前公司内部Git版本管理系统的首选方案。
> 
> 对于一个版本管理系统而言，如何去部署、升级、备份和迁移是一个必须考虑的问题，而Docker由于其容器特性，非常符合我们的需求，最重要的是它不会对现有服务器环境造成任何影响，这里选择Docker的方式部署GitLab



### 一. 确保机器上安装了docker并启动

```
# 安装docker
yum install docker
# 启动docker
systemctl start docker
```



> 安装完成后建议配置加速器(比如阿里云镜像加速), 否则拉取镜像会非常慢, 特别是gitlab镜像, 还是非常大的



### 二. 拉取镜像并启动

```
#拉取镜像
docker pull gitlab/gitlab-ce
# 启动
docker run --detach \
     --hostname 192.168.133.128 \
     --publish 8443:443 --publish 9001:80 --publish 2222:22 \
     --name gitlab \
     --memory 4g \
     --restart always \
     --volume /home/gitlab/config:/etc/gitlab \
     --volume /home/gitlab/logs:/var/log/gitlab \
     --volume /home/gitlab/data:/var/opt/gitlab \
     docker.io/gitlab/gitlab-ce
```
> --host 配置你宿主机的地址
> 
> --name 配置你的容器名称
> 
> --publish 暴露了容器的三个端口, 分别是https对应的443， http对应80以及ssh对应的22(如果不需要配置https, 可以不暴露)
>
> --memory 限制容器最大内存暂用4G, 这是官方推荐的
> 
> --volume 指定挂载目录, 这个便于我们在本地备份和修改容器的相关数据

### 三. 修改配置文件并重启

```
# 打开挂载的配置目录
vi /srv/gitlab/config/gitlab.rb

###################################################
# 添加外部请求的域名(如果不支持https, 可以改成http)
external_url 'https://gitlab.yinnote.com'
# 修改gitlab对应的时区 
gitlab_rails['time_zone'] = 'PRC'
# 开启邮件支持 
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'gitlab@yinnote.com'
gitlab_rails['gitlab_email_display_name'] = 'Yinnote GitLab'
# 配置邮件参数
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.mxhichina.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "gitlab@yinnote.com"
gitlab_rails['smtp_password'] = "xxxxxx"
gitlab_rails['smtp_domain'] = "yinnote.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false        
###################################################
```

(选配) 如果配置了https, 需要导入证书


```
# 进入挂载配置目录
cd /srv/gitlab/config
# 创建密钥文件夹, 并放入证书
mkdir ssl
# 内容如下
```


重启服务


```
# 方法一: 重启容器(其中xxxxxx是容器id)
docker restart xxxxxx

# 方法二: 登陆容器, 重启配置
docker exec -it  xxxxxx bash   
gitlab-ctl reconfigure
```

> 1、通过ssh方式拉取代码时, 记住端口号是2222, 不是默认的22
> 
> 2、如果没有配置https, 是无法通过https路径拉取代码的
> 
### 四.访问host：port
说明：启动gitlab，页面显示502，原因可能gitlab需要内存至少2g，最好大于2g。还可能是端口号冲突导致。

```
#打印gitlab日志
docker logs - f gitlab
```
##  附：git命令

```
git init                                                  # 初始化本地git仓库（创建新仓库）
git config --global user.name "xxx"                       # 配置用户名
git config --global user.email "xxx@xxx.com"              # 配置邮件
git config --global color.ui true                         # git status等命令自动着色
git config --global color.status auto
git config --global color.diff auto
git config --global color.branch auto
git config --global color.interactive auto
git clone git+ssh://git@192.168.53.168/VT.git             # clone远程仓库
git status                                                # 查看当前版本状态（是否修改）
git add xyz                                               # 添加xyz文件至index
git add .                                                 # 增加当前子目录下所有更改过的文件至index
git commit -m 'xxx'                                       # 提交
git commit --amend -m 'xxx'                               # 合并上一次提交（用于反复修改）
git commit -am 'xxx'                                      # 将add和commit合为一步
git rm xxx                                                # 删除index中的文件
git rm -r *                                               # 递归删除
git log                                                   # 显示提交日志
git log -1                                                # 显示1行日志 -n为n行
git log -5
git log --stat                                            # 显示提交日志及相关变动文件
git log -p -m
git show dfb02e6e4f2f7b573337763e5c0013802e392818         # 显示某个提交的详细内容
git show dfb02                                            # 可只用commitid的前几位
git show HEAD                                             # 显示HEAD提交日志
git show HEAD^                                            # 显示HEAD的父（上一个版本）的提交日志 ^^为上两个版本 ^5为上5个版本
git tag                                                   # 显示已存在的tag
git tag -a v2.0 -m 'xxx'                                  # 增加v2.0的tag
git show v2.0                                             # 显示v2.0的日志及详细内容
git log v2.0                                              # 显示v2.0的日志
git diff                                                  # 显示所有未添加至index的变更
git diff --cached                                         # 显示所有已添加index但还未commit的变更
git diff HEAD^                                            # 比较与上一个版本的差异
git diff HEAD -- ./lib                                    # 比较与HEAD版本lib目录的差异
git diff origin/master..master                            # 比较远程分支master上有本地分支master上没有的
git diff origin/master..master --stat                     # 只显示差异的文件，不显示具体内容
git remote add origin git+ssh://git@192.168.53.168/VT.git # 增加远程定义（用于push/pull/fetch）
git branch                                                # 显示本地分支
git branch --contains 50089                               # 显示包含提交50089的分支
git branch -a                                             # 显示所有分支
git branch -r                                             # 显示所有原创分支
git branch --merged                                       # 显示所有已合并到当前分支的分支
git branch --no-merged                                    # 显示所有未合并到当前分支的分支
git branch -m master master_copy                          # 本地分支改名
git checkout -b master_copy                               # 从当前分支创建新分支master_copy并检出
git checkout -b master master_copy                        # 上面的完整版
git checkout features/performance                         # 检出已存在的features/performance分支
git checkout --track hotfixes/BJVEP933                    # 检出远程分支hotfixes/BJVEP933并创建本地跟踪分支
git checkout v2.0                                         # 检出版本v2.0
git checkout -b devel origin/develop                      # 从远程分支develop创建新本地分支devel并检出
git checkout -- README                                    # 检出head版本的README文件（可用于修改错误回退）
git merge origin/master                                   # 合并远程master分支至当前分支
git cherry-pick ff44785404a8e                             # 合并提交ff44785404a8e的修改
git push origin master                                    # 将当前分支push到远程master分支
git push origin :hotfixes/BJVEP933                        # 删除远程仓库的hotfixes/BJVEP933分支
git push --tags                                           # 把所有tag推送到远程仓库
git fetch                                                 # 获取所有远程分支（不更新本地分支，另需merge）
git fetch --prune                                         # 获取所有原创分支并清除服务器上已删掉的分支
git pull origin master                                    # 获取远程分支master并merge到当前分支
git mv README README2                                     # 重命名文件README为README2
git reset --hard HEAD                                     # 将当前版本重置为HEAD（通常用于merge失败回退）
git rebase
git branch -d hotfixes/BJVEP933                           # 删除分支hotfixes/BJVEP933（本分支修改已合并到其他分支）
git branch -D hotfixes/BJVEP933                           # 强制删除分支hotfixes/BJVEP933
git ls-files                                              # 列出git index包含的文件
git show-branch                                           # 图示当前分支历史
git show-branch --all                                     # 图示所有分支历史
git whatchanged                                           # 显示提交历史对应的文件修改
git revert dfb02e6e4f2f7b573337763e5c0013802e392818       # 撤销提交dfb02e6e4f2f7b573337763e5c0013802e392818
git ls-tree HEAD                                          # 内部命令：显示某个git对象
git rev-parse v2.0                                        # 内部命令：显示某个ref对于的SHA1 HASH
git reflog                                                # 显示所有提交，包括孤立节点
git show HEAD@{5}
git show master@{yesterday}                               # 显示master分支昨天的状态
git log --pretty=format:'%h %s' --graph                   # 图示提交日志
git show HEAD~3
git show -s --pretty=raw 2be7fcb476
git stash                                                 # 暂存当前修改，将所有至为HEAD状态
git stash list                                            # 查看所有暂存
git stash show -p stash@{0}                               # 参考第一次暂存
git stash apply stash@{0}                                 # 应用第一次暂存
git grep "delete from"                                    # 文件中搜索文本“delete from”
git grep -e '#define' --and -e SORT_DIRENT
git gc
git fsck
```




