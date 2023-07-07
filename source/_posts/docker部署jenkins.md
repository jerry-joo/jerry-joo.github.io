---
title: docker部署Jenkins
mathjax: true
categories: docker
tags: jenkins
---
docker部署jenkins
<!--more-->
## 一.在docker中安装jenkins
本文以jenkins:2.60.3为例子

### 1.拉取jenkins镜像
```
docker pull jenkins/jenkins:2.60.3 
```
### 2.运行jenkins容器

```
docker run -it -uroot -p 15200:8080 --name jenkins_test \
-v /root/jenkins/jenkins_home:/var/jenkins_home \
-v /root/mavenrepo:/root/mavenrepo \
-v /usr/local/git:/usr/local/git \
-v /usr/bin/docker:/usr/bin/docker \
-v /var/run/docker.sock:/var/run/docker.sock \
--net mynet \
 jenkins/jenkins:2.60.3
```
说明：

```
-p 15200:8080
````
将容器内8080端口向外暴露，连接的外部端口为15200，即用户可以访问
http://<font color = 'red'>自己的服务器ip </font>:15200 来访问jenkins,这些端口都是可以自定义的。

```
-v /root/jenkins/jenkins_home:/var/jenkins_home
```
将jenkins_home挂载到本地服务器，可以直接修改本地文件同步修改容器内，不用进入容器

```
-v /root/mavenrepo:/root/mavenrepo
```
挂载maven仓库到本地，因为本文部署的项目为web端应用，涉及到java和vue，所以需要使用到maven和nodejs。

在此处挂载maven仓库的目的是为了方便找到依赖文件，如果不设置，作者也不知道下载的依赖文件会去哪。

本文使用jenkins配置maven的方法为<font color = 'red'> 代理</font>,即在jenkinsfile中设置代理镜像来达到maven打包的目的，这个方法好处是方便，不需要下载一个maven到jenkins容器中，只要提供一个配置文件即可，下文中会提到配置文件。后续nodejs也同理

```
-v /usr/local/git:/usr/local/git
```
与上面同意，将服务器的git直接挂载到jenkins容器中，免去安装git（这步的做法有待商榷，似乎jenkins自带git？）

```
-v /usr/bin/docker:/usr/bin/docker 
-v /var/run/docker.sock:/var/run/docker.sock 
```
将本地的docker环境直接映射到jenkins容器中，可以在jenkins容器中直接使用docker，<font color="red">注意</font>此处的docker就是jenkins容器使用的docker,为后续打包完成后的部署提供巨大的便利。但是小心内存不够用！


```
--net mynet
```
接入自定义的docker网络，可省略。

### 3.进入jenkins

访问http://<font color = 'red'>自己的服务器ip </font>:15200 根据步骤初始化，中途会要求密码

jenkins容器启动后会生成一个初始密码，在<font color="red">容器内</font>/var/jenkins_home/scrects/ 目录下的initilAdminPassword 文件中
可以直接进入容器加载到该目录查看
<font color="red">或者</font> 由于之前将容器的jenkins_home目录挂载到了本地/root/jenkins/jenkins_home/下，可以直接访问本地服务器的/root/jenkins/jenkins_home/scrects/来查看密码，感受到挂载数据卷的好处了吧！

初始化时最好安装插件

推荐安装

1.blue Ocean

更简洁的错误输出

2.maven Integration *必装

方便构建

3.Docker Pipeline *必装


4.nodejs *必装

前端构建运行npm指令需要

5.pipline *必装

流水线的插件

6.ssh publish

由于选择了docker部署，不需要用ssh传到服务器，可不装，再次提醒docker部署方法是直接在本机部署，注意内存消耗

7.Localization Support Plugin  &  Localization: Chinese (Simplified)

汉化

## 二.jenkins配置

### 1.maven配置

上面讲到了不需要安装maven，只需要提供一个maven的配置文件settings.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <pluginGroups>
  </pluginGroups>
  <proxies>
  </proxies>
  <servers>
  </servers>
  <mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
  </mirrors>

  <localRepository>/root/mavenrepo</localRepository>

  <profiles>
	<profile>
		<id>jdk-1.8</id>
		<activation>
			<jdk>1.8</jdk>
		</activation>
		<properties>
			<maven.compiler.source>1.8</maven.compiler.source>
			<maven.compiler.target>1.8</maven.compiler.target>
			<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
		</properties>
	</profile>
  </profiles>
</settings>
```

此配置文件采用了aliyun的镜像仓库

本地的仓库设置，与上面挂载的地址一致，注意此处对应的是容器内的目录

```
<localRepository>/root/mavenrepo</localRepository>
```

将此配置文件放到本地目录/root/jenkins/jenkins_home/<font color="red">appconfig/maven/</font>setting.xml,红色部分可以修改，因为挂载了数据卷，也就是将此配置文件放在容器/var/jenkins_home/appconfig/maven/settings.xml

完成此步可以在jenkins全局配置中配置maven pom的地址，要选择自定义路径。(不设置也没关系，jenkinsfile会出手，但是ali镜像必须设置，不然下载不到依赖)

### 2.nodejs配置

不需要jenkinsfile会出手。

