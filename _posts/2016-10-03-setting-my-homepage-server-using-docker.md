---
layout: post
title:  "Docker 활용기 - 개인 홈페이지 서버 환경 구축"
date:   2016-10-03 17:30:00 +0900
tags: [docker]
---

기존에 AWS에 올려두었던 개인 포트폴리오 홈페이지 서버를 구글 클라우드를 공부할 겸 옮기기로 결정했다. 막상 옮기려니 오래전에 셋팅해두었던 서버여서 잘 기억도 나지 않고 당시 가이드 문서도 상세히 적어놓지 않아서 새로 구축하기가 번거로웠다. 그래서 이 참에 docker 환경으로 구성해서 다음번에 이전을 할 일이 생기더라도 간편하게 서버를 구축할 수 있도록 하기로 마음먹게 되었다.


## Docker 이미지 만들기

먼저 nginx를 제외하고 tomcat과 jenkins에 대해서만 생각을 하기로 하고 설계를 했다.

![](http://yonghochoi.github.io/images/docker/homepage_architect.png)

먼저 홈페이지 서버와 jenkins 서버를 docker 이미지로 만든 후 Docker Hub의 내 개인 계정에 push를 했다. 이 과정에서 jenkins 서버 이미지를 만들며 삽질을 많이 했었다. Docker hub에 official로 올라가 있는 jenkins를 받아서 사용을 했었는데,  jenkins에 item을 만들어서 설정을 완료하고 commit을 하여 새로운 이미지를 만들면 다음 번에 이 이미지를 사용하여 컨테이너를 생성했을 때 내용이 그대로 남아있을 것이라고 생각했는데 초기화 되는 것이었다.

 이 문제 때문에 몇 일간 삽질을 했다. Dockerfile을 자세히 들여다보니 jenkins의 홈 디렉토리인 /var/jenkins_home 디렉토리가 VOLUME으로 지정되어 있었다. 볼륨으로 사용할 경우 호스트의 디렉토리와 매핑이 되기 때문에 컨테이너의 변경 사항이 반영되지 않는다. 그래서 내린 결론은 ubuntu 서버에 직접 톰캣과 젠킨스를 wget으로 받아서 서버를 구축하고 이미지로 만들자는 것이었다.


* Jenkins 서버의 Dockerfile

  {% highlight shell %}
  FROM ubuntu:14.04

  MAINTAINER Yongho Choi <yongho1037@gmail.com>

  ENV TOMCAT_VERSION 8.0.37

  # Set locales
  RUN locale-gen en_GB.UTF-8
  ENV LANG en_GB.UTF-8
  ENV LC_CTYPE en_GB.UTF-8

  # Fix sh
  RUN rm /bin/sh && ln -s /bin/bash /bin/sh

  # Install dependencies
  RUN apt-get update && \
  apt-get install -y git build-essential curl wget software-properties-common

  # Install JDK 8
  RUN \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  add-apt-repository -y ppa:webupd8team/java && \
  apt-get update && \
  apt-get install -y oracle-java8-installer wget unzip tar && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /var/cache/oracle-jdk8-installer

  # Define commonly used JAVA_HOME variable
  ENV JAVA_HOME /usr/lib/jvm/java-8-oracle

  # Get Tomcat
  RUN wget --quiet --no-cookies http://apache.rediris.es/tomcat/tomcat-8/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz -O /tmp/tomcat.tgz && \
  tar xzvf /tmp/tomcat.tgz -C /opt && \
  mv /opt/apache-tomcat-${TOMCAT_VERSION} /opt/tomcat && \
  rm /tmp/tomcat.tgz && \
  rm -rf /opt/tomcat/webapps/examples && \
  rm -rf /opt/tomcat/webapps/docs && \
  rm -rf /opt/tomcat/webapps/ROOT

  # Add admin/admin user
  # ADD tomcat-users.xml /opt/tomcat/conf/

  ENV CATALINA_HOME /opt/tomcat
  ENV PATH $PATH:$CATALINA_HOME/bin

  # jenkins download
  RUN wget --quiet --no-cookies http://mirrors.jenkins-ci.org/war-stable/latest/jenkins.war -O /${CATALINA_HOME}/webapps/jenkins.war

  EXPOSE 8080
  EXPOSE 8009

  WORKDIR /opt/tomcat

  # Launch Tomcat
  CMD ["/opt/tomcat/bin/catalina.sh", "run"]
  {% endhighlight %}


* 홈페이지(tomcat) 서버의 Dockerfile

  {% highlight shell %}
  FROM ubuntu:14.04                                                               
  MAINTAINER Yongho Choi <yongho1037@gmail.com>                                                                                                                
  ENV TOMCAT_VERSION 8.0.37                                                                                                                                    
  # Set locales                                                    
  RUN locale-gen en_GB.UTF-8                                     
  ENV LANG en_GB.UTF-8                                            
  ENV LC_CTYPE en_GB.UTF-8                                                                                                                                    
  # Fix sh                                                        
  RUN rm /bin/sh && ln -s /bin/bash /bin/sh                                                                                                                  
  # Install dependencies                                          
  RUN apt-get update && \                                          
  apt-get install -y git build-essential curl wget software-properties-common                                                                                
  # Install JDK 8                                                  
  RUN \                                                            
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \                                                    
  add-apt-repository -y ppa:webupd8team/java && \                  
  apt-get update && \                                              
  apt-get install -y oracle-java8-installer wget unzip tar && \    
  rm -rf /var/lib/apt/lists/* && \                                
  rm -rf /var/cache/oracle-jdk8-installer                          

  # Define commonly used JAVA_HOME variable                        
  ENV JAVA_HOME /usr/lib/jvm/java-8-oracle                       

  # Get Tomcat                                                                
  RUN wget --quiet --no-cookies http://apache.rediris.es/tomcat/tomcat-8/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz -O /tmp/tomcat.tgz && \
  tar xzvf /tmp/tomcat.tgz -C /opt && \                                                                                                                        
  mv /opt/apache-tomcat-${TOMCAT_VERSION} /opt/tomcat && \                                     
  rm /tmp/tomcat.tgz && \                                                                    
  rm -rf /opt/tomcat/webapps/examples && \                                                   
  rm -rf /opt/tomcat/webapps/docs && \                                                       
  rm -rf /opt/tomcat/webapps/ROOT                                                            

  ENV CATALINA_HOME /opt/tomcat                                                               ENV PATH $PATH:$CATALINA_HOME/bin                                                           

  EXPOSE 8080                                                                                
  EXPOSE 8009                                                                                 

  WORKDIR /opt/tomcat                                                                         

  # Launch Tomcat                                                                 
  CMD ["/opt/tomcat/bin/catalina.sh", "run"]
  {% endhighlight %}


## docker-compose 활용

이 후 두 개의 컨테이너를 따로 구동 시키지 않고 한번에 관리하기 위해 docker-compose를 사용하였다. 생각보다 쉽게 구성을 할 수 가 있었는데, docker-compose.yml만 작성하면 여러 개의 docker 컨테이너를 구동시킬 수가 있고, docker 간 통신을 위한 links를 설정할수도 있다.



* docker-compose.yml

  {% highlight shell %}
  version: '2'                 
  services:                    
    jenkins:                   
      image: yonghochoi/jenkins
      ports:                   
       - "8080:8080"           
      links:                   
       - tomcat                
    tomcat:                    
      image: yonghochoi/tomcat
      ports:                   
       - "80:8080"
       - "2222:22"
  {% endhighlight %}



처음에는 image 대신 build를 사용하여 Dockerfile로 컨테이너를 생성했었는데, Dockerfile을 따로 보관해야 하는 불편함이 있어서 Docker Hub에 올린 뒤 image를 사용해서 컨테이너를 생성했다.  

여기서 발생한 이슈는 jenkins 설정 정보를 이미지화 하면 이 이미지에 계정정보와 배포하려는 서버의 각종 정보들이 그대로 남게 된다는 것이었는데, Docker Hub에서 무료로 개인 계정으로 사용할 수 있는 private repository가 1개 주어지므로 이를 활용하기로 했다. 그래서 jenkins는 private repository를 사용하고, 홈페이지 서버는 톰캣만 wget으로 받아서 구동시켜놓은 상태로 public repository로 만들었다.

## ssh를 통해 젠킨스에서 홈페이지(tomcat) 서버로 배포

위의 .yml 파일에서 tomcat 서버에 ssh port를 따로 포워딩 한 이유는 jenkins에서 ssh를 통해 배포할 때 links에 설정한 이름으로 연결이 될 것으로 생각했었는데, 연결할 수가 없었다. 방법이 잘못 되어서 그런 것일 수도 있기 때문에 더 찾아봐야겠지만 우선은 ssh port를 열고 이를 통해 배포를 하도록 설정했다.  이 과정에서도 한가지 이슈가 있었는데 기본적으로 서버 이미지에 sshd가 설치되어 있지 않아서 연결 할 수가 없었다.


* sshd 설치

  {% highlight shell %}
  $ apt-get update && apt-get upgrade && apt-get install -y openssh-server
  {% endhighlight %}

  * apt-get updpate가 되어있지 않으면 설치가 안됨

* /var/run/sshd 디렉토리 생성

  {% highlight shell %}
  $ mkdir /var/run/sshd
  {% endhighlight %}

  * 해당 디렉토리가 존재하지 않으면 "Missing privilege separation directory: /var/run/sshd" 에러가 발생한다.

* 실행

  {% highlight shell %}
  $ /usr/sbin/sshd -D
  {% endhighlight %}



sshd를 설치하는 내용을 추가하기 위해 Dockerfile을 수정하고 이미지에 반영하여 다시 Docker Hub에 push를 했다.



* sshd 설치 내용 Dockerfile에 반영

  {% highlight shell %}
  RUN apt-get update && apt-get upgrade \
      && apt-get -y install openssh-server

  RUN mkdir /var/run/sshd

  CMD ["/usr/sbin/sshd", "-D"]
  {% endhighlight %}



외부에서 root 계정으로는 ssh로 접속할 수가 없으므로 새로운 계정을 만들어서 접속할 수 있도록 해주어야 하는데 홈페이지 서버에 관련된 모든 작업을 이 계정이 수행하도록 하기로 결정했다. 그래서 유저 계정을 추가하는 과정과 권한을 부여하는 과정을 Dockerfile에 반영했다.



* 유저 계정 관련 내용 Dockerfile에 반영

  {% highlight shell %}
  RUN adduser --disabled-password --gecos "" <계정명> \
      && echo '<계정명>:<패스워드>' | chpasswd \
      && mkdir <작업디렉토리>

  RUN chown -R <계정명>:<계정명> <작업디렉토리>
  {% endhighlight %}



계정명과 패스워드가 Dockerfile에 포함되는데 Docker hub에 이 repository가 public이기 때문에 보안상의 문제가 발생하였다. 고민 끝에 초기 계정과 패스워드를 설정하고 반드시 변경할 것을 Dockerfile에 명시하기로 했다.



* 갱신된 Dockerfile

  {% highlight shell %}
  FROM ubuntu:14.04

  MAINTAINER Yongho Choi <yongho1037@gmail.com>

  # Set locales
  RUN locale-gen en_GB.UTF-8
  ENV LANG en_GB.UTF-8
  ENV LC_CTYPE en_GB.UTF-8

  # Fix sh
  RUN rm /bin/sh && ln -s /bin/bash /bin/sh

  # Install dependencies
  RUN apt-get update -y && apt-get upgrade -y && \
      apt-get install -y git build-essential curl wget software-properties-common openssh-server

  # Install JDK 8
  RUN \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  add-apt-repository -y ppa:webupd8team/java && \
  apt-get -y update && \
  apt-get install -y oracle-java8-installer wget unzip tar && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /var/cache/oracle-jdk8-installer

  # Define commonly used JAVA_HOME variable
  ENV JAVA_HOME /usr/lib/jvm/java-8-oracle

  # add user and working directory make
  ENV HOME_PATH home/service

  # Important! : You should change passwd. (default xhazot)
  RUN adduser --disabled-password --gecos "" tomcat \
      && echo 'tomcat:xhazot' | chpasswd \
      && mkdir /${HOME_PATH} \
      && mkdir /${HOME_PATH}/tmp \
      && mkdir /var/run/sshd

  # Get Tomcat
  ENV TOMCAT_VERSION 8.0.37

  RUN wget --quiet --no-cookies http://apache.rediris.es/tomcat/tomcat-8/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz -O /${HOME_PATH}/tmp/tomcat.tgz && \
  tar xzvf /${HOME_PATH}/tmp/tomcat.tgz -C /${HOME_PATH} && \
  mv /${HOME_PATH}/apache-tomcat-${TOMCAT_VERSION} /${HOME_PATH}/tomcat && \
  rm /${HOME_PATH}/tmp/tomcat.tgz && \
  rm -rf /${HOME_PATH}/tomcat/webapps/examples && \
  rm -rf /${HOME_PATH}/tomcat/webapps/docs && \
  rm -rf /${HOME_PATH}/tomcat/webapps/ROOT

  ENV CATALINA_HOME /${HOME_PATH}/tomcat
  ENV PATH $PATH:$CATALINA_HOME/bin

  EXPOSE 8080
  EXPOSE 8009

  EXPOSE 22

  WORKDIR /${HOME_PATH}/tomcat

  RUN chown -R tomcat:tomcat /${HOME_PATH}

  # Launch Tomcat
  CMD ["/home/service/tomcat/bin/catalina.sh", "run"]

  # Launch sshd
  CMD ["/usr/sbin/sshd", "-D"]
  {% endhighlight %}

## 마치며

이제 모든 설정은 마무리 되었고, docker-compose.yml 파일만으로 [내 개인 홈페이지] 서버 환경을 구축할 수 있었다. 현재는 완전한 자동화는 아니지만 해야할 일이 거의 없으므로 이 전에 비해서는 훨씬 편리해졌다.


해야 할일을 요약하면

1. [docker 설치]
2. docker-compose.yml을 이용하여 도커 컨테이너를 생성
3. tomcat server의 패스워드 변경
4. jenkins 설정에서 ssh ip 변경

설치과정을 제외한 설정만 보면 5분도 걸리지 않을 작업들만 남는다. 앞으로 개선사항으로는 tomcat 서버 앞단에 nginx 서버를 두어 로드밸런싱 및 암호화 적용을 하고, database 서버도 구축하여 기능을 개선하면 좋을 것 같다. (nginx와 database도 마찬가지로 docker로 구성.)

## Tip

이번 작업을 진행하면서 Github의 Project 기능을 사용해봤는데, 작업 관리하는데 굉장히 만족스러웠다. 해당 프로젝트 관련 이슈사항들이 한곳에 몰리기 때문에 편리하기도 했고 재미 있기도 했다.

진행 예정인 작업들과 진행 중인 작업 내용들을 칸반 형태로 작성하고 이리저리 옮겨가며 작업 상황을 기록할 수 있어서 우선순위를 정하거나 진행상황을 한눈에 볼 수 있었고, 이슈화 시켜서 조금더 세부적인 사항들을 정리할 수 있었다.

* Github Projects 기능
![](http://yonghochoi.github.io/images/docker/github-projects.png)

* Github Issues 기능
![](http://yonghochoi.github.io/images/docker/github-issues.png)


[내 개인 홈페이지]: http://yongho-choi.com
[docker 설치]: https://docs.docker.com/engine/installation/linux/ubuntulinux/
