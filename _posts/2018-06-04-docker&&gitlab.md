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