## 三.jenkinsfile编写

### 1.java jenkinsfile
```
pipeline{
    agent any

    environment {
      WS = "${WORKSPACE}"
      IMAGE_NAME = "auto-schedule-ui" //自己应用的名字，下面可以引用
    }
 
    //定义流水线的加工流程
    stages {
        //流水线的所有阶段
        stage('1.环境检查'){
            steps {
               sh 'pwd && ls -alh'
               sh 'printenv'
               sh 'docker version'
               sh 'java -version'
               sh 'git --version'
            }
        }
 
        stage('2.编译'){

            //本教程关键，利用docker镜像代理maven，同时指定仓库地址
            agent {
                docker {
                    image 'maven:3-alpine'
                    args '-v maven-repository:/root/mavenrepo'
                 }
            }

            steps {
               sh 'pwd && ls -alh'
               sh 'mvn -v'
               sh 'cd ${WS} && mvn clean package -s "/var/jenkins_home/appconfig/maven/settings.xml" -Dmaven.test.skip=true'
            }
        }
 
        stage('3.打包'){
            steps {
               sh 'pwd && ls -alh'
               sh 'echo ${WS}'

                //docker 的shell脚本，用Dockerfile来构建了docker镜像，dockerfile需要手动编写 ${IMAGE_NAME}在上面
               sh 'docker build -t ${IMAGE_NAME} -f Dockerfile ${WS}/${IMAGE_NAME}/target/' 
              
            }
        }
 
        stage('4.部署'){
            
            steps {
               sh 'pwd && ls -alh'
               // 删除容器和虚悬镜像
               sh 'docker rm -f ${IMAGE_NAME} || true && docker rmi $(docker images -q -f dangling=true) || true'
               //docker创建容器 ${IMAGE_NAME}同上
               sh 'docker run -d -p 8300:8300 --net mynet --name ${IMAGE_NAME} -v /mydata/logs/${IMAGE_NAME}:/logs/${IMAGE_NAME} ${IMAGE_NAME}'
            }
        }
    }
}

```
此jenkinsfile可以作为模板

个人理解jenkinsfile就是一个脚本文件，代替人类执行重复任务，但是编写shell时必须多加检查，做好备份（作者因为删除指令，就删库跑路过，只能重新开始）


### 2.java Dockerfile
```
# FROM java:8
FROM anapsix/alpine-java:8_server-jre_unlimited
# 将当前目录下的jar包复制到docker容器的/目录下
COPY *.jar /app.jar
# 运行过程中创建一个xx.jar文件
RUN touch /app.jar
 
ENV TZ=Asia/Shanghai JAVA_OPTS="-Xms128m -Xmx256m -Djava.security.egd=file:/dev/./urandom"
ENV PARAMS="--spring.profiles.active=prod"
 
# 指定docker容器启动时运行jar包
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -jar /app.jar $PARAMS" ]
```

### 3.npm jenkinsfile
npm版,与上述类似替换stage即可
```
stage('1.环境检查'){
            steps {
               sh 'pwd && ls -alh'
               sh 'printenv'
               sh 'docker version'
               sh 'git --version'
            }
        }
 
        stage('2.编译'){
            agent {
                docker {
                    image 'node:14.15.0-alpine'
                 }
            }
            steps {
               sh 'pwd && ls -alh'
               sh 'node -v'
               sh 'cd ${WS} && npm install --registry=https://registry.npmmirror.com --no-fund && npm run build'
            }
        }
 
        stage('3.打包'){
            steps {
               sh 'pwd && ls -alh'
               sh 'docker build --build-arg PROFILE=${PROFILE} -t ${IMAGE_NAME} .'
            }
        }
 
        stage('4.部署'){
            // 删除容器和虚悬镜像
            steps {
               sh 'pwd && ls -alh'
               sh 'docker rm -f ${IMAGE_NAME} || true && docker rmi $(docker images -q -f dangling=true) || true'
               sh 'docker run -d -p 15195:80 --name ${IMAGE_NAME} --net mynet ${IMAGE_NAME}'
            }
        }

```
唯一注意点是，查看清楚项目package.json的构建指令是什么，再修改stage2中的构建脚本，比如package.json中build脚本为
```
"build-prod": "vue-cli-service build"
```
则改动为(在结尾)
```
sh 'cd ${WS} && npm install --registry=https://registry.npmmirror.com --no-fund && npm run build-prod'
```
### 4.npm Dockerfile(使用了nginx)

```
FROM nginx:alpine3.17
 
# 构建参数,在Jenkinsfile中构建镜像时定义
ARG PROFILE
 
# 将dist文件中的内容复制到 `/usr/share/nginx/html/` 这个目录下面
COPY dist/  /usr/share/nginx/html/
 
# 用本地配置文件来替换nginx镜像里的默认配置
COPY nginx/nginx-${PROFILE}.conf /etc/nginx/nginx.conf
 
# 以前台形式持续运行
CMD ["nginx", "-g", "daemon off;"]
```
需要手动创建nginx的配置文件，例如此处从上面的jenkinsfile中得知PROFILE=prod，需要创建nginx/nginx-prod.conf文件夹及文件