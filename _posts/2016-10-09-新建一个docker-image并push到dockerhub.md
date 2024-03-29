---
title: (Docker) 新建一个Docker Image并push到dockerhub
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [October 9, 2016](https://web.archive.org/web/20201020193709/https://void-shana.moe/linux/%e6%96%b0%e5%bb%ba%e4%b8%80%e4%b8%aadocker-image%e5%b9%b6push%e5%88%b0dockerhub.html "9:20 am") 
[VOID001](https://web.archive.org/web/20201020193709/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [0 Comment](https://web.archive.org/web/20201020193709/https://void-shana.moe/linux/%e6%96%b0%e5%bb%ba%e4%b8%80%e4%b8%aadocker-image%e5%b9%b6push%e5%88%b0dockerhub.html#respond)





Preface
-------


软件开发测试过程中的环境配置一直是令人头疼的事情，Docker正是为了解决这种问题而诞生的一套工具，现在已经发展成为了一个生态，使用Docker，可以轻松的部署你的项目，并且保证不同slave机器的环境配置完全一致，当你提交一个patch(commit)之后，想要测试项目的运行的时候， 只需要docker run <your-image>就可以轻松的搭建好项目运行环境，并且访问配置好的服务器端口，就可以访问到刚刚修改后的项目了~ 同时docker结合DroneCI TravisCI等CI平台，使得软件测试也变得十分容易 NEUOJ的健壮性测试，就是在自建的私有DroneCI服务器上运行的


但是因为之前我是直接使用的centos的镜像，没有进行修改，所以之前我们每一次运行测试，都需要下载一大堆软件包，然后才开始运行代码测试， 既费时又费流量(每一次都需要安装200MB的东西)


![hhhh](https://web.archive.org/web/20201020193709im_/https://voidisprogramer.com/wp-content/uploads/2016/10/HHHH-1024x1011.png)


因而自己新建一个Docker镜像包含这些所有的软件包能省去很多时间以及流量


新建一个Docker镜像
------------


使用docker build命令可以轻松的创建Docker Image


创建docker image需要Dockerfile, Dockerfile描述了这个image要用到的base镜像， 你可以从一个已经存在的镜像开始创建， 也可以Build From Scratch


Dockerfile的每一个指令的格式如下



```
INSTRUCTION args
```

常用的INSTRUCTION有


FROM: 表示镜像的base是什么,scratch的话表示无base


RUN: 执行Shell命令,并将结果打包到images中


CMD: docker run images的时候执行的命令


MAINTAINER: 维护者


以下是一个示例Dockerfile



```
# NEUOJ Test Script Ver1.0
FROM centos:7
MAINTAINER VOID001 <zhangjianq[[email protected]](/web/20201020193709/https://void-shana.moe/cdn-cgi/l/email-protection)>

#- uname -a
RUN yum install wget -y
RUN rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
RUN yum install git -y
RUN yum install openssh-clients -y
RUN rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm # Add php7 Source
RUN yum repolist -y

#Just Test!
RUN yum install mariadb -y

RUN yum -y install php70w.x86\_64 php70w-bcmath.x86\_64 php70w-cli.x86\_64     php70w-common.x86\_64 php70w-dba.x86\_64 php70w-devel.x86\_64 php70w-embedded.x86\_64 php70w-enchant.x86\_64 php70w-fpm.x86\_64 php70w-gd.x86\_64 php70w-imap.x86\_64 php70w-interbase.x86\_64 php70w-intl.x86\_64 php70w-ldap.x86\_64 php70w-mbstring.x86\_64 php70w-mcrypt.x86\_64 php70w-mysql.x86\_64 php70w-odbc.x86\_64 php70w-opcache.x86\_64 php70w-pdo.x86\_64 php70w-pdo\_dblib.x86\_64 php70w-pear.noarch php70w-pecl-apcu.x86\_64 php70w-pecl-apcu-devel.x86\_64 php70w-pecl-imagick.x86\_64 php70w-pecl-imagick-devel.x86\_64 php70w-pecl-xdebug.x86\_64 php70w-pgsql.x86\_64 php70w-phpdbg.x86\_64 php70w-process.x86\_64 php70w-pspell.x86\_64 php70w-recode.x86\_64 php70w-snmp.x86\_64 php70w-soap.x86\_64 php70w-tidy.x86\_64 php70w-xml.x86\_64 php70w-xmlrpc.x86\_64
```

然后 生成Docker Image的指令为


docker build -t <username>/<imagename> /path/to/your/Dockerfile/directory


e.g docker build -t void001/neuoj-test .


然后 image 就生成好啦~


将Docker镜像push到Docker Registry
-----------------------------


镜像生成好了，我们可以用 docker images来从查看 为了使我们的镜像全网络都可以用，我们需要将他push到一个docker registry上，可以理解为和github类似，只不过github托管的是code，docker registry(也叫dockerhub)托管的是镜像 docker官方的registry是 hub.docker.com 我们也可以用docker自己在本地搭建docker registry服务器，这里就不做介绍了


在dockerhub注册一个账号之后，我们在本地使用如下指令就可以登录自己的dockerhub账号


docker login


登录的时候需要提供用户名和密码 然后就会将你的credential保存在本地的一个文件里(~/.docker/config.json) 然后你就可以向docker reg 中你的repo进行推送操作了


这里注意，之前这个镜像的名字一定要为 yourname/your-image 而不能单单是 your-image 不然会导致直接推送到官方的docker lib/image下，而你是没有这个权限的，所以推送失败QAQ确定好镜像名字之后，直接使用


docker push yourname/your-image 就可以将镜像推送到docker registry了~


![hhhh](https://web.archive.org/web/20201020193709im_/https://voidisprogramer.com/wp-content/uploads/2016/10/HHHH-1-1024x422.png)


然后就可以在自己的.drone.yml里配置使用我的neuoj-test镜像作为测试用镜像了，已经预装好了所有NEUOJ运行需要的依赖，因而只需要运行测试就可以而不需要每次都安装一大堆软件包了~(镜像下载完之后，会存到本地的docker的lib下，因而只需要pull一次镜像，以后就可以很快的从本地get到这个镜像)






---


[archlinux](https://web.archive.org/web/20201020193709/https://void-shana.moe/category/linux/archlinux), [golang](https://web.archive.org/web/20201020193709/https://void-shana.moe/category/golang), [Linux](https://web.archive.org/web/20201020193709/https://void-shana.moe/category/linux) [C. Linux](https://web.archive.org/web/20201020193709/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20201020193709/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20201020193709/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20201020193709/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20201020193709/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20201020193709/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20201020193709/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20201020193709/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
Post navigation
---------------
[NEXT  
tmux 终端效率利器](https://web.archive.org/web/20201020193709/https://void-shana.moe/linux/tmux-%e7%bb%88%e7%ab%af%e6%95%88%e7%8e%87%e5%88%a9%e5%99%a8.html)
[PREVIOUS 
An Encrypted Message](https://web.archive.org/web/20201020193709/https://void-shana.moe/uncategorized/an-encrypted-message.html)

            